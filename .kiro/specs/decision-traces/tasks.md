# Implementation Plan: Decision Traces

- [ ] 1. Add the trace classes and SHACL shape to the KB ontology — KB repo
  - _Requirements: R1.1, R1.2_
- [ ] 2. Implement trace storage and `kb_record_decision` validation — KB repo
  - _Requirements: R1, R1.3_
- [ ] 3. Record review-queue decisions as traces — KB repo
  - _Requirements: R2.1_
- [ ] 4. Extract declared decisions from public sources with unknown reasoning fields — KB repo
  - _Requirements: R5.1_
- [ ] 5. Identify decision nodes in the governed envelope and add the record-then-commit step — Aircrew-LangChain repo
  - _Requirements: R3.1–R3.3_
- [ ] 6. Generate synthetic milestone and funding decisions (after spec `ensim-acquisition-missions`) — ENSIM repo
  - _Requirements: R4.1_
- [ ] 7. Implement CQ10 and precedent filtering by origin — KB repo
  - _Requirements: R4.2, R6_
