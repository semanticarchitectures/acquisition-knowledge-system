# Requirements Document: Decision Traces (Context Graph)

- Owning repos: knowledge base (schema and store); Aircrew-LangChain (agent capture); ENSIM (synthetic institutional decisions); ENCAP (console capture, later)
- Status: Draft

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

1. WHEN a LangGraph decision node commits an action THEN THE node SHALL first call
   `kb_record_decision` and SHALL include the returned trace ID in its state.
2. THE trace SHALL carry `runRef` linking to the execution telemetry for that run.
3. IF `kb_record_decision` fails THEN THE node SHALL not commit the action, and SHALL surface the
   failure.

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
