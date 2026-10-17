# 0001. An umbrella repo holds specs and architecture; code stays in component repos

- Status: Accepted
- Date: 2026-10-17

## Context

AKS spans ENCAP, the knowledge base, ENSIM, Turnpoint and Aircrew-LangChain. The reference
architecture's next steps said to commit it to the ENCAP repo as `docs/ARCHITECTURE.md`. That puts the
system-level intent inside one component and makes the others depend on it for their own context.
Cross-component specs (the KB-to-consumer MCP contract, decision traces) have no natural home in any
single component.

## Decision

Create `acquisition-knowledge-system` as the umbrella repo. It holds Kiro steering, cross-cutting
specs, the reference architecture and ADRs. It holds no component code or data. Component repos keep
their own code and their own component-internal specs, and link back to this repo.

## Options considered

1. **Architecture in the ENCAP repo** (the earlier plan). Simple, but ENCAP becomes the place every
   other component looks for system intent, against the one-directional rule.
2. **Umbrella repo with specs only** (chosen).
3. **Umbrella repo that also holds the KB code.** Rejected: it would merge a component into the
   umbrella and blur the "corpus is data only" boundary.
4. **Git submodules of every component.** Rejected: couples release cycles and adds friction for
   little benefit; links pinned to commits give traceability without it.

## Consequences

- Specs here must name the repo each task lands in.
- A spec that becomes internal to one component moves to that repo.
- Component repos should add a pointer to this repo in their README and AGENTS.md.
