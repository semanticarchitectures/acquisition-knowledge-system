---
inclusion: always
---

# Technology standards

## Shared standards (all components)

| Concern | Standard |
| --- | --- |
| Agent and tool interface | Model Context Protocol (MCP). Every capability is an MCP tool or API first. |
| Document representation | OKF Markdown (Google Cloud's Open Knowledge Format), one bundle per echelon layer |
| Upper ontology | BFO + CCO, imported unchanged and pinned |
| Provenance | PROV-O |
| Validation | SHACL |
| Licensing | Apache 2.0 with NOTICE; third-party source license recorded per source |
| Models | Reached through a model port (thin interface), so the IL5 enclave model is a configuration change |

## Platform ports (adapters, not rewrites)

| Port | Phase 1 | Phase 2 (IL5) |
| --- | --- | --- |
| Storage | SQLite, Oxigraph files | Foundry datasets (projection) |
| Model endpoint | Commercial API | Enclave-hosted model |
| Identity | Local roles and scoped tokens | Enclave identity |
| Tools | MCP | Foundry functions or AIP |

## Framework boundary

- **LangGraph** is used for agent orchestration and decision-trace capture, in consumer repos only.
- **Knowledge-graph construction does not depend on LangChain.** Extraction, entity resolution and
  validation are plain code driven by the ontology and SHACL shapes.
- The graph model, ontology and store are chosen independently of any LLM framework.

## Per-component stacks

See `portfolio.md`. Do not assume one component's stack for another.

## Testing expectations

- Competency questions are golden-answer tests run in CI.
- Extraction has a versioned gold set and a regression suite; models and prompts are pinned.
- Every agent answer path has a citation check: the cited passage must support the claim.
