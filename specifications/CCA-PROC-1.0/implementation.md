---
id: IS-005
title: Process Foundation Implementation Specification
version: "1.0"
status: Draft
implementation_milestone: IM-005
---

# IS-005 Process Foundation Implementation Specification

## 1. Purpose and authority

IM-005 implements the Process Foundation exactly as defined by:

1. API-002-HPP;
2. API-002;
3. SP-003;
4. `requirements.yaml`; and
5. this implementation specification.

Authority descends in that order. `conformance.md` defines evidence and
assessment but does not change implementation behavior.

No implementation convenience may introduce architecture, public behavior, or
public declarations absent from the higher authorities.

## 2. Implementation target

The implementation target is:

```text
repositories/cca-core/
```

This specification package creates no C++ implementation. IM-005 is a separate
future implementation change.

Process code belongs in the logical Layer 4 Process boundary even when it
shares a physical repository with other CCA foundations. Physical placement
MUST NOT create a Runtime-to-Process dependency.

## 3. Required public modules

Implement exactly these public Process concepts:

- `ProcessDefinition`;
- `ExecutionContext`;
- `ExecutionState`;
- `ExecutionResult`; and
- `ProcessEngine`.

`ProcessEngine` is the only public Process service.

The public namespace is:

```cpp
namespace cca::process
```

Public declarations MAY be divided into focused headers under:

```text
include/cca/process/
```

The complete umbrella header is:

```text
include/cca/process/process.hpp
```

Every public declaration MUST match API-002-HPP.

## 4. Dependency boundaries

The Process implementation MAY depend on:

- the CCA-REP-1.0 public API;
- the CCA-RF-1.0 Runtime Foundation public composition API; and
- lower-layer shared CCA engineering utilities that do not change Process
  semantics.

The implementation MUST NOT:

- modify CCA-REP-1.0 declarations or behavior;
- place Process behavior in Representation classes;
- make Runtime Foundation code depend on Process;
- expose an internal Runtime Provider through Process public headers;
- use compiler-internal types as Process inputs;
- use mutable global state; or
- introduce a dependency on an excluded subsystem.

## 5. Definition materialization

The public `ProcessDefinition` constructor MUST:

1. validate the source using `cca::representation::ValidationService`;
2. throw `std::invalid_argument` if validation fails;
3. count entities, relationships, and all properties;
4. copy identifiers in the API-002 canonical order; and
5. retain no source document reference or view.

The internal representation of the copied plan is implementation-defined.

Materialization MUST use public CCA-REP-1.0 collection order. It MUST NOT use:

- pointer ordering;
- unordered-container iteration order;
- sorted identifier order;
- relationship graph traversal;
- metadata;
- type-name dispatch; or
- property-name dispatch

to determine the canonical sequence.

The empty valid document MUST produce zero counts and an empty sequence.

## 6. Execution implementation

`ExecutionContext` MUST begin Ready with an empty trace.

For a Ready context, the implementation MUST:

1. prepare the complete trace or the capacity required to produce it without
   violating the strong exception guarantee;
2. establish Running;
3. visit every planned identifier exactly once in stored order;
4. establish Completed; and
5. return a Success result owning an equivalent trace.

The implementation MUST NOT invoke user code, perform I/O, publish a required
event, or infer control flow during visitation.

Non-Ready context rejection MUST occur before any state or trace mutation.

Direct document execution MUST preserve Representation validation diagnostics
exactly. It MAY use a private validated-materialization helper to avoid
performing validation twice. That helper is not public API.

## 7. Result implementation

`ExecutionResult` invariants MUST be enforced at construction.

No public constructor is provided. Private factories, constructors, or helper
functions MAY be used.

The implementation MUST NOT produce:

- Success with a non-Completed state;
- InvalidRepresentation with a non-Failed state;
- Success with diagnostics;
- InvalidRepresentation with a non-empty trace; or
- a public result code not declared by API-002-HPP.

Messages MUST be deterministic for equivalent input. Exact message prose is
implementation-defined.

## 8. Runtime integration

The implementation MUST supply one private Runtime Provider that exposes
`cca::process::ProcessEngine` as a typed Service Contract.

For each hosting Runtime instance:

- the contract cardinality is `ExactlyOne`;
- exactly one Provider binding is registered before Runtime Freeze;
- the Provider participates in the Runtime dependency graph;
- provider construction uses the Runtime Foundation composition mechanism;
- provider start and stop use the Runtime lifecycle;
- public resolution occurs through the typed Service Registry; and
- the Provider and its outcomes remain instance-local.

The concrete contract wrapper and Provider type are implementation-defined and
MUST remain outside `include/cca/process/`.

If a concrete CCA-RF-1.0 implementation requires an internal Provider to derive
from the public contract surface, that inheritance is an implementation
adapter. It does not make the Provider public and does not add ProcessEngine
state.

Runtime composition and hosted consumers MUST invoke the resolved contract
only while the Runtime is Running. A resolved ProcessEngine reference follows
the lifetime of its internal Provider and MUST NOT be retained after Provider
or Runtime destruction. Direct ProcessEngine behavior has no Runtime
association.

Process input rejection returns `ExecutionResult` and MUST NOT fail the Runtime
instance. Provider lifecycle failure returns the Runtime implementation's
standard lifecycle failure result and MUST NOT be converted to
`ExecutionResult`.

## 9. Internal freedom

Subject to the public contract, the implementation may choose:

- PImpl layout;
- containers;
- allocation strategy;
- plan storage;
- private factories;
- validation/materialization helpers;
- trace construction algorithm;
- internal Runtime contract wrapper; and
- private Provider composition.

These choices MUST NOT change:

- ownership;
- source lifetime independence;
- counts;
- canonical order;
- state transitions;
- result invariants;
- diagnostics;
- exception behavior;
- Runtime cardinality; or
- thread-safety obligations.

## 10. Required automated tests

Every CCA-PROC-001 through CCA-PROC-038 requirement and every normative
API-002 behavior MUST map to at least one automated test.

Required test suites:

- Architecture boundary;
- Public API declarations;
- ProcessDefinition validation;
- ProcessDefinition ownership and lifetime;
- Definition counts and ordering;
- ExecutionState;
- ExecutionContext lifecycle and isolation;
- ExecutionResult invariants;
- ProcessEngine direct execution;
- ProcessEngine overload equivalence;
- Diagnostics and deterministic ordering;
- Exception safety;
- Thread safety;
- Runtime Service Contract integration;
- Runtime instance isolation; and
- Exclusion and public-surface audit.

Tests MUST include:

- the exact empty-document public example;
- valid mutable, internally Validated, and Frozen input;
- invalid input with multiple ordered diagnostics;
- multiple entities and relationships with multiple properties;
- self-relationships, cycles, and disconnected graph shapes;
- definitions executed after source mutation, rollback, and destruction;
- context re-execution rejection;
- repeated equivalent runs;
- concurrent supported operations;
- two independent Runtime instances;
- ExactlyOne cardinality validation;
- Provider start before hosted calls during Running;
- Provider stop after the final hosted call;
- Process input failure that leaves Runtime Running; and
- Provider lifecycle failure that follows the Runtime failure path.

Requirement evidence MUST identify the test or review artifact that satisfies
each item. Test names alone are not evidence unless the asserted behavior is
reviewable.

## 11. Build and static quality

The implementation MUST:

- use the repository's approved C++ language level;
- compile on every supported platform;
- compile warning-free;
- treat warnings as errors;
- pass applicable formatting and static-analysis gates; and
- preserve valid public header self-containment.

No warning suppression may conceal a Process conformance defect.

## 12. Documentation and examples

The implementation MUST provide:

- generated or maintained API documentation;
- an architecture summary distinguishing Process from Runtime lifecycle;
- a programming guide for definitions, contexts, states, and results;
- Runtime Service Contract registration and resolution guidance;
- thread-safety documentation;
- requirement-to-test conformance evidence; and
- examples for direct and Runtime-hosted execution.

The direct example MUST include the exact API form:

```cpp
RepresentationDocument model;
ProcessEngine engine;
ExecutionResult result = engine.execute(model);
```

Examples MUST NOT imply scheduling, workflow, persistence, networking, or
action execution.

## 13. Deliverables

IM-005 deliverables are:

- public headers;
- source files;
- private Runtime integration;
- unit and integration tests;
- API documentation;
- architecture documentation;
- direct and Runtime-hosted examples; and
- conformance evidence.

Implementation work MUST NOT modify the CCA-PROC-1.0 package to make a
non-conforming implementation appear conforming. Specification changes follow
CCA-ENG-1.0 governance.

## 14. Stop conditions

Implementation MUST stop when:

1. two authoritative package documents directly contradict one another;
2. API-002 behavior cannot be implemented without changing API-002-HPP;
3. CCA-REP-1.0 public behavior is insufficient for a required operation; or
4. CCA-RF-1.0 integration would require a seventh Runtime component or an
   upward Runtime dependency.

An implementation detail such as container selection, allocation, helper
classes, or private algorithms is not a stop condition.

When stopped, report:

- affected document and section;
- exact contradiction or unavailable contract;
- impacted requirements; and
- the smallest proposed specification correction.

Do not invent public architecture in implementation.

## 15. Explicit exclusions

IM-005 MUST NOT implement:

- scheduling;
- persistence or serialization;
- networking;
- distributed execution;
- BPMN;
- workflow-engine behavior;
- MemoryOS;
- Studio or GUI behavior;
- Artificial Intelligence;
- plugins;
- user-defined actions or callbacks;
- Process Event Bus schemas;
- cancellation, retry, timeout, pause, or resume; or
- code generation.

No Git commit or tag is part of this implementation specification.
