# Tasks — AKS Simulation Interface (D2 layer)

**Spec:** `.kiro/specs/simulation-interface/tasks.md`
**Implements:** [ADR 0012](../../docs/adr/0012-aks-simulation-interface.md); satisfies
requirements.md R1–R6 via design.md.

Tasks are sequenced by dependency. Umbrella tasks define the shared contract; component tasks
land in each component repo's `.kiro/specs/` (per the AKS conventions) and are cited here by
the repo they land in. Each task cites the requirements it satisfies.

## Phase A — Shared contract (umbrella)

- [ ] 1. Author the **D2 envelope JSON schema** in the umbrella (`schema/` under this spec):
  common fields (`envelopeVersion`, `id`, `messageType`, `timestamp`, `missionId`,
  `producedBy`, `doctrineSource`, `source`, `destination`) generalized from ENSIM
  `common.schema.json`; reference ENSIM's five `c2-interfaces` schemas by `$id` for their
  payloads. _Requirements: R3.1, R3.2, R3.5, R3.6_
- [ ] 2. Add the **MissionOrder** message type schema (payload = Aircrew `mission-order`
  fields: roster, nets, route, drop_zone, cargo, timeline). _Requirements: R3.3_
- [ ] 3. Write the **C2SIM mapping table** as a design artifact: per-field D2→C2SIM intent;
  flag `doctrineSource` (and any other) as open questions for the follow-on spec.
  _Requirements: R3.4_
- [ ] 4. Document the **boundary contract v1** (StdioBridge line protocol + the three inherited
  Portico constraints) as the versioned AKS federate interface. _Requirements: R2.1–R2.3_
- [ ] 5. Define the **envelope↔GMNS-node addressing** (`source`/`destination` semantics) and
  the **acceptance scenario config** (federation name, node map, the one mission). _Requirements:
  R4.2, R6.1_

## Phase B — Component adapters (each in its own repo)

- [ ] 6. **ENSIM** (`ensim/.kiro/specs/`): finish `sim-services` to drive the federate for the
  acceptance mission; publish TaskingRequest→MissionReport as envelope messages. _Requirements:
  R1, R3.2, R5.1_
- [ ] 7. **GMNS** (`GlobalMobileNetworkSim/.kiro/specs/`): MATLAB federate shim that calls
  StdioBridge, ingests an envelope as a `C2Message`, carries it over the DES, and re-emits /
  reports delivery outcome. Incremental to the ADR 0011 C2SIM work; honors the three Portico
  constraints. _Requirements: R1.3, R4.1, R4.3, R5.1, R5.3_
- [ ] 8. **Aircrew-LangChain** (`aircrew-langchain/.kiro/specs/`): Python adapter that emits
  MissionOrder envelopes and consumes tasking/report messages; reuses `order.py` validation.
  _Requirements: R1.3, R3.3, R5.1_
- [ ] 9. **Turnpoint** (`turnpoint/.kiro/specs/`): Python adapter for plan handoff into the
  envelope. _Requirements: R1.3, R5.1_

## Phase C — Validation and acceptance

- [ ] 10. **Schema CI:** extend ENSIM's `validate.mjs`/`check-refs.mjs` to the envelope +
  MissionOrder; wire into umbrella CI. _Requirements: R3_
- [ ] 11. **Cross-language boundary test:** two-federate publish/subscribe (ENSIM template)
  extended to the Python adapter, proving the StdioBridge contract holds cross-language.
  _Requirements: R2, R1.3_
- [ ] 12. **End-to-end acceptance test:** the four-component single-mission scenario; assert
  content integrity (minus GMNS delivery effects) and `doctrineSource` on every message. This
  is the D2 conformance gate. _Requirements: R6_

## Phase D — Follow-on handoff (not built here)

- [ ] 13. Open the **D4/D5 content-profile spec** (SISO C2SIM message profile, UCI at
  Turnpoint→AOC, AOC-DFO mapping per ADR 0010, envelope→C2SIM migration), seeded by the Phase A
  mapping table and open questions. Written when the GMNS C2SIM gateway exists. _ADR 0012_

## Dependencies
Phase A before B (components build against the contract). Task 6 (ENSIM sim-services) and
task 7 (GMNS shim) are the longest poles. Phase C task 12 requires 6–9. Task 13 is deferred
to D4/D5 and blocks nothing at D2.

## Notes
- Each component task is recorded here for traceability but is **owned and detailed in its
  component repo's `.kiro/specs/`** per the conventions; this file does not duplicate those.
- No C2SIM/UCI message structure is specified at D2 (ADR 0012 scope); only the JSON envelope,
  shaped to converge.
