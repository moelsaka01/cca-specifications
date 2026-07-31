---
id: API-003
title: Persistence Public API
version: "1.0"
status: Draft
cpp_contract: API-003-HPP
---

# API-003 Persistence Public API

## 1. Purpose and authority

This document defines the behavior of the declarations in `api.hpp.md`.
`api.hpp.md` takes precedence for declarations. The public namespace is
`cca::persistence`.

## 2. Ownership model

`PersistencePackage` owns a complete copied RepresentationDocument, metadata,
ordered Workspace Policy identifiers, persisted ProcessDefinitions, format, and
copied ExecutionContextSnapshot values. It is immutable through its public API.
`PersistenceResult` owns its message and, on success, its package.
`PersistenceEngine` owns no execution or Workspace state.

## 3. PersistenceFormat

`Canonical` is the sole format value in version 1.0. It identifies the
CCA-defined package profile; it does not prescribe an external serialization
mechanism.

## 4. PersistenceMetadata

Metadata contains the Workspace name and description. Empty strings are valid.
Values are copied at construction and returned read-only.

## 5. ExecutionContextSnapshot

A snapshot contains one public Process `ExecutionState` and an owned ordered
trace of Representation identifiers. `processDefinitionIndex()` identifies the
corresponding ProcessDefinition in the containing package; the snapshot does
not duplicate that definition. It contains no Runtime state, Provider,
registry, thread, sink, or implementation detail.

## 6. PersistencePackage

The public constructor copies a complete Workspace, metadata, ordered Policy
identifiers, ProcessDefinitions, format, and context snapshots into an
independently owned package and validates the
construction preconditions. This is the construction path available to
Providers that already possess a public package value.

The package accessors return read-only views of complete owned state. Semantic
object ordering and metadata ordering are preserved exactly. A package created
by a successful `save` is valid and immutable.

## 7. PersistenceResult

`succeeded()` is true only for a complete operation. `code()` and `message()`
are deterministic and stable for equivalent outcomes. `package()` is non-null
only on success and remains owned by the result.

## 8. PersistenceEngine operations

### save

`save` observes the supplied Workspace and metadata, copies all required state,
Policies, ProcessDefinitions, and context snapshots, and returns one complete
package. It does not mutate the source document or its lifecycle. Invalid
source state produces failure with no package.

### validate

`validate` checks package completeness, format, semantic validity, ordering, and
context snapshot admissibility. It is side-effect free and never changes the
package. Success does not create Runtime or Services.

### load

`load` is the provider-independent restoration boundary: it accepts a complete
package supplied by a Provider or caller, validates it, and returns an
independently owned copy of Workspace state, Policies, ProcessDefinitions, and
context snapshots. Failure returns no package and never exposes a partial
restoration. Loading does not start Runtime or Services.

## 9. Error behavior

Operations report ordinary invalid input through a failed `PersistenceResult`.
Equivalent invalid inputs use the same code, message, and diagnostic ordering.
Allocation or other standard exceptions may propagate only where the language
contract requires; no partial package is observable.

## 10. Thread safety

Distinct engines and packages may be used concurrently. Concurrent mutation is
not applicable because packages are immutable and engines own no mutable state.
