# Implementation Plan: Knowledge Base Prototype

All tasks land in the knowledge base repo (`pokb`) unless marked otherwise. Phase gates are from the
design narrative; durations there are proposed, not committed.

## Phase 0 — Foundations

- [ ] 1. Create the `pokb` repo from the layout in the design narrative, with AGENTS.md pointing to this umbrella repo
  - _Requirements: R1, R2_
- [ ] 2. Build the Phase 0 source inventory in `sources/registry.yaml` for L0–L4, focused on what can support the CQs at L3 and L4
  - Prefer budget justification exhibits, contract award notices and command releases for L4
  - _Requirements: R1.1, R8.1_
- [ ] 3. Confirm or revise CQ1–CQ10 against the inventory; drop or reword any CQ no source supports
  - _Requirements: R8.1_
- [ ] 4. Draft ontology v0: import pinned BFO/CCO; add only the mid-level classes the confirmed CQs need; align names with ENSIM's org-doctrine schema
  - _Requirements: R3, R6.1; steering: ontology.md_
- [ ] 5. Write SHACL shapes with passing and failing fixtures for each class and n-ary relation
  - _Requirements: R5.1_
- [ ] 6. Write failing golden-answer tests for every confirmed CQ against a hand-built fixture graph
  - _Requirements: R8.2_
- [ ] 7. **Gate 0:** every CQ is backed by at least one registered source. Decide ADR 0002, 0003 and 0004.

## Phase 1 — Thin slice (organizational structure only)

- [ ] 8. Implement `ingest.register` with refusal of unregistered and non-public sources
  - _Requirements: R1.2, R1.3, R1.4_
- [ ] 9. Implement `ingest.parse` behind a process boundary (ENCAP's service if ready, else Docling) and `ingest.normalize` to OKF with stable anchors
  - _Requirements: R2.1–R2.4_
- [ ] 10. Implement tier 1 rules and gazetteer for organizations and office symbols
  - _Requirements: R3.1_
- [ ] 11. Implement tier 2 extraction for reporting and assignment relations only, with SHACL-generated schemas and pinned model and prompt
  - _Requirements: R3.2–R3.5_
- [ ] 12. Implement the SHACL gate and commit to per-source-version named graphs in Oxigraph
  - _Requirements: R5, R6.3_
- [ ] 13. Implement the graph read tools `kb_entity`, `kb_cq`, `kb_cite` per spec `kb-mcp-contract` (no passage index; ADR 0009)
  - _Requirements: R8.3; kb-mcp-contract R1–R3_
- [ ] 14. **Gate 1:** CQ1 and CQ2 pass with citations that resolve to OKF anchors.

## Phase 2 — Breadth

- [ ] 15. Extend extraction to all relation families in the ontology
  - _Requirements: R3_
- [ ] 16. Implement entity resolution with deterministic matching, adjudication and reversible merges
  - _Requirements: R4_
- [ ] 17. Build the review queue artifact and durable review log; record decisions as traces
  - _Requirements: R9; decision-traces R2_
- [ ] 18. Implement validity intervals, supersession and unsupported-fact flagging
  - _Requirements: R6_
- [ ] 19. Implement scope enforcement at query time from an ENCAP-issued scope (dev: policy file in ENCAP's format), with scope tests
  - _Requirements: R7_
- [ ] 20. Annotate the gold set (about 10 sources) and score extraction and ER
  - _Requirements: R11.1, R11.2_
- [ ] 21. **Gate 2:** 8 of 10 CQs pass; committed-claim precision ≥ 0.90 (proposed target).

## Phase 3 — Context graph

- [ ] 22. Implement the trace store and `kb_record_decision` per spec `decision-traces`
  - _Requirements: decision-traces R1, R3_
- [ ] 23. Add the decision-node hook in Aircrew-LangChain (lands in that repo)
  - _Requirements: decision-traces R3_
- [ ] 24. Generate synthetic acquisition decisions from ENSIM (depends on spec `ensim-acquisition-missions`; lands in ENSIM)
  - _Requirements: decision-traces R4_
- [ ] 25. **Gate 3:** CQ9 and CQ10 pass on synthetic data.

## Phase 4 — Integration

- [ ] 26. Export the versioned snapshot; ENCAP reads it at a pinned version (ENCAP side lands in ENCAP)
  - _Requirements: R10.1_
- [ ] 27. Implement the Foundry dataset export and load it into a Foundry developer instance
  - _Requirements: R10.2_
- [ ] 28. Run the rebuild test from OKF + registry + review log
  - _Requirements: R10.3_
- [ ] 29. Run the graph-vs-vector comparison and the citation faithfulness check; write the evaluation report
  - _Requirements: R11.3, R11.4_
- [ ] 30. **Gate 4:** demo, evaluation report, and a CUI pilot go/no-go memo.
