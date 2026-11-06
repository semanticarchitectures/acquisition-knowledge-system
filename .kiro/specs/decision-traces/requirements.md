# Requirements Document: Decision Traces (Context Graph)

- Owning repos: knowledge base (schema and store); Aircrew-LangChain (agent capture); ENSIM (synthetic institutional decisions); ENCAP (console capture, later)
- Status: Requirements approved (2026-11-06)
- Revision of 2026-11-06: R1.4 added, R3 rewritten and R7 added so that traces are written to files
  until the KB trace store exists, as the integrated plan decides and Kevin confirmed. `design.md`
  is revised to match (awaiting approval); `tasks.md` follows once the design is approved.

## Introduction

A decision trace is a first-class record of how context turned into action: who decided, in what
role, under which authority and version, using which facts, with what exceptions and precedents, and
with what outcome. Traces accumulate into the context graph. They are distinct from execution
telemetry (for example LangSmith traces), which they link to but do not replace.

## Requirements

### Requirement 1: Trace schema

**User Story:** As an auditor, I want every recorded decision to answer who, under what authority,
using what, and why, so that precedent is searchable and decisions can be replayed.

#### Acceptance Criteria

1. THE schema SHALL define a trace as a PROV-O activity with: `decisionPoint`, `decidedBy`, `inRole`,
   `underAuthority` (issuance and version), `used` (claim and entity IDs), `optionsConsidered`,
   `exception`, `outcome`, `precedent`, `atTime`, `runRef` and `origin`.
2. THE schema SHALL be expressed as a SHACL shape in the KB ontology.
3. IF a required field is unknown THEN THE recorder SHALL leave it empty and mark it unknown, and
   SHALL NOT fill it with an inferred value.
4. THE schema SHALL also be published as a JSON Schema with the same fields, so that a trace written
   before the KB exists can be validated where it is written.

### Requirement 2: Reviewer decisions

**User Story:** As a project lead, I want reviewer accept and reject choices captured, so that the
context graph holds real decisions from Phase 2.

#### Acceptance Criteria

1. WHEN a reviewer accepts or rejects a review item THEN THE KB SHALL record a trace with origin
   `reviewer`.

### Requirement 3: Agent decisions

**User Story:** As an agent developer, I want decision nodes to record traces before acting, so that
agent decisions are auditable at the moment they are made.

#### Acceptance Criteria

1. WHEN a LangGraph decision node commits an action THEN THE node SHALL first record the trace
   through the trace-recording interface and SHALL include the returned trace ID in its state.
2. THE trace SHALL carry `runRef` linking to the execution record for that run: the run and
   checkpoint IDs, and the LangSmith run ID when tracing is on.
3. IF recording the trace fails THEN THE node SHALL not commit the action, and SHALL surface the
   failure.
4. WHILE the KB trace store does not exist THE trace-recording interface SHALL write each trace to a
   file with the run's records, one trace per line, in the format of R1.4.
5. WHEN `kb_record_decision` is available THEN THE trace-recording interface SHALL call it instead,
   and no decision node SHALL change.
6. IF a field's value would be a KB identifier that does not exist yet (for example `inRole` or
   `underAuthority.issuance`) THEN THE recorder SHALL write the local identifier it has (for example
   the ENSIM role ID), SHALL list the field as pending resolution, and SHALL NOT invent a KB
   identifier.

### Requirement 4: Synthetic institutional decisions

**User Story:** As an evaluator, I want acquisition decisions generated against real authorities, so
that the schema is tested before real CUI decisions exist.

#### Acceptance Criteria

1. WHERE ENSIM's acquisition mission set exists THE generator SHALL produce milestone and funding
   decisions that cite authorities present in the KB graph.
2. THE KB SHALL store synthetic traces with origin `synthetic` and SHALL never return them as real
   precedent unless the caller asks for synthetic data.

### Requirement 5: Declared decisions

**User Story:** As an analyst, I want decisions reported in public sources captured honestly, so that
missing reasoning is visible rather than invented.

#### Acceptance Criteria

1. WHEN the extractor finds a decision reported in a public source THEN THE KB SHALL record a trace
   with origin `declared`, with outcome and authority filled and reasoning fields marked unknown.

### Requirement 6: Precedent query

**User Story:** As an agent, I want to find similar past decisions, so that I can cite precedent.

#### Acceptance Criteria

1. THE KB SHALL support CQ10 (inputs, policy version and precedents for a given decision).
2. THE KB SHALL return precedents filtered by origin, with the origin shown in every result.

### Requirement 7: Importing traces written to files

**User Story:** As a project lead, I want traces recorded before the KB existed to join the context
graph, so that decisions from D1 and D2 are on the record at D3.

#### Acceptance Criteria

1. WHEN the KB trace store exists THEN THE KB SHALL import traces written to files, resolve each
   pending local identifier to a KB identifier, and validate each trace against the shape of R1.2.
2. IF an imported trace fails validation, or a pending identifier cannot be resolved, THEN THE KB
   SHALL reject that trace with the failing field and SHALL leave the file unchanged.
