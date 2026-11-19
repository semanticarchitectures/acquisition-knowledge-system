# Requirements — AKS Simulation Interface (D2 layer)

**Spec:** `.kiro/specs/simulation-interface/requirements.md`
**Decision:** implements [ADR 0012](../../docs/adr/0012-aks-simulation-interface.md).
**Scope:** the **D2 transport-and-interim-content layer** only — an HLA federation for
time/state transport plus a C2SIM-shaped JSON message envelope. The D4/D5 conformant
C2SIM/UCI content profile is committed by ADR 0012 but specified by a **follow-on spec**,
not here.

## Introduction

At D2 ("at scale, over a realistic network," 2027-08-31) four components couple: ENSIM,
GMNS, Turnpoint, and Aircrew-LangChain exchange scenarios, orders, tasking, and reports over
a simulated degraded network. ADR 0012 fixed the design: one HLA federation (Portico 2.1.4)
for time-and-state synchronization, joined through ENSIM's existing JSON-over-stdio
`federation-kernel` boundary, carrying a JSON message envelope generalized from ENSIM's five
`c2-interfaces` message types and Aircrew's `mission-order` format, shaped to converge on
SISO C2SIM later.

This spec turns that decision into buildable requirements. It does not re-decide the
architecture; where a requirement seems to, defer to ADR 0012.

Grounding (read at current `main`, 2026-11-18):
- ENSIM has a working `federation-kernel` (Portico 2.1.4) exposing a `StdioBridge`
  (`connect`/`publish`/`subscribe`/`tick`/`resign`, one JSON object per line) and a FOM that
  carries any record as a single `Record` object with `EntityId`/`EntityType`/`RecordJson`.
- ENSIM's five `c2-interfaces` types — TaskingRequest, ValidationRecord, ATOLine,
  ExecutionStatus, MissionReport — share a `common.schema.json` with `entityId`, `actor`
  (role/c2Node/organization), and a **required** `doctrineSource` citation.
- Aircrew's `mission-order` (v1) carries roster, nets, route, cargo, timeline, and was
  explicitly "offered as input to this decision."

---

### Requirement 1 — One HLA federation as the transport

**User Story:** As an AKS component, I want to exchange state with the other simulations over
one shared HLA federation, so that time and state stay synchronized without point-to-point
coupling.

**Acceptance Criteria:**

1.1. THE SYSTEM SHALL define one named AKS federation that all four D2 components join, using
Portico pinned at `portico-2.1.4` (the version ENSIM's `federation-kernel` already vendors).

1.2. THE SYSTEM SHALL carry every exchanged message as the existing FOM `Record` object
(`EntityId`, `EntityType`, `RecordJson`), so the FOM changes only when a new entity/message
*type* is added, not per field — consistent with ENSIM's canonical-schema rule.

1.3. WHERE a component is not Java (GMNS/MATLAB, Turnpoint/Python, Aircrew/Python), THE
SYSTEM SHALL let it join through the JSON-over-stdio `StdioBridge` contract rather than an
embedded JVM or JNI.

1.4. THE SYSTEM SHALL NOT require HLA time-management or synchronization points at D2; the
federation is used for publish/subscribe state exchange, matching ENSIM's current federate
(time management is a later, separately-specified concern).

### Requirement 2 — The federate boundary contract

**User Story:** As a component developer in any language, I want one documented local
interface to the federation, so that I can join without learning HLA.

**Acceptance Criteria:**

2.1. THE SYSTEM SHALL adopt ENSIM's `StdioBridge` line protocol as the AKS federate boundary
contract: newline-delimited JSON, commands `connect`, `publish`, `subscribe`, `tick`,
`resign`; each reply carries `ok` and, on failure, `error`; `tick` returns `records` reflected
since the previous tick.

2.2. THE boundary contract SHALL be documented in this spec's design as a versioned contract
that components code against, so that an ENSIM-internal change to how Portico is wrapped does
not break other components as long as the line protocol holds.

2.3. THE design SHALL record the three empirically-found Portico 2.1.4 operational constraints
already documented in `federation-kernel` (stagger federate startup ~3s; subscribe-before-
publish, no history replay; UTF-8 string encoding, not `HLAunicodeString`) as adapter
requirements, so each component's adapter inherits them rather than rediscovering them.

### Requirement 3 — The D2 message envelope

**User Story:** As a component exchanging C2 and plan content, I want one envelope schema
shared across the federation, so that any component can read any message and the content maps
onto SISO C2SIM later without rework.

**Acceptance Criteria:**

3.1. THE SYSTEM SHALL define a single JSON message envelope with common fields generalized
from ENSIM's `common.schema.json`: a stable kebab-case `id`, a `messageType` discriminator, an
RFC 3339 `timestamp`, a `missionId`, a `producedBy` actor (role / C2 node / organization), and
a `doctrineSource` citation array.

3.2. THE envelope SHALL carry the five ENSIM message types (TaskingRequest, ValidationRecord,
ATOLine, ExecutionStatus, MissionReport) as the D2 content set for the request → validate →
tasking → execution → report sequence, reusing ENSIM's schemas rather than redefining them.

3.3. THE envelope SHALL accommodate Aircrew's `mission-order` content (roster, nets, route,
cargo, timeline) as a message type, resolving Aircrew's private format into the shared envelope
per ADR 0012.

3.4. EACH envelope field SHALL be named and structured to map 1:1 onto a SISO C2SIM
construct where one exists; the design SHALL record the intended C2SIM mapping per field, so
the D4/D5 follow-on is refinement, not redesign. WHERE no clean C2SIM construct exists for a
field, THE design SHALL flag it as an open mapping question for the follow-on spec.

3.5. THE SYSTEM SHALL require a `doctrineSource` citation on every message, preserving ENSIM's
rule that C2 content traces to public doctrine.

3.6. THE envelope SHALL version itself (`envelopeVersion`), so the D2 JSON profile and the
later C2SIM-conformant profile are distinguishable on the wire.

### Requirement 4 — GMNS carries the traffic as the network layer

**User Story:** As the AKS network testbed, GMNS must transport envelope messages over its
simulated degraded links, so that D2 measures real C2 traffic under latency and outage.

**Acceptance Criteria:**

4.1. THE SYSTEM SHALL let GMNS ingest an envelope message, carry it as a GMNS `C2Message`
payload over its simulated links (subject to latency, outage, and ICAM/AUTH gating), and
re-emit it to the federation on delivery.

4.2. THE envelope SHALL carry enough addressing (source/destination mapped to GMNS nodes) for
GMNS to route it, specified in the design as the envelope↔GMNS-node mapping.

4.3. WHERE GMNS drops or delays a message (outage, AUTH failure, no route), THE delivery
outcome SHALL be reported back to the federation as an ExecutionStatus/MissionReport record,
so degraded-network effects are observable end-to-end.

### Requirement 5 — Per-component adapters

**Acceptance Criteria:**

5.1. THE SYSTEM SHALL define, per component, a federate adapter against the boundary contract:
ENSIM (exists), GMNS (MATLAB adapter — incremental to its ADR 0011 C2SIM work), Turnpoint
(Python adapter), Aircrew-LangChain (Python adapter).

5.2. EACH adapter requirement SHALL name the component repo it lands in, so the task is
recorded in that repo's `.kiro/specs/` per the AKS conventions, not here.

5.3. NO adapter SHALL embed another component; each joins only the federation (per
`portfolio.md` boundary rule).

### Requirement 6 — End-to-end acceptance test

**User Story:** As the D2 gate, I want one scenario that exercises the whole interface, so that
"the interface works" is a passing test, not an assertion.

**Acceptance Criteria:**

6.1. THE SYSTEM SHALL define one acceptance scenario in which a single mission flows end to
end across all four components over the federation: a TaskingRequest originates, is validated,
becomes an ATOLine, is executed with C2 traffic carried over GMNS's degraded network, and
returns a MissionReport — all as envelope messages.

6.2. THE acceptance test SHALL assert message integrity (content unchanged end to end except
for GMNS-modeled delivery effects) and that every message carried a `doctrineSource`.

6.3. THE acceptance test SHALL be the interface's D2 conformance criterion; a component is
"D2-coupled" when it passes its leg of this scenario.

### Out of scope (follow-on spec, per ADR 0012)

- The SISO C2SIM message profile and the envelope→C2SIM migration.
- UCI at the Turnpoint→AOC boundary and its AOC-DFO class mapping (ADR 0010).
- HLA time management / synchronization points.
- The `sa-codecs` shared package (ADR 0008, deferred).
