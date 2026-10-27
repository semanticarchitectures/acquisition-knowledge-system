# Requirements Document: Knowledge Base Prototype

- Owning repo: knowledge base (`pokb`, working name; repo TBD)
- Consumers: ENCAP, Aircrew-LangChain, evaluation harness
- Design narrative: [docs/architecture/kb-prototype-architecture.md](../../../docs/architecture/kb-prototype-architecture.md)
- Related decisions: ADR 0002, 0003, 0004, 0009
- Status: Draft (revised 2026-10-26 for ADR 0009: ENCAP owns the passage index and access model)

## Introduction

The knowledge base turns a registered public corpus into an ontology-conformant, fully cited
knowledge graph of the Mission Planning program office and the echelons above it (L0–L4). Agents
query it, and write decision traces back to it, through MCP. This spec covers the Phase 1 (public)
prototype.

The ten competency questions (CQ1–CQ10) in the design narrative are part of these requirements by
reference. They are confirmed or revised at the end of Phase 0 (R8.1).

## Requirements

### Requirement 1: Source registry and provenance

**User Story:** As a reviewer, I want every source recorded with its origin, release basis and
license, so that every claim in the graph can be traced and legally reused.

#### Acceptance Criteria

1. THE registry SHALL record, for each source: URL, issuing organization, publication date,
   retrieval date, content hash, public-release basis, license, echelon layer and `supersedes`.
2. IF a file has no registry entry THEN THE pipeline SHALL refuse to ingest it.
3. IF a source's marking is anything other than public THEN THE pipeline SHALL refuse to ingest it
   and log the refusal.
4. WHEN a registered source's content hash changes THEN THE pipeline SHALL record a new source
   version rather than overwrite the old one.

### Requirement 2: Parsing and OKF normalization

**User Story:** As an agent, I want every passage I cite to have a stable anchor, so that a citation
still resolves after the corpus is rebuilt.

#### Acceptance Criteria

1. THE pipeline SHALL convert each source to OKF Markdown with stable section anchors, preserving
   headings, tables and page numbers.
2. THE pipeline SHALL run PDF extraction in an isolated process.
3. WHEN the same source version is parsed twice THEN THE pipeline SHALL produce identical anchors.
4. THE pipeline SHALL write one OKF bundle per echelon layer (L0–L4).

### Requirement 3: Ontology-guided extraction

**User Story:** As an ontology engineer, I want extraction constrained to the ontology, so that the
graph uses one consistent vocabulary across documents.

#### Acceptance Criteria

1. THE extractor SHALL run a rules-and-gazetteer tier before the LLM tier.
2. WHEN extracting from a section THEN THE extractor SHALL supply the model with the relevant ontology
   slice and a JSON schema generated from the applicable SHACL shapes.
3. THE extractor SHALL return, for each candidate claim, the source span, a confidence and the shape
   it fills.
4. IF a candidate relation does not match an ontology property THEN THE extractor SHALL reject it.
5. THE extractor SHALL pin model version and prompt hash for every run and record them with each
   claim.

### Requirement 4: Entity resolution

**User Story:** As a program office user, I want one organization to be one entity, so that answers
do not split or merge organizations incorrectly.

#### Acceptance Criteria

1. THE resolver SHALL match deterministically on office symbol, exact alias and known rename before
   any similarity matching.
2. IF a candidate match scores below the auto-merge threshold THEN THE resolver SHALL send it to the
   review queue and SHALL NOT merge it.
3. THE resolver SHALL make every merge reversible and record who or what approved it.

### Requirement 5: Validation gate

**User Story:** As a consumer, I want only valid claims committed, so that queries do not return
malformed facts.

#### Acceptance Criteria

1. WHEN a candidate claim violates a SHACL shape THEN THE gate SHALL reject it and log the violation.
2. WHEN a valid candidate's confidence is below the commit threshold THEN THE gate SHALL send it to the
   review queue.
3. THE gate SHALL commit each accepted claim to the named graph of its source version, linked to its
   OKF anchor.

### Requirement 6: Time and supersession

**User Story:** As an analyst, I want to see the organization as it stood on a given date, so that
answers do not present stale structure as current.

#### Acceptance Criteria

1. THE graph SHALL represent changeable relationships (assignment, reporting line, portfolio
   membership, funding) as n-ary nodes with `validFrom` and `validTo`.
2. WHEN a source is superseded THEN THE pipeline SHALL mark its named graph superseded and SHALL flag,
   not delete, every fact left with no non-superseded support.
3. WHEN a source changes THEN THE pipeline SHALL regenerate only that source's named graph.

### Requirement 7: Echelon scoping

**User Story:** As a governance officer, I want each agent limited to its echelon and those above it,
so that the boundary carries into the CUI phase.

#### Acceptance Criteria

1. THE graph SHALL assign every named graph to exactly one echelon layer, L0 to L4.
2. THE KB SHALL accept the caller's scope as decided by ENCAP's access model and SHALL NOT keep a
   policy store of its own (ADR 0009). In development, scope comes from a static policy file in
   ENCAP's access-control format.
3. WHILE a client is scoped to layer Ln THE KB SHALL return facts only from Ln and layers above it.
4. IF a query would return a fact outside the caller's scope THEN THE KB SHALL omit it and SHALL NOT
   reveal that it exists.
5. IF a call arrives with no scope THEN THE KB SHALL refuse it.

### Requirement 8: Competency questions

**User Story:** As the project lead, I want the CQs to be executable tests, so that progress is
measured by answers, not by pipeline features.

#### Acceptance Criteria

1. WHEN Phase 0 closes THEN THE project SHALL have a confirmed CQ list, each CQ backed by at least one
   registered source.
2. THE KB SHALL implement each CQ as a parameterized, versioned SPARQL template with a golden-answer
   test.
3. WHEN a CQ answer is returned THEN THE KB SHALL attach claim IDs that resolve to OKF anchors.
4. IF a CQ cannot be answered from the graph THEN THE KB SHALL return an explicit gap, not a partial
   answer presented as complete.

### Requirement 9: Review queue

**User Story:** As a reviewer, I want to see each uncertain claim with its source passage, so that I
can accept or reject it quickly.

#### Acceptance Criteria

1. THE review queue SHALL show each item's candidate, its source passage and the reason it was queued.
2. WHEN a reviewer accepts or rejects an item THEN THE KB SHALL record the choice as a decision trace
   (see spec `decision-traces`).
3. THE review log SHALL be stored so that a graph rebuild reapplies every past decision (ADR 0003).

### Requirement 10: Snapshot and projection

**User Story:** As an ENCAP or evaluation maintainer, I want a pinned snapshot, so that evaluation
runs are reproducible.

#### Acceptance Criteria

1. THE KB SHALL export a versioned snapshot: N-Quads of all named graphs plus the OKF bundles they
   cite. ENCAP builds the passage index from those bundles (ADR 0009).
2. WHERE the Foundry adapter is enabled THE KB SHALL export one dataset per class and per link type,
   with source and claim IDs as columns.
3. THE KB SHALL be rebuildable from the OKF bundles, the registry and the review log alone.

### Requirement 11: Evaluation

**User Story:** As the project lead, I want to know whether the graph earns its cost, so that the CUI
pilot decision rests on evidence.

#### Acceptance Criteria

1. THE project SHALL maintain a versioned gold set and score extraction precision and recall per
   relation type.
2. THE project SHALL score entity resolution with pairwise precision and recall.
3. THE project SHALL compare answers from the same agent with and without KB tools on about 40
   CQ-derived questions, and report single-hop and multi-hop results separately.
4. THE project SHALL check citation faithfulness: each cited passage supports its claim.
