# Vision and goal

- Status: Draft for approval
- Date: 2026-10-18
- Owner: Kevin Kelly

## Goal

**Show, end to end and with evidence, that AI can speed up how the Air Force conceives, acquires,
fields and operates capability, while staying safe, secure and reliable under human authority.**

AKS is first a **capability demonstration**. It succeeds when it produces convincing evidence, not
when it is adopted. Every claim the demonstration makes must be backed by something a skeptical
reviewer can inspect: a citation, a decision trace, a test result or an evaluation score.

## Horizons

The vision runs further than the first build. Each horizon reuses what the one before it built.

| Horizon | Focus | What it demonstrates |
| --- | --- | --- |
| **Near** (now; Phase 1) | Capability acquisition with AI assistance inside the Air Force acquisition system. First tenant: the Mission Planning program office | Agents that are grounded, cited and governed. A knowledge graph and context graph of a real acquisition organization |
| **Mid** | Mission planning for **unmanned aircraft** (Turnpoint), with agents planning missions inside granted authority | AI that acts, not only advises, inside a bounded envelope with a full audit trail |
| **Far** | **Organizing autonomous operations at scale.** The software between unmanned mission plans and operations and the **Air Operations Center (AOC)**, for a future with orders of magnitude more unmanned missions than manned | Machine-to-machine command and control with humans on the loop, still under human authority and still auditable |

## One vision across levels and lifecycle

The end state spans the **strategic, operational and tactical** levels and the **capability
lifecycle** from concept development through production, then into operations, whose results feed
back into the next concept.

What makes it one vision is one shared model, not one application (see
[ADR 0005](adr/0005-one-model-many-apps.md)): a single ontology, knowledge graph and decision-trace
record that every app and agent reads and writes. A requirement, the acquisition decision that funded
it, the software that implements it, the missions it planned and the results of those missions are
all nodes in the same graph, linked by cited relationships.

| Lifecycle stage | Strategic | Operational | Tactical |
| --- | --- | --- | --- |
| Concept and requirements | Enterprise needs and policy (KB L0–L2, ENSIM enterprise) | Mission needs at the AOC (AOC-DFO) | Not applicable |
| Acquisition and development | Portfolio and funding decisions (KB L3, decision traces) | Program office execution (KB L4, ENCAP agents) | Software built against traced requirements (spec-driven development) |
| Production and fielding | Fielding decisions | Integration with the AOC (AOC-DFO interface) | Released planning software (Turnpoint) |
| Operations and feedback | Lessons into policy and portfolio | Unmanned mission tasking and deconfliction with the AOC; C2 traffic under degraded comms (GMNS) | Mission plans executed in simulation (Aircrew-LangChain, ENSIM) |

## The demonstration thread

Across all three horizons, one story runs through every component:

1. **Concept.** A need appears: the AOC cannot plan and deconflict many times more unmanned missions
   with today's processes.
2. **Requirements and acquisition.** Agents grounded in the KB trace the need to its authorities and
   a likely acquisition pathway. Each milestone and funding decision is a decision trace. ENSIM
   supplies synthetic decisions before real ones exist.
3. **Development.** The capability is built: an unmanned planning feature in Turnpoint, developed
   spec-driven with AI assistance, each task traced to a requirement.
4. **Operations.** Turnpoint plans unmanned missions and passes them to the AOC through an interface
   modeled with AOC-DFO, as real UCI messages carried over a simulated, degraded network (GMNS).
   Agents fly the missions in simulation.
5. **Feedback.** Mission results return to the graph as evidence against the original requirement,
   which closes the loop.

Every step leaves evidence that is cited, traced and checked against authority. That evidence is the
demonstration.

## Safe, secure and reliable: properties the demonstration must show

These are claims the demonstration must back with evidence, not adjectives. Risk classification and
reliability criteria come from the AFMC AI governance policy (v2).

| Property | What the demo must show | Mechanism |
| --- | --- | --- |
| **Safe** | AI never acts beyond the authority it was granted. Humans keep decision authority. Autonomy stays inside a defined envelope | Governed envelope, authorization by role, two-person control, outputs labeled as decision support |
| **Secure** | Data never crosses a boundary without review. Every client sees only its scope | Classification markings, echelon scopes, scoped tokens, a reviewed promotion step |
| **Reliable** | Answers are cited and checked. Gaps are declared, not filled in. Changes do not cause silent regressions | CQ golden tests, citation faithfulness checks, evaluation against ENSIM, pinned models and prompts |
| **Traceable** (the means to all three) | Every decision, by person or agent, can be replayed: who, under what authority, using what | Decision traces, provenance, call logs |

## Guardrails on the vision

1. **One model, many apps; not one app.** The levels differ in tempo, assurance needs,
   classification and users. A single application would be forced to the lowest common denominator.
   ([ADR 0005](adr/0005-one-model-many-apps.md))
2. **AI-accelerated, human-decided.** Acquisition decisions and decisions to use force carry legal
   authority that AI does not hold. "AI-driven" describes speed and scale, never the locus of
   authority.
3. **Depth before breadth.** One thread demonstrated end to end is worth more than five horizons each
   half built. The near horizon stays the active work; the mid and far horizons shape design choices
   now, but are not built yet.
4. **Tactical execution needs deterministic, verifiable behavior.** Language-model agents belong at
   planning and supervision. At the tactical edge, AI acts as bounded autonomy inside verified
   envelopes, not as a language model in a real-time control loop.
5. **Public data limits the far horizon.** AOC interface and unmanned C2 details are largely
   restricted. The far horizon is shown with public doctrine, surrogates (as AOC-DFO uses NATO
   STANAGs) and fictional ENSIM elements, all labeled as such.

## Open questions

- [ ] **Name.** "Acquisition Knowledge System" describes only the near horizon. Keep it, or rename the
      umbrella to fit the full vision?
- [ ] **Turnpoint's goal.** Its current goal is FalconView parity, a workflow for manned aircrew.
      Planning unmanned missions at scale needs batch planning, deconfliction, autonomy envelopes and
      interfaces between machines. Should Turnpoint's goals be revised in its own repo?
- [ ] **Demonstration audience.** Who must the demonstration convince first: program office leadership,
      PEO C3BM, AFMC, or the AI governance community? The answer sets which evidence comes first.
- [ ] **Unmanned scale target.** What multiple of today's mission volume should the far-horizon
      simulation demonstrate?
