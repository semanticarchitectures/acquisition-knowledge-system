# Program definition: inputs to date

- Status: Inputs being collected (not yet a spec). Becomes the seed of the realizable program
  documentation set when that backlog candidate is promoted
  ([integrated plan, backlog](../integrated-plan.md#backlog-not-baselined)).
- Owner: Kevin Kelly
- Last input: 2026-11-08 (security operations scope)

Every statement below is Kevin's input unless marked *(proposed)*. Proposed items are
recommendations awaiting his decision.

## The program

- **What it acquires:** a mission planning system for large-scale autonomous operations that
  integrates with current mission planning and operations. (2026-11-08)
- **Character:** fictional in name and clearly labeled, but **realizable**. Needs, constraints,
  standards and integration points are real and public; each element is marked public,
  representative or fictional. (2026-11-08)
- **Reference implementation:** Turnpoint. Test environment: GMNS, ENSIM, Aircrew-LangChain. AOC side:
  AOC-DFO. (2026-11-08)
- **Documents:** CONOPS, capability and technical requirements, acquisition strategy, systems
  engineering plan, test strategy, security and program protection guidance, and the rest of the set
  the chosen acquisition pathway calls for. Each follows a cited public template and is replaceable by
  official documents where they exist, in the CUI environment only. (2026-11-08)

## Requirements baseline

- **Start from FAA Part 108** (with Part 146, automated data service providers). Military
  requirements are written as traced deltas against this civil baseline, so a commercial Part 108
  operator can see exactly what a military system adds. (2026-11-08)
- Part 108 is still proposed (NPRM 2025-08-07; final rule under OIRA review since 2026-07-10;
  statutory deadline 2026-12-07). Requirements pin the text and section they derive from and are
  re-diffed when the final rule publishes.
- Industry consensus standards Part 108 relies on are cited, never reproduced (license).

## CONOPS: mission set

- **Mission set: contested logistics.** (2026-11-08)
- **Unclassified missions:** logistics and security operations in a remote area to provide
  humanitarian assistance. (2026-11-08)
- **Mission lifecycle:** (2026-11-08)
  1. Prepositioning materials
  2. Deploying manned and unmanned airlift that transports the materials to where they are needed
  3. Sustainment
  4. Redeployment

- **Security operations are observation and non-violent means, such as talking.** (2026-11-08) No
  weapons employment or use of force, consistent with Turnpoint's rules.

### Proposed elaboration *(proposed, 2026-11-08; for Kevin's review)*

| Lifecycle phase | Planning problem for the system | Where AKS already has it |
| --- | --- | --- |
| Prepositioning | Where to stage which materials, given expected demand, access and risk | Portfolio and resource evaluation use case (vision) |
| Deployment | Airlift planning for mixed manned and unmanned fleets; airspace integration and deconfliction; handoff to the AOC | ENSIM Mission 1 (C-17 humanitarian airdrop) and Mission 2 (UAS deployment); Aircrew-LangChain; Turnpoint Phases 5–7; AOC-DFO |
| Sustainment | Recurring resupply at volume: many autonomous sorties, replanning as demand and conditions change | GMNS (degraded links at volume); Turnpoint batch planning |
| Redeployment | Retrograde of people, materials and aircraft; drawing down the operation | Not yet modeled |

- **"Contested", kept unclassified:** degraded or denied communications and navigation, congested
  airspace shared with civil and relief aircraft, damaged infrastructure, weather, and ground security
  threats, all with notional parameters.

- **Part 108 fit:** humanitarian operations often run in civil airspace alongside civil and relief
  drones, and a domestic disaster-response variant would fall directly under Part 108. That makes the
  civil baseline operationally real, not only a reference point.

## Open inputs

- [ ] Scale: sorties per day, number of aircraft, number of delivery points.
- [ ] Human roles: where humans approve, supervise or only monitor.
- [ ] Cargo and vehicle classes: payload weights and ranges for the unmanned airlift.
- [ ] Integration points: which current planning and operations functions the system must work with.
- [ ] Setting: host-nation scenario (as ENSIM Mission 1) or a domestic disaster-response variant, or
      both.
- [x] Security-operations scope: observation and non-violent means, such as talking (2026-11-08).
