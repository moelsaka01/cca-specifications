# CCA-RF-1.0 Conformance Standard

**Standard:** CCA-RF-1.0  
**Version:** 1.0  
**Status:** Published  
**Implementation milestone:** IM-003

This document defines how an implementation demonstrates conformance to the
[CCA Runtime Foundation Standard](README.md). It supplies an assessment
structure and does not implement or extend the Runtime architecture.

Related records:

- [Normative requirements](requirements.yaml)
- [Approved architecture decisions](decisions.md)
- [CCA Engineering Handbook](../CCA-ENG-1.0/README.md)
- [Specification governance](../../governance/README.md)

## 1. Conformance target

A conformance target is one identified implementation and version of the
headless `cca-runtime` host, including its Runtime instance model, six Runtime
Foundation components, programming-model documentation, and retained
verification evidence.

Conformance applies to the complete CCA-RF-1.0 boundary. An implementation
cannot claim CCA-RF-1.0 conformance for selected components or selected
lifecycle states while omitting another required obligation.

Every requirement in [requirements.yaml](requirements.yaml) has priority
`required`. A conformant target MUST satisfy all forty requirements.

## 2. Conformance levels

Conformance levels describe evidence maturity. They do not create partial
architecture profiles.

| Level | Name | Criteria | Permitted claim |
|---|---|---|---|
| CL0 | Declared | Target and version are identified; assessment is not complete | No conformance claim |
| CL1 | Assessed | Architecture and requirement checklists are complete; gaps and evidence are recorded | Assessment only; not conformant while any requirement is unmet |
| CL2 | Conformant | Every architecture item and CCA-RF-001 through CCA-RF-040 pass with attributable evidence | `CCA-RF-1.0 Conformant` |
| CL3 | Independently verified | CL2 is satisfied and the evidence set is reproduced or reviewed by an independent assessor | `CCA-RF-1.0 Independently Verified` |

Only CL2 and CL3 are conformance claims. A waiver, planned fix, equivalent
name, or partially implemented lifecycle does not satisfy a required
obligation.

## 3. Architecture checklist

### 3.1 Host and instance model

- [ ] The official host artifact is named `cca-runtime`.
- [ ] The host operates headlessly.
- [ ] One host can create and operate multiple Runtime instances.
- [ ] Each Runtime owns an independent execution context.
- [ ] No global Runtime singleton exists.
- [ ] Runtime behavior and dependency access use no mutable global Runtime
  state.
- [ ] Lifecycle, configuration, events, services, failures, and cleanup are
  attributable to one Runtime instance.

### 3.2 Layers and Runtime Foundation composition

- [ ] The architecture publishes L5 Applications.
- [ ] The architecture publishes L4 Domain Engines containing Representation,
  Process, and Persistence.
- [ ] The architecture publishes L3 Runtime Foundation.
- [ ] The architecture publishes L2 Platform Abstraction.
- [ ] The architecture publishes L1 Operating System.
- [ ] Every inter-layer dependency points from a higher-numbered layer to a
  lower-numbered layer.
- [ ] The Runtime Foundation contains exactly Lifecycle Manager, Service
  Registry, Dependency Injector, Event Bus, Configuration Manager, and
  Observability.

### 3.3 Lifecycle and freeze

- [ ] Successful execution follows `Constructed -> Initializing -> Configuring
  -> Registering Services -> Resolving Dependencies -> Validating -> Runtime
  Freeze -> Starting -> Running -> Stopping -> Stopped -> Destroyed`.
- [ ] Runtime Freeze completes after validation and before service startup.
- [ ] Runtime topology and composition are immutable after Runtime Freeze.
- [ ] Invalid lifecycle transitions are rejected and observable.
- [ ] Failure follows `Failed -> Rollback -> Destroyed`.

### 3.4 Services and communication

- [ ] Required collaboration uses Dependency Injector.
- [ ] Asynchronous notification uses Event Bus.
- [ ] Service Registry resolution is compile-time type-safe.
- [ ] No string-based service lookup is exposed.
- [ ] Consumers resolve Service Contracts rather than internal Provider types.
- [ ] Providers remain internal to the Runtime Foundation.
- [ ] Every Service Contract declares `ExactlyOne`, `ZeroOrOne`, or
  `OneOrMore`.
- [ ] Provider counts are validated against the declared cardinality before
  Runtime Freeze.

### 3.5 Startup, shutdown, and failure

- [ ] The complete service dependency graph is computed before startup.
- [ ] The graph is validated and converted into dependency levels before
  Runtime Freeze.
- [ ] Startup levels execute sequentially.
- [ ] Services within one level may start concurrently.
- [ ] The next level does not begin until the current level completes.
- [ ] Shutdown stops dependents before their providers.
- [ ] Failure enters the explicit Failed state.
- [ ] Rollback accounts for work completed before failure.
- [ ] Cleanup is deterministic and terminates in Destroyed.

### 3.6 Programming model and observability

- [ ] Published programming-model documentation covers Runtime API philosophy,
  lifecycle, service registration, dependency declaration, error reporting,
  configuration, observability, and thread safety.
- [ ] Thread-safety documentation covers multiple instances, permitted
  same-level startup concurrency, and asynchronous Event Bus notification.
- [ ] Observability identifies instance-scoped lifecycle, validation, startup,
  shutdown, notification, failure, rollback, and cleanup outcomes.

## 4. Requirement checklist

### 4.1 Foundation and governance

- [ ] CCA-RF-001 - Single Runtime Foundation boundary
- [ ] CCA-RF-002 - Architecture-first contracts
- [ ] CCA-RF-003 - Deterministic boundary outcomes
- [ ] CCA-RF-004 - Dependency Injection for required collaboration
- [ ] CCA-RF-005 - No mutable global state
- [ ] CCA-RF-006 - Explicit lifecycle contract
- [ ] CCA-RF-007 - Service Contract resolution
- [ ] CCA-RF-008 - Explicit Event Bus boundary
- [ ] CCA-RF-009 - Documented configuration model
- [ ] CCA-RF-010 - Documented observability model
- [ ] CCA-RF-011 - Runtime API philosophy
- [ ] CCA-RF-012 - Conformance evidence

### 4.2 Host, isolation, and composition

- [ ] CCA-RF-013 - Headless Runtime
- [ ] CCA-RF-014 - Official Runtime host
- [ ] CCA-RF-015 - Multiple Runtime instances
- [ ] CCA-RF-016 - Independent execution contexts
- [ ] CCA-RF-017 - No global Runtime singleton
- [ ] CCA-RF-018 - Exact Runtime Foundation components
- [ ] CCA-RF-019 - Canonical CCA layer model
- [ ] CCA-RF-020 - Inter-layer dependency direction

### 4.3 Lifecycle and services

- [ ] CCA-RF-021 - Normal Runtime lifecycle sequence
- [ ] CCA-RF-022 - Runtime immutability after Freeze
- [ ] CCA-RF-023 - Freeze before service startup
- [ ] CCA-RF-024 - Compile-time type-safe service resolution
- [ ] CCA-RF-025 - No string-based service lookup
- [ ] CCA-RF-026 - Internal Providers
- [ ] CCA-RF-027 - Contract provider cardinality
- [ ] CCA-RF-028 - Supported provider cardinalities
- [ ] CCA-RF-029 - Event Bus asynchronous notification

### 4.4 Ordering, failure, and programming model

- [ ] CCA-RF-030 - Startup dependency graph
- [ ] CCA-RF-031 - Sequential startup dependency levels
- [ ] CCA-RF-032 - Optional same-level startup concurrency
- [ ] CCA-RF-033 - Reverse dependency shutdown
- [ ] CCA-RF-034 - Explicit Failed state
- [ ] CCA-RF-035 - Failure Rollback
- [ ] CCA-RF-036 - Deterministic failure cleanup
- [ ] CCA-RF-037 - Service registration documentation
- [ ] CCA-RF-038 - Dependency declaration documentation
- [ ] CCA-RF-039 - Error reporting documentation
- [ ] CCA-RF-040 - Thread-safety documentation

## 5. Runtime compliance matrix

Complete the assessment metadata and every matrix row. `Pass`, `Fail`, and
`Not assessed` are the permitted result values.

| Assessment field | Value |
|---|---|
| Implementation | |
| Implementation version | |
| `cca-runtime` artifact reference | |
| Evidence root | |
| Assessor | |
| Assessment date | |
| Claimed level | CL0 |

| Requirement | Primary verification | Result | Evidence reference |
|---|---|---|---|
| CCA-RF-001 | Architecture review | Not assessed | |
| CCA-RF-002 | Documentation review | Not assessed | |
| CCA-RF-003 | Conformance test | Not assessed | |
| CCA-RF-004 | Architecture review | Not assessed | |
| CCA-RF-005 | Implementation review | Not assessed | |
| CCA-RF-006 | Conformance review | Not assessed | |
| CCA-RF-007 | Conformance test | Not assessed | |
| CCA-RF-008 | Architecture review | Not assessed | |
| CCA-RF-009 | Documentation review | Not assessed | |
| CCA-RF-010 | Documentation review | Not assessed | |
| CCA-RF-011 | Documentation review | Not assessed | |
| CCA-RF-012 | Conformance audit | Not assessed | |
| CCA-RF-013 | Conformance test | Not assessed | |
| CCA-RF-014 | Artifact inspection | Not assessed | |
| CCA-RF-015 | Conformance test | Not assessed | |
| CCA-RF-016 | Isolation test | Not assessed | |
| CCA-RF-017 | Implementation review | Not assessed | |
| CCA-RF-018 | Architecture review | Not assessed | |
| CCA-RF-019 | Architecture review | Not assessed | |
| CCA-RF-020 | Dependency review | Not assessed | |
| CCA-RF-021 | Lifecycle test | Not assessed | |
| CCA-RF-022 | Lifecycle test | Not assessed | |
| CCA-RF-023 | Lifecycle test | Not assessed | |
| CCA-RF-024 | Compile-time test | Not assessed | |
| CCA-RF-025 | Interface review | Not assessed | |
| CCA-RF-026 | Architecture review | Not assessed | |
| CCA-RF-027 | Contract review | Not assessed | |
| CCA-RF-028 | Contract test | Not assessed | |
| CCA-RF-029 | Architecture review | Not assessed | |
| CCA-RF-030 | Startup test | Not assessed | |
| CCA-RF-031 | Startup test | Not assessed | |
| CCA-RF-032 | Architecture review | Not assessed | |
| CCA-RF-033 | Shutdown test | Not assessed | |
| CCA-RF-034 | Failure test | Not assessed | |
| CCA-RF-035 | Failure test | Not assessed | |
| CCA-RF-036 | Failure test | Not assessed | |
| CCA-RF-037 | Documentation review | Not assessed | |
| CCA-RF-038 | Documentation review | Not assessed | |
| CCA-RF-039 | Documentation review | Not assessed | |
| CCA-RF-040 | Documentation review | Not assessed | |

## 6. Verification guidance

### 6.1 Evidence properties

Evidence MUST be attributable to the assessed implementation and version. It
MUST identify its input conditions, expected result, observed result, and
verification date. Automated evidence MUST be reproducible from versioned
inputs. Review evidence MUST identify the reviewed artifact and reviewer.

Evidence from one Runtime instance MUST NOT be used to conceal behavior in
another instance. Concurrency evidence MUST retain instance and dependency
level identity.

### 6.2 Architecture and documentation review

Review the published component inventory, layer model, dependency direction,
Runtime API philosophy, lifecycle contract, service registration model,
dependency declarations, error reporting, configuration, observability, and
thread-safety guarantees. Compare the result directly with
[decisions.md](decisions.md) and the normative diagrams.

### 6.3 Compile-time and interface verification

Demonstrate that Service Contract type mismatches fail at compile time.
Inspect the Service Registry interface to show that service identity and
resolution accept no string key. Show that consumers receive contracts while
Provider types remain internal.

### 6.4 Lifecycle and isolation verification

Exercise the complete successful lifecycle and record every state in order.
Attempt structural mutation after Runtime Freeze and verify deterministic
rejection. Operate at least two Runtime instances and demonstrate independent
configuration, registry, Event Bus, lifecycle, failure, and cleanup outcomes.

### 6.5 Startup and shutdown verification

Capture the resolved dependency graph and its levels. Verify that each level
completes before the next begins. If same-level concurrency is used, verify
that it does not cross a dependency edge. During shutdown, verify every edge
in reverse: a dependent stops before its provider.

### 6.6 Failure and rollback verification

Induce failures at representative lifecycle and startup points. Verify the
transition to Failed, then Rollback, then Destroyed. Repeat equivalent cases
and compare cleanup responsibilities and observable outcomes for equivalence.

## 7. Assessment result

The assessor MUST record:

- the completed architecture checklist;
- the completed requirement checklist and compliance matrix;
- every failed or unassessed item;
- the evidence references;
- the resulting CL0, CL1, CL2, or CL3 level; and
- for CL3, the independent assessor and reproduced evidence.

An assessment record is append-only evidence for its identified implementation
version. A later assessment supersedes it by reference; it does not rewrite the
earlier result.

