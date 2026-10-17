# Design Document: KB MCP Contract

- Requirements: [requirements.md](requirements.md)
- Hosting: ADR 0004 (proposed option C: library + dev server; ENCAP mounts the library)
- Status: Draft

## Overview

The tools are implemented once in the KB repo as a Python library (`pokb.mcp.tools`). A FastMCP dev
server exposes them for the prototype and evaluation. ENCAP mounts the same library behind its own
runtime and identity in production. ENCAP is TypeScript; it reaches the library over MCP, not by
import.

## Common envelope

Every result:

```json
{
  "contract_version": "0.1.0",
  "as_of": "2026-10-17",
  "scope": {"lowest_layer": "L4", "permission": "read"},
  "results": [ ... ],
  "gaps": [ {"question": "...", "reason": "no supporting claim", "logged_as": "gap-..."} ]
}
```

Every fact in `results` carries `claim_ids: [...]` (R2.1) and `supported: true|false` (R2.2).

## Tools

| Tool | Input | Output (in `results`) |
| --- | --- | --- |
| `kb_search` | `query`, `layers?`, `k?` | Passages: `anchor`, `text`, `source_version`, `score` |
| `kb_entity` | `name_or_id`, `hops (1–2)`, `as_of?` | Entity, types, typed links with `claim_ids` |
| `kb_cq` | `cq_id`, `params`, `as_of?` | Rows defined by the CQ template, each with `claim_ids` |
| `kb_path` | `from`, `to`, `max_hops`, `as_of?` | Paths; each edge with `claim_ids` |
| `kb_cite` | `claim_id` | `source_version`, `anchor`, `issuer`, `published`, `superseded_by?` |
| `kb_record_decision` | Decision trace (spec `decision-traces`) | `trace_id` |
| `kb_sparql` | `query` (SELECT/CONSTRUCT only), `timeout_ms` | Bindings |

Full JSON Schemas live in the KB repo at `src/pokb/mcp/schemas/` and are the authority; this table
is a summary.

## Error model

`{"error": {"code": "invalid_input|out_of_scope|timeout|not_found", "field": "...", "message": "..."}}`.
`out_of_scope` is used only for tool-level permission failures (R3.2, R3.3); facts outside scope are
omitted, not reported (knowledge-base-prototype R7.3).

## Testing strategy

- Contract tests run against both the dev server and ENCAP's mount, from the same fixture graph.
- Schema snapshot tests fail on any unversioned change.
- Scope tests per R3.
