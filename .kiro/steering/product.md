---
inclusion: always
---

# Product: Acquisition Knowledge System (AKS)

## Goal

**Demonstrate how AI can architect and simulate Air Force enterprise operations, at scale and
realistically, and build on that simulation to show how AI can be used in the enterprise, while
staying safe, secure and reliable under human authority.**

Two layers, in order: (1) AI architects and simulates the enterprise (ENSIM, AOC-DFO, GMNS,
Aircrew-LangChain); (2) AI is used in that enterprise (KB, ENCAP, Turnpoint). Audience: Air Force and
contractor personnel, who will judge realism against how the enterprise actually works.

AKS is first a **capability demonstration**. Judge every design choice by whether it produces
evidence a skeptical reviewer can inspect: citations, decision traces, test results, evaluation
scores. Full statement: [docs/vision.md](../../docs/vision.md).

## Horizons

| Horizon | Focus |
| --- | --- |
| Near | AI-assisted capability acquisition in the Air Force acquisition system; Mission Planning program office (PAE C3BM) |
| Mid | Mission planning for unmanned aircraft (Turnpoint), agents acting inside granted authority |
| Far | Organizing autonomous operations at scale: the interface between unmanned mission plans and operations and the AOC |

One vision across the strategic, operational and tactical levels and the capability lifecycle is
realized as **one shared model, many apps** (ADR 0005, Accepted), not one application. Active work follows the two-layer order in
[docs/integrated-plan.md](../../docs/integrated-plan.md): simulate the enterprise first, then use AI in it.
The horizons describe scope, not sequence.

## What it is today

AKS composes a family of existing projects into one capability. It gives an Air Force acquisition
organization citation-grounded AI agents over a layered public corpus, and a knowledge graph and
context graph that model the organization's structure, mission, relationships, resources and
operations.

The first tenant is the **Mission Planning program office** under **PAE C3BM**.

AKS adds no new platform. Each component keeps its own repo and role (see `portfolio.md`). This
repo holds the shared product intent, architecture, cross-cutting specs and decisions.

## Users

| User | Needs |
| --- | --- |
| Program office staff (PM, engineers, contracting, finance) | Grounded answers about policy, structure, funding and process, each with a citation |
| AI Facilitator Teams (embedded engineer/PM pairs) | A reusable way to stand up the capability in a new program office |
| AI agents (ENCAP agents, LangGraph agents, Claude) | Typed, scoped, cited access to the knowledge graph through MCP |
| Reviewers and governance | Review queues, decision traces and audit of every answer |

## Properties the demonstration must show

Safe (AI never exceeds granted authority; humans decide), secure (no unreviewed boundary crossing;
scoped access), reliable (cited, checked, regression-tested; gaps declared), and traceable (every
decision replayable). See the table in [docs/vision.md](../../docs/vision.md).

AI-accelerated, human-decided: "AI-driven" never means AI holds acquisition or use-of-force authority.

## Principles (from the reference architecture)

1. **Every claim cites a source.** An uncited claim is a defect, not a style issue.
2. **Flag gaps, never invent.** When the corpus has no answer, agents say so and log the gap.
3. **Hard classification boundary.** Every document, graph and agent carries a marking. Nothing
   crosses a boundary except through a reviewed promotion step.
4. **License and provenance tracked** for every source.
5. **One-directional dependencies.** Components are cross-referenced, never merged.
6. **Agents first, humans verify.** Every capability is an API or MCP tool before it is a screen.
7. **Platform-agnostic core.** Apache 2.0. Foundry/Envision and other enclaves attach through
   adapters, never rewrites.

## Phases

| Phase | Environment | Data |
| --- | --- | --- |
| 1 Public | Unclassified cloud or local; optional Foundry developer instance | Public L0–L4 |
| 2 CUI pilot | Envision (Foundry on IL5) or another IL5 enclave | Public layers + CUI program layer and project-level scope |
| 3 Higher | Classified enclave, not yet designed | Adds classified layers; public layers mirrored in, never out |

## Current focus

Phase 1, public data only. The active spec is `knowledge-base-prototype`: prove that the public
corpus becomes an ontology-conformant, fully cited knowledge graph of the Mission Planning program
office that agents query, and write decisions back to, through MCP.

## Non-goals (Phase 1)

- CUI, internal program documents or classified material
- A production UI
- Foundry as the system of record
- Agent outputs presented as determinations. Outputs are **decision support with citations**;
  acquisition decisions carry legal authority an agent does not hold.
