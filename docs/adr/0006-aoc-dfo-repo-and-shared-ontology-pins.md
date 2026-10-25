# 0006. AOC-DFO gets its own repo; all project ontologies share one BFO/CCO pin

- Status: Accepted (2026-10-22); rationale amended by ADR 0007 (2026-10-25): GMNS is now an AKS
  component, and the repository model is decided by ADR 0008
- Date: 2026-10-22
- Resolves: AOC-DFO open decisions OD-2 (repository home) and OD-3 (CCO version pin)
- Related: ADR 0001, ADR 0005

## Context

AOC-DFO's spec (September 2026) left two decisions open that now affect the shared model, not just
AOC-DFO:

- **OD-2:** a standalone `AOCDataFlowOntology` repo, or a module in an ontology monorepo alongside
  GMNS.
- **OD-3:** pin CCO to the GMNS version, or move GMNS and AOC-DFO to the current CCO release together.

Under ADR 0005 the ontology is the shared model. AOC-DFO (operations) and the KB ontology
(acquisition) must align, and the far horizon depends on AOC-DFO modeling the AOC side of the
unmanned-to-AOC interface. Ontologies pinned to different upper-ontology versions cannot be joined
without remapping.

## Decision

1. **OD-2: standalone repo.** AOC-DFO lives in its own repo under `semanticarchitectures`, following
   the component pattern of ADR 0001. It is not merged into a monorepo with GMNS.
2. **OD-3: one shared pin.** AOC-DFO, GMNS and the KB ontology move together to one current BFO 2020
   and CCO release. The pin is recorded once, in this repo's ontology steering ("Pinned versions"),
   and each component's imports follow it.

## Options considered

| Decision | Option | Outcome |
| --- | --- | --- |
| OD-2 | Standalone repo | Chosen: matches every other component; independent release |
| OD-2 | Ontology monorepo with GMNS | Rejected: couples GMNS, which is outside AKS, to AKS's release cycle |
| OD-3 | Match GMNS's existing pin | Rejected: freezes the shared model at an older CCO and leaves the KB on a different one |
| OD-3 | Shared move to current CCO | Chosen: one pin for every ontology in the shared model |

## Consequences

- The exact CCO release is selected and recorded before KB or AOC-DFO Phase 0 scaffolding. Until then
  the pin table reads TBD.
- GMNS needs an upgrade to the shared pin. That work happens in the GMNS repo, as a joint amendment
  as the AOC-DFO spec anticipated.
- A later pin change is made once, here, and propagated to every component, never changed in one
  component alone.
- OD-1 (Tier 2 access) and OD-4 (Foundry object types) remain AOC-DFO decisions, to be settled in its
  own repo.
