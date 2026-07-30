# CCA-PROC-1.0 Conformance Standard

**Standard:** CCA-PROC-1.0  
**Version:** 1.0  
**Status:** Draft — ready for independent review  
**Implementation milestone:** IM-005

This document defines how an implementation demonstrates conformance to the
CCA Process Foundation Standard. It supplies assessment structure and evidence
expectations. It does not implement or extend Process architecture.

Related records:

- [SP-003 Process Foundation Standard](SP-003.md)
- [Normative requirements](requirements.yaml)
- [API-002 Process Public API](api.md)
- [API-002-HPP Process Public C++ API](api.hpp.md)
- [IS-005 Implementation Specification](implementation.md)
- [Version history](CHANGELOG.md)
- [CCA-ENG-1.0](../CCA-ENG-1.0/README.md)
- [CCA-RF-1.0](../CCA-RF-1.0/README.md)
- [CCA-REP-1.0](../CCA-REP-1.0/SP-002.md)

## 1. Conformance target

A conformance target is one identified implementation and version of the
complete CCA-PROC-1.0 boundary, including:

- the five public Process concepts;
- direct ProcessEngine behavior;
- definition materialization;
- execution state, context, result, and trace behavior;
- Runtime Foundation Service Contract integration;
- automated tests;
- public documentation and examples; and
- retained verification evidence.

Conformance is all-or-nothing. An implementation cannot claim CCA-PROC-1.0
conformance for direct execution while omitting Runtime integration, or for
Runtime integration while changing the public API.

Every requirement in `requirements.yaml` has priority `required`. A conformant
target MUST satisfy all thirty-eight requirements and every normative API-002
behavior.

## 2. Conformance levels

Conformance levels describe evidence maturity. They do not define partial
architecture profiles.

| Level | Name | Criteria | Permitted claim |
|---|---|---|---|
| CL0 | Declared | Target and version are identified; assessment is incomplete | No conformance claim |
| CL1 | Assessed | All checklists and matrix rows have evidence or explicit gaps | Assessment only |
| CL2 | Conformant | Every package, architecture, API, requirement, build, and evidence item passes | `CCA-PROC-1.0 Conformant` |
| CL3 | Independently verified | CL2 evidence is reproduced or independently reviewed | `CCA-PROC-1.0 Independently Verified` |

Only CL2 and CL3 are conformance claims. A waiver, planned implementation,
similar name, or partial test suite does not satisfy a required obligation.

## 3. Specification-package consistency gate

Before implementation assessment, reviewers verify:

- [ ] SP-003, requirements, API-002, API-002-HPP, IS-005, this conformance
  standard, and the changelog all identify CCA-PROC version 1.0.
- [ ] API-002-HPP is the sole public declaration authority.
- [ ] API-002 behavior matches API-002-HPP signatures.
- [ ] Every requirement has a unique identifier, normative source,
  verification method, expected evidence, IM-005 milestone, status, and ADR
  field.
- [ ] Every requirement appears exactly once in the checklist and compliance
  matrix.
- [ ] Relative links resolve.
- [ ] No existing CCA-ENG-1.0, CCA-RF-1.0, or CCA-REP-1.0 contract is
  redefined.
- [ ] The package is visibly Draft and makes no unreviewed publication or
  approval claim.

Failure of this gate blocks implementation-readiness approval.

## 4. Architecture checklist

### 4.1 Boundary and dependency direction

- [ ] Process is represented as an L4 Domain Engine.
- [ ] Process consumes CCA-REP-1.0 `RepresentationDocument` state.
- [ ] Process depends downward on the CCA-RF-1.0 Runtime Foundation.
- [ ] Runtime Foundation has no upward dependency on Process.
- [ ] ProcessEngine is not represented as a seventh Runtime Foundation
  component.
- [ ] The requested Compiler-to-Representation-to-Process-to-Runtime flow is
  described as processing and dependency flow, not a replacement layer model.
- [ ] Representation remains passive and owns its document state.

### 4.2 Execution profile

- [ ] Execution is synchronous structural visitation.
- [ ] Execution produces an ordered identity trace.
- [ ] Execution has no external side effects.
- [ ] No executable meaning is inferred from type names, property names,
  values, metadata, relationship type, or relationship direction.
- [ ] Relationship cycles, self-relationships, and disconnected components do
  not create scheduling behavior.
- [ ] A valid empty document is a successful empty execution.

### 4.3 Runtime integration

- [ ] Each hosting Runtime declares one typed ProcessEngine Service Contract.
- [ ] The ProcessEngine contract cardinality is ExactlyOne.
- [ ] The Provider remains internal to Runtime composition.
- [ ] The Provider participates in the validated dependency graph.
- [ ] Provider startup and shutdown follow Runtime lifecycle.
- [ ] Hosted execution occurs only while the owning Runtime is Running.
- [ ] Process execution state does not advance Runtime lifecycle.
- [ ] No global Runtime or Process singleton exists.
- [ ] Multiple Runtime instances retain independent ProcessEngine Providers and
  outcomes.

### 4.4 Excluded architecture

- [ ] No scheduling subsystem exists.
- [ ] No persistence or serialization subsystem exists.
- [ ] No networking or distributed execution exists.
- [ ] No BPMN or workflow engine exists.
- [ ] No MemoryOS, Studio, GUI, or Artificial Intelligence behavior exists.
- [ ] No plugin, user action, callback, or Process event schema exists.
- [ ] No cancellation, retry, timeout, pause, resume, or code generation
  exists.

## 5. Public API and behavior checklist

### 5.1 Public declarations

- [ ] Public declarations are in `cca::process`.
- [ ] Focused headers exist under `<cca/process/>`.
- [ ] `<cca/process/process.hpp>` exposes the complete API.
- [ ] Public declarations match API-002-HPP exactly.
- [ ] Public Process concepts are ProcessDefinition, ExecutionContext,
  ExecutionState, ExecutionResult, and ProcessEngine.
- [ ] ProcessEngine is the only public Process service.
- [ ] No public Provider type exists.
- [ ] The required direct example compiles without modification.

### 5.2 ProcessDefinition

- [ ] Construction validates through CCA-REP-1.0.
- [ ] Invalid construction throws `std::invalid_argument`.
- [ ] Valid construction copies the execution order and all three counts.
- [ ] Definition order follows entity/property then relationship/property
  insertion order.
- [ ] Counts satisfy the API invariant.
- [ ] No source document reference, view, iterator, or pointer is retained.
- [ ] Source mutation, rollback, and destruction do not change the definition.
- [ ] Metadata does not change definition behavior.

### 5.3 ExecutionContext and state

- [ ] ExecutionState contains exactly Ready, Running, Completed, and Failed.
- [ ] A new context owns its definition, is Ready, and has an empty trace.
- [ ] Successful context execution follows Ready to Running to Completed.
- [ ] A context is neither copyable nor movable.
- [ ] A non-Ready execution attempt throws `std::logic_error`.
- [ ] Rejected execution leaves context state and trace unchanged.
- [ ] Separate contexts remain isolated.

### 5.4 ExecutionResult

- [ ] Result codes are exactly Success and InvalidRepresentation.
- [ ] Success is Completed with empty message, no diagnostics, and complete
  trace.
- [ ] InvalidRepresentation is Failed with deterministic non-empty message,
  complete validation diagnostics, and empty trace.
- [ ] `succeeded()` is true exactly for Completed Success.
- [ ] Result values own message, trace, and diagnostics.
- [ ] Result values are copyable and movable.

### 5.5 ProcessEngine

- [ ] ProcessEngine is concrete, default-constructible, and stateless.
- [ ] Direct document execution validates before materialization.
- [ ] Definition execution creates a new Ready context.
- [ ] Context execution is one-shot and synchronous.
- [ ] All overloads produce equivalent terminal results for equivalent valid
  input.
- [ ] Successful trace equals definition execution order exactly.
- [ ] Direct execution never modifies or freezes the source document.
- [ ] Document validation failure returns a result rather than a Process
  exception.
- [ ] Document, definition, and context operations satisfy the strong exception
  guarantee.

### 5.6 Thread safety

- [ ] Completed definitions and results support concurrent const access.
- [ ] One engine supports concurrent calls with distinct contexts and valid
  document synchronization.
- [ ] One context requires external synchronization.
- [ ] Shared Representation document access follows CCA-REP-1.0 synchronization
  rules.
- [ ] Runtime-hosted operation follows the Runtime's documented thread-safety
  contract.

## 6. Requirement checklist

### 6.1 Architecture and scope

- [ ] CCA-PROC-001 - Process Foundation boundary
- [ ] CCA-PROC-002 - Preserve canonical Runtime dependency direction
- [ ] CCA-PROC-003 - No additional Runtime Foundation component
- [ ] CCA-PROC-004 - Passive Representation input
- [ ] CCA-PROC-005 - Deterministic Process outcomes
- [ ] CCA-PROC-006 - No mutable global state
- [ ] CCA-PROC-007 - Excluded subsystem boundary
- [ ] CCA-PROC-008 - No implicit executable taxonomy

### 6.2 Input and definition

- [ ] CCA-PROC-009 - RepresentationDocument input contract
- [ ] CCA-PROC-010 - Read-only input validation
- [ ] CCA-PROC-011 - Representation lifecycle independence
- [ ] CCA-PROC-012 - Invalid direct input result
- [ ] CCA-PROC-013 - Invalid definition construction
- [ ] CCA-PROC-014 - Owned definition plan
- [ ] CCA-PROC-015 - Definition source-lifetime independence
- [ ] CCA-PROC-016 - Canonical structural execution order
- [ ] CCA-PROC-017 - Relationships do not schedule execution
- [ ] CCA-PROC-018 - Metadata neutrality

### 6.3 State, context, result, and engine

- [ ] CCA-PROC-019 - Exact execution state set
- [ ] CCA-PROC-020 - Successful context lifecycle
- [ ] CCA-PROC-021 - One-shot ExecutionContext
- [ ] CCA-PROC-022 - ExecutionContext isolation
- [ ] CCA-PROC-023 - Successful result invariants
- [ ] CCA-PROC-024 - Invalid result invariants
- [ ] CCA-PROC-025 - Preserve validation diagnostic order
- [ ] CCA-PROC-026 - Synchronous execution
- [ ] CCA-PROC-027 - Complete successful trace
- [ ] CCA-PROC-028 - Stateless public ProcessEngine
- [ ] CCA-PROC-029 - Execute overload equivalence
- [ ] CCA-PROC-030 - Strong exception guarantee

### 6.4 Runtime, quality, and conformance

- [ ] CCA-PROC-031 - ExactlyOne Runtime Service Contract
- [ ] CCA-PROC-032 - Internal ProcessEngine Provider
- [ ] CCA-PROC-033 - Runtime lifecycle separation
- [ ] CCA-PROC-034 - Runtime dependency injection and instance isolation
- [ ] CCA-PROC-035 - Process and Runtime failure separation
- [ ] CCA-PROC-036 - Documented Process thread safety
- [ ] CCA-PROC-037 - Exact public C++ declarations
- [ ] CCA-PROC-038 - Complete automated conformance coverage

## 7. Process compliance matrix

Complete the assessment metadata and every matrix row. `Pass`, `Fail`, and
`Not assessed` are the permitted result values.

| Assessment field | Value |
|---|---|
| Implementation | |
| Implementation version | |
| Runtime integration artifact | |
| Evidence root | |
| Assessor | |
| Assessment date | |
| Claimed level | CL0 |

| Requirement | Primary verification | Result | Evidence reference |
|---|---|---|---|
| CCA-PROC-001 | Architecture review | Not assessed | |
| CCA-PROC-002 | Architecture review | Not assessed | |
| CCA-PROC-003 | Architecture review | Not assessed | |
| CCA-PROC-004 | Conformance test | Not assessed | |
| CCA-PROC-005 | Determinism test | Not assessed | |
| CCA-PROC-006 | Implementation review | Not assessed | |
| CCA-PROC-007 | Scope review | Not assessed | |
| CCA-PROC-008 | Semantic-neutrality test | Not assessed | |
| CCA-PROC-009 | Compile-time test | Not assessed | |
| CCA-PROC-010 | Validation test | Not assessed | |
| CCA-PROC-011 | Lifecycle test | Not assessed | |
| CCA-PROC-012 | Validation test | Not assessed | |
| CCA-PROC-013 | Exception test | Not assessed | |
| CCA-PROC-014 | Ownership test | Not assessed | |
| CCA-PROC-015 | Lifetime test | Not assessed | |
| CCA-PROC-016 | Ordering test | Not assessed | |
| CCA-PROC-017 | Ordering test | Not assessed | |
| CCA-PROC-018 | Semantic-neutrality test | Not assessed | |
| CCA-PROC-019 | Compile-time test | Not assessed | |
| CCA-PROC-020 | State-transition test | Not assessed | |
| CCA-PROC-021 | State-transition test | Not assessed | |
| CCA-PROC-022 | Isolation test | Not assessed | |
| CCA-PROC-023 | Result test | Not assessed | |
| CCA-PROC-024 | Result test | Not assessed | |
| CCA-PROC-025 | Diagnostic-order test | Not assessed | |
| CCA-PROC-026 | API test | Not assessed | |
| CCA-PROC-027 | Trace test | Not assessed | |
| CCA-PROC-028 | API review | Not assessed | |
| CCA-PROC-029 | Equivalence test | Not assessed | |
| CCA-PROC-030 | Exception-safety test | Not assessed | |
| CCA-PROC-031 | Runtime-integration test | Not assessed | |
| CCA-PROC-032 | API and architecture review | Not assessed | |
| CCA-PROC-033 | Runtime-lifecycle test | Not assessed | |
| CCA-PROC-034 | Runtime-isolation test | Not assessed | |
| CCA-PROC-035 | Failure-path test | Not assessed | |
| CCA-PROC-036 | Thread-safety test | Not assessed | |
| CCA-PROC-037 | API-conformance test | Not assessed | |
| CCA-PROC-038 | Traceability audit | Not assessed | |

## 8. Verification guidance

### 8.1 Evidence properties

Every evidence item MUST identify:

- implementation and version;
- requirement or API behavior;
- test, review, or command performed;
- explicit input and initial state;
- expected outcome;
- observed outcome;
- assessment date; and
- durable artifact reference.

Automated evidence MUST be reproducible. Review evidence MUST identify the
reviewer or review record. A claim without attributable evidence is Not
assessed.

### 8.2 Package and API verification

Verify YAML syntax, stable requirement IDs, relative links, version agreement,
and authority statements.

Compile-time verification MUST cover:

- every API-002-HPP signature;
- construction, copy, move, and deletion traits;
- exact enumerator names;
- const and noexcept qualifiers;
- umbrella and focused header self-containment; and
- absence of public Provider or excluded interfaces.

### 8.3 Definition and ownership verification

Definition tests MUST use documents with multiple entities, relationships, and
properties so the exact mixed order is observable.

Lifetime tests MUST execute a materialized definition after:

- unrelated source mutation;
- source transaction rollback;
- source semantic-object removal; and
- source document destruction.

Tests MUST prove no retained source reference is required.

### 8.4 State, result, and diagnostic verification

State tests MUST observe Ready before execution and Completed after success.
If an internal test seam is needed to observe the transient Running state, it
MUST remain private test infrastructure.

Result tests MUST compare all fields, not only `succeeded()`.

Invalid input tests MUST contain multiple diagnostics with identical and
different diagnostic codes. The Process result fingerprint MUST equal direct
CCA-REP-1.0 validation, including stable within-code insertion order.

### 8.5 Determinism verification

Repeat equivalent cases in fresh objects and compare:

- definition order and counts;
- state transitions;
- result state and code;
- message;
- trace; and
- diagnostics.

Vary object addresses, allocation history, metadata, semantic type names,
property values, relationship direction, cycles, and disconnected shape where
the standard says those factors do not change structural order.

### 8.6 Runtime integration verification

Runtime evidence MUST show:

- typed ExactlyOne contract declaration;
- one internal Provider per Runtime instance;
- successful Runtime validation and Freeze;
- Provider startup before hosted execution;
- hosted execution only while Running;
- no resolved ProcessEngine reference retained beyond Provider lifetime;
- Provider shutdown through Runtime lifecycle after hosted execution;
- two Runtime instances with isolated engines and outcomes;
- input rejection that leaves Runtime Running; and
- Provider lifecycle failure that follows CCA-RF-1.0 failure and rollback.

The evidence MUST show that no Runtime Foundation component was added.

### 8.7 Exception and thread-safety verification

Exception tests SHOULD use deterministic allocation-failure injection where
the implementation provides a private seam. Complete state snapshots are
required before and after a thrown operation.

Thread-safety evidence MUST cover:

- concurrent const definition reads;
- concurrent const result reads;
- concurrent calls on one engine with distinct contexts;
- two Runtime instances executing independently; and
- documented external synchronization for a shared context or source
  document.

### 8.8 Build, documentation, and exclusion verification

Retain evidence for:

- warning-free warnings-as-errors builds on supported platforms;
- formatting and static-analysis gates;
- unit and integration test results;
- public API documentation;
- direct and Runtime-hosted examples;
- requirement-to-test traceability; and
- a source/public-header search confirming all explicit exclusions.

## 9. Assessment result

Record one final result:

| Field | Value |
|---|---|
| Specification consistency gate | Not assessed |
| Architecture checklist | Not assessed |
| Public API and behavior checklist | Not assessed |
| Requirements passed | 0 / 38 |
| Build and static quality | Not assessed |
| Evidence reproducible | Not assessed |
| Unmet requirements disclosed | No |
| Final level | CL0 |
| Final claim | None |

An assessor may record CL2 only when every item is Pass. CL3 additionally
requires independent reproduction or review of the complete CL2 evidence set.
