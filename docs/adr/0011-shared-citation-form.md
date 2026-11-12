# 0011. One citation form across the shared model

- Status: Proposed (2026-11-11)
- Date: 2026-11-11
- Refines: ontology steering, "Modeling patterns → Provenance"
- Related: ADR 0005 (one model, many apps), ADR 0006 (shared pins), AOC-DFO Amendment 002 (B1, OQ-1)

## Context

Product principle 1 says every claim cites a source, and the ontology steering says each source
version is a named graph described with PROV-O and each claim links to the anchor it came from. It
does not say which property links a claim to its source, or how the anchor is written. AOC-DFO had
to settle both in Phase 1 and found two problems with the obvious answer (Amendment 002, B1):

1. **`prov:wasDerivedFrom` on a class leaves OWL 2 DL.** PROV-O declares it an object property.
   Asserting it on a class makes it an annotation property too, and OWL 2 DL forbids one IRI being
   both. Any component that does this breaks DL for every graph that loads it with PROV-O, and the
   KB imports PROV-O.
2. **A separate anchor property loses its pairing.** A term citing two sources with two
   `anchor` values cannot say which anchor belongs to which source.

The KB will merge terms and claims from AOC-DFO, GMNS and later components. If each cites its own
way, the KB must translate citations or cannot check them uniformly.

## Decision

Every component in the shared model cites the same way:

| Cites | Property | Object |
| --- | --- | --- |
| Classes and properties (TBox) | `dcterms:source` (annotation property) | an anchored source-version IRI |
| Individuals and claims (ABox) | `prov:wasDerivedFrom` | an anchored source-version IRI |

- **Anchored source-version IRI:** `<component base>/source/<source-version id>#<anchor>`. The
  fragment is the anchor, so every citation carries its own. No separate anchor property.
- **Source version:** each `<component base>/source/<id>` (no fragment) is one edition of one
  source, a `prov:Entity` in its own named graph, carrying at least its edition, location,
  retrieval time, content hash, handling caveat and any `supersededBy` link.
- **Anchors** are chosen by the component that owns the source (paragraph numbers for AFMANs, type
  names for NIEM, OKF section anchors for the KB) and must resolve to text the component has
  ingested. Each component checks that in CI.
- Supersession stays as the steering says: a fact is supported while at least one non-superseded
  source version asserts it.

## Options considered

| Option | Outcome |
| --- | --- |
| `prov:wasDerivedFrom` everywhere, plus an anchor property (AOC-DFO design as first approved) | Rejected: leaves OWL 2 DL on classes; anchors lose their source |
| Each component chooses its own form | Rejected: the KB would translate or skip checks; contrary to ADR 0005 |
| `dcterms:source` for terms, `prov:wasDerivedFrom` for claims, anchor in the IRI (chosen) | DL-safe, one form to check, already implemented and tested in AOC-DFO |
| Reified citation nodes (one node per citation with source and anchor properties) | Rejected for now: heavier queries and more triples for no gain at current scale; revisit if a citation needs its own metadata (confidence, quoted text) |

## Consequences

- AOC-DFO already conforms (Amendment 002, accepted 2026-11-11; SHACL shapes and the citation gate
  in its Phase 1).
- GMNS cites nothing yet; it adopts this form when it first does. No change now.
- The KB's claims cite OKF sections as `…/source/<OKF doc version>#<section anchor>`. The KB owns
  the graph and mints those IRIs; the anchors resolve to passages in ENCAP's index (ADR 0009).
- The ontology steering's provenance bullet is updated to name the properties and IRI form.

## Open question

- **Shared sources.** Two components citing the same publication (for example the KB and AOC-DFO
  both citing AFDP 3-0.1) would mint two source-version IRIs under different bases. Options: a
  shared source registry in the umbrella, or `owl:sameAs` links in an alignment module. Not decided
  here; no component cites a shared source yet.
