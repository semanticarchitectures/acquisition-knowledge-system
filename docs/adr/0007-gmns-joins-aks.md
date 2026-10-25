# 0007. GMNS joins AKS as the network transport and C2 messaging testbed

- Status: Accepted (2026-10-25)
- Date: 2026-10-25
- Amends: ADR 0006 (its rationale that GMNS is "outside AKS")
- Related: ADR 0005, ADR 0008

## Context

GMNS (GlobalMobileNetworkSim) is a discrete-event simulation of a mobile communications network:
fiber, GEO, LEO and line-of-sight links, outages, LLM agent emulation and an identity layer. Its
Python port spec makes it a standards-conformant C2 interoperability testbed:

- Emulated mission systems exchange real UCI messages (Open Arsenal schemas, a bounded profile of
  about 10–20 message types). Each message's serialized size drives its transmission delay through
  the simulated network.
- A SISO C2SIM (SISO-STD-019) gateway, a separate spec after the port's Milestone M2, handles scenario
  initialization, tasking and reporting with external simulations.
- Each event is rendered twice: conformant UCI on the wire, a BFO/CCO record in the log.

The AKS far horizon is machine-to-machine C2 between unmanned mission plans and the AOC, at many
times today's volume. AOC-DFO models what flows; nothing in AKS yet models whether and when those
flows arrive over a degraded network. ADR 0006 described GMNS as outside AKS, which no longer fits.

## Decision

GMNS becomes an AKS component. Its role: **the network transport and standards-conformant C2
messaging testbed**. It carries the demonstration thread's unmanned-to-AOC traffic (step 4) as real
UCI messages under latency and outage, and supplies evidence for the "reliable" property in the
vision.

GMNS keeps its own repo (`semanticarchitectures/GlobalMobileNetworkSim`) and its own specs. It follows
the shared BFO/CCO pin (ADR 0006).

## Options considered

1. **Keep GMNS outside AKS** and depend on it informally. Rejected: the far horizon needs it, and an
   informal dependency escapes the shared-model and pin rules.
2. **GMNS as an AKS component** (chosen).

## Consequences

- ADR 0006's decisions stand. Its rejection of a monorepo with GMNS stands too, but for a different
  reason: the repository model is now decided by ADR 0008, not by GMNS being outside AKS.
- **Alignment task (umbrella level):** GMNS's UCI-to-CCO mapping must use AOC-DFO's message classes
  (InformationArtifactType and its subclasses) rather than define its own, so one message is modeled
  one way.
- **GMNS's move to the shared pin is a CCO 1.x to 2.2 IRI migration.** Its ontology files are not in
  the public repo yet, so the migration cannot be verified from here.
- **C2SIM as the simulation interface across AKS** (ENSIM, Aircrew-LangChain, GMNS, Turnpoint) is a
  candidate decision, deferred until the C2SIM gateway exists. Today the gateway is specified, not
  built, and the UCI profile is not yet selected.
- The vision's demonstration thread names GMNS as the transport in step 4.
