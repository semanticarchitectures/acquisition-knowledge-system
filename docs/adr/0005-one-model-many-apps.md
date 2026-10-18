# 0005. One shared model, many apps: how the single vision is realized

- Status: Proposed
- Date: 2026-10-18
- Related: [docs/vision.md](../vision.md)

## Context

The goal ([vision](../vision.md)) is a capability that works at the strategic, operational and tactical
levels with a single vision across the capability lifecycle, from concept development through
production and into operations. Read literally, that could mean one application.

The levels differ in ways a single application would have to absorb:

| Dimension | Strategic | Operational | Tactical |
| --- | --- | --- | --- |
| Tempo | Months to years | Hours to days | Seconds to minutes |
| Assurance | Advisory; cited answers are enough | Supervised planning | Deterministic, verifiable behavior |
| Classification | Mostly public or CUI | CUI to classified | Often classified |
| Users | Leadership, program offices | AOC planners | Mission systems and crews, manned or unmanned |

## Decision

Realize the single vision as **one shared semantic model with many purpose-built apps and agents** on
top of it:

- **Shared:** the ontology (BFO/CCO + KB + AOC-DFO, aligned), the knowledge graph, the decision-trace
  record, provenance, and the MCP contracts.
- **Separate:** each app and agent, with the tempo, assurance level, deployment and classification
  its level needs (ENCAP console, Turnpoint, the agents, future AOC interface software).
- **Unified view:** where a single place to look is wanted, it is a view over the shared model, such as
  the ENCAP personnel console, not a monolith.

## Options considered

1. **One application across all levels.** One codebase and UX. Rejected: it would bind tactical
   assurance and classification to strategic tooling, or the reverse.
2. **Independent apps, no shared model.** Each fits its level. Rejected: the lifecycle thread breaks;
   a requirement cannot be traced to the missions that tested it.
3. **Shared model, many apps** (chosen).

## Consequences

- The ontology and its alignment across acquisition (KB) and operations (AOC-DFO) become the most
  important assets. Alignment work moves earlier.
- "Singular" is enforced by the model and the traces, not by the user interface.
- Each new app must read and write through the shared contracts; no app keeps a private copy of shared
  facts.
- Across classification levels the model is replicated, never merged: public layers are mirrored in,
  never out (reference architecture, Phase 3).
