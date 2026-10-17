---
inclusion: always
---

# Conventions

## Spec-driven workflow

Every non-trivial change starts as a spec and moves through three gated phases. A human approves
each phase before the next begins.

1. **Requirements** (`requirements.md`): user stories with acceptance criteria in EARS notation.
2. **Design** (`design.md`): architecture, components and interfaces, data models, error handling,
   testing strategy. Link to `docs/architecture` instead of restating it.
3. **Tasks** (`tasks.md`): discrete, checkable tasks, sequenced by dependency, each citing the
   requirements it satisfies and the repo it lands in.

When requirements change, update `requirements.md` first, then refine `design.md`, then resync
`tasks.md`. Never edit `tasks.md` alone to unblock work; that is how specs drift from code.

Spec status lives in `docs/specs-index.md`: Draft, Requirements approved, Design approved,
In progress, Done, Moved.

## EARS patterns

| Pattern | Form |
| --- | --- |
| Ubiquitous | THE SYSTEM SHALL <response> |
| Event-driven | WHEN <trigger> THEN THE SYSTEM SHALL <response> |
| State-driven | WHILE <state> THE SYSTEM SHALL <response> |
| Unwanted behavior | IF <condition> THEN THE SYSTEM SHALL <response> |
| Optional feature | WHERE <feature is included> THE SYSTEM SHALL <response> |

Name the actual component instead of "the system" when the requirement belongs to one (for example,
"THE extraction pipeline SHALL").

## Architecture decision records

- Write an ADR for any choice that is costly to reverse, crosses components, or departs from the
  reference architecture.
- Status: Proposed, Accepted, Superseded by NNNN.
- An ADR lists the options considered and the consequences, including the bad ones.

## Project-family rules (apply in every component)

- Cite sources. Per-claim citation in any synthesized output.
- Hard classification boundary. Public-only in Phase 1.
- Track license and provenance for every source.
- Flag gaps rather than invent. A missing fact is a one-line open question, not a guess.
- Cross-reference components; never merge them.

## Writing

- Lead with the decision or answer. Short sentences, plain words.
- Specifics over adjectives: numbers with units, names, dates.
- Proposed targets and durations are labeled as proposed.
