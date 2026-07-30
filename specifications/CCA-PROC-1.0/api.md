---
id: API-002
title: Process Public API
version: "1.0"
status: Draft
derived_from:
  - SP-003
  - CCA-PROC-1.0 requirements
cpp_contract: API-002-HPP
---

# API-002 Process Public API

## 1. Purpose and authority

This document defines the normative behavior of the Process Foundation public
C++ API.

`api.hpp.md` is the sole authority for public C++ declarations. If declaration
text in another package document differs, `api.hpp.md` takes precedence. This
document defines the behavior of those declarations.

The public namespace is:

```cpp
namespace cca::process
```

The implementation MUST provide focused headers below `<cca/process/>` and the
complete umbrella header:

```cpp
#include <cca/process/process.hpp>
```

The public API depends on the CCA-REP-1.0 umbrella header:

```cpp
#include <cca/representation/representation.hpp>
```

No Runtime implementation type appears in the public Process API. Runtime
hosting is a composition responsibility described in Section 9.

## 2. Ownership and value model

`ProcessDefinition` owns a materialized execution plan. It does not own or
retain the source `RepresentationDocument`. It is copyable and movable.

`ExecutionContext` owns one definition value, one current state, and one
mutable trace. It is neither copyable nor movable.

`ExecutionResult` owns its message, trace, and diagnostics. It is copyable and
movable.

`ProcessEngine` owns no per-execution state. It is default-constructible,
copyable, and movable.

The caller owns every source `RepresentationDocument`. All document parameters
are observed through `const` references.

## 3. ExecutionState

The exact public states are:

| Enumerator | Meaning | Terminal |
|---|---|---|
| `Ready` | A context exists and has not begun execution | No |
| `Running` | Structural visitation is in progress | No |
| `Completed` | Every planned identifier was visited successfully | Yes |
| `Failed` | The execution request was rejected or failed | Yes |

Constructing an `ExecutionContext` establishes `Ready`.

A successful context execution follows:

```text
Ready -> Running -> Completed
```

No other successful transition is permitted. Process states do not represent
or modify CCA-RF-1.0 Runtime lifecycle states.

## 4. ProcessDefinition

### 4.1 Construction

```cpp
explicit ProcessDefinition(
    const cca::representation::RepresentationDocument& document);
```

Construction validates `document` using CCA-REP-1.0 behavior.

If validation fails, the constructor throws `std::invalid_argument`. The
exception message is deterministic for equivalent invalid input, but its exact
wording is not a compatibility contract.

If validation succeeds, construction copies:

- the canonical execution order;
- entity count;
- relationship count; and
- total property count.

The source document may be Mutable, internally Validated, or Frozen.
Construction does not freeze or otherwise modify it.

Construction materializes the complete plan before it succeeds. Later source
mutation, rollback, or destruction has no effect on the definition.

### 4.2 Canonical order

The canonical `executionOrder()` is:

1. each entity identifier in document insertion order;
2. immediately after an entity, each of its property identifiers in owner
   insertion order;
3. each relationship identifier in document insertion order after all
   entities and entity properties; and
4. immediately after a relationship, each of its property identifiers in
   owner insertion order.

Every identity-bearing object contributes exactly one identifier.

Relationship endpoints do not add trace entries and do not define execution
precedence. Metadata, type names, property names, values, and relationship
direction do not alter this order.

### 4.3 Counts

`entityCount()` equals the number of entities in the materialized source state.

`relationshipCount()` equals the number of relationships in the materialized
source state.

`propertyCount()` equals the combined number of properties owned by all
entities and relationships.

The following invariant always holds:

```text
executionOrder().size()
    == entityCount() + relationshipCount() + propertyCount()
```

### 4.4 Copy, move, and returned references

Copying a definition creates an equivalent independent value. Moving transfers
its value.

A reference returned by `executionOrder()` remains valid until the supplying
definition is assigned, moved from, or destroyed.

## 5. ExecutionContext

### 5.1 Construction

```cpp
explicit ExecutionContext(ProcessDefinition definition);
```

Construction takes a definition by value and owns the resulting value.
The initial state is `ExecutionState::Ready`, and `trace()` is empty.

### 5.2 Observation

`definition()` returns the context-owned definition.

`state()` returns the current Process execution state.

`trace()` returns the context-owned live trace collection.

A reference to the trace collection remains valid for the lifetime of the
context. Executing the context may invalidate trace iterators and references to
trace elements.

### 5.3 One-shot behavior

Only a Ready context may be passed to `ProcessEngine::execute`.

Executing a Ready context appends every identifier from
`definition().executionOrder()` exactly once and in order, then establishes
Completed.

Passing a Running, Completed, or Failed context to `execute(context)` throws
`std::logic_error` before mutation. The context remains unchanged.

## 6. ExecutionResult

### 6.1 Codes

`ExecutionResult::Code` contains exactly:

| Code | Required state |
|---|---|
| `Success` | `Completed` |
| `InvalidRepresentation` | `Failed` |

### 6.2 Success

For a successful result:

- `succeeded()` is true;
- `state()` is `ExecutionState::Completed`;
- `code()` is `ExecutionResult::Code::Success`;
- `message()` is empty;
- `diagnostics()` is empty; and
- `trace()` equals the executed definition's canonical execution order.

### 6.3 Invalid Representation input

When `execute(document)` receives an invalid document:

- `succeeded()` is false;
- `state()` is `ExecutionState::Failed`;
- `code()` is `ExecutionResult::Code::InvalidRepresentation`;
- `message()` is deterministic and non-empty;
- `diagnostics()` equals the complete CCA-REP-1.0 validation diagnostics,
  including their order; and
- `trace()` is empty.

No implementation-specific exception is used for this expected domain
outcome.

### 6.4 Ownership and returned views

The result owns all returned data.

The view returned by `message()` remains valid until the result is assigned,
moved from, or destroyed.

References returned by `trace()` and `diagnostics()` remain valid until the
result is assigned, moved from, or destroyed.

## 7. ProcessEngine

### 7.1 Direct document execution

```cpp
ExecutionResult execute(
    const cca::representation::RepresentationDocument& document) const;
```

This overload:

1. validates the document without modifying it;
2. returns an InvalidRepresentation result when validation fails;
3. materializes a `ProcessDefinition` when validation succeeds;
4. creates a Ready `ExecutionContext`; and
5. delegates to context execution.

### 7.2 Definition execution

```cpp
ExecutionResult execute(
    const ProcessDefinition& definition) const;
```

This overload copies the definition into a new Ready context and delegates to
context execution.

### 7.3 Context execution

```cpp
ExecutionResult execute(
    ExecutionContext& context) const;
```

This overload enforces the one-shot precondition, performs synchronous
structural visitation, changes the context from Ready through Running to
Completed, and returns an equivalent Completed result.

On success:

```text
context.trace() == result.trace()
context.state() == result.state() == Completed
```

### 7.4 Overload equivalence

For a valid document state:

```cpp
const ProcessDefinition definition{document};

engine.execute(document)
```

and:

```cpp
engine.execute(definition)
```

produce equivalent terminal results.

For a definition:

```cpp
ExecutionContext context{definition};

engine.execute(context)
```

produces a terminal result equivalent to `engine.execute(definition)`.

Object addresses and allocation layout do not participate in equivalence.

### 7.5 Synchronous operation

Every execute overload is synchronous. It returns only after producing a
terminal result or before propagating a documented standard exception.

The API defines no future, task, callback, cancellation token, retry policy,
timeout, or scheduling handle.

## 8. Public example

The minimal public example is:

```cpp
#include <cca/process/process.hpp>
#include <cca/representation/representation.hpp>

using cca::process::ExecutionResult;
using cca::process::ProcessEngine;
using cca::representation::RepresentationDocument;

RepresentationDocument model;

ProcessEngine engine;

ExecutionResult result = engine.execute(model);
```

A default empty document is valid under CCA-REP-1.0. The result therefore has:

```text
succeeded()   == true
state()       == ExecutionState::Completed
code()        == ExecutionResult::Code::Success
trace().size() == 0
```

## 9. Runtime Foundation integration

The source-level Process API is usable directly. Direct use does not create,
own, start, stop, or locate a Runtime instance.

A conforming IM-005 implementation additionally exposes `ProcessEngine` inside
each hosting CCA-RF-1.0 Runtime instance as:

- one typed Service Contract;
- `ExactlyOne` cardinality; and
- one internal Provider.

The Provider type is not part of API-002-HPP.

Runtime-hosted consumers resolve `ProcessEngine` through the owning Runtime's
typed Service Registry after successful Runtime startup. They do not construct
or locate a global engine.

Runtime composition and hosted consumers MUST invoke the resolved ProcessEngine
contract only while the owning Runtime is Running. The resolved reference is
Provider-owned and MUST NOT be retained beyond the Provider or Runtime
lifetime. Directly constructed engines have no Runtime association and execute
with identical Process semantics.

Process execution state remains per call. It does not mutate Runtime
configuration, service registration, dependency edges, Runtime Freeze, or
Runtime lifecycle.

Invalid document execution is returned as `ExecutionResult`. Provider startup
or shutdown failure remains a Runtime Foundation failure.

## 10. Determinism and input stability

The implementation MUST use only CCA-REP-1.0 public insertion order and owned
identifier values to construct the execution plan.

The implementation MUST NOT use:

- pointer ordering;
- hash-container iteration order;
- wall-clock time;
- randomness;
- filesystem or network state;
- environment variables;
- mutable process-global state; or
- thread scheduling

to determine acceptance, order, state, result, trace, or diagnostics.

The caller MUST externally synchronize source document access during
validation and materialization. Once a definition is materialized, the source
document is no longer accessed through that definition.

## 11. Failures and exception safety

Public Process operations expose only standard C++ exceptions.

| Condition | Public outcome |
|---|---|
| Invalid direct document input | Failed InvalidRepresentation result |
| Invalid `ProcessDefinition` constructor input | `std::invalid_argument` |
| Non-Ready context execution | `std::logic_error` |
| Standard-library allocation failure | May propagate |

Definition construction and all execute overloads provide the strong exception
guarantee.

If an exception propagates:

- the source document is unchanged;
- caller-owned definitions are unchanged;
- caller-owned contexts are unchanged; and
- no partial result is returned.

## 12. Thread safety

Concurrent const access to a completed `ProcessDefinition` or
`ExecutionResult` is supported.

Concurrent calls on one stateless `ProcessEngine` are supported when each call
uses:

- a distinct `ExecutionContext`; and
- document access that is valid under external CCA-REP-1.0 synchronization.

Concurrent access to one `ExecutionContext` requires external
synchronization.

Runtime-hosted use also follows the hosting Runtime's documented thread-safety
contract.

## 13. Compatibility and exclusions

Binary compatibility is not required for IM-005. API-002-HPP establishes the
source-compatibility baseline for CCA-PROC 1.x.

Exact diagnostic and exception message prose is not source-compatible.
Enumerators, method signatures, result-code meanings, state transitions,
ordering, and ownership behavior are source and behavioral contracts.

API-002 defines no scheduling, persistence, serialization, networking,
distributed execution, BPMN, workflow engine, MemoryOS, Studio, Artificial
Intelligence, plugin, action, callback, event-schema, cancellation, retry,
timeout, or code-generation interface.
