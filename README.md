# Acquisition Knowledge System (AKS)

Umbrella repo for a family of projects that give an Air Force acquisition organization
citation-grounded AI agents over a layered public corpus, backed by a knowledge graph and a context
graph of the organization's structure, mission, relationships, resources and operations.
The first tenant is the Mission Planning program office (PAE C3BM).

This repo holds **intent, architecture, cross-cutting specs and decisions**. It holds no component
code. Code lives in the component repos.

## Components

| Component | Role | Repo |
| --- | --- | --- |
| ENCAP | Runtime: synthesis, index, operating model, agents, console | TBD |
| Knowledge base (`pokb`, working name) | Corpus and knowledge/context graph | TBD |
| ENSIM | Test enterprise for evaluation | [ENSIM](https://github.com/semanticarchitectures/ENSIM) |
| Turnpoint | Mission planning tool agents call over MCP | [turnpoint](https://github.com/semanticarchitectures/turnpoint) |
| Aircrew-LangChain | LangGraph agents; first decision-trace host | [Aircrew-LangChain](https://github.com/semanticarchitectures/Aircrew-LangChain) |
| AOC-DFO | Reference ontology for alignment | TBD |

Dependency rules: [.kiro/steering/portfolio.md](.kiro/steering/portfolio.md).

## How this repo works

It follows Kiro-style spec-driven development:

- **`.kiro/steering/`** — standing context every agent session loads: product, portfolio, tech,
  structure, conventions, and ontology rules.
- **`.kiro/specs/<name>/`** — one spec per feature or cross-repo contract, each with
  `requirements.md` (EARS), `design.md` and `tasks.md`, approved phase by phase.
- **`docs/architecture/`** — the reference architecture and source design documents.
- **`docs/adr/`** — architecture decision records.

Start with [docs/specs-index.md](docs/specs-index.md).

## Start here

1. [Reference architecture](docs/architecture/reference-architecture.md)
2. [KB prototype architecture](docs/architecture/kb-prototype-architecture.md)
3. [Spec: knowledge-base-prototype](.kiro/specs/knowledge-base-prototype/requirements.md)
4. Open decisions: [ADR 0002](docs/adr/0002-rdf-graph-store-oxigraph.md),
   [0003](docs/adr/0003-authority-split-okf-and-graph.md),
   [0004](docs/adr/0004-where-the-kb-mcp-server-runs.md)

## Boundary

Phase 1 is **public, unclassified data only**. Do not commit CUI, internal program documents or
classified material to this repo or any component repo.

## License

Apache 2.0. See [LICENSE](LICENSE).
