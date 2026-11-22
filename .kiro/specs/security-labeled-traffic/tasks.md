# Tasks — Security-Labeled Traffic and Cross Domain Solutions

**Spec:** `.kiro/specs/security-labeled-traffic/tasks.md`
**Implements:** [AKS ADR 0013](../../docs/adr/0013-security-labeled-traffic-and-cross-domain-solutions.md);
satisfies requirements.md R1–R14 via design.md C1–C11 (both approved 2026-11-22).

Tasks are sequenced by dependency. Umbrella tasks (Phase A) build the portable contract in this
spec's folder. Component tasks land in each component repo's `.kiro/specs/` per the AKS
conventions and are cited here by the repo they land in; the component spec owns the detail.
Each task cites the requirements it satisfies and the design section it builds.

## Phase A — Portable contract (umbrella, this repo)

- [ ] 1. **Configuration schema** `schema/label-config.schema.json` (JSON Schema 2020-12): levels,
  caveats with `type` and `partners`, `partners`, `latticeGate`, `defaultLabel`, `abacRules` with
  optional `caveat`, `cdsPolicies` (rules, `allowList`, `sanitizeRules`), `review`, required
  `provenance` (≥ 1) and `simMarker`. _Requirements: R2.1, R2.4, R12.3 · Design: C2_
- [ ] 2. **Guardrail deny-list** `schema/guardrail-denylist.json` and a lint script
  (`scripts/lint-label-config.mjs`) that checks display names and rule text against it and that
  level display names start with `simMarker`. _Requirements: R12.1, R12.2 · Design: C10_
- [ ] 3. **US Government configuration** `configs/us-govt-representative.json`: four levels
  (U < CUI < S < TS analogues), PII/PHI handling caveats, `REL-PARTNER`, `latticeGate: true`,
  `defaultLabel` = U analogue, `review` {n: 2, agentsMustDiffer: true}, the `low-high` CDS policy
  from design C5, and a provenance list of public sources. Uses the `aocdfo:` placeholder prefix
  until task 16. _Requirements: R2.2, R2.5, R2.7, R12.3 · Design: C2, C5_
- [ ] 4. **Coalition configuration** `configs/coalition-representative.json`: three levels, one
  handling caveat, `REL-A` / `REL-AB` / `REL-BC` over partners {a, b, c}, a matching CDS policy.
  _Requirements: R2.6 · Design: C2_
- [ ] 5. **Envelope `d2.2` schema delta** in `.kiro/specs/simulation-interface/schema/`: optional
  `label`, `derivedFrom`, `sanitization`, `approvals`, `qos`; `envelopeVersion` enum adds
  `"d2.2"`; `"d2.1"` fixtures still validate. Add the five rows to the `simulation-interface`
  C2SIM mapping table as open mapping questions. _Requirements: R5.1–R5.3, R5.5, R9.1 · Design: C7_
  - Depends on `simulation-interface` task 1 (the D2 envelope schema) existing; if it has not
    landed, this task authors the delta as a separate `envelope-d2.2.schema.json` that `$ref`s the
    base.
- [ ] 6. **Test vectors** `vectors/algebra.json`, `vectors/decisions.json`, `vectors/cds.json`,
  `vectors/review.json`, covering at minimum: join/dominance including unrestricted, single,
  nested and disjoint releasability; a stage 1 deny an ABAC rule would permit; a per-caveat ABAC
  deny; each fail-closed cause; each CDS action; review sets person+agent, person+person,
  agent(X)+agent(Y), agent(X)+agent(X), single reject, timeout, `requireHuman`. Each vector names
  its configuration. _Requirements: R14.2 · Design: Testing strategy_
- [ ] 7. **Umbrella CI**: validate both configurations and all vectors against their schemas, run
  the guardrail lint over `configs/`, and add negative fixtures (unknown IRI, duplicate level,
  untyped caveat, unknown domain, deny-list hit, missing provenance) that must fail.
  _Requirements: R2.4, R12.2 · Design: C2, C10_

## Phase B — GMNS engine (`GlobalMobileNetworkSim/.kiro/specs/`)

Recorded as one GMNS component spec (proposed name `security-labeled-traffic`) that cites this
spec; the sub-tasks below are its expected shape, detailed there.

- [ ] 8. **`icam.LabelConfig` + `icam.LabelAlgebra`**: load and validate a configuration
  (schema, referential, guardrail lint — same deny-list file, vendored by pin); parse, join,
  dominates, joinAll, render with the partner-set releasability reading; error ids
  `configInvalid`, `unknownTerm`. Parameterized `matlab.unittest` over `vectors/algebra.json`, plus
  the seeded-random lattice-law test. _Requirements: R1, R2.1–R2.4, R2.7, R12.2, R14.1, R14.3 ·
  Design: C1, C2_
- [ ] 9. **Scenario domains**: `ScenarioLoader` accepts `labelConfig`, enclave `level` /
  `partners`, entity `agentModel`, and `cds_gateway` nodes; `nodeSpansDomains` error; bypass-link
  warning. _Requirements: R4.1, R4.3, R8.2 · Design: C3, C5, C6_
- [ ] 10. **`icam.LabeledDecision`**: stage 1 lattice gate (local, fail-closed), stage 2 per-caveat
  PDP calls, deny-overrides, result `{decision, stage, reason, labelRendered, defaultApplied}`;
  `PolicyDecisionPoint.evaluate` gains optional `caveats` and rule `caveat` matching; cache key
  includes caveat; `PolicyEnforcementPoint` routes through it when a configuration is loaded.
  Parameterized test over `vectors/decisions.json`. _Requirements: R3, R5.4 · Design: C4_
- [ ] 11. **Router enforcement**: `RoutingEngine` refuses a cross-domain hop to a non-dominating
  domain except via a `cds_gateway`; receivers run the labeling-violation check.
  _Requirements: R4.2, R6.1, R6.2 · Design: C3, C7_
- [ ] 12. **`network.CdsGateway`**: five-step pipeline, fail-closed causes (`queueOverflow`,
  `malformed`, `noRule`, `gatewayDown`), allow-list, sanitize (new envelope, sanitization record,
  `ProvenanceGraph` `sanitizedBy` edge), latency distributions, `OutageEngine` availability.
  Parameterized test over `vectors/cds.json`. _Requirements: R7 · Design: C5_
- [ ] 13. **`icam.ReviewCoordinator`**: review cases, `REVIEW_REQUEST` / `REVIEW_DECISION`
  C2_Messages through the network, distinct-entity and `agentsMustDiffer` counting,
  `requireHuman`, any-reject, timeout; `reviewerNotEligible` loader check; deterministic stub
  reviewer for tests; LLM reviewers via `+agent` as an optional mode. Parameterized test over
  `vectors/review.json`. _Requirements: R8 · Design: C6_
- [ ] 14. **Observability**: Event_Log entry types and the Statistics_Report **Labels** section
  (per domain, per gateway, derivation depth, defaults applied); report header states the engine
  is a simulation. _Requirements: R5.4, R6.3, R11, R12.4 · Design: C8, C10_
- [ ] 15. **Regression guard**: run existing `tests/icam/*` and `SimControllerICAMTest` unchanged;
  Event_Log diff on existing scenarios with no `labelConfig` must be empty; grep test that `+icam`
  and `+network` contain no `aocdfo:` IRI or shipped display name. _Requirements: R3.7, R13.4,
  R2.3 · Design: Correctness properties 7, 8_

## Phase C — Other components

- [ ] 16. **AOC-DFO label terms** (`aoc-dfo/.kiro/specs/`): add `LabelLevel`,
  `HandlingCaveat`, `ReleasabilityCaveat` and one individual per term in both configurations,
  aligned under CCO's information-content-entity hierarchy; no ordering or release rules in OWL.
  Then, back in this repo, swap the `aocdfo:` placeholder for the pinned namespace in tasks 3, 4
  and 6. _Requirements: R2.2 · Design: C9_
- [ ] 17. **ENCAP alignment note** (`encap/.kiro/specs/`): record the `labelIri` match field
  contract in ENCAP's access-control spec as a future option; no implementation now.
  _Requirements: R10 · Design: C11_
- [ ] 18. **GMNS QoS spec handoff** (`GlobalMobileNetworkSim/.kiro/specs/`): open the separate QoS
  spec that consumes the envelope `qos` field; until it lands GMNS carries `qos` unchanged.
  _Requirements: R9.2, R9.3 · Design: C7_

## Phase D — Acceptance

- [ ] 19. **Acceptance scenario** `scenarios/labeled_cds/` in GMNS: `low_net` (U analogue),
  `high_net` (S analogue), `CDS_1`, medic and non-medic roles, two human reviewers and agent
  reviewers with models X, X and Y; assertions per R13.2, stated once per configuration and run
  under both by swapping only `labelConfig`. _Requirements: R13.1–R13.3 · Design: Testing strategy_

## Dependencies

Phase A tasks 1–2 come first; 3, 4 and 6 need 1; 7 needs 1–6. Task 5 depends on
`simulation-interface` task 1 or uses its fallback. Phase B needs tasks 1–4 and 6 (vendored at a
pinned umbrella commit). Within Phase B: 8 → 9 → 10 → 11; 12 needs 10; 13 needs 12; 14 needs
10–13; 15 runs throughout and gates the merge of each. Task 16 can run in parallel with Phase B
(placeholders keep Phase B unblocked). Tasks 17 and 18 block nothing. Task 19 needs 8–14.

## Notes

- Component tasks are recorded here for traceability but are **owned and detailed in the
  component repo's `.kiro/specs/`**; this file does not duplicate them.
- Proposed GMNS spec name `security-labeled-traffic` is proposed, not decided; GMNS may fold it
  into `matlab-network-sim` as a new requirement group instead.
