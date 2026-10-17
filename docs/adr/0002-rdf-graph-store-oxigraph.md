# 0002. Embedded RDF store (Oxigraph) for the knowledge and context graph

- Status: Proposed
- Date: 2026-10-17

## Context

The KB prototype must answer multi-hop competency questions (CQ1, CQ5, CQ7, CQ9) with provenance,
time validity and echelon scoping, while staying aligned with BFO/CCO and AOC-DFO. ENCAP's earlier
position was "embedded SQLite (FTS + vectors) first, no graph DB until a query pattern demands one".
The CQs are that query pattern, and they are present from Phase 1.

## Decision

Use pyoxigraph (embedded, file-based, SPARQL 1.1, named graphs) for entities, claims and decision
traces. Keep SQLite FTS5 + sqlite-vec for passage retrieval. No database server in Phase 1.

## Options considered

| Option | For | Against |
| --- | --- | --- |
| Oxigraph (chosen) | Native named graphs for provenance, supersession and echelon scope; SPARQL property paths; BFO/CCO with no translation; embedded | Weaker GraphRAG tooling than Neo4j; smaller community |
| Neo4j property graph | Strong GraphRAG ecosystem (neo4j-graphrag, LangChain integrations), graph algorithms | Breaks BFO/CCO alignment; provenance moves into edge properties; adds a server |
| OKF `relations:` + SQLite only | No new technology | Multi-hop, time-scoped and provenance queries become application code |

## Consequences

- Revisit only on a measured need: traversal volume or graph-algorithm requirements SPARQL cannot meet.
- Phase 2 portability to the IL5 enclave is unconfirmed; the Foundry export path is the fallback
  (see ADR 0003).
- ENCAP's no-graph-DB position stands for ENCAP's own document graph; this decision is scoped to the
  KB.
