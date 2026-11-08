# Implementation Plan: Decision Traces

- Requirements: [requirements.md](requirements.md) (approved 2026-11-06)
- Design: [design.md](design.md) (approved 2026-11-07)
- Status: Proposed 2026-11-07, awaiting approval. Renumbered from the earlier draft, in which no task
  had started, to put the work before D1 first.

Each task names the repo it lands in. Dates are the milestone a task serves, not commitments.

## Before D1: agent traces to files (Aircrew-LangChain Phase 2)

- [ ] 1. Pin the trace JSON Schema from `design.md` by umbrella commit and validate every trace
      against it where it is written — Aircrew-LangChain repo
  - _Requirements: R1.4_
- [ ] 2. Implement the trace-recording interface with the file backend: one `traces_<run>.jsonl` per
      run with the run records, a durable write before the ID is returned, IDs
      `trace:aircrew-langchain/<run>/<n>` — Aircrew-LangChain repo
  - _Requirements: R3.1, R3.3, R3.4_
- [ ] 3. Identify the decision nodes in the governed envelope and add record-then-commit: assemble the
      trace from the envelope and the mission order (`decidedBy`, `inRole`, `underAuthority`, `used`
      as state paths), list local identifiers in `pending`, store the trace ID in state, and commit
      only after the record succeeds — Aircrew-LangChain repo
  - _Requirements: R1.3, R3.1, R3.3, R3.6_
- [ ] 4. Fill `runRef` with the run and checkpoint IDs, and the LangSmith run ID when tracing is on —
      Aircrew-LangChain repo
  - _Requirements: R3.2_
- [ ] 5. Tests: a scripted mission writes one schema-valid trace per decision node, each ID present in
      the final state and linked to its run and checkpoint; a failed write commits nothing and is
      reported in the run's events — Aircrew-LangChain repo
  - _Requirements: R3.1–R3.4, R3.6_

## D3: the KB trace store and import

- [ ] 6. Add the trace classes and SHACL shape to the KB ontology — KB repo
  - _Requirements: R1.1, R1.2_
- [ ] 7. Adopt the `design.md` trace schema as `kb_record_decision`'s input schema, and test that every
      schema-valid trace with no `pending` fields is valid under the SHACL shape — KB repo
  - _Requirements: R1.2, R1.4_
- [ ] 8. Implement trace storage and `kb_record_decision` validation — KB repo
  - _Requirements: R1, R1.3_
- [ ] 9. Resolve `pending` identifiers in `kb_record_decision`; reject a trace whose identifier cannot be
      resolved, naming the field; make recording idempotent by trace ID; mark a replayed trace
      uncommitted when its run's final state does not hold its ID — KB repo
  - _Requirements: R7.1, R7.2_
- [ ] 10. Add the KB backend to the trace-recording interface and a one-time replay of each traces
       file through `kb_record_decision`; decision nodes do not change — Aircrew-LangChain repo
  - _Requirements: R3.5, R7.1_
- [ ] 11. Replay test from a recorded traces file, including an uncommitted trace and an unresolvable
       pending identifier — Aircrew-LangChain repo, against the KB's contract test server
  - _Requirements: R7.1, R7.2_

## Other origins

- [ ] 12. Record review-queue decisions as traces — KB repo
  - _Requirements: R2.1_
- [ ] 13. Extract declared decisions from public sources with unknown reasoning fields — KB repo
  - _Requirements: R5.1_
- [ ] 14. Generate synthetic milestone and funding decisions through `kb_record_decision` (after spec
       `ensim-acquisition-missions`) — ENSIM repo
  - _Requirements: R4.1_
- [ ] 15. Implement CQ10 and precedent filtering by origin — KB repo
  - _Requirements: R4.2, R6_
