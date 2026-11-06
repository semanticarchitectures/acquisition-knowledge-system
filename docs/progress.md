# Progress against the baseline

- Baseline: [integrated plan](integrated-plan.md), commit `38ff131` (tag `plan-baseline-2026-10-26` to be added from your machine)
- Capacity: Kevin at half time, directing AI coding agents; working rule of at most three active streams
- Updated: 2026-11-06

The baseline dates never change. **Forecast** is the current best estimate; **status** is one of
On track, At risk, Late, Done. Each weekly update adds a log entry and revises the tables.

## Demonstration milestones

| ID | Milestone | Baseline | Forecast | Status | Note |
| --- | --- | --- | --- | --- | --- |
| D0 | Foundations aligned | 2026-11-30 | 2026-11-30 | At risk | ADRs 0009, 0010 accepted; GMNS spec and ontology on a branch; 7 PRs open, none merged; 0002–0004 and 0008 open |
| D1 | The AI-built enterprise, running | 2027-03-31 | 2027-04-30 | At risk | Aircrew-LangChain Phase 1 built 2026-11-06 (branch, gate passed, not merged); ENSIM audit and ENCAP Phase 2 gate not begun; AOC-DFO Phases 1–4 not started |
| D2 | At scale, over a realistic network | 2027-08-31 | 2027-12-15 | At risk | GMNS port spec, ontology and CCO 2.2 pin ready for review (PRs #6, #7); core port not started |
| D3 | AI in the enterprise | 2027-12-15 | 2028-03-31 | At risk | Follows D2 |
| D4 | Plan to AOC over a degraded network | 2028-05-31 | 2028-08-31 | At risk | Follows D3 |
| D5 | Closed loop | 2028-10-31 | 2029-01-31 | At risk | Follows D4 |

Forecasts past D1 reflect the three-stream rule at half time, which the baseline dates predate.

## D0 checklist

- [x] ADR 0009 accepted (2026-10-26); umbrella specs revised to match
- [ ] ADRs 0002, 0003, 0004, 0008 decided
- [ ] Umbrella-link PRs merged: [ENSIM #1](https://github.com/semanticarchitectures/ensim/pull/1),
      [ENCAP #1](https://github.com/semanticarchitectures/encap/pull/1),
      [Aircrew-LangChain #1](https://github.com/semanticarchitectures/aircrew-langchain/pull/1),
      [GMNS #5](https://github.com/semanticarchitectures/GlobalMobileNetworkSim/pull/5),
      [Turnpoint #1](https://github.com/semanticarchitectures/turnpoint/pull/1) (AOC-DFO already linked)
- [ ] Turnpoint goal revised: decision 0018 in [Turnpoint #1](https://github.com/semanticarchitectures/turnpoint/pull/1)
- [ ] GMNS Python port spec and ontology files committed: on branch `specs/python-port` ([GMNS #6](https://github.com/semanticarchitectures/GlobalMobileNetworkSim/pull/6)); shared CCO pin in [GMNS #7](https://github.com/semanticarchitectures/GlobalMobileNetworkSim/pull/7); merge pending
- [x] Decision traces written to files until the KB exists: confirmed (2026-11-06); `decision-traces`
      requirements revision drafted, awaiting approval
- [x] ADR 0010 accepted (2026-11-06): GMNS keeps its message classes, aligns to AOC-DFO later

## Active streams (cap: three)

| Stream | Component | Baseline window | State |
| --- | --- | --- | --- |
| D0 decisions and quick starts | Umbrella, Turnpoint, GMNS | 2026-10-27 to 2026-11-30 | Active: GMNS items done on branches; reviews and decisions pending |
| AOC-DFO Phases 1–4 | AOC-DFO | 2026-10-26 to 2026-12-31 | Behind: 8 of 64 tasks done (Phase 0); only design 3.7 for ADR 0010 since 2026-10-24 |
| Aircrew-LangChain Phase 1 (mission order, identity) | Aircrew-LangChain | 2026-11-01 to 2026-12-31 | Active: chosen 2026-11-06; built the same day on branch `phase-1/mission-order`, gate passed; merge pending |

## Log

### Week of 2026-10-26

- Plan re-sequenced to the two-layer goal and baselined; capacity confirmed at half time.
- ADR 0009 accepted: ENCAP owns the passage index and access model; `knowledge-base-prototype` and
  `kb-mcp-contract` revised (no KB index; scope issued by ENCAP; `kb_search` moved to ENCAP).
- Turnpoint decision 0018 drafted (unmanned aircraft and the AOC interface) in Turnpoint #1.
- Umbrella links added in five component repos as pull requests; ENCAP's PR also amends its plan for
  ADR 0009.
- Portfolio roles for ENSIM and Aircrew-LangChain updated to the re-sequenced plan.

### Week of 2026-10-26, continued (2026-10-29)

- Use case added: focused acquisition portfolio and resource evaluation (vision), recorded in the
  integrated plan's backlog. No change to the baseline or forecasts.

### Week of 2026-11-02 (update 2026-11-06)

- GMNS moved: Python-port specs, ontology and Claude Code governance on branch `specs/python-port`
  (GMNS #6, 2026-11-03); ontology moved to the shared CCO 2.2 pin (GMNS #7). GMNS was already on
  CCO 2.x, so the pin move was a bump, not the IRI migration the plan expected.
- ADR 0010 accepted: GMNS keeps `gmns:C2Message`; AOC-DFO aligns it in `aoc-align` once AOC-DFO has
  message classes (its Phase 3). ADR 0007 and ontology steering corrected.
- No pull request merged yet: the five umbrella-link PRs (open since 2026-10-26) and GMNS #6, #7.
- No commits on ENSIM, ENCAP, Aircrew-LangChain or Turnpoint since the baseline; the Nov 1 starts
  (ENSIM realism audit, ENCAP Phase 2 gate, Aircrew-LangChain Phase 1) have not begun.
- AOC-DFO Phases 1–4 have not started, 1.5 of 9.5 baseline weeks in.
- Forecasts: D0 to At risk (review backlog); D1 forecast moved to 2027-04-30.
- 2026-11-06: Aircrew-LangChain Phase 1 chosen as the third active stream. Built the same day on
  branch `phase-1/mission-order` (commit `94b8c71`, not merged): mission orders with a roster and
  nets, checked against ENSIM's operating model at `aa7b714`; agent ids in the
  `<Type>V<version>.Instance<n>` form the `decision-traces` spec uses; authority keyed by role.
  Gate passed: scripted scorecards unchanged from Phase 0, a two-loadmaster run, invalid orders
  rejected with their reasons. live2 recorded as the `GeneralistV1` baseline.
- Mission orders do not come from ENSIM yet, as the plan assumes for Aircrew-LangChain Phases 1–2:
  ENSIM's step-6 "Mission tasking" artifact has a name and no content. Orders are written in
  Aircrew-LangChain's format (its `docs/mission-order.md`) and validated against ENSIM's records;
  the format is offered as input to the simulation-interface ADR (due 2027-01-31).
- 2026-11-06: decision traces go to files in the trace format until the KB trace store exists,
  through one recording interface so decision nodes do not change when `kb_record_decision`
  arrives (confirmed by Kevin). The `decision-traces` requirements are revised to match (R1.4, R3,
  R7: proposed, awaiting approval); R3.1 previously required `kb_record_decision`, which contradicted
  the plan.
