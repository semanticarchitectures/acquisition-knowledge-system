# Acquisition Knowledge System (AKS)

**Goal:** demonstrate how AI can architect and simulate Air Force enterprise operations, at scale and
realistically, and build on that simulation to show how AI can be used in the enterprise, while
staying safe, secure and reliable under human authority. AKS is first a capability demonstration,
for Air Force and contractor personnel. See [docs/vision.md](docs/vision.md).

The first use of AI in the enterprise gives an Air Force acquisition organization citation-grounded AI agents over a
layered public corpus, backed by a knowledge graph and a context graph of the organization's
structure, mission, relationships, resources and operations. The first tenant is the Mission Planning
program office (PAE C3BM). Later horizons extend the same shared model to mission planning for
unmanned aircraft and to organizing autonomous operations with the Air Operations Center.

This repo holds **intent, architecture, cross-cutting specs and decisions**. It holds no component
code. Code lives in the component repos.

## How the pieces fit

The single vision is realized as **one shared model, many apps** ([ADR 0005](docs/adr/0005-one-model-many-apps.md)):
one ontology, knowledge graph and decision-trace record (BFO 2020 and CCO 2.2, pinned once for every
component), with purpose-built apps and agents on top. One demonstration thread runs through them:

1. **Concept:** a need appears (the AOC cannot handle many times more unmanned missions).
2. **Acquisition:** agents grounded in the knowledge base trace it to authorities; each decision is
   recorded as a trace (KB, ENCAP, ENSIM).
3. **Development:** the capability is built in Turnpoint, spec-driven with AI assistance.
4. **Operations:** Turnpoint plans unmanned missions and hands them to the AOC (AOC-DFO) as UCI
   messages over a degraded network (GMNS); agents fly them in simulation (Aircrew-LangChain, ENSIM).
5. **Feedback:** results return to the graph as evidence against the original requirement.

## Components

| Component | Role | Status (2026-10-25) | Repo |
| --- | --- | --- | --- |
| Knowledge base (`pokb`, working name) | Corpus and knowledge/context graph | Spec only; critical path | TBD |
| ENCAP | Runtime: synthesis, index, operating model, agents, console | OKF format and synthesis agent built; Phase 2 gate in progress | [encap](https://github.com/semanticarchitectures/encap) |
| AOC-DFO | AOC data-flow ontology; KB alignment target; AOC side of the far-horizon interface | Requirements approved; Phase 0 done | [aoc-dfo](https://github.com/semanticarchitectures/aoc-dfo) |
| ENSIM | Test enterprise for evaluation | Missions 1–3 modeled; Mission 1 runs | [ENSIM](https://github.com/semanticarchitectures/ENSIM) |
| Aircrew-LangChain | LangGraph agents; first decision-trace host | Baseline done; Phase 1 next | [Aircrew-LangChain](https://github.com/semanticarchitectures/Aircrew-LangChain) |
| Turnpoint | Mission planning tool agents call over MCP; unmanned planning next | Phases 1–3 done | [turnpoint](https://github.com/semanticarchitectures/turnpoint) |
| GMNS | Network transport and C2 messaging testbed (UCI, planned C2SIM) | MATLAB; Python port spec not yet committed | [GlobalMobileNetworkSim](https://github.com/semanticarchitectures/GlobalMobileNetworkSim) |

Dependency rules: [.kiro/steering/portfolio.md](.kiro/steering/portfolio.md).

## Milestones

Six demonstration milestones in the goal's order: simulation first, then AI used in the enterprise.
Each is gated by component milestones. Dates are proposed. Full plan,
timeline and dependencies: [docs/integrated-plan.md](docs/integrated-plan.md).

| ID | Milestone | Layer | Proposed |
| --- | --- | --- | --- |
| D0 | Foundations aligned | | Nov 2026 |
| D1 | The AI-built enterprise, running | 1: AI architects and simulates | Mar 2027 |
| D2 | At scale, over a realistic network | 1: AI architects and simulates | Aug 2027 |
| D3 | AI in the enterprise: grounded and on the record | 2: AI used in the enterprise | Dec 2027 |
| D4 | AI in operations: plan to AOC over a degraded network | 2: AI used in the enterprise | May 2028 |
| D5 | Closed loop | Both | Oct 2028 |

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
2. [Integrated milestone plan](docs/integrated-plan.md) and [progress against it](docs/progress.md)
3. [Reference architecture](docs/architecture/reference-architecture.md)
4. [KB prototype architecture](docs/architecture/kb-prototype-architecture.md)
5. [Spec: knowledge-base-prototype](.kiro/specs/knowledge-base-prototype/requirements.md)
6. Open decisions (needed for D0): [ADR 0002](docs/adr/0002-rdf-graph-store-oxigraph.md),
   [0003](docs/adr/0003-authority-split-okf-and-graph.md),
   [0004](docs/adr/0004-where-the-kb-mcp-server-runs.md),
   [0009](docs/adr/0009-encap-owns-index-and-access-model.md),
   [0008](docs/adr/0008-separate-repos-and-shared-packages.md)

## Boundary

Phase 1 is **public, unclassified data only**. Do not commit CUI, internal program documents or
classified material to this repo or any component repo.

## License

Apache 2.0. See [LICENSE](LICENSE).
