# 0013. Security-labeled traffic streams and Cross Domain Solutions (representative, public)

- Status: **Proposed** (2026-11-21) — captures the decisions to make before a spec is written
- Relates: [ADR 0007](0007-gmns-joins-aks.md) and the GMNS `matlab-network-sim` ICAM layer (identity, credential, access management); [ADR 0012](0012-aks-simulation-interface.md) (the simulation-interface envelope that will carry labels); [ADR 0005](0005-one-model-many-apps.md) (one shared model); the vision's "Secure" property (classification markings, echelon scopes, a reviewed promotion step)
- Scope note: **representative and public only.** This models the *structure* of US Government data security — not any real classified ruleset, accreditation artifact, or control baseline.

## Context

A new high-level objective (Kevin, 2026-11-21): model **network traffic streams whose identities
carry security requirements and quality of service**, mirroring the US Government approach to data
security — broad classifications plus detailed data-type restrictions (PII, HIPAA, FOUO, SECRET),
access controls, and QoS on network/cyber and C2 traffic — **with gateways between networks that
mirror US Government Cross Domain Solutions (CDS)**, and the requirements for securely sharing
approved data across domains.

This is not greenfield. AKS already has the pieces this builds on:
- **GMNS ICAM** already models identity, credentials, authentication, and a policy
  decision/enforcement point that gates C2 message routing. Security labels + release rules are an
  extension of this policy engine, not a new one.
- **The ADR 0012 envelope** already carries `producedBy` (actor identity) and a required
  `doctrineSource`. A classification/handling label is a natural new envelope field.
- **The vision's "Secure" property** already names "classification markings, echelon scopes, scoped
  tokens, a reviewed promotion step" — this objective makes that property concrete and testable.
- GMNS already has nodes, links, and a `+security` package (adversarial/ policy analysis).

The decision direction is set (Kevin, 2026-11-21): **extend ICAM + the envelope**, and model the
structure **faithfully but with fictional/public rules only**. What remains are the design questions
a spec must answer, recorded here before the spec is written.

## Decision (direction)

**Model security-labeled traffic and CDS gateways as an extension of the GMNS ICAM/policy layer and
the ADR 0012 envelope, using a representative, public label set and fictional release rules.** No
real classification guide, control baseline, or accreditation content enters the repo (consistent
with the handling rules already in GMNS and AOC-DFO).

Shape of the model (to be specified, not fixed here):
- **Labels on traffic.** Each C2 message / traffic stream carries a handling label: a broad level
  (representative analogues of UNCLASSIFIED / FOUO / SECRET) **plus** zero or more data-type caveats
  (representative analogues of PII, HIPAA, PHI, FOUO). Labels are envelope fields (extends ADR 0012).
- **Networks as domains.** GMNS networks are tagged with a domain security level; a node belongs to
  a domain. Traffic may flow within a domain subject to the policy engine.
- **CDS gateways.** A new GMNS node type sits between two domains and enforces a **release policy**:
  it inspects a message's label, applies representative release rules (allow / deny / sanitize /
  hold-for-review), and models the latency and failure modes of a real guard. This mirrors the CDS
  pattern (a reviewed, accredited boundary) in behavior, not in accreditation content.
- **QoS by identity.** A traffic stream's identity also carries QoS requirements (priority,
  bandwidth, latency class), which GMNS's existing link/outage model honors or degrades — so
  "security requirements and QoS defined by stream identity" is one model, not two.
- **Enforcement by ICAM.** GMNS's existing policy decision/enforcement point evaluates the combined
  identity+label+QoS against policy, reusing its ABAC-style rule mechanism rather than a new engine.

## Open questions (the spec must decide these)

1. **MAC vs. ABAC.** Are labels enforced as a **mandatory** lattice (Bell–LaPadula: no read-up, no
   write-down, a formal dominance relation) or as **attribute-based** policy (the ICAM engine's
   current style)? Real classification is mandatory; GMNS's ICAM is attribute-based. The spec must
   choose, or define how a lattice overlays the ABAC engine. *This is the central design question.*
2. **Label model.** How many broad levels, which representative caveats, and how caveats compose
   (e.g., SECRET//PII). Define the label algebra (dominance, combination, the "high-water mark" of a
   derived message).
3. **CDS release-rule model.** What release actions (allow/deny/sanitize/hold), how "sanitize" is
   represented (field redaction? downgrade with evidence?), and what a "reviewed, approved" cross-
   domain transfer requires in the model (a human-in-the-loop approval event, mirroring two-person
   control?).
4. **Derived-information labeling.** When processing creates new information from labeled inputs
   (this connects to the Foundry objective, ADR 0014), what label does the output carry? High-water
   mark by default — confirm.
5. **Scope of "mirror CDS."** How faithfully to model the guard's behavior (filter types, content
   inspection, transfer protocols) vs. keeping it a representative black box. Stay public; decide
   depth.
6. **Relationship to KB/ENCAP echelon scopes.** The vision's "echelon scopes, scoped tokens" (ENCAP,
   ADR 0009) and this network-layer labeling are two expressions of the same concern. Are they one
   model or two aligned ones? (Likely align, not merge — different layers.)
7. **AOC-DFO alignment.** Should handling labels be ontology terms aligned to the shared BFO/CCO
   model (so a label is a first-class modeled thing, not just a string), consistent with ADR 0005?

## Hard guardrails (non-negotiable, carry into the spec)

- **Public and representative only.** No real classification guide, CDS accreditation artifact,
  control baseline (e.g., specific Raise-the-Bar requirements), or marking that could be mistaken for
  genuine classified guidance. Labels are fictional analogues, clearly marked as such.
- **No tool that could be mistaken for a real accreditation or marking system.** The model
  demonstrates structure and behavior for a simulation; it is not a classification tool.
- Inherits the repos' existing "Phase 1 is public data only" rule.

## Consequences

- A new umbrella objective and vision entry (makes the "Secure" property concrete).
- A future `.kiro/specs/security-labeled-traffic/` spec (cross-cutting; enforcement lands in GMNS),
  gated on resolving the open questions above — especially Q1 (MAC vs. ABAC).
- Envelope extension (ADR 0012): label fields, to be added in the follow-on, not retrofitted now.
- Connects to ADR 0014 (Foundry): labeled data ingested and enriched must carry labels through.

## Alternatives considered

| Option | Why not (now) |
| --- | --- |
| New standalone security subsystem, loosely coupled to GMNS | Rejected (Kevin): duplicates the ICAM identity/policy work already specified; labels and QoS belong on the traffic GMNS already models. |
| Abstract multi-level-security mechanism, US Govt as just one instance | Reasonable, but the objective specifically asks to mirror the US Govt approach (PII/HIPAA/FOUO/SECRET, CDS); model that structure directly, kept public. |
| Deep real-doctrine mapping (ICD 503, Raise-the-Bar, real control baselines) | Rejected: crosses the public-only guardrail and risks producing something mistakable for real accreditation content. |
| Write the full spec now | Rejected (Kevin): the MAC-vs-ABAC question (Q1) is a real decision; specifying before deciding repeats the drift pattern. Objective + this ADR first. |
