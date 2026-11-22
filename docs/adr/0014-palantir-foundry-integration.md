# 0014. Palantir Foundry as an operator-facing consumer of simulation data

- Status: **Proposed** (2026-11-21) — captures the decisions to make before a spec is written
- Relates: [ADR 0012](0012-aks-simulation-interface.md) (the envelope/records Foundry ingests), [ADR 0013](0013-security-labeled-traffic-and-cross-domain-solutions.md) (labels Foundry must carry through), [ADR 0005](0005-one-model-many-apps.md) (one shared ontology — Foundry's Ontology must reconcile with it), [ADR 0009](0009-encap-owns-index-and-access-model.md) (ENCAP's console — Foundry overlaps its operator-facing role); the vision's "Operations and feedback" lifecycle row
- Scope note: uses **Palantir Foundry free Developer Edition**; its limits and terms bound what this can be.

## Context

A new high-level objective (Kevin, 2026-11-21): **ingest AKS simulation data into Palantir Foundry
(free Developer Edition), process it into new information, and let a human operator view and
manipulate that information through Foundry.**

Where it fits AKS: this is an **operations-and-feedback / operator-facing consumer** — the output
edge of the demonstration thread (D5 "closed loop," the vision's "Operations and feedback" row). It
reads simulation output (GMNS network performance, ENSIM mission state, Aircrew agent traces,
AOC-DFO mission threads); it does not feed back into the simulations (consistent with the
dependency-direction rule in `portfolio.md`: nothing flows back into the sims).

It is largely additive — it disturbs no existing component decision — but it raises reconciliation
questions with two things AKS already has: the **shared ontology** (ADR 0005) and **ENCAP's operator
console** (ADR 0009). Foundry brings its own Ontology and its own operator UI; the ADR exists to
decide how those relate rather than silently duplicate.

The direction is set (Kevin, 2026-11-21): define the **objective and the open questions** now;
scope the implementation later.

## Decision (direction)

**Adopt Palantir Foundry (Developer Edition) as an operator-facing consumer of AKS simulation data,
specified later; this ADR records the integration decisions to settle first.** The integration is
one-directional (sim → Foundry), carries ADR 0013 handling labels through, and reconciles with the
AKS shared ontology rather than forking it.

Shape (to be specified, not fixed here):
- **Ingestion.** A pipeline takes AKS simulation output — the ADR 0012 envelope records and/or GMNS
  run outputs — into Foundry datasets.
- **Enrichment ("new information").** A transform in Foundry derives new information from the
  ingested data (e.g., network-health rollups, mission-outcome correlations, degraded-comms impact
  on C2 delivery). This is the "process into new information" step.
- **Operator view/manipulate.** A Foundry application (Workshop/Object views) lets a human operator
  explore and act on the derived information.
- **Labels carried through.** Data ingested under ADR 0013 handling labels keeps those labels in
  Foundry; derived information takes the high-water-mark label (ADR 0013 D4).

## Open questions (the spec must decide these)

1. **Developer Edition limits.** What does the free tier actually allow — data volume, Ontology
   object/type counts, pipeline/transform availability, external connectivity, and the **license
   terms** on using it for a government-adjacent demonstration? This gates everything; verify first.
2. **Ingestion path.** How does sim data reach Foundry — file upload, the Foundry API/SDK, a
   connector? What the Developer Edition permits (Q1) constrains this. Batch (per sim run) or
   streaming?
3. **Ontology mapping (the ADR 0005 reconciliation).** AKS has a BFO/CCO shared ontology; Foundry
   has its own Ontology (Objects/Links/Actions). Do AKS entities map onto Foundry Objects
   one-to-one? Is the Foundry Ontology a *projection* of the shared model (shared model stays
   canonical) or a parallel model? *This is the central design question* — it decides whether
   Foundry integrates with "one model, many apps" or becomes an eighth model.
4. **Relationship to ENCAP's console (ADR 0009).** ENCAP owns the operator-facing personnel console.
   Foundry is also operator-facing. Are they different audiences/purposes (ENCAP = acquisition
   agents/governance; Foundry = operational sim-data analysis), or do they overlap? Decide the
   boundary so they are complementary, not competing.
5. **What "manipulate" means.** Read-only operator views, or write-back actions (Foundry Actions)
   that change state? If write-back, where does it go — Foundry only, or does it violate the
   no-feedback-into-sims rule? Likely Foundry-only; confirm.
6. **Demo vs. reference integration.** Is this a one-off proof (ingest one run, one enrichment, one
   view) or a first-class, maintained AKS consumer? Affects how much is built and specified.
7. **Hosting/data posture.** Developer Edition is Palantir-hosted. Only public/representative AKS
   data (Phase 1 rule) goes in. Confirm no CUI/sensitive data reaches a hosted third party — and
   note the interplay with ADR 0013: labeled data in a hosted environment is itself a modeling point.

## Consequences

- A new umbrella objective and vision entry (operator-facing consumption; the D5 closed loop made
  concrete with a real tool).
- A future `.kiro/specs/foundry-integration/` spec, gated on Q1 (Developer Edition limits) and Q3
  (Ontology reconciliation).
- A dependency to record in `portfolio.md`: Foundry reads sim output; nothing flows back.
- Interplay with ADR 0013: handling labels must survive ingestion and enrichment.

## Alternatives considered

| Option | Why not (now) |
| --- | --- |
| Full reference integration, fully specified now | Rejected (Kevin): Developer Edition limits and the Ontology-reconciliation question are unknowns; specify after deciding. |
| Build the operator view in ENCAP instead of Foundry | Possible, but the objective is specifically to demonstrate Foundry integration (a widely-used government data platform); that external-tool interop is the point. ADR records the ENCAP boundary (Q4) rather than avoiding Foundry. |
| Skip the shared-ontology reconciliation; let Foundry have its own model | Rejected: that makes Foundry an eighth parallel model, against ADR 0005. Q3 must be answered. |
