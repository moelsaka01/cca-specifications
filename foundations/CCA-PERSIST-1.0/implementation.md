---
id: IS-006
title: Persistence Foundation Implementation Specification
version: "1.0"
status: Draft
implementation_milestone: IM-006
---

# IS-006 Persistence Foundation Implementation Specification

## 1. Purpose and authority

Implementations SHALL follow API-003-HPP, API-003, SP-004, and
`requirements.yaml` in that order. This document constrains private choices and
does not add public behavior.

## 2. Required implementation boundary

Implementation belongs in the Persistence boundary and may depend on the
public Representation and Process APIs. It MUST NOT introduce a dependency
from Runtime to Persistence or expose Runtime implementation types.

## 3. Ownership and copying

The package implementation SHALL own copied semantic state, ordered Policy
identifiers, and ProcessDefinitions. Source objects, metadata, and context
traces SHALL remain independent after save. Package and result invariants SHALL
be established before publication to callers.

## 4. Deterministic representation

Use the Representation public ordering as the canonical traversal order. Do not
sort, deduplicate, or normalize semantically meaningful order. Context snapshots
retain input order and reference a valid package ProcessDefinition index.
Equivalent inputs SHALL produce byte-independent equivalent
public values and diagnostics; no external encoding is prescribed.

## 5. Validation and atomicity

Validation SHALL be read-only and deterministic. Save and load SHALL construct
temporary complete values and publish them only after all checks succeed.
Failure SHALL not expose a partially constructed package.

## 6. Runtime exclusion

No Runtime lifecycle, registry, Provider, thread, logger, metric, health, or
other implementation state may appear in a package or public declaration.
Loading SHALL be a provider-independent data operation and SHALL NOT start
Services. It SHALL restore Policies, ProcessDefinitions, and snapshots as
independent package-owned state.

## 7. Provider boundary

External preservation infrastructure, if supplied, belongs behind Providers.
Providers may consume the public package contract but may not add Persistence
concepts, alter package semantics, or define architecture.

## 8. Error and exception safety

Invalid values return the specified failed result. Copy and move operations
preserve value semantics. Standard allocation exceptions may propagate, but a
failed operation MUST leave all caller-owned inputs unchanged and MUST NOT
publish partial output.

## 9. Verification obligations

The implementation test suite SHALL cover every CCA-PERSIST requirement,
including source immutability, complete semantic ordering, package independence,
validation purity, Runtime exclusion, context fidelity, deterministic failures,
and concurrent use of independent engines and packages.
