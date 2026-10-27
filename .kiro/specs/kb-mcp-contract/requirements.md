# Requirements Document: KB MCP Contract

- Owning repos: knowledge base (implements the tools as a library and dev server); ENCAP (mounts the library in production, per ADR 0004 option C)
- Consumers: ENCAP agents, Aircrew-LangChain, Claude, evaluation harness
- Related decisions: ADR 0004, 0009
- Status: Draft (revised 2026-10-26 for ADR 0009)

## Introduction

One contract defines how every consumer reads the knowledge graph and writes decision traces. It is
specified once here so that the KB's dev server and ENCAP's production hosting behave identically.

## Requirements

### Requirement 1: Tool set

**User Story:** As an agent developer, I want a small, stable set of typed tools, so that agents
query the graph without writing SPARQL.

#### Acceptance Criteria

1. THE contract SHALL define the tools `kb_entity`, `kb_cq`, `kb_path`, `kb_cite`,
   `kb_record_decision` and `kb_sparql`, each with a JSON input and output schema. Passage search is
   ENCAP's tool, not part of this contract (ADR 0009).
2. THE contract SHALL be versioned with semantic versioning; a breaking change SHALL increment the
   major version.
3. WHEN a client calls a tool with an invalid input THEN THE server SHALL return a structured error
   naming the invalid field.

### Requirement 2: Citation in every result

**User Story:** As a reviewer, I want every fact an agent receives to carry its source, so that no
uncited claim can enter an answer.

#### Acceptance Criteria

1. THE server SHALL attach to every returned fact a claim ID that `kb_cite` resolves to a source
   version, OKF anchor and supersession status.
2. IF a fact's only support is superseded THEN THE server SHALL mark it as unsupported in the result.
3. WHEN a tool finds no answer THEN THE server SHALL return a gap object, and SHALL log it to the gap
   backlog.

### Requirement 3: Scopes and access

**User Story:** As a governance officer, I want each client limited to its echelon layers and
permissions, so that the CUI boundary works the same way later.

#### Acceptance Criteria

1. THE server SHALL take each call's scope (lowest readable layer L0–L4, read or write permission,
   expert flag) from ENCAP's access model, and SHALL NOT define scopes of its own (ADR 0009).
2. WHILE a client is read-only THE server SHALL reject `kb_record_decision`.
3. THE server SHALL expose `kb_sparql` only to tokens with expert scope, read-only and time-limited.
4. THE server SHALL log every call with client, tool, inputs, scope and returned claim IDs.
5. WHERE the KB dev server runs without ENCAP THE server SHALL read scopes from a static policy file
   in ENCAP's access-control format.

### Requirement 4: Temporal queries

**User Story:** As an analyst, I want to ask as of a date, so that I can see past structure.

#### Acceptance Criteria

1. WHERE a tool accepts `as_of` THE server SHALL return only facts whose validity interval contains
   that date.
2. IF `as_of` is omitted THEN THE server SHALL use the current date and SHALL state it in the result.
