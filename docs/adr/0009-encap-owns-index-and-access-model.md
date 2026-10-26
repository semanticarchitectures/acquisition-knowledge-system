# 0009. ENCAP owns the passage index and the access model; the KB owns the graph

- Status: Proposed — needs Kevin's decision (D0)
- Date: 2026-10-26
- Extends: ADR 0004 (where the KB MCP server runs)
- Related: ADR 0001, ADR 0003, ADR 0005; [integrated plan](../integrated-plan.md)

## Context

ENCAP and the knowledge base (KB) each plan the same two pieces for the same corpus:

| Piece | ENCAP plan | KB spec |
| --- | --- | --- |
| Passage index | Phase 4 `document-graph/`: SQLite (FTS + vectors) over OKF bundles, with a query API | `store.index`: SQLite FTS5 + sqlite-vec over OKF sections (knowledge-base-prototype design) |
| Access model | Phase 5 `access-control/`: which `Role` documents may reach which documents and capabilities, enforced at the query API, denials logged | Scoped MCP tokens mapped to echelon layers and read or write permission (kb-mcp-contract R3) |

Both components are on the critical path to D3. Building both twice costs time on both paths and
leaves two definitions of who may see what, which is the opposite of what the "secure" property in
the vision has to show.

The pieces are not symmetric. ENCAP's own plan makes it the owner of runtime and governance, and its
console must enforce access natively. The KB's distinctive asset is the RDF graph (entities,
n-ary relations, claims, decision traces), not the passage index. Both already share OKF bundles and
section anchors as the citation unit (ADR 0003).

## Decision

1. **ENCAP owns the passage index.** ENCAP's `document-graph/` indexes the KB's OKF bundles, one per
   echelon layer. Passage search for every consumer is ENCAP's. The KB drops `store.index`.
2. **ENCAP owns the access model.** Roles, assignments and rules are data in ENCAP's
   `access-control/`, keyed to `Role` documents in the operating model. ENCAP is the **policy
   decision point**: it decides a caller's scope (lowest readable echelon layer, read or write).
3. **The KB owns the graph and enforces scope on it.** The KB's graph tools (`kb_entity`, `kb_cq`,
   `kb_path`, `kb_cite`, `kb_record_decision`, `kb_sparql`) accept a scope decided by ENCAP and
   enforce it on named graphs. The KB is a **policy enforcement point** for its own data, not a
   second policy store.
4. **One contract.** `kb_search` leaves the KB MCP contract and becomes an ENCAP tool. The remaining
   tools keep the contract in spec `kb-mcp-contract`, now carrying an ENCAP-issued scope.
5. **Development server.** For prototyping and evaluation (ADR 0004, option C), the KB's dev server
   reads a static policy file **in ENCAP's access-control format**, never a format of its own.

## Options considered

| Option | For | Against |
| --- | --- | --- |
| A. ENCAP owns index and access; KB owns graph and enforces scope (chosen) | Matches ENCAP's role as runtime and governance owner; one policy definition; the graph-vs-vector evaluation uses ENCAP's index as the vector baseline, which is the system a user would actually get | KB hybrid queries and the evaluation depend on ENCAP's Phase 4; cross-language contract (TypeScript and Python) |
| B. KB owns index and access; ENCAP consumes them | KB Phase 1 is not blocked by ENCAP | Inverts governance: ENCAP's console would depend on the KB for its own access decisions; ENCAP's Phases 4–5 become wasted work |
| C. Each keeps its own (status quo) | No coordination needed | Two indexes and two access models for one corpus; duplicated work on both critical paths; two answers to "who may see this" |
| D. ENCAP owns access, KB owns the index | Single policy store | Still two places that index the same OKF; ENCAP's document graph without its index has little left |

## Consequences if accepted

- **Timing fits.** KB Phases 1–2 answer CQs from the graph and cite OKF anchors; they do not need
  passage search. KB hybrid queries and the Phase 4 evaluation need ENCAP's index, planned for
  Feb–Mar 2027, ahead of KB Phase 4 (May–Jun 2027).
- **Fallback if ENCAP's index slips.** The KB evaluation reads OKF bundles directly, as ENCAP's own
  Phase 2–3 CQ runner does. No second index is built.
- **Spec changes (after acceptance):**
  - `knowledge-base-prototype`: remove `store.index`; R7 becomes enforcement of an ENCAP-issued scope;
    tasks 13 and 19 change accordingly.
  - `kb-mcp-contract`: remove `kb_search`; R3 scopes come from ENCAP's access model; dev server reads
    ENCAP's policy format.
  - ENCAP plan: Phase 4 indexes multiple OKF bundles (one per echelon layer); Phase 5 rules express
    echelon-layer scope and read or write permission; ENCAP issues scopes the KB tools accept.
- **Echelon rule preserved.** "An agent reads its own layer and every layer above, never below"
  becomes a rule in ENCAP's access model and is enforced in both places: by ENCAP on passages, by the
  KB on graph facts.
- **Dependency direction.** ENCAP consumes the KB's bundles and graph tools; the KB depends on ENCAP
  only for the scope it receives at call time, not at build time. Portfolio arrows are unchanged
  (ENCAP → KB).
