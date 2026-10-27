# Specs index

| Spec | Purpose | Owning repo(s) | Status |
| --- | --- | --- | --- |
| [knowledge-base-prototype](../.kiro/specs/knowledge-base-prototype/requirements.md) | Public corpus to cited knowledge graph of the Mission Planning program office | Knowledge base | Draft |
| [kb-mcp-contract](../.kiro/specs/kb-mcp-contract/requirements.md) | One MCP contract for every KB consumer | Knowledge base, ENCAP | Draft |
| [decision-traces](../.kiro/specs/decision-traces/requirements.md) | Context graph: decision trace schema and capture paths | Knowledge base, Aircrew-LangChain, ENSIM | Draft |
| [ensim-acquisition-missions](../.kiro/specs/ensim-acquisition-missions/requirements.md) | Acquisition mission set for evaluation | ENSIM | Draft (requirements only) |
| [aoc-dfo](https://github.com/semanticarchitectures/aoc-dfo/tree/main/.kiro/specs/aoc-dfo) (in component repo) | AOC data flow ontology, mission-thread layer, KB alignment target | AOC-DFO | Requirements approved (v1.1, 2026-10-24); Phase 0 scaffolding done |

Statuses: Draft, Requirements approved, Design approved, In progress, Done, Moved (with link).

## Decisions

| ADR | Title | Status |
| --- | --- | --- |
| [0001](adr/0001-umbrella-repo-specs-only.md) | Umbrella repo holds specs; code stays in components | Accepted |
| [0002](adr/0002-rdf-graph-store-oxigraph.md) | Embedded RDF store (Oxigraph) | Proposed |
| [0003](adr/0003-authority-split-okf-and-graph.md) | OKF for documents; RDF graph for entities, claims, traces | Proposed |
| [0004](adr/0004-where-the-kb-mcp-server-runs.md) | Where the KB MCP server runs | Proposed |
| [0005](adr/0005-one-model-many-apps.md) | One shared model, many apps | Accepted |
| [0006](adr/0006-aoc-dfo-repo-and-shared-ontology-pins.md) | AOC-DFO standalone repo; one shared BFO/CCO pin | Accepted (rationale amended by 0007) |
| [0007](adr/0007-gmns-joins-aks.md) | GMNS joins AKS as network transport and C2 messaging testbed | Accepted |
| [0008](adr/0008-separate-repos-and-shared-packages.md) | Separate repos plus shared packages, or a monorepo | Proposed |
| [0009](adr/0009-encap-owns-index-and-access-model.md) | ENCAP owns passage index and access model; KB owns the graph | Accepted |
