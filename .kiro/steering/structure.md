---
inclusion: always
---

# Repository structure

```
.kiro/
  steering/            # standing context loaded into every agent session
    product.md         # what AKS is, users, principles, phases
    portfolio.md       # components, repos, dependency rules
    tech.md            # shared standards, ports, framework boundary
    structure.md       # this file
    conventions.md     # spec workflow, EARS, ADRs, writing rules
    ontology.md        # ontology rules (loaded for .ttl and ontology specs)
  specs/
    <spec-name>/
      requirements.md  # user stories + EARS acceptance criteria
      design.md        # architecture, interfaces, data models, errors, tests
      tasks.md         # sequenced, checkable tasks linked to requirements
docs/
  architecture/        # reference architecture and source design documents
  adr/                 # architecture decision records, numbered
  specs-index.md       # every spec, its status and owning repo(s)
AGENTS.md              # conventions for any coding agent (Kiro, Claude Code, others)
README.md
LICENSE
```

## What goes where

- **Steering** holds facts that are true across specs. If you find yourself repeating something in
  two specs, it belongs in steering.
- **Specs** hold one feature or one cross-repo contract each. Specs here are cross-cutting or for a
  component that does not have its own repo yet.
- **docs/architecture** holds narrative documents. Specs link to them by section; they do not
  restate them.
- **docs/adr** records decisions with alternatives and consequences. A spec may cite an ADR; it may
  not silently contradict one.

## Naming

- Spec folders: kebab-case nouns (`knowledge-base-prototype`, `kb-mcp-contract`).
- ADRs: `NNNN-short-title.md`, numbered in order, never renumbered.
- Requirement IDs: `R<n>.<m>` within a spec; tasks cite them as `_Requirements: R2.1, R2.3_`.
