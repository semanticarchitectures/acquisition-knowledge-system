# Progress against the baseline

- Baseline: [integrated plan](integrated-plan.md), commit `38ff131` (tag `plan-baseline-2026-10-26` to be added from your machine)
- Capacity: Kevin at half time, directing AI coding agents; working rule of at most three active streams
- Updated: 2026-10-26

The baseline dates never change. **Forecast** is the current best estimate; **status** is one of
On track, At risk, Late, Done. Each weekly update adds a log entry and revises the tables.

## Demonstration milestones

| ID | Milestone | Baseline | Forecast | Status | Note |
| --- | --- | --- | --- | --- | --- |
| D0 | Foundations aligned | 2026-11-30 | 2026-11-30 | On track | ADR 0009 accepted; 0002–0004 and 0008 open; umbrella-link PRs open |
| D1 | The AI-built enterprise, running | 2027-03-31 | 2027-03-31 | At risk | Four streams feed D1; the three-stream rule means one waits |
| D2 | At scale, over a realistic network | 2027-08-31 | 2027-12-15 | At risk | GMNS port not started; three-stream rule pushes it later |
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
- [ ] GMNS Python port spec and ontology files committed
- [ ] Decision traces written to files until the KB exists: confirmed

## Active streams (cap: three)

| Stream | Component | Baseline window | State |
| --- | --- | --- | --- |
| D0 decisions and quick starts | Umbrella, Turnpoint, GMNS | 2026-10-27 to 2026-11-30 | Active |
| AOC-DFO Phases 1–4 | AOC-DFO | 2026-10-26 to 2026-12-31 | Active |
| Third stream from Nov 1 | Choose one: ENSIM realism audit, ENCAP Phase 2 gate, or Aircrew-LangChain Phase 1 | from 2026-11-01 | Not chosen |

## Log

### Week of 2026-10-26

- Plan re-sequenced to the two-layer goal and baselined; capacity confirmed at half time.
- ADR 0009 accepted: ENCAP owns the passage index and access model; `knowledge-base-prototype` and
  `kb-mcp-contract` revised (no KB index; scope issued by ENCAP; `kb_search` moved to ENCAP).
- Turnpoint decision 0018 drafted (unmanned aircraft and the AOC interface) in Turnpoint #1.
- Umbrella links added in five component repos as pull requests; ENCAP's PR also amends its plan for
  ADR 0009.
- Portfolio roles for ENSIM and Aircrew-LangChain updated to the re-sequenced plan.
