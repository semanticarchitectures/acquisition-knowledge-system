# 0004. Where the KB MCP server runs

- Status: Proposed — needs a decision before KB Phase 1
- Date: 2026-10-17
- Departs from: reference architecture, "ENCAP is the only runtime" and "Corpus: data only"

## Context

The KB prototype design makes one MCP server the only interface to the KB, used by ENCAP, LangGraph
agents (Aircrew-LangChain), Claude and evaluation scripts. The reference architecture says ENCAP is
the only runtime and the corpus is data only. If the KB repo ships a server, it becomes a second
runtime. If it does not, Aircrew-LangChain and the evaluation harness cannot reach the graph without
standing up ENCAP.

## Options

| Option | For | Against |
| --- | --- | --- |
| A. ENCAP hosts the KB tools; KB repo ships data + a query library only | Keeps one runtime and the data-only boundary; one governance point | Aircrew-LangChain and evaluation need ENCAP running; slows Phase 1 |
| B. KB repo ships its own MCP server | Fastest path for Phase 1; independent testing | Second runtime with its own access control; governance split |
| C. KB repo ships the tool implementations as a library plus a dev-only server; ENCAP mounts the same library in production (recommended) | Phase 1 speed; one implementation; production stays single-runtime | Two hosting modes to keep in step |

## Recommendation

Option C. The contract is the same in both modes and is specified once in
`.kiro/specs/kb-mcp-contract`. The dev server is for the prototype and evaluation only and is not
deployed to an enclave.

## Consequences if accepted

- The reference architecture's "data only" boundary becomes "data plus a query library".
- Scoped tokens in the dev server are a prototype of ENCAP's role-based access, not a second
  identity system.
