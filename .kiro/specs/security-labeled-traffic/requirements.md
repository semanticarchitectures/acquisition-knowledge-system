# Requirements — Security-Labeled Traffic and Cross Domain Solutions

**Spec:** `.kiro/specs/security-labeled-traffic/requirements.md`
**Decision:** implements [AKS ADR 0013](../../docs/adr/0013-security-labeled-traffic-and-cross-domain-solutions.md)
(Accepted 2026-11-22, decisions D1–D7). Extends [AKS ADR 0012](../../docs/adr/0012-aks-simulation-interface.md)
(the D2 envelope) and the `simulation-interface` spec.
**Scope:** a general-purpose label / policy / release-gateway **engine**, a representative and
public **US Government configuration** loaded into it, the envelope fields that carry labels, and
the GMNS behavior that enforces them. Cross-cutting: enforcement lands in GMNS, the schema and
configuration format in this repo, label terms in AOC-DFO, and label-aware scope rules in ENCAP.

## Introduction

AKS ADR 0013 decided to model network traffic whose identity defines both its security handling
and its quality of service, with gateways between networks that mirror US Government Cross Domain
Solutions (CDS). The design separates a reusable engine from the policy it enforces, the way real
multi-level and cross-domain systems separate the guard from the approved policy.

This spec turns D1–D7 into buildable requirements. It does not re-decide them; where a
requirement seems to, defer to AKS ADR 0013.

Grounding (read at GMNS `main` 4fffc02, 2026-11-14, and AKS `main`, 2026-11-22):
- GMNS `+icam` has a `PolicyDecisionPoint` that walks rules **in order, first match wins**, matching
  `(enclave, role, messageType)` strings with `*` wildcards, and falls back to the enclave's
  `failPolicy` (`open` | `closed`) when no rule matches. The `airdrop_icam` scenario sets
  `failPolicy: "open"`.
- GMNS already encodes classification as a string convention, `messageType = 'data_item:<classification>'`
  (GMNS requirement R36), evaluated by exact or wildcard match. There is no level ordering.
- GMNS **Enclaves** are named security domains with their own policy and trust anchors; an Entity
  (human or Non_Person_Entity, including AI agents) holds Role_Bindings in one or more Enclaves.
- GMNS has no QoS model (no priority, latency class, or bandwidth reservation on traffic) — see
  open question O3.
- The D2 envelope (`simulation-interface` design) carries `envelopeVersion` (`"d2.1"`), `id`,
  `messageType`, `timestamp`, `missionId`, `producedBy`, `doctrineSource[]`, `source`, `destination`.

Terms used here: a **label** is `(level, caveats)`. A **domain** is a GMNS Enclave that carries a
level (R4). A **configuration** is a data file the engine loads; the engine itself knows no scheme.

---

### Requirement 1 — Label model and algebra (engine)

**User Story:** As a simulation engineer, I want one label model that any configuration can fill
in, so that the same engine can run the US Government scheme, a coalition scheme, or a commercial
one.

**Acceptance Criteria:**

1.1. THE engine SHALL represent a label as one **level** plus zero or more **caveats**, where the
levels and caveats are declared by the loaded configuration, not by the engine.

1.2. THE engine SHALL require each caveat to be typed as **handling** (restricts which subjects
may see it) or **releasability** (restricts which domains or partners it may go to).

1.3. THE engine SHALL compute the **join** of two labels as: the higher level; the union of
handling caveats; the intersection of releasability caveats.

1.4. THE engine SHALL compute **dominance** as: label A dominates label B WHEN A's level ≥ B's
level AND A's handling caveats ⊇ B's AND A's releasability caveats ⊆ B's.

1.5. IF a label names a level or caveat that the loaded configuration does not declare, THEN THE
engine SHALL reject the label with an error naming the unknown term, and SHALL NOT treat it as
the lowest level.

1.6. THE engine SHALL provide join and dominance as pure functions with property-based tests for
the lattice laws (join is commutative, associative and idempotent; dominance is a partial order;
the join of A and B dominates both).

### Requirement 2 — Configurations (engine / policy split)

**User Story:** As a demonstrator, I want the US Government model to be a file the engine loads,
so that I can show the same engine under a second configuration without code changes.

**Acceptance Criteria:**

2.1. THE engine SHALL load exactly one configuration per scenario from a versioned data file
whose format this spec's design defines (levels and their order, caveats and their types,
lattice-gate on/off, ABAC rules, CDS release policies, review rules, default label for unlabeled
traffic).

2.2. THE configuration SHALL identify every level and caveat by an **AOC-DFO IRI** (D7) and MAY
give each a short display name. THE engine SHALL treat IRIs as opaque identifiers.

2.3. THE configuration SHALL own the level order, caveat types, and release rules; THE engine
SHALL NOT read any of these from the ontology at run time.

2.4. WHEN a configuration is loaded, THE engine SHALL validate it against its JSON schema and
reject it with a descriptive error on: an undeclared IRI reference, a cycle or tie in the level
order, a caveat with no type, or a CDS policy that references an unknown domain.

2.5. THE SYSTEM SHALL ship the **US Government configuration** with: four levels representing
UNCLASSIFIED < CUI < SECRET < TOP SECRET; handling caveats PII and PHI; one releasability caveat
`REL-PARTNER`; lattice gate on; two-entity review (R8). FOUO SHALL NOT appear as a separate term;
it is folded into the CUI analogue (D2).

2.6. THE SYSTEM SHALL ship **one second configuration** (coalition or commercial — see O2) that
runs on the same engine with no code change, as the proof of the engine/policy split.

2.7. EVERY display name in a shipped configuration SHALL carry the marker `SIM`, and every label
rendered by any AKS component (logs, reports, consoles) SHALL show it.

### Requirement 3 — Policy decision: lattice gate, then ABAC (D1)

**User Story:** As a policy author, I want mandatory level rules that no permissive rule can
override, so that the model behaves like real mandatory access control.

**Acceptance Criteria:**

3.1. THE GMNS `PolicyDecisionPoint` SHALL evaluate a labeled request in two stages: **stage 1**,
the lattice gate (R3.2); **stage 2**, the existing ordered ABAC rules, evaluated only if stage 1
permits.

3.2. WHILE the configuration has the lattice gate on, THE PDP stage 1 SHALL deny a read WHEN the
subject's domain level does not dominate the label (no read-up), and SHALL deny a flow WHEN the
destination domain does not dominate the label (no write-down), unless the flow passes through a
CDS gateway (R7).

3.3. THE PDP SHALL combine stages by **deny-overrides**: a stage 1 deny SHALL be final regardless
of any stage 2 permit.

3.4. WHERE the configuration turns the lattice gate off, THE PDP SHALL evaluate stage 2 alone;
WHERE the configuration supplies no ABAC rules, THE PDP SHALL decide on stage 1 alone and permit
what it permits.

3.5. THE ABAC stage SHALL keep GMNS's existing first-match semantics and `*` wildcards, and SHALL
gain the ability to match on handling caveats (for example, "permit role `medic` to read PHI").

3.6. THE PDP SHALL evaluate stage 1 locally, without a network round-trip, and SHALL fail closed:
IF stage 1 cannot be evaluated (no configuration, unknown term), THEN THE PDP SHALL deny,
irrespective of the enclave's `failPolicy`.

3.7. WHERE a scenario loads no configuration, THE PDP SHALL behave exactly as it does today,
including the `data_item:<classification>` convention, so existing GMNS scenarios and tests pass
unchanged.

3.8. EVERY decision SHALL return the deciding stage and a reason (the lattice relation that
failed, or the matched rule index), extending GMNS's existing `{decision, reason}` result.

### Requirement 4 — Domains

**Acceptance Criteria:**

4.1. THE GMNS scenario format SHALL let an Enclave declare a **level** and an optional set of
releasability caveats, making it a domain; an Enclave with no level SHALL be unlabeled and
behave as today.

4.2. THE GMNS router SHALL NOT deliver a labeled message from one domain to another whose label
does not dominate the message's label except through a CDS gateway node (R7).

4.3. IF a scenario connects two domains with a link that bypasses every CDS gateway, THEN THE
scenario loader SHALL warn, and the router SHALL enforce R4.2 on that link.

### Requirement 5 — Envelope label fields (extends AKS ADR 0012)

**User Story:** As any AKS component, I want the label to travel with the message, so that every
hop and every consumer can enforce and display it.

**Acceptance Criteria:**

5.1. THE D2 envelope SHALL gain an optional `label` field `{level: IRI, caveats: [IRI]}` and
increment `envelopeVersion` to `"d2.2"`; a `"d2.1"` message SHALL remain valid.

5.2. THE envelope SHALL gain an optional `derivedFrom` list of envelope `id`s and an optional
`sanitization` record reference (R7.4), so a lowered label always points to the event that
lowered it.

5.3. THE envelope SHALL gain an optional `approvals` list of review events (R8.3).

5.4. WHEN a message has no `label`, THE engine SHALL apply the configuration's declared default
label (see O1), and SHALL record that the default was applied.

5.5. THE `simulation-interface` C2SIM mapping table SHALL gain a row per new field, flagged as an
open mapping question where C2SIM has no construct.

### Requirement 6 — Derived information (D4)

**Acceptance Criteria:**

6.1. WHEN a component produces a message from one or more labeled inputs, THE component SHALL
label the output with the join (R1.3) of all input labels and list the inputs in `derivedFrom`.

6.2. THE SYSTEM SHALL lower a label only through a recorded CDS sanitization (R7.4) or an
approved review (R8); IF a message's label is lower than the join of its `derivedFrom` inputs
without such a record, THEN any receiving AKS component SHALL reject it as a labeling violation.

6.3. THE GMNS Statistics_Report SHALL count messages per label and per derivation depth. The
engine cannot decide whether a label is higher than the content needs, so this count is a
signal of over-labeling drift, not a verdict.

### Requirement 7 — CDS gateway node (D3, D5)

**User Story:** As a simulation engineer, I want a gateway node between domains that behaves like
a representative cross-domain guard, so that cross-domain sharing and its delays and failures show
up in the simulation.

**Acceptance Criteria:**

7.1. THE GMNS scenario format SHALL define a **CDS gateway** node type that connects exactly two
domains and loads a release policy from the configuration.

7.2. WHEN a message reaches a CDS gateway, THE gateway SHALL, in order: validate the envelope
against the ADR 0012 schema; check the label against the release policy; apply the per-message-type
**field allow-list**; then take exactly one release action — **allow**, **deny**, **sanitize**, or
**hold-for-review**.

7.3. THE gateway SHALL fail closed: IF validation fails, the release policy has no matching rule,
or the gateway's queue is full, THEN THE gateway SHALL deny and record the cause
(`malformed`, `noRule`, `queueOverflow`).

7.4. WHEN the action is **sanitize**, THE gateway SHALL emit a **new** message with the redacted
fields removed, a label lowered as the named rule specifies, `derivedFrom` = the original `id`,
and a **sanitization record** (rule id, fields removed, input label, output label, gateway id,
time); THE original SHALL NOT cross.

7.5. THE gateway SHALL model processing latency from a configurable distribution and an
availability / failure schedule, using GMNS's existing outage mechanism where possible.

7.6. THE gateway SHALL NOT model any real CDS product, real content-inspection engine, or real
transfer protocol (D5, guardrail).

### Requirement 8 — Two-entity review (D3)

**User Story:** As a policy author, I want held transfers to need approval from more than one
independent reviewer, where a reviewer can be a person or an AI agent, so that the model covers
human–AI and AI–AI control as well as two-person control.

**Acceptance Criteria:**

8.1. WHEN the action is **hold-for-review**, THE gateway SHALL hold the message until it has
approvals from **N distinct reviewer entities**, where N is set in the configuration (US
Government configuration: N = 2), or until a configured timeout, after which it SHALL deny.

8.2. A reviewer entity SHALL be a GMNS ICAM Entity of kind **person** or **agent** (a
Non_Person_Entity that is an AI agent) holding a reviewer Role_Binding in the gateway's
enclave. Valid sets include person + agent, two persons, and two agents.

8.3. EACH approval event SHALL record reviewer entity id, kind, decision (approve | reject), time,
and the message `id`; approval events SHALL travel as GMNS C2_Messages subject to network delay,
and SHALL be attached to the released message's `approvals` list.

8.4. ANY single **reject** SHALL deny the transfer.

8.5. THE engine SHALL count two entities as distinct WHEN their ICAM identities differ;
WHERE the configuration sets `agentsMustDiffer` THE engine SHALL additionally require two agent
reviewers to declare different `model` values. (Whether the US Government configuration sets it
is O4.)

8.6. WHERE the configuration sets `requireHuman`, THE engine SHALL require at least one approval
from a person entity.

### Requirement 9 — QoS by stream identity

**User Story:** As a network analyst, I want a stream's identity to carry its QoS as well as its
label, so that security and service level are one model.

**Acceptance Criteria:**

9.1. THE envelope or GMNS stream definition SHALL carry a QoS class: priority (ordinal), latency
class, and minimum bandwidth.

9.2. WHILE a link is congested or degraded, THE GMNS router SHALL serve higher-priority traffic
first and SHALL record when a stream's latency class is missed.

9.3. THE QoS class SHALL be independent of the label: THE engine SHALL NOT infer priority from
level, and a configuration MAY map labels to default QoS classes explicitly.

### Requirement 10 — Alignment with ENCAP echelon scopes (D6)

**Acceptance Criteria:**

10.1. THE label model SHALL NOT encode echelon layer, and ENCAP's echelon scope SHALL NOT encode
level or caveats; they are two independent axes.

10.2. WHERE ENCAP adopts labels, ENCAP's access-control rule format SHALL reference labels by the
same AOC-DFO IRIs (R2.2), so both layers share one vocabulary.

### Requirement 11 — Observability

**Acceptance Criteria:**

11.1. THE GMNS Event_Log SHALL record every labeled decision (stage, reason), every CDS action,
every sanitization record, and every review event.

11.2. THE GMNS Statistics_Report SHALL add, per domain and per CDS gateway: permits and denies by
stage, actions by type, review hold times, review timeouts, and fail-closed causes.

11.3. THE records SHALL be exportable unchanged to downstream consumers (for example AKS ADR 0014,
Foundry), labels included.

### Requirement 12 — Guardrails (AKS ADR 0013, non-negotiable)

**Acceptance Criteria:**

12.1. THE SYSTEM SHALL contain only representative, public label terms and fictional release
rules; no real classification guide, accreditation artifact, control baseline, or real marking.

12.2. THE configuration validator SHALL reject a configuration whose display names or rule text
contain a real dissemination-control or banner-marking string from a deny-list maintained in this
spec's design, so a shipped configuration cannot be mistaken for real marking guidance.

12.3. EVERY shipped configuration SHALL include a provenance note naming the public sources it was
modeled on (per the project-family citation and provenance rules).

12.4. NO component SHALL present the engine as a classification, marking, or accreditation tool;
user-facing text SHALL describe it as a simulation.

### Requirement 13 — Acceptance scenario

**User Story:** As the gate for this capability, I want one scenario that exercises every decision,
so that "it works" is a passing test.

**Acceptance Criteria:**

13.1. THE SYSTEM SHALL define one GMNS scenario with at least two domains (a low and a high
analogue) joined by one CDS gateway, carrying labeled envelope traffic.

13.2. THE scenario SHALL exercise and assert: a lattice-gate deny that an ABAC rule would have
permitted; an ABAC caveat deny (PHI to an unauthorized role); each CDS action (allow, deny,
sanitize, hold-for-review); a two-entity approval by person + agent; a two-entity approval by two
agents; a single-reject deny; a review timeout; a high-water-mark derived message; and a
fail-closed queue overflow.

13.3. THE same scenario SHALL run under the second configuration (R2.6) with only the
configuration file changed, and its assertions SHALL be stated per configuration.

13.4. Existing GMNS ICAM tests SHALL pass unchanged (R3.7).

### Open questions (flagged, not decided here)

- **O1 — Default label for unlabeled traffic** in the US Government configuration: lowest level
  (keeps D2 `"d2.1"` traffic flowing) or reject unlabeled traffic once a configuration is loaded
  (safer, breaks D2 compatibility). Proposed: lowest level, with the default recorded per R5.4.
- **O2 — Second configuration:** coalition (releasability-heavy, exercises `REL-*` intersection)
  or commercial (handling-heavy, for example customer data and trade secrets). Proposed: coalition,
  because it exercises the releasability algebra the US configuration only touches once.
- **O3 — QoS home:** GMNS has no QoS model today. Is R9 part of this spec or a separate GMNS spec
  this one depends on? Proposed: a separate GMNS component spec; R9 here states only the interface.
- **O4 — `agentsMustDiffer`** in the US Government configuration: do two AI reviewers need
  different models to count as independent? Proposed: on, so AI–AI review means two different
  models, not two instances of one.
- **O5 — Where the engine lives:** in GMNS `+icam` (MATLAB) only, or also as a cross-language
  shared package so ENCAP and ENSIM can evaluate labels too. AKS ADR 0008 defers shared packages;
  proposed: MATLAB in GMNS now, with the configuration format and test vectors in this repo as the
  portable contract.

### Out of scope

- Real classification guidance, CDS accreditation, or control baselines (guardrail).
- Foundry ingestion of labeled data (AKS ADR 0014, its own spec).
- C2SIM / UCI security-marking constructs (the D4/D5 content-profile follow-on to AKS ADR 0012).
- ENCAP implementing label-aware access rules (R10 states the alignment; ENCAP's own spec builds it).
