# Acquisition Knowledge System (AKS)

**Goal:** show, end to end and with evidence, that AI can speed up how the Air Force conceives,
acquires, fields and operates capability, while staying safe, secure and reliable under human
authority. AKS is first a capability demonstration. See [docs/vision.md](docs/vision.md).

The near horizon gives an Air Force acquisition organization citation-grounded AI agents over a
layered public corpus, backed by a knowledge graph and a context graph of the organization's
structure, mission, relationships, resources and operations. The first tenant is the Mission Planning
program office (PAE C3BM). Later horizons extend the same shared model to mission planning for
unmanned aircraft and to organizing autonomous operations with the Air Operations Center.

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
| GMNS | Network transport and C2 messaging testbed (UCI, planned C2SIM) | [GlobalMobileNetworkSim](https://github.com/semanticarchitectures/GlobalMobileNetworkSim) |
| AOC-DFO | Reference ontology for alignment; AOC side of the far-horizon interface | [aoc-dfo](https://github.com/semanticarchitectures/aoc-dfo) |

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

1. [Vision and goal](docs/vision.md)
2. [Reference architecture](docs/architecture/reference-architecture.md)
3. [KB prototype architecture](docs/architecture/kb-prototype-architecture.md)
4. [Spec: knowledge-base-prototype](.kiro/specs/knowledge-base-prototype/requirements.md)
5. Open decisions: [ADR 0002](docs/adr/0002-rdf-graph-store-oxigraph.md),
   [0003](docs/adr/0003-authority-split-okf-and-graph.md),
   [0004](docs/adr/0004-where-the-kb-mcp-server-runs.md),
   [0005](docs/adr/0005-one-model-many-apps.md)

## Boundary

Phase 1 is **public, unclassified data only**. Do not commit CUI, internal program documents or
classified material to this repo or any component repo.

## License

Apache 2.0. See [LICENSE](LICENSE).
