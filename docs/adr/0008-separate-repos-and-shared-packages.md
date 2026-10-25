# 0008. Separate repos plus versioned shared packages, or a monorepo

- Status: Proposed — needs Kevin's decision
- Date: 2026-10-25
- Related: ADR 0001, ADR 0006, ADR 0007

## Context

Two repository models are in play:

- **AKS (ADR 0001, 0006):** each component has its own repo; the umbrella holds specs and decisions.
- **GMNS Python port, Amendment A1:** prepares for an eventual Semantic Architectures monorepo, with
  three extraction seams that carry no GMNS-specific imports: `sa-semantics` (ontology and SHACL
  tooling), `sa-codecs` (UCI and C2SIM codecs) and `sa-compare`.

Those seams are useful beyond GMNS. The KB and AOC-DFO need `sa-semantics` (pin checking, SHACL,
catalog resolution). Turnpoint would need `sa-codecs` to emit UCI. The question is how they are
shared.

## Options

| Option | For | Against |
| --- | --- | --- |
| A. Separate repos; seams published as versioned packages (recommended) | Consistent with ADR 0001 and every existing component; each repo releases independently; shared code changes are explicit version bumps | A shared change needs a release, then a bump in each consumer; versions can drift |
| B. Semantic Architectures monorepo containing all components | Shared changes are atomic; one CI run tests every consumer | Reverses ADR 0001; mixes public and private work in one repo boundary; couples release cycles; large migration of five existing repos |
| C. Hybrid: separate component repos plus one `sa-shared` repo holding the seams | Shared code in one place, components stay separate | Same release mechanics as A with one more repo |

## Recommendation

Option A, with the seams living in GMNS until a second consumer needs them, then moving to their own
repo (which becomes option C). The Amendment A1 seam discipline (no GMNS-specific imports) already
makes that move cheap. Pin-sensitive packages such as `sa-semantics` must read the shared pin from the
umbrella's ontology steering, not carry their own.

## Consequences if accepted

- GMNS Amendment A1's monorepo-readiness requirements stay, reframed as package-readiness.
- No component migrates repos.
- The first cross-repo package (likely `sa-semantics`, needed by the KB and AOC-DFO) gets its own
  repo when its second consumer starts using it.
