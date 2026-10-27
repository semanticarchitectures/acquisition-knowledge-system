# Implementation Plan: KB MCP Contract

- [ ] 1. Decide ADR 0004 (hosting) — umbrella repo (ADR 0009, access model, accepted 2026-10-26)
- [ ] 2. Write JSON Schemas for all six tools and the common envelope — KB repo
  - _Requirements: R1.1, R2.1_
- [ ] 3. Implement read tools (`kb_entity`, `kb_cq`, `kb_cite`) in `pokb.mcp.tools` — KB repo
  - _Requirements: R1, R2, R4_
- [ ] 4. Implement scope enforcement from ENCAP-issued scopes, the dev policy file reader (ENCAP's format) and call logging — KB repo
  - _Requirements: R3_
- [ ] 5. Implement `kb_path` and `kb_sparql` (expert scope) — KB repo
  - _Requirements: R1.1, R3.3_
- [ ] 6. Implement `kb_record_decision` per spec `decision-traces` — KB repo
  - _Requirements: R1.1, R3.2_
- [ ] 7. Write the contract test suite against a fixture graph — KB repo
- [ ] 8. Mount the library behind ENCAP's runtime and run the same contract tests — ENCAP repo
  - _Requirements: R1.2_
