# AGENTS.md

Instructions for any coding agent working in this repo (Kiro, Claude Code or others).

## Read first

The files in `.kiro/steering/` are the standing context for this repo. Kiro loads them
automatically; other agents should read `product.md`, `portfolio.md`, `conventions.md`, `tech.md`
and `structure.md` before changing anything, and `ontology.md` before touching ontology work.

## What this repo is

An umbrella for specs, architecture and decisions. It contains **no component code or data**
(ADR 0001). If a task asks you to write code, it belongs in the component repo the task names.

## Rules

- Follow the spec workflow in `conventions.md`: requirements, then design, then tasks, each approved
  before the next. Never edit `tasks.md` alone.
- Requirements use EARS. Tasks cite requirement IDs and the repo they land in.
- A spec may not contradict an Accepted ADR. To change a decision, write a new ADR.
- Cite sources. Flag gaps rather than invent. Track license and provenance.
- Phase 1 is public data only. Never add CUI or internal program material.
- Do not add dependencies against the arrows in `portfolio.md` without an ADR.
- Repo URLs marked TBD stay TBD until a human supplies them.
- Keep `docs/specs-index.md` current when a spec changes status.
