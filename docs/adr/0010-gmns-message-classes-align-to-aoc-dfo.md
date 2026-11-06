# 0010. GMNS keeps its message classes and aligns them to AOC-DFO later

- Status: Accepted (2026-11-06)
- Date: 2026-11-06
- Amends: [ADR 0007](0007-gmns-joins-aks.md), consequence "Alignment task (umbrella level)"
- Related: ADR 0005, ADR 0006

## Context

ADR 0007 says GMNS's UCI-to-CCO mapping "must use AOC-DFO's message classes (InformationArtifactType
and its subclasses) rather than define its own, so one message is modeled one way." Two facts found
since make that rule unworkable as written:

1. **AOC-DFO has no InformationArtifactType class.** Its mission-thread design (Amendment 001, A6)
   made "information artifact type" a role a class plays, flagged with
   `dfo:isInformationArtifactType true`, to avoid a metaclass.
2. **AOC-DFO has no message classes yet.** They come in its Phase 3 (tasks 3.x). GMNS's ontology,
   now public (branch `specs/python-port`), already defines `gmns:C2Message` ⊑ CCO `Message`, and
   its Python port needs a message class before AOC-DFO will have one.

The goal behind the rule still stands: one message, modeled one way, across the shared model.

## Decision

- GMNS **keeps `gmns:C2Message`** (and any message subclasses its CQs need) for now.
- When AOC-DFO's message-content classes exist, GMNS's message classes are **aligned to them** with
  `rdfs:subClassOf` or `owl:equivalentClass`, with a written rationale. The axioms go in AOC-DFO's
  `aoc-align` module, which no core module imports.
- Both ontologies keep their messages under CCO `Message` (Media Content Entity) so the alignment is a
  refinement, not a remodel. A GMNS message class that cannot sit under an AOC-DFO class, or the
  reverse, is a divergence and needs an ADR.

## Options considered

| Option | Outcome |
| --- | --- |
| Keep ADR 0007's rule: GMNS uses AOC-DFO's classes only | Rejected: the class it names does not exist, and the classes it means do not exist yet, so GMNS would block on AOC-DFO Phase 3 |
| GMNS defines its own message classes with no alignment duty | Rejected: two models of one message, which ADR 0005 exists to prevent |
| Keep GMNS's classes now; align in `aoc-align` once AOC-DFO's exist (chosen) | Neither repo blocks; one model is restored by the alignment |

## Consequences

- AOC-DFO task 5.3 (alignment) includes GMNS message classes once AOC-DFO task 3.6 has produced message-content classes.
- Until then, the shared model temporarily has two message hierarchies under CCO `Message`. The
  alignment closes the gap; nothing else depends on it before AKS milestone D2.
- GMNS should not add message classes beyond what its CQs need, to keep the later alignment small.
- The dependency direction is unchanged: alignment axioms live in AOC-DFO's `aoc-align`, which reads
  GMNS's ontology at a pinned commit; GMNS imports nothing from AOC-DFO.
