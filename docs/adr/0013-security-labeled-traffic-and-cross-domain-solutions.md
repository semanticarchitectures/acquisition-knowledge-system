# 0013. Security-labeled traffic streams and Cross Domain Solutions (representative, public)

- Status: **Accepted** (2026-11-22) — open questions resolved (Kevin, 2026-11-22); proposed 2026-11-21
- Relates: [ADR 0007](0007-gmns-joins-aks.md) and the GMNS `matlab-network-sim` ICAM layer (identity, credential, access management); [ADR 0012](0012-aks-simulation-interface.md) (the simulation-interface envelope that will carry labels); [ADR 0005](0005-one-model-many-apps.md) (one shared model); the vision's "Secure" property (classification markings, echelon scopes, a reviewed promotion step)
- Scope note: **abstract mechanism + faithfully-configured public instance.** Build a general-purpose label/policy/gateway *engine*, then load a representative, public *configuration* that mirrors the US Government data-security model. The engine knows labels, a dominance relation, and release actions; it does not hardcode any specific classification scheme. Never a real classified ruleset, accreditation artifact, or control baseline.

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

The decision direction is set (Kevin, 2026-11-21): **extend ICAM + the envelope**, and — refined
2026-11-21 — build an **abstract enforcement mechanism** loaded with a **faithfully-configured,
public US Government instance**, rather than hardcoding one label set into the model. The design
questions recorded when this ADR was proposed were resolved on 2026-11-22 (see *Resolved design
decisions*); the spec now builds on those answers.

## Decision (direction)

**Model security-labeled traffic and CDS gateways as an extension of the GMNS ICAM/policy layer and
the ADR 0012 envelope, as a two-part design: a general-purpose enforcement *mechanism* and a
representative, public *configuration* that mirrors the US Government data-security model.** No real
classification guide, control baseline, or accreditation content enters the repo (consistent with
the handling rules already in GMNS and AOC-DFO).

**Why mechanism + configuration (refined 2026-11-21).** A structure-faithful model risks being
specific to one instantiation — a demo that shows exactly one thing and does not generalize. A
purely abstract model risks demonstrating nothing a government reviewer recognizes. Separating the
two resolves the tension: the *engine* (label algebra, policy decision point, release gateway) is
reusable and testable and knows nothing about any specific scheme; the *US Government model* is a
public, representative configuration loaded into it (the label set, the dominance rules, the CDS
release policies). This is also **how real MLS/cross-domain systems are built** — the reference
monitor / guard is separate from the accreditor-approved policy it enforces — so the separation is
*more* faithful to the real architecture, not less. It lets the demonstration show the same engine
under more than one configuration (the US Government representative set; a coalition or commercial
variant), which is a stronger capability story for both a government sponsor and a partner than a
single hardwired lattice.

Shape of the model (details fixed by the resolved decisions below and by the spec):
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
  identity+label+QoS against policy. It gains a mandatory lattice gate in front of its existing
  ABAC-style rule mechanism (D1), rather than a new engine.

## Resolved design decisions (Kevin, 2026-11-22)

These answer the seven questions recorded when the ADR was proposed (Q1–Q7, kept as D1–D7 so
earlier citations still map). Each distinguishes the **engine** (general-purpose) from the **US
Government configuration** (representative, public) where the split matters.

**D1 — Enforcement composition: lattice gate, then ABAC, deny-overrides.** *(was Q1, the central
question)*
- *Engine:* a policy decision is a two-stage pipeline. Stage 1 is an optional **mandatory lattice
  gate**: the subject's (or destination domain's) level must dominate the label's level for a read,
  and a flow may not move to a non-dominating domain except through a CDS gateway (no read-up, no
  write-down). Stage 2 is the existing GMNS ICAM **ABAC rule set**, evaluated only if stage 1
  passes. Combination is **deny-overrides**: no ABAC permit can grant what the lattice denies. A
  configuration may disable stage 1 (pure ABAC) or supply no ABAC rules (pure lattice) — so "both,
  as configurations" holds.
- *US Govt configuration:* both stages on. Levels are enforced by the lattice; handling caveats
  (PII, PHI) are ABAC rules on the subject's authorizations and purpose (need-to-know). This is the
  MAC-over-discretionary layering real systems use.
- *Rejected:* expressing the lattice as ordinary ABAC rules (makes "mandatory" a convention, not a
  guarantee); a pure product lattice of level × caveat set (cannot express purpose/role conditions).

**D2 — Label model and algebra.** *(was Q2)*
- *Engine:* a label is `(level, caveats)`. The configuration declares a totally ordered set of
  levels and a set of caveats, each typed **handling** (restricts who may see it) or
  **releasability** (restricts which domains/partners it may go to). Algebra: **join** of two labels =
  max level, union of handling caveats, *intersection* of releasability caveats. **Dominance** =
  level ≥, handling caveats ⊇, releasability caveats ⊆.
- *US Govt configuration:* four levels, representative analogues of UNCLASSIFIED < CUI < SECRET <
  TOP SECRET; handling caveats PII and PHI (the HIPAA-regulated data type); one releasability caveat
  analogous to coalition release (`REL-PARTNER`). FOUO is folded into the CUI analogue, matching the
  public US practice that replaced FOUO with CUI. Every rendered label carries a visible `SIM`
  marker (guardrail).

**D3 — CDS release actions and review.** *(was Q3)*
- *Engine:* four release actions — **allow**, **deny**, **sanitize**, **hold-for-review**.
  **Sanitize** is field redaction under a named rule: it emits a *new* message with the redacted
  fields removed and a lowered label, plus a **sanitization record** (rule id, fields removed, input
  label, output label) linked to the original. **Hold-for-review** parks the message until the
  required approval events arrive or a timeout denies it.
- *Review is **two-entity**, not two-person.* An approval event names a reviewer **entity**, which
  may be a person or an AI agent; a hold requires approvals from *N distinct entities* (N is a
  configuration parameter). Valid pairs include person + AI, two people, or two different AI agents.
  Each approval records the entity's identity and kind (`person` | `agent`). The spec must define
  what makes two AI entities "distinct" (separate identity at minimum; whether a different model or
  provider is required is a configuration choice) and may let a configuration require at least one
  person.
- *US Govt configuration:* N = 2 (two-entity review), mirroring two-person control while letting the
  demonstration exercise human–AI and AI–AI review.

**D4 — Derived information: high-water mark, confirmed.** *(was Q4)*
- Output of any processing (including ADR 0014 enrichment in Foundry) takes the **join** (D2) of all
  input labels. A label can only go down through an explicit, recorded event — a CDS sanitize (D3)
  or an approved review — never implicitly. Over-labeling drift is accepted as the faithful
  behavior; the spec may add a reporting metric for it.

**D5 — Guard depth: parameterized black box.** *(was Q5)*
- The CDS gateway models: label check against the release policy (D1–D3), envelope schema
  validation (ADR 0012 schema), and a **field allow-list filter** per message type; plus
  configurable **latency distribution** and **failure modes** (fail-closed, queue overflow,
  malformed-message reject). It does not model real filter products, real content-inspection
  engines, or real transfer protocols (guardrail).

**D6 — ENCAP echelon scopes: align, not merge.** *(was Q6)*
- Echelon scope (ADR 0009: which organizational layer an agent may read) and handling labels are
  **two independent axes** at different layers (KB/ENCAP access vs. network traffic). They share one
  vocabulary: ENCAP access-control rules may reference handling labels (D7 IRIs), and the envelope
  carries both `producedBy` echelon context and the label. Neither model encodes the other.

**D7 — Labels as ontology terms; configuration owns the rules.** *(was Q7)*
- Levels and caveats are **AOC-DFO terms (IRIs)** aligned to the shared BFO/CCO model, so a label is
  a modeled thing per ADR 0005 and is visible to ADR 0014's ontology reconciliation. The
  **configuration file references those IRIs and owns the order, caveat types, and release rules**;
  the engine treats IRIs as opaque identifiers. The ontology gives meaning; the configuration gives
  behavior. Representative, public terms only.

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
  now unblocked: it specifies D1–D7 (engine interfaces, the US Govt configuration file, the CDS
  gateway node, and the two-entity review event).
- Envelope extension (ADR 0012): label fields (level and caveat IRIs per D7), sanitization-record
  links, and review approval events (D3), added in the follow-on spec, not retrofitted now.
- AOC-DFO: new representative label terms (D7), added through that repo's normal ontology process.
- Connects to ADR 0014 (Foundry): labeled data ingested and enriched must carry labels through.

## Alternatives considered

| Option | Verdict |
| --- | --- |
| **Abstract mechanism + faithfully-configured public instance** | **Chosen (refined 2026-11-21).** The engine is general-purpose and reusable; the US Government model is a public, representative configuration it loads. Resolves the structure-faithful-vs-abstract tension, mirrors how real MLS/CDS systems separate guard from policy, and lets the demo show more than one configuration. |
| Structure-faithful only (hardcode the US Govt label set/CDS semantics into the model) | Superseded by the chosen option: faithful but specific to one instantiation — a demo that generalizes poorly and a reusable capability lost. Kept as the *configuration* half of the chosen option, not the whole model. |
| Purely abstract mechanism, US Govt as just one unspecified instance | Superseded: the objective specifically asks to mirror the US Govt approach (PII/HIPAA/FOUO/SECRET, CDS). The chosen option keeps the abstract engine *and* delivers a recognizable US Govt instance, rather than leaving it unspecified. |
| New standalone security subsystem, loosely coupled to GMNS | Rejected (Kevin): duplicates the ICAM identity/policy work already specified; labels and QoS belong on the traffic GMNS already models. |
| Deep real-doctrine mapping (ICD 503, Raise-the-Bar, real control baselines) | Rejected: crosses the public-only guardrail and risks producing something mistakable for real accreditation content. |
| Write the full spec now | Rejected at proposal (Kevin, 2026-11-21): the enforcement-model question (Q1) was a real decision; specifying before deciding repeats the drift pattern. Resolved as D1 on 2026-11-22, so the spec can now proceed. |
| Two-*person* review only | Refined (Kevin, 2026-11-22) to two-*entity* review (D3): reviewers may be people or AI agents, so the model covers human–AI and AI–AI control, not only human pairs. |
