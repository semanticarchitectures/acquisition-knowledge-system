# 0012. The AKS simulation interface: HLA transport now, C2SIM/UCI content profile at D4/D5

- Status: Accepted (2026-11-16)
- Relates: [ADR 0005](0005-one-model-many-apps.md) (one shared model), [ADR 0007](0007-gmns-joins-aks.md) (GMNS as the network/C2 testbed), [ADR 0010](0010-gmns-message-classes-align-to-aoc-dfo.md) (GMNS↔AOC-DFO message alignment), [ADR 0011](0011-gmns-stays-matlab-uci-c2sim-native.md) (GMNS native UCI/C2SIM roadmap)
- Supersedes the "simulation-interface ADR (due 2027-01-31)" placeholder referenced in `portfolio.md`, `integrated-plan.md`, and the component specs.

## Context

D2 ("at scale, over a realistic network," 2027-08-31) is the first deliverable in which the
simulations couple: ENSIM, Aircrew-LangChain and GMNS exchange scenarios, orders, reports and
messages over a simulated degraded network. The portfolio flagged a "simulation-interface ADR"
as the first gate for this, framed as a choice among HLA, SISO C2SIM, and bespoke formats.

Reading the components at their current `main` shows the decision is not a single-protocol
choice, because three components exchange three different *kinds* of traffic:

- **ENSIM** needs distributed **time-and-state synchronization** (mission timeline, ATO state,
  C2-node hand-offs). It already has a **working HLA federation** (Portico 2.1.4), verified
  end-to-end across two real federate processes. Its `federation-kernel` boundary is
  deliberately **JSON-over-stdio/HTTP, not JNI**, so non-Java peers can join. Its C2 interfaces
  are schema-only today (five message types).
- **GMNS** does **C2 message transport** under degraded links with ICAM/AUTH semantics
  (AUTH_REQUEST/RESPONSE, cert renewal). It is MATLAB-authoritative and will build UCI then a
  SISO C2SIM gateway natively (ADR 0011).
- **Turnpoint** does **mission-plan hand-off** — MCP/REST now, UCI at the D4 AOC boundary.
- **Aircrew-LangChain** has its own `mission-order` format (v1), explicitly "offered as input
  to this decision."

HLA is a *transport/time-management* standard; SISO C2SIM is a *message-content* standard. They
are not alternatives at the same layer — C2SIM is commonly carried over HLA. The real question
is how to stage a transport layer and a content layer against the D2 → D5 schedule without
stranding working code or blocking D2 on capabilities that do not exist yet.

Constraints taken as given:
- ENSIM's HLA federation and JSON boundary are built and verified; discarding them is pure cost.
- The GMNS SISO C2SIM gateway does not exist until D4/D5 (ADR 0011 roadmap: UCI M2 → C2SIM).
- Components join the simulation through this interface and never embed one another
  (`portfolio.md`).
- Stacks are polyglot: TypeScript+Java (ENSIM), MATLAB (GMNS), Python (Turnpoint, Aircrew).

## Decision

**The AKS simulation interface is a two-layer design, staged to the schedule:**

1. **Transport layer (from D2): an HLA federation.** Every simulation joins one HLA federation
   (Portico 2.1.4) for time-and-state synchronization, through ENSIM's existing
   JSON-over-stdio/HTTP `federation-kernel` boundary contract — **not** raw HLA APIs — so
   Python and MATLAB peers join without JNI. ENSIM is already a federate; GMNS, Turnpoint and
   Aircrew-LangChain join through per-component adapters against the JSON boundary.

2. **Content layer (D2 interim → D4/D5 conformant): JSON now, converging on C2SIM/UCI.** At D2,
   C2 and plan/order content crosses the federation as a **JSON message envelope**, generalized
   from Aircrew-LangChain's `mission-order` format and ENSIM's five `c2-interfaces` message
   types. This envelope is **designed field-for-field to map onto SISO C2SIM**, so the later
   move is refinement, not rework. At D4/D5, when the GMNS native C2SIM gateway lands, **SISO
   C2SIM (SISO-STD-019) becomes the conformant cross-simulation content standard and UCI the
   content standard at the Turnpoint → AOC boundary**, mapped to AOC-DFO message classes per
   ADR 0010.

**Scope of this ADR:** it fixes the direction and specifies the **D2 transport-and-interim-
content layer in full**. The **C2SIM/UCI content profile is recorded here as the committed
D4/D5 direction but specified by a follow-on ADR** when the GMNS C2SIM gateway is real. This
keeps D2 unblocked and defers the C2SIM message-profile detail to the point where it can be
written against an existing gateway.

### What D2 requires (specified now)
- **RTI:** Portico, pinned at `portico-2.1.4` (Linux/Windows release assets; no macOS binary,
  no HLA-4 — accepted constraints, already documented in ENSIM).
- **Federate boundary contract:** ENSIM's JSON-over-stdio/HTTP `federation-kernel` interface is
  the contract every component codes against; raw HLA/JNI is an ENSIM-internal detail, not part
  of the interface.
- **D2 content envelope:** a versioned JSON schema in the umbrella (a cross-cutting spec,
  `.kiro/specs/simulation-interface/` per the conventions), covering the request → validate →
  tasking → execution → report sequence; field names and structure chosen to map 1:1 onto
  C2SIM later.
- **Per-component adapters and owners:** ENSIM (done); GMNS (MATLAB federate shim — incremental
  to its ADR 0011 C2SIM work); Turnpoint (Python adapter); Aircrew-LangChain (Python adapter).
- **Acceptance test:** one mission exchanged end-to-end across all four components over the
  federation, as the interface's conformance criterion for D2.

### What D4/D5 requires (direction committed, detail deferred to a follow-on ADR)
- SISO C2SIM as the cross-simulation content standard, delivered by the GMNS C2SIM gateway.
- UCI at the Turnpoint → AOC boundary, mapped to AOC-DFO message classes (ADR 0010).
- A follow-on ADR specifies the C2SIM/UCI message profile and the JSON-envelope → C2SIM
  migration, written once the gateway exists.

## Options considered

| Option | Verdict |
| --- | --- |
| **A. HLA transport + UCI/C2SIM payloads (C2SIM-over-HLA), all at once** | The right end state, but specifying the C2SIM profile now means writing it against a gateway that does not exist until D4/D5, and risks blocking D2. Chosen *in staged form* as Option D. |
| **B. C2SIM as the single interface, no HLA** | Rejected: discards ENSIM's working HLA federation; C2SIM provides no distributed time management, so time-sync would be rebuilt bespoke; and the C2SIM gateway does not exist until ~2028, so choosing it now blocks D2. |
| **C. Bespoke JSON bus; standards only at the external edge** | Rejected: invents a fourth format when too many was the problem; no time management; weakens the standards-conformance story that justifies the GMNS UCI/C2SIM and AOC-DFO investment. |
| **D. HLA + JSON transport now (D2); C2SIM/UCI content profile at D4/D5 (chosen)** | Strands no working code, does not block D2 on an unbuilt gateway, and keeps the full standards story; matches ENSIM's JSON-not-JNI boundary, ADR 0011's UCI→C2SIM roadmap, and the shared-model rule. |

## Consequences

- **D2 is unblocked** using only what exists (ENSIM's HLA federation + a JSON envelope). The
  interface is no longer a dangling dependency in `portfolio.md` / `integrated-plan.md`.
- **No working code is discarded.** ENSIM's federation and boundary become the portfolio
  interface rather than an ENSIM-internal detail.
- **The standards story holds.** Real SISO C2SIM and UCI remain the content standards at D4/D5,
  where conformance is actually demonstrated; the D2 JSON envelope is explicitly a way-station
  shaped to converge on them.
- **New work created:** a cross-cutting `simulation-interface` spec in the umbrella (D2 envelope
  + boundary contract); a GMNS MATLAB federate shim; Python adapters in Turnpoint and
  Aircrew-LangChain; and a follow-on ADR for the C2SIM/UCI content profile before D4.
- **Accepted constraints:** Portico has no HLA-4 and no macOS binary; builds run on
  Linux/Windows. Revisit only if an HLA-4 RTI or macOS support becomes a hard requirement.
- **Aircrew-LangChain's `mission-order` format** is adopted as a basis for the D2 envelope
  rather than kept as a private format — resolving one of the "three simulation interfaces"
  the integrated plan flagged.
- **`sa-codecs`** (ADR 0008's deferred shared package for UCI/C2SIM codecs) gains a concrete
  future consumer: the D4/D5 content profile. Still deferred; noted for traceability.

## Follow-on actions (not part of this ADR)

- Write `.kiro/specs/simulation-interface/` in the umbrella (D2 envelope schema, boundary
  contract, per-component adapter requirements, end-to-end acceptance test).
- Update `portfolio.md`, `integrated-plan.md`, `progress.md` to reference ADR 0012 in place of
  the "simulation-interface ADR (due 2027-01-31)" placeholder.
- Record component-adapter tasks in each component repo's `.kiro/specs/` (per the conventions).
