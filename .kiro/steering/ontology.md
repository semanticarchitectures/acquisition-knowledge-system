---
inclusion: fileMatch
fileMatchPattern: "**/*.{ttl,owl,rq,shacl}"
---

# Ontology rules

Loaded when working on ontology, shape or SPARQL files, and on any spec that changes the ontology.

## Tiers

1. **Upper:** BFO and CCO, imported unchanged and pinned by version. Never edit imported files.
2. **Mid-level (KB ontology):** subclassed from CCO. Only what a competency question needs.
   A new class or property must name the CQ that requires it.
3. **Shapes:** one SHACL shape per class and per n-ary relation. Shapes gate extraction, validate
   commits and generate the extractor's JSON schema.

## Alignment

- Align with **AOC-DFO** so the two graphs join without remapping.
- Reuse **ENSIM's org-doctrine schema** names where they exist (Organization, Role, C2Node,
  DoctrineProcess, Decision), so the real enterprise and the simulated one are described alike and
  evaluation compares like with like. Record any divergence in an ADR.

## Modeling patterns

- **Time:** relationships that change (assignment, reporting line, portfolio membership, funding)
  are n-ary nodes with `validFrom` and `validTo`. Never state structure without a validity interval.
- **Provenance:** each source version is its own named graph described with PROV-O. Each claim links
  to the OKF section anchor it came from.
- **Echelons:** L0 U.S. Government, L1 DoD, L2 Department of the Air Force, L3 Portfolio, L4 Program
  office. Each is a named-graph set. An agent reads its own layer and every layer above, never below.
- **Supersession:** a fact is supported while at least one non-superseded source asserts it.

## Change control

Ontology changes go through review like code: a PR with the CQ that motivates the change, a ROBOT
QC report, and passing shape tests.
