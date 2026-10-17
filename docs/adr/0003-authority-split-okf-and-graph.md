# 0003. OKF is authoritative for documents; the RDF graph is authoritative for entities, claims and traces

- Status: Proposed
- Date: 2026-10-17
- Departs from: reference architecture, "the OKF bundle remains the source of truth" and "layers link
  upward with typed relations: edges in OKF frontmatter"

## Context

The reference architecture treats the OKF bundle as the single source of truth, with typed relations
held as edges in OKF frontmatter. The KB prototype design adds an RDF graph of entities, n-ary
relationships with validity intervals, claims and decision traces. Two stores that both claim to be
authoritative for the same fact will drift.

## Decision

Split authority by kind of fact:

| Fact | Authoritative store | Notes |
| --- | --- | --- |
| Document text, sections and anchors | OKF bundle | The citation unit for every consumer |
| Document-to-document links (implements, governed-by, supersedes between documents) | OKF `relations:` frontmatter | Drives the re-review walk in the reference architecture |
| Entities, entity relationships, claims, validity intervals | RDF graph | Every claim points to an OKF anchor |
| Decision traces | RDF graph | |
| Foundry objects | Neither; a projection of both | Rebuilt by export |

The OKF bundle stays the one component that is never replaced. The RDF graph can always be rebuilt
from the OKF bundle, the registry and the review log.

## Options considered

1. **OKF only** (reference architecture as written). Entity-level, time-scoped queries become
   application code over frontmatter; CQ7 and CQ8 are hard.
2. **Authority split** (chosen).
3. **Graph only.** Loses the human-readable, diffable document layer ENCAP is built on.

## Consequences

- The rebuild-from-OKF property must be tested in CI.
- Review decisions must be stored durably outside the graph so a rebuild does not lose them.
- The reference architecture's principle stands; this ADR narrows "source of truth" to documents.
