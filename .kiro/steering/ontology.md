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

## Pinned versions (shared by every project ontology, ADR 0006)

| Ontology | Pinned release | Source and integrity | Used by |
| --- | --- | --- | --- |
| BFO 2020 | Tag `release-2024-01-29` (selected 2026-10-22) | [BFO-ontology/BFO-2020](https://github.com/BFO-ontology/BFO-2020) commit `044490fc5100ffed6df7d4d15cbc167698b6fdee`, file `src/owl/bfo-core.ttl`, SHA-256 `6944e2c96a0d8709dc15ae5364792019e208061eda3f32311af4f3b8d8443a3b`. License CC BY 4.0 | KB ontology, AOC-DFO, GMNS |
| CCO | `v2.2`, version IRIs `https://www.commoncoreontologies.org/2026-08-13/…` (selected 2026-10-22) | [CommonCoreOntology/CommonCoreOntologies](https://github.com/CommonCoreOntology/CommonCoreOntologies) commit `0bc7d33e1bc09fd4693366119ab4e03cb0340042`, file `src/cco-merged/CommonCoreOntologiesMerged.ttl`, SHA-256 `a9453382b25b40781c181d6ba44981f228abb76d90d548846713adba9baadb42`. License BSD 3-Clause | KB ontology, AOC-DFO, GMNS |

Change a pin here first, then in each component. Never change a pin in one component alone.

Notes on the current pins:

- **BFO has no dated version IRI.** Its `owl:versionIRI` is the unversioned PURL
  `http://purl.obolibrary.org/obo/bfo/2020/bfo-core.ttl`, and CCO 2.2 imports that same PURL. Each
  component vendors the pinned file and resolves the PURL to it through an XML catalog
  (`catalog-v001.xml`), and CI checks the SHA-256 above. Never resolve the PURL over the network.
- **Why the 2024 BFO tag, not master.** BFO master carries untagged, annotation-only fixes from
  Sep 2025 (definitions aligned to the ISO 21838-2 text; no axiom changes). The tag
  `fol-release-2026-08-06` is the first-order-logic axiomatization, not the OWL file. Revisit when BFO
  tags a new OWL release.
- **CCO 2.x uses opaque IRIs** (`…/ont00000958` = Information Content Entity). Specs may write
  label shorthand such as `cco:InformationContentEntity`; ontology files must use the IRIs. Moving a
  CCO 1.x model (GMNS) to 2.2 is an IRI migration, not a version bump.
- **CCO 2.2 dropped some 1.x classes** specs may name: there is no Directive Information Content
  Entity (ICE subclasses are Descriptive, Designative, Prescriptive, Representational, Media Content
  Entity, Ideology, Religion), no Military Organization, and no Organization Role.
- Import only the merged core file. CCO extensions (Modal Relation, Familial Relations, Barcode) are
  not part of the pin; adding one needs a CQ and an update to this table.
- **CCO's merged file embeds BFO** (it has no `owl:imports`). Checked 2026-10-22: the embedded BFO
  is logically identical to the pinned tag; it differs only in annotations (scope-note links from a
  Feb 2024 BFO commit) and adds CCO's "Process Profile" (`BFO_0000144`). Importing both files is
  consistent: AOC-DFO's full closure reasons clean under HermiT (ROBOT 1.9.8).
- **GMNS:** its ontology files are not in the public GMNS repo (checked 2026-10-22, all branches),
  so its move to this pin cannot be verified yet.

## Alignment

- Align with **AOC-DFO** so the two graphs join without remapping.
- **GMNS's UCI-to-CCO mapping** uses AOC-DFO's message classes (InformationArtifactType and its
  subclasses) rather than its own, so one message is modeled one way (ADR 0007).
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
