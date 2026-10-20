# Specs index

| Spec | Purpose | Owning repo(s) | Status |
| --- | --- | --- | --- |
| [knowledge-base-prototype](../.kiro/specs/knowledge-base-prototype/requirements.md) | Public corpus to cited knowledge graph of the Mission Planning program office | Knowledge base | Draft |
| [kb-mcp-contract](../.kiro/specs/kb-mcp-contract/requirements.md) | One MCP contract for every KB consumer | Knowledge base, ENCAP | Draft |
| [decision-traces](../.kiro/specs/decision-traces/requirements.md) | Context graph: decision trace schema and capture paths | Knowledge base, Aircrew-LangChain, ENSIM | Draft |
| [ensim-acquisition-missions](../.kiro/specs/ensim-acquisition-missions/requirements.md) | Acquisition mission set for evaluation | ENSIM | Draft (requirements only) |

Statuses: Draft, Requirements approved, Design approved, In progress, Done, Moved (with link).

## Decisions

| ADR | Title | Status |
| --- | --- | --- |
| [0001](adr/0001-umbrella-repo-specs-only.md) | Umbrella repo holds specs; code stays in components | Accepted |
| [0002](adr/0002-rdf-graph-store-oxigraph.md) | Embedded RDF store (Oxigraph) | Proposed |
| [0003](adr/0003-authority-split-okf-and-graph.md) | OKF for documents; RDF graph for entities, claims, traces | Proposed |
| [0004](adr/0004-where-the-kb-mcp-server-runs.md) | Where the KB MCP server runs | Proposed |
| [0005](adr/0005-one-model-many-apps.md) | One shared model, many apps | Accepted |
