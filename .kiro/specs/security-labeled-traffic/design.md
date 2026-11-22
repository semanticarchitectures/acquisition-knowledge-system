# Design — Security-Labeled Traffic and Cross Domain Solutions

**Spec:** `.kiro/specs/security-labeled-traffic/design.md`
**Implements:** [AKS ADR 0013](../../docs/adr/0013-security-labeled-traffic-and-cross-domain-solutions.md)
(D1–D7). Satisfies requirements.md R1–R14 (approved 2026-11-22).

## Overview

Three artifacts, two homes:

- **The engine** (GMNS `+icam`, MATLAB — R14): label algebra, configuration loader, a lattice gate
  in front of the existing `PolicyDecisionPoint`, a CDS gateway node, and a review coordinator.
- **The portable contract** (this repo, under `schema/`, `configs/`, `vectors/` in this spec's
  folder): the configuration JSON schema, the US Government and coalition configurations, the
  envelope `d2.2` additions, the guardrail deny-list, and language-neutral test vectors.
- **Label terms** (AOC-DFO): representative IRIs that the configurations reference (D7).

The design principle is the ADR's: **the engine knows labels, an order, and release actions;
the configuration knows the scheme.** Nothing in the MATLAB code names a level or caveat.

Grounding is in requirements.md (GMNS `main` 4fffc02). The hooks this design uses:
`icam.PolicyDecisionPoint.evaluate` (first-match rules, `{decision, reason}`),
`icam.PolicyEnforcementPoint.checkSend` / `checkReceive`, `icam.CredentialCache` (per-entity
decision cache with TTL), `icam.EntityRegistry` (`entityType` = `human` | `NPE`),
`network.OutageEngine`, `network.RoutingEngine`, `fabric.ProvenanceGraph`, `io.ScenarioLoader`,
`io.ReportWriter`.

## Architecture

```
  scenario.json ──► io.ScenarioLoader ──► icam.LabelConfig  (loads configs/<name>.json, validates
                    (enclave levels,                         against schema, runs guardrail lint)
                     CDS nodes)                  │
                                                 ▼
  envelope d2.2 ──► icam.PolicyEnforcementPoint.checkSend / checkReceive
   (label, qos,        │
    derivedFrom)       ▼
                 icam.LabeledDecision  ── stage 1: icam.LatticeGate (local, fail-closed)
                       │                     uses icam.LabelAlgebra (join, dominates)
                       │               ── stage 2: icam.PolicyDecisionPoint.evaluate (existing,
                       │                     first-match, + caveat matching)
                       ▼               ── combine: deny-overrides
                 {decision, stage, reason}
                       │
  network.RoutingEngine ── cross-domain hop? ──► network.CdsGateway (new node type)
                                                   validate → release policy → allow-list
                                                   → allow | deny | sanitize | hold
                                                                         │
                                                         icam.ReviewCoordinator
                                                         (N distinct entities, C2_Message
                                                          approvals, timeout, any-reject)
  Event_Log / Statistics_Report ◄── every decision, action, sanitization record, review event
```

## Components and interfaces

### C1 — `icam.LabelAlgebra` (R1)

Static, pure functions over a loaded `LabelConfig`. A label in MATLAB is a struct
`{level: string IRI, handling: string[] IRIs, relCaveats: string[] IRIs, releasableTo: string[]
partner ids}`. Caveats are split by type at parse time, so the algebra never looks types up.

**Releasability is computed on partner sets.** Each releasability caveat in the configuration
denotes a set of partners, and every set implicitly includes `home`. A label's `releasableTo` is
the intersection of its releasability caveats' partner sets, or **all partners** when it has none
(no caveat = unrestricted). R1.3's "intersection of releasability caveats" and R1.4's "⊆" are
evaluated on `releasableTo`; that is what keeps the join a lattice join (joining an unrestricted
message with a `REL-PARTNER` message gives `REL-PARTNER`, not unrestricted). On the wire a joined
label carries the union of the inputs' releasability IRIs, and `render` drops any IRI whose
partner set is a superset of another in the label, so `REL-AB` joined with `REL-ABC` renders as
`REL-AB`.

| Function | Result |
| --- | --- |
| `parse(cfg, envelopeLabel)` | struct, or error `netsim:label:unknownTerm` naming the IRI (R1.5) |
| `join(cfg, a, b)` | `level = max(rank)`, `handling = union`, `relCaveats = union`, `releasableTo = intersect` (R1.3) |
| `dominates(cfg, a, b)` | `rank(a) ≥ rank(b) && all(ismember(b.handling, a.handling)) && all(ismember(a.releasableTo, b.releasableTo))` (R1.4) |
| `joinAll(cfg, labels)` | fold of `join`; used for high-water mark (R6.1) |
| `render(cfg, label)` | display string with the `SIM` marker, e.g. `SIM-SECRET//PII//REL-PARTNER` (R2.7) |

Ranks come from the configuration's level order (index in the ordered array).

This partner-set reading is the one subtle point in the algebra; the test vectors cover the
unrestricted, single-caveat, nested, and disjoint (`releasableTo = {home}`) cases.

### C2 — `icam.LabelConfig` and the configuration format (R2, R12)

Loaded once per scenario from `scenario.labelConfig` (a path). Schema:
`schema/label-config.schema.json` (JSON Schema 2020-12). Shape:

```json
{
  "configVersion": "1.0",
  "name": "us-govt-representative",
  "simMarker": "SIM",
  "provenance": [{"title": "...", "url": "...", "note": "public source modeled on"}],
  "levels": [
    {"iri": "aocdfo:LabelLevel_U",   "display": "SIM-UNCLASSIFIED"},
    {"iri": "aocdfo:LabelLevel_CUI", "display": "SIM-CUI"},
    {"iri": "aocdfo:LabelLevel_S",   "display": "SIM-SECRET"},
    {"iri": "aocdfo:LabelLevel_TS",  "display": "SIM-TOP SECRET"}
  ],
  "caveats": [
    {"iri": "aocdfo:Caveat_PII", "display": "PII", "type": "handling"},
    {"iri": "aocdfo:Caveat_PHI", "display": "PHI", "type": "handling"},
    {"iri": "aocdfo:Caveat_RelPartner", "display": "REL-PARTNER", "type": "releasability",
     "partners": ["partner-a"]}
  ],
  "partners": ["home", "partner-a"],
  "latticeGate": true,
  "defaultLabel": {"level": "aocdfo:LabelLevel_U", "caveats": []},
  "abacRules": [
    {"enclave": "*", "role": "medic", "messageType": "*", "caveat": "aocdfo:Caveat_PHI", "decision": "permit"},
    {"enclave": "*", "role": "*",     "messageType": "*", "caveat": "aocdfo:Caveat_PHI", "decision": "deny"}
  ],
  "cdsPolicies": [ { "...": "see C5" } ],
  "review": {"n": 2, "timeoutSec": 900, "agentsMustDiffer": true, "requireHuman": false}
}
```

Notes:
- `levels` is ordered low → high; order is the rank (R2.1). Ties and cycles are impossible by
  construction (an array); the validator rejects duplicate IRIs (R2.4).
- IRIs use the `aocdfo:` prefix as a placeholder; the real namespace comes from the AOC-DFO pin
  (C9). The engine never dereferences them (R2.2, R2.3).
- **Validation order (R2.4, R12.2):** JSON Schema → referential checks (every IRI in rules,
  policies and `defaultLabel` is declared; every CDS policy names declared domains) →
  **guardrail lint** (C10). Any failure is `netsim:label:configInvalid` with the path and cause,
  and halts scenario loading — consistent with GMNS's existing `ScenarioLoader` halt-on-error
  behavior (GMNS R22.5).
- **US Government configuration** `configs/us-govt-representative.json` (R2.5): the levels and
  caveats above, `latticeGate: true`, `review.n: 2`, `agentsMustDiffer: true`, `defaultLabel` =
  the U analogue.
- **Coalition configuration** `configs/coalition-representative.json` (R2.6): three levels
  (`SIM-PUBLIC < SIM-COALITION < SIM-NATIONAL`), one handling caveat, and three releasability
  caveats over partners `{a, b, c}` with overlapping sets — `REL-A` = {a}, `REL-AB` = {a, b},
  `REL-BC` = {b, c} (each plus `home`). A join of `REL-AB` and `REL-BC` is releasable to {b};
  a join of `REL-A` and `REL-BC` is releasable to `home` only — the cases the scenario asserts.

### C3 — Domains in the scenario (R4)

`ScenarioLoader` accepts two optional enclave fields:

```json
{"enclaveId": "high_net", "cacheTtlSec": 300, "failPolicy": "closed",
 "level": "aocdfo:LabelLevel_S", "partners": ["home"]}
```

An enclave with `level` is a **domain**; its label is `{level, handling: all declared handling
caveats, releasableTo: partners}` (default `["home"]`). A domain may hold any handling caveat;
ABAC decides who inside it may read them. So a message reaches a partner domain only if that
partner is in the message's `releasableTo`. A node's domain is the domain of the enclave its entities belong to;
a node whose entities span two domains is an error unless it is a CDS gateway
(`netsim:label:nodeSpansDomains`). R4.3's bypass warning is computed after loading: for every
link whose endpoints are in different domains, warn if neither endpoint is a CDS gateway.

### C4 — `icam.LabeledDecision`: lattice gate, then ABAC (R3)

New function `icam.LabeledDecision.evaluate(cfg, pdp, request)` where `request` adds `label`,
`srcDomain`, `dstDomain`, `operation` (`send` | `receive`) to the PDP's existing arguments.

```
if no cfg loaded:                         -> pdp.evaluate(...) unchanged         (R3.7)
label = cfg.defaultLabel if absent; log defaultApplied                           (R5.4)
stage 1 (if cfg.latticeGate):
   try
     receive: require dominates(domainLabel(dst), label)          no read-up
     send:    require dominates(domainLabel(dstDomain), label)
              unless next hop is a CdsGateway                      no write-down (R3.2)
   catch any error -> deny, stage=1, reason="fail-closed: <cause>"               (R3.6)
   if fails -> return {deny, stage:1, reason:"<dst> does not dominate <label>"}
stage 2 (if cfg.abacRules non-empty or no cfg):
   r = pdp.evaluate(..., caveats=label.handling)                                 (R3.5)
   return {r.decision, stage:2, reason:r.reason}
else return {permit, stage:1, reason:"lattice permits; no ABAC rules"}           (R3.4)
```

- **Deny-overrides (R3.3)** is structural: stage 2 runs only after a stage 1 permit.
- **Caveat matching (R3.5).** `PolicyDecisionPoint.evaluate` gains an optional `caveats`
  argument and rules gain an optional `caveat` field. A rule with a `caveat` matches only if that
  IRI is in the request's handling caveats. To keep "all caveats must be authorized" semantics
  with first-match rules, `LabeledDecision` calls `evaluate` **once per handling caveat** (plus
  once with no caveat for the message type) and denies if any call denies. Existing rules
  (no `caveat` field) behave exactly as today.
- **Stage 1 never touches the network or the `CredentialCache`** (R3.6). The cache key for stage 2
  results gains the caveat IRI, so a cached PHI decision is never reused for a non-PHI request.
- **Fail-closed is independent of `failPolicy` (R3.6).** `failPolicy` still governs stage 2 when
  the PDP node is unreachable over the network; stage 1 has no unreachable state.
- Result struct (R3.8): `{decision, stage (0 = no config | 1 | 2), reason, labelRendered,
  defaultApplied}`.
- `PolicyEnforcementPoint.checkSend` / `checkReceive` call `LabeledDecision.evaluate` instead of
  `pdp.evaluate` when a configuration is loaded; the denied-log gains `stage` and `labelRendered`.

### C5 — `network.CdsGateway` node type (R7)

Scenario declaration:

```json
{"nodeId": "CDS_1", "type": "cds_gateway", "domains": ["low_net", "high_net"],
 "policy": "low-high", "queueCapacity": 50,
 "latency": {"dist": "lognormal", "muSec": 1.2, "sigma": 0.4},
 "outages": [{"startSec": 3600, "durationSec": 600}]}
```

Release policy (in the configuration's `cdsPolicies`):

```json
{"policyId": "low-high",
 "rules": [
   {"direction": "high->low", "messageType": "MissionReport",
    "match": {"maxLevel": "aocdfo:LabelLevel_S", "handling": ["aocdfo:Caveat_PII"]},
    "action": "sanitize", "sanitizeRule": "strip-pii", "outputLabel": {"level": "aocdfo:LabelLevel_U", "caveats": []}},
   {"direction": "high->low", "messageType": "ExecutionStatus",
    "match": {"maxLevel": "aocdfo:LabelLevel_S"}, "action": "hold"},
   {"direction": "low->high", "messageType": "*", "match": {}, "action": "allow"}
 ],
 "allowList": {"MissionReport": ["id","messageType","timestamp","missionId","producedBy",
                                 "doctrineSource","payload.summary","payload.status"]},
 "sanitizeRules": {"strip-pii": {"removeFields": ["payload.crewRoster", "payload.casualties"]}}}
```

Pipeline per message (R7.2), first-match on rules:

| Step | On failure |
| --- | --- |
| 1. queue admission | `deny`, cause `queueOverflow` (R7.3) |
| 2. envelope schema validation (ADR 0012 `d2.2` schema) | `deny`, cause `malformed` |
| 3. release-policy rule match (direction, messageType, label within `match`) | `deny`, cause `noRule` |
| 4. allow-list: drop any field not listed for the type (for `allow` / `hold` results) | — |
| 5. action | allow → forward; deny → drop + log; sanitize → C5.1; hold → C6 |

- **C5.1 Sanitize (R7.4):** apply `removeFields`, then the allow-list; build a new envelope with a
  fresh `id`, `label = outputLabel`, `derivedFrom = [originalId]`, `sanitization = {recordId}`;
  write the sanitization record to the Event_Log and the `fabric.ProvenanceGraph` (an edge
  original → sanitized, typed `sanitizedBy`). The original is dropped at the gateway.
- **Latency and failures (R7.5):** processing delay sampled per message from `latency` (normal,
  lognormal, or fixed); gateway availability uses `network.OutageEngine` with the node's
  `outages`. While the gateway is out, arriving messages are denied with cause `gatewayDown`
  (fail-closed), not queued.
- **Directions.** A rule's `direction` is relative to the two domains' ranks (`low->high`,
  `high->low`); a gateway between equal-rank domains with different releasability uses
  `a->b` / `b->a` by domain id.
- **No product modeling (R7.6):** the pipeline is exactly the five steps above; there is no
  content-inspection engine, file-type filter, or transfer protocol.

### C6 — `icam.ReviewCoordinator`: two-entity review (R8)

- On `hold`, the coordinator creates a review case `{caseId, messageId, gatewayId, openedAt,
  deadline = openedAt + review.timeoutSec, approvals: []}` and sends a `REVIEW_REQUEST`
  C2_Message to each entity holding the `reviewer` Role_Binding in the gateway's enclave (R8.2).
- Reviewers answer with `REVIEW_DECISION` C2_Messages `{caseId, reviewerId, kind, model?, decision,
  time}`; they travel through the network, so latency and outages apply (R8.3).
- **Reviewer kind.** `EntityRegistry` entities gain an optional `agentModel` string. Kind is
  `person` when `entityType = human`; `agent` when `entityType = NPE` **and** `agentModel` is set.
  An NPE without `agentModel` (a sensor, a platform) cannot hold the reviewer role — the loader
  rejects that binding (`netsim:label:reviewerNotEligible`), mirroring GMNS's existing
  `netsim:icam:policyViolation` check for human-restricted roles.
- **Release rule:**
  ```
  any decision == reject                         -> deny  (R8.4)
  distinct = unique(approvals.reviewerId)        (R8.5)
  if agentsMustDiffer: among agent approvals, count unique(agentModel) instead of ids
  if requireHuman and no person approval: not yet satisfied (R8.6)
  if count(distinct) >= review.n              -> release (with allow-list applied)
  at deadline                                  -> deny, cause reviewTimeout (R8.1)
  ```
  Concretely with `n = 2, agentsMustDiffer = true`: person + agent ✔; person + person ✔;
  agent(model X) + agent(model Y) ✔; agent(X) + agent(X) ✘ (counts as one).
- The released envelope's `approvals` carries every approval event (R8.3).
- In GMNS, agent reviewers are driven by the existing `+agent` package (`AgentRegistry`,
  `LLMClient`); in scripted scenarios a deterministic stub reviewer emits fixed decisions so the
  acceptance test is reproducible.

### C7 — Envelope `d2.2` additions (R5, R6, R9)

Added to `simulation-interface`'s envelope schema; all optional so `d2.1` stays valid (R5.1):

| Field | Type | Req. | C2SIM mapping (D4/D5 intent) |
|---|---|---|---|
| `envelopeVersion` | `"d2.1"` \| `"d2.2"` | R5.1 | unchanged intent |
| `label` | `{level: IRI, caveats: [IRI]}` | R5.1 | **open mapping Q** — C2SIM has no security-marking construct in its core; candidate: extension element |
| `derivedFrom` | `[envelope id]` | R5.2, R6.1 | **open mapping Q** |
| `sanitization` | `{recordId, gatewayId}` | R5.2, R7.4 | **open mapping Q** |
| `approvals` | `[{caseId, reviewerId, kind, model?, decision, time}]` | R5.3, R8.3 | **open mapping Q** |
| `qos` | `{priority: int ≥ 0, latencyClass: enum, minBandwidthKbps: number}` | R9.1 | **open mapping Q** |

Each row is added to the `simulation-interface` C2SIM mapping table as an open question (R5.5).
`latencyClass` values are `realtime | interactive | bulk` (an interface only; the GMNS QoS spec
gives them meaning, R9.2). Until then GMNS carries `qos` unchanged.

**Labeling-violation check (R6.2).** On receipt, a component with the configuration loaded
checks `dominates(label, joinAll(labels of derivedFrom))` when the inputs are known to it; if
not, and there is no `sanitization` or approved `approvals` on the message, it rejects with
`netsim:label:labelingViolation`. Inputs a component has never seen are skipped (the check is
best-effort per component; GMNS sees every message so it is the authoritative checker).

### C8 — Observability (R11, R5.4, R6.3)

Event_Log entries (one struct per event, existing GMNS log): `labelDecision`, `cdsAction`,
`sanitization`, `reviewRequest`, `reviewDecision`, `reviewOutcome`, `defaultLabelApplied`,
`labelingViolation`. Statistics_Report adds a **Labels** section:

- per domain: permits / denies by stage; defaults applied; messages per rendered label;
  messages per derivation depth (length of the longest `derivedFrom` chain) — R6.3;
- per CDS gateway: actions by type; fail-closed causes; review hold time (p50, p95, max);
  timeouts; approvals by reviewer kind.

Records keep their IRIs and rendered labels, so an export (for example AKS ADR 0014) carries
labels unchanged (R11.3).

### C9 — AOC-DFO label terms (D7, R2.2)

A small representative module in AOC-DFO, through that repo's own spec process:
`LabelLevel` and `HandlingCaveat` / `ReleasabilityCaveat` classes, aligned under CCO's
information-content-entity hierarchy, with one individual per level and caveat used by the two
shipped configurations. The ordering and release rules are **not** modeled in OWL (D7: the
configuration owns behavior). Until the module lands, configurations use the `aocdfo:`
placeholder prefix, and a task swaps in the pinned namespace.

### C10 — Guardrails (R12)

- **Deny-list** `schema/guardrail-denylist.json`: case-insensitive tokens that are real
  dissemination controls or banner-line forms the shipped configurations must not contain, for
  example `NOFORN`, `ORCON`, `REL TO`, `FVEY`, `SCI`, `SAP`, `TS//`, `S//`, `//NF`, `IMCON`, and
  the bare words `CLASSIFIED BY` / `DERIVED FROM` / `DECLASSIFY ON`. Display names must also start
  with the configuration's `simMarker` (for levels) so a bare `SECRET` cannot appear.
- The lint runs at load time in GMNS (C2) **and** in this repo's CI over `configs/` (R12.2).
- `provenance` is required by the schema with at least one entry (R12.3).
- User-facing strings produced by GMNS reports say "simulated handling label"; the report header
  states the engine is a simulation, not a marking or accreditation tool (R12.4).

The deny-list is itself public (all tokens appear in public marking guides); it lists strings to
*avoid*, and no guidance on how to apply them.

### C11 — ENCAP alignment (R10)

No ENCAP code in this spec. The contract: ENCAP's `access-control/` rule format MAY add a
`labelIri` match field referencing the same AOC-DFO IRIs; ENCAP's echelon scope stays separate.
The envelope already carries `producedBy` (echelon context) and `label` side by side (D6). A
follow-on task in ENCAP's `.kiro/specs/` picks this up when ENCAP adopts labels.

## Data flow: one cross-domain message

1. A high-domain entity sends a `MissionReport` labeled `SIM-SECRET//PII` toward a low-domain node.
2. `checkSend`: stage 1 sees the next hop is `CDS_1`, so no-write-down is deferred to the gateway;
   stage 2 ABAC permits. The router forwards to `CDS_1`.
3. `CDS_1`: admitted, schema-valid, matches the `sanitize/strip-pii` rule. A new message labeled
   `SIM-UNCLASSIFIED`, `derivedFrom = [orig]`, `sanitization = {rec-17}` leaves for the low domain; the
   original is dropped.
4. `checkReceive` at the low node: stage 1 `dominates(low_net, SIM-UNCLASSIFIED)` ✔; labeling-
   violation check passes because `sanitization` is present.
5. Event_Log: one `labelDecision` per check, one `cdsAction`, one `sanitization`.

## Correctness properties

1. `join` is commutative, associative, idempotent; `dominates` is reflexive, antisymmetric,
   transitive; `dominates(join(a,b), a)` and `dominates(join(a,b), b)` (R1.6).
2. No stage 2 permit can produce a final permit after a stage 1 deny (R3.3).
3. With a configuration loaded, no labeled message is delivered to a domain that does not
   dominate its label, except as a CDS output with a sanitization record or approvals (R4.2, R6.2).
4. Every lowered label is traceable to exactly one sanitization record or review outcome (R6.2).
5. A held message is released only with ≥ N distinct qualifying approvals and zero rejects (R8).
6. Every CDS failure path denies (R7.3).
7. With no configuration loaded, GMNS behavior is byte-identical on existing scenarios (R3.7).
8. The engine contains no level or caveat literal (R2.3) — checked by a grep test over `+icam`
   and `+network` for `aocdfo:` and the shipped display names.

## Testing strategy

- **Test vectors (R14.2)** in `vectors/`, JSON, one file per area:
  `algebra.json` (label pairs → join, dominance — including the empty-releasability cases),
  `decisions.json` (request + config → stage, decision, reason code),
  `cds.json` (message + policy → action, cause, output label),
  `review.json` (approval sequences → release | deny | pending).
  GMNS runs them through a `matlab.unittest` parameterized test (R14.3).
- **Property tests (R1.6):** randomized labels over both shipped configurations, asserting
  property 1; MATLAB has no property-testing library in GMNS today, so the test generates seeded
  random labels in a loop (seed logged for reproduction).
- **Config tests (R2.4, R12):** each invalid-config case (unknown IRI, duplicate level, untyped
  caveat, unknown domain, deny-list hit, missing provenance) yields the expected error id.
- **Regression (R3.7, R13.4):** the existing `tests/icam/*` and `tests/sim/SimControllerICAMTest`
  suites run unchanged; property 7 is asserted by diffing Event_Logs of existing scenarios with
  and without this code.
- **Acceptance (R13):** scenario `scenarios/labeled_cds/` in GMNS — two domains (`low_net` at U
  analogue, `high_net` at S analogue), one `CDS_1`, a medic and a non-medic role, two human and
  two agent reviewers (agents with models X, X, and Y available). Assertions per R13.2, stated
  once per configuration (R13.3). Agent reviewers use the deterministic stub.
- **Umbrella CI:** schema-validate both configs and all vectors; run the guardrail lint.

## Error identifiers (new, GMNS convention `netsim:<area>:<type>`)

| Identifier | Condition |
| --- | --- |
| `netsim:label:configInvalid` | configuration fails schema, referential, or guardrail check |
| `netsim:label:unknownTerm` | a label names an undeclared IRI |
| `netsim:label:nodeSpansDomains` | a non-CDS node's entities belong to two domains |
| `netsim:label:reviewerNotEligible` | a reviewer binding on an NPE without `agentModel` |
| `netsim:label:labelingViolation` | a lowered label without a sanitization record or approvals |

Bypass links (R4.3) are a warning, not an error.

## Open questions (for tasks, not blocking design)

- **Explicit `releasableTo` on the wire:** the envelope carries caveat IRIs and the receiver
  computes partner sets from its configuration. Revisit if Foundry ingestion (AKS ADR 0014) needs
  the computed set without the configuration.
- **AOC-DFO class placement**: exact CCO parent for `LabelLevel` (information content entity vs.
  a designative ICE); decided in AOC-DFO's spec, not here.
- **Agent reviewer prompts**: what an LLM reviewer is shown (envelope fields after allow-list
  only, label rendered) — left to the GMNS `+agent` task; the acceptance test uses stubs.
