# 0011. GMNS stays MATLAB; UCI and C2SIM are built natively, Python port deferred

- Status: Accepted (2026-11-12)
- Amends: [ADR 0007](0007-gmns-joins-aks.md) (GMNS's role and stack), [ADR 0008](0008-separate-repos-and-shared-packages.md) (the shared-seam rationale that assumed the Python port)
- Relates: [ADR 0010](0010-gmns-message-classes-align-to-aoc-dfo.md) (GMNS message classes — unaffected)

## Context

ADR 0007 brought GMNS into AKS as the network-transport and standards-conformant C2
messaging testbed, on the assumption — carried from the GMNS `python-port` spec set — that
a Python port would become the authoritative implementation and the MATLAB version would be
frozen as a comparison baseline (python-port Amendment A2). The umbrella plan was built on
that: D2 lists "GMNS Python port (M1) coupled to the simulations" and D4 lists "GMNS UCI
profile (M2)," both keyed to the Python port's milestones.

On review of the GMNS repo (2026-11-12), the port had not started — no `src/gmns/`, no
`pyproject.toml` — while the MATLAB implementation had instead grown substantially (new
`+fabric`, `+data`, and `+security` packages; Phase 10–11 work). GMNS's own `CLAUDE.md`
still asserted "Python is the authoritative implementation," which was false. GMNS
**Amendment A3** (2026-11-12) corrected this: MATLAB is authoritative and the Python port is
deferred.

A3 leaves the umbrella with a gap. GMNS exists in AKS *for* the standards-conformant C2
capabilities — real UCI message exchange and the planned C2SIM gateway — which are D4/D5
deliverables and the point of the far-horizon thread (Turnpoint → AOC over GMNS). If the
Python port is the only path to those capabilities and the port is deferred, D2 loses its
network layer and D4 loses UCI.

## Decision

**GMNS remains MATLAB, and the UCI and C2SIM capabilities are built natively in MATLAB.**
The Python port is deferred, not cancelled; if it is ever reactivated it must be re-scoped
against the then-current MATLAB codebase (GMNS Amendment A3).

Consequences for the capabilities AKS depends on:

- **Network transport (D2).** Delivered by the current MATLAB engine, coupled to the
  simulations over the simulation interface (the simulation-interface ADR, due Jan 2027),
  not over a Python MCP server. "GMNS M1" in the plan is redefined as *MATLAB network layer
  ready for coupling*, which largely exists today, materially reducing the D2 schedule risk.
- **UCI (D4).** Built in MATLAB: emulated mission systems exchange real UCI messages whose
  serialized size drives transmission delay over the simulated links. "GMNS M2" is redefined
  as the *MATLAB UCI profile and codec*.
- **C2SIM (D5).** Built in MATLAB as the cross-simulation interface, consistent with the
  simulation-interface ADR.
- **ADR 0010 (message classes) is unaffected.** `gmns:C2Message` and its alignment to
  AOC-DFO's message-content classes are ontology work independent of the implementation
  language.

## Consequences

- **ADR 0007 amended:** GMNS's stack is "MATLAB (Python port deferred)," not "Python port,
  MATLAB baseline frozen." Its role and boundary are otherwise unchanged — it still runs as
  its own simulation, reached over the simulation interface / C2SIM, never embedded.
- **ADR 0008 amended:** the three shared seams (`sa-semantics`, `sa-codecs`, `sa-compare`)
  were motivated by the Python port's Amendment A1. With the port deferred, GMNS no longer
  produces them. AKS's need for `sa-semantics` (pin checking, SHACL) is real and now traces
  to the KB and AOC-DFO, not to GMNS; where it is built is reopened and left to a future ADR.
  The seam-readiness constraints in the python-port spec are dormant with the rest of it.
- **Schedule.** The "largest schedule risk" framing changes: there is no large Python port to
  build (risk down), but UCI and C2SIM in MATLAB are their own builds and remain real work
  on the D4/D5 critical path (risk persists, relocated). The integrated plan's D2 input is
  lighter; its D4/D5 inputs are comparable in size but in MATLAB.
- **Standards-conformance evidence.** The python-port spec framed conformance as
  first-principles verification against a frozen oracle. With MATLAB authoritative and
  carrying UCI/C2SIM, conformance evidence comes from schema-validating emitted UCI/C2SIM
  messages against their published schemas, in GMNS's own spec — not from this ADR.

## Alternatives considered

| Option | Why not |
| --- | --- |
| Reactivate the Python port on its original schedule | The port never started, MATLAB has grown past the spec, and the port is the largest single build in the plan; forcing it keeps D2 at maximum risk |
| Defer the port *and* the UCI/C2SIM capabilities | Rejected: UCI/C2SIM are the reason GMNS is in AKS (D4/D5); dropping them removes GMNS's purpose |
| Build UCI/C2SIM in MATLAB now; port later if ever (chosen) | Delivers the capabilities AKS depends on without blocking on a large port; keeps the port as a dormant, re-scopable option |
