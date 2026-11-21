# Design — AKS Simulation Interface (D2 layer)

**Spec:** `.kiro/specs/simulation-interface/design.md`
**Implements:** [ADR 0012](../../docs/adr/0012-aks-simulation-interface.md). Satisfies
requirements.md R1–R6.

## Overview

Two layers (ADR 0012), of which this spec builds the first:

- **Transport:** one HLA federation (Portico 2.1.4), joined through ENSIM's existing
  `federation-kernel` `StdioBridge` (JSON-over-stdio). Reused, not rebuilt.
- **Content (D2):** a versioned JSON **message envelope** generalized from ENSIM's
  `c2-interfaces` common schema and five message types, plus Aircrew's `mission-order`,
  shaped to map onto SISO C2SIM for the D4/D5 follow-on.

The design principle is **reuse over invention**: ENSIM already built the transport and most
of the envelope. This spec names them the AKS-wide contract and fills the gaps (addressing for
GMNS, the mission-order message type, the C2SIM mapping table, the adapters, the acceptance
test).

## Architecture

```
            ┌─────────────────────────── AKS HLA federation (Portico 2.1.4) ───────────────────────────┐
            │  FOM: single Record object { EntityId, EntityType, RecordJson }                          │
            └──────────────────────────────────────────────────────────────────────────────────────────┘
                 ▲ StdioBridge (JSON/line)  ▲                      ▲                        ▲
                 │                          │                      │                        │
        ┌────────┴────────┐       ┌─────────┴────────┐    ┌────────┴────────┐      ┌────────┴────────┐
        │ ENSIM           │       │ GMNS adapter     │    │ Turnpoint       │      │ Aircrew-LangChain│
        │ (federate +     │       │ (MATLAB)         │    │ adapter (Py)    │      │ adapter (Py)    │
        │  sim-services)  │       │  carries envelope│    │  plan handoff   │      │  mission orders │
        │ EXISTS          │       │  over degraded   │    │                 │      │                 │
        └─────────────────┘       │  links as C2Msg  │    └─────────────────┘      └─────────────────┘
                                   └──────────────────┘
      All content is the D2 JSON envelope, carried in RecordJson.
```

## Components and interfaces

### D1 — Transport: the federation and boundary contract
- **Federation:** one named AKS federation (name fixed in the acceptance scenario config).
  Portico `portico-2.1.4`, vendored by ENSIM's `scripts/setup-portico.sh`. No HLA time
  management at D2 (R1.4).
- **FOM:** reuse ENSIM's single-`Record` FOM (`EntityId`, `EntityType`, `RecordJson`). An AKS
  envelope message is one `Record` with `EntityType` = its `messageType` and `RecordJson` =
  the envelope JSON. The FOM changes only when a new `messageType` is added (R1.2).
- **Boundary contract (R2):** ENSIM's `StdioBridge` line protocol is the AKS federate
  interface. Newline-delimited JSON:
  - `in {"cmd":"connect","federationName":…,"federateName":…}`
  - `in {"cmd":"publish","entityType":<messageType>,"entityId":<id>,"record":<envelope>}`
  - `in {"cmd":"subscribe"}` · `in {"cmd":"tick","timeoutSeconds":N}` · `in {"cmd":"resign"}`
  - `out {…,"ok":true|false,"error"?:…}`; `tick` → `records:[{entityId,entityType,record},…]`.
  Documented here as **boundary contract v1**; an ENSIM-internal change to the Portico wrap
  does not break components while the line protocol holds.
- **Inherited Portico constraints (R2.3)** — every adapter MUST honor, carried verbatim from
  `federation-kernel` (found empirically against 2.1.4):
  1. **Stagger startup ~3s** — the federate that creates the federation settles before others
     join, or Portico's JGroups coordinator can NPE. (`CrossFederatePublishSubscribeTest` is
     the template.)
  2. **Subscribe before publish** — HLA gives no history replay; a late subscriber sees only
     future updates.
  3. **UTF-8 strings** — Portico 2.1.4's `HLAunicodeString` round-trip is broken; strings use
     plain UTF-8 (both ends are AKS code, so wire-standard encoding isn't required).

### D2 — The message envelope
Common fields (generalized from ENSIM `common.schema.json`):

| Field | Type | From | C2SIM mapping (D4/D5 intent) |
|---|---|---|---|
| `envelopeVersion` | const `"d2.1"` | new (R3.6) | distinguishes D2-JSON from the later C2SIM profile |
| `id` | kebab-case string | ENSIM `entityId` | C2SIM message UUID |
| `messageType` | enum | ENSIM `messageType` | C2SIM message category |
| `timestamp` | RFC 3339 | ENSIM | C2SIM `<DateTime>` |
| `missionId` | kebab-case string | ENSIM | C2SIM `OrderID` / task context |
| `producedBy` | actor {roleId?, c2NodeId?, organizationId?} | ENSIM `actor` | C2SIM `<Sender>` / `ForceSide` |
| `doctrineSource[]` | {publication, section?, url?, note?} | ENSIM, **required** | no direct C2SIM field — **open mapping Q** for follow-on |
| `source` / `destination` | node ref | **new (R4.2)** | C2SIM `<Receiver>` addressing |
| `payload` | per-type object | ENSIM type schemas | C2SIM message body |

Message types (`messageType` discriminator):
- **TaskingRequest, ValidationRecord, ATOLine, ExecutionStatus, MissionReport** — reuse
  ENSIM's `c2-interfaces` schemas unchanged as the `payload` (R3.2). ENSIM is the canonical
  owner; this spec references them by `$id`, does not fork them.
- **MissionOrder** — new message type whose `payload` is Aircrew's `mission-order` fields
  (roster, nets, route, drop_zone, cargo, timeline) (R3.3). Aircrew's loader/validator
  (`src/airdrop_alpha/order.py`) is the reference; the envelope adds only the common fields.

**C2SIM mapping table (R3.4)** — the per-field column above is the D2→C2SIM intent. The one
flagged gap (`doctrineSource` has no C2SIM home) is an explicit open question handed to the
follow-on spec, not resolved here. This table is the artifact that makes D4/D5 refinement.

### D3 — GMNS as the carrier (R4)
- GMNS's adapter subscribes to the federation, and for a message addressed node-to-node, wraps
  the envelope as a GMNS `C2Message` payload and runs it through the DES (latency, outage,
  ICAM/AUTH per the `matlab-network-sim` spec).
- **Envelope↔GMNS-node mapping:** `source`/`destination` carry a GMNS node id (or a
  role/c2Node id the GMNS scenario maps to a node). Defined in the acceptance scenario config.
- On delivery, GMNS re-publishes the message (optionally annotated with measured latency); on
  drop/delay/AUTH-fail/no-route, GMNS publishes an ExecutionStatus or MissionReport recording
  the outcome (R4.3), so degraded-network effects are observable to the whole federation.

### D4 — Adapters (R5), each a task in its own repo
| Component | Adapter | Lands in | State |
|---|---|---|---|
| ENSIM | `federation-kernel` + `sim-services` | ensim (`.kiro/specs/`) | federate exists; sim-services pending |
| GMNS | MATLAB federate shim calling StdioBridge | GMNS (`.kiro/specs/`) | new; incremental to ADR 0011 C2SIM |
| Turnpoint | Python adapter | turnpoint (`.kiro/specs/`) | new |
| Aircrew-LangChain | Python adapter | aircrew-langchain (`.kiro/specs/`) | new; emits MissionOrder |

Per the conventions, each adapter is specified in its component repo; this umbrella spec owns
only the shared contract and the mapping/acceptance artifacts.

## Correctness properties
1. Any component can parse any envelope message by `messageType` alone (one schema set).
2. An envelope message is exactly one FOM `Record`; a new `messageType` is the only reason to
   change the FOM.
3. Content is unchanged end to end except for GMNS-modeled delivery effects (R6.2).
4. Every message carries a non-empty `doctrineSource` (R3.5, R6.2).
5. Every envelope field either has a recorded C2SIM mapping or is a flagged open question
   (R3.4) — no field is undecided-and-silent.

## Testing strategy
- **Schema tests:** reuse ENSIM's `c2-interfaces/scripts/validate.mjs` + `check-refs.mjs`
  against the envelope; add the MissionOrder type and the common-field additions.
- **Boundary tests:** a two-federate publish/subscribe test per the ENSIM template, extended to
  a non-Java adapter (Python) to prove the StdioBridge contract holds cross-language.
- **End-to-end acceptance (R6):** the four-component single-mission scenario; the D2 gate.

## Open questions for the follow-on (D4/D5) spec
- `doctrineSource` → C2SIM: no native construct; carry as extension or drop at the C2SIM
  boundary?
- HLA time management: Mission 1 is discrete-event; at scale (D2 "concurrent missions") does
  the federation need time management, and if so does that change the boundary contract?
- UCI at Turnpoint→AOC and its AOC-DFO mapping (ADR 0010) — entirely follow-on.
