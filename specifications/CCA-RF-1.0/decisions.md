# CCA-RF-1.0 Architecture Decisions

This document summarizes the eight approved architecture decisions that define
CCA Runtime Foundation Standard 1.0. It is an explanatory index, not a
replacement for the normative [standard](README.md),
[requirements](requirements.yaml), or [conformance criteria](conformance.md).

## ADR-003-001 — Runtime host and instance isolation

**Decision:** The runtime host is `cca-runtime`. It is headless and may host
multiple Runtime Foundation instances. Each instance is isolated, owns its
state and dependencies, and is composed explicitly. A process-wide Runtime
Foundation singleton is prohibited.

**Rationale:** A headless host separates runtime operation from presentation.
Independent instances make ownership, testing, embedding, and failure
containment explicit without hidden process-global coupling.

**Consequences:**

- callers select and retain the instance with which they interact;
- dependencies and mutable state cannot be shared implicitly through a
  singleton;
- cleanup, diagnostics, and lifecycle outcomes are attributable to one
  instance; and
- user interfaces, applications, and domain engines remain outside the host's
  Runtime Foundation responsibility.

**Reference:** ADR-003-001; [layer model](diagrams/layer-model.mmd);
[requirements](requirements.yaml); [conformance criteria](conformance.md).

## ADR-003-002 — Layer model and Runtime Foundation composition

**Decision:** The architecture has five dependency layers:

1. L5 Applications;
2. L4 Domain Engines, containing Representation, Process, and Persistence;
3. L3 Runtime Foundation;
4. L2 Platform Abstraction; and
5. L1 Operating System.

Inter-layer dependencies point from a higher-numbered layer to a
lower-numbered layer. The Runtime Foundation contains exactly six components:
Lifecycle Manager, Service Registry, Dependency Injector, Event Bus,
Configuration Manager, and Observability.

**Rationale:** A fixed layer vocabulary and dependency direction prevent
implementation structure from silently redefining architecture. The exact
six-component foundation assigns each required responsibility to a reviewable
boundary.

**Consequences:**

- Applications depend downward through Domain Engines;
- Domain Engines depend on the Runtime Foundation;
- the Runtime Foundation depends on Platform Abstraction;
- Platform Abstraction depends on the Operating System;
- diagrams and implementations must not add a seventh Runtime Foundation
  component; and
- this decision does not prohibit relationships within one layer.

**Reference:** ADR-003-002; [layer model](diagrams/layer-model.mmd);
[requirements](requirements.yaml); [conformance criteria](conformance.md).

## ADR-003-003 — Lifecycle, freeze point, and immutability

**Decision:** Each Runtime Foundation instance follows this normal lifecycle:

```text
Constructed -> Initializing -> Configuring -> Registering Services
-> Resolving Dependencies -> Validating -> Runtime Freeze -> Starting
-> Running -> Stopping -> Stopped -> Destroyed
```

At Runtime Freeze, the validated runtime structure becomes immutable for the
remainder of that run. Registrations and the resolved dependency structure
cannot be mutated after the freeze point.

**Rationale:** An explicit lifecycle makes acceptance, rejection, startup, and
cleanup observable. Freezing only after configuration, registration,
resolution, and validation ensures startup operates on a stable graph.

**Consequences:**

- lifecycle transitions are owned by the Lifecycle Manager;
- mutation that would change the validated runtime structure must occur before
  Runtime Freeze;
- startup cannot repair or extend the graph implicitly; and
- a new run or instance is required to apply structural changes after freeze.

**Reference:** ADR-003-003;
[runtime lifecycle](diagrams/runtime-lifecycle.mmd);
[requirements](requirements.yaml); [conformance criteria](conformance.md).

## ADR-003-004 — Typed service registry contracts

**Decision:** The Service Registry stores internal providers against
compile-time typed contracts. Contract lookup is type-safe and must not use
string identifiers. A contract declares one of three cardinalities:
`ExactlyOne`, `ZeroOrOne`, or `OneOrMore`.

**Rationale:** Typed contracts prevent spelling-dependent lookup and make
provider requirements checkable before startup. Explicit cardinality defines
whether absence or multiplicity is valid.

**Consequences:**

- providers remain internal to the Runtime Foundation composition;
- consumers resolve contracts by type rather than by string;
- registration and validation enforce the declared cardinality; and
- cardinality violations produce deterministic validation failure before
  Runtime Freeze.

**Reference:** ADR-003-004;
[service registry](diagrams/service-registry.mmd);
[requirements](requirements.yaml); [conformance criteria](conformance.md).

## ADR-003-005 — Required and asynchronous communication

**Decision:** Required collaboration is supplied through the Dependency
Injector. Asynchronous communication is carried by the Event Bus.

**Rationale:** Required dependencies must be visible in the composition graph,
while asynchronous interaction needs an explicit decoupled boundary. Keeping
the mechanisms distinct avoids hidden service location and implicit global
communication.

**Consequences:**

- a required collaborator participates in dependency validation and startup
  ordering;
- Event Bus publication does not replace required dependency declaration;
- asynchronous producers and consumers interact through the Event Bus; and
- neither mechanism authorizes mutable global state.

**Reference:** ADR-003-005;
[runtime communication](diagrams/runtime-communication.mmd);
[requirements](requirements.yaml); [conformance criteria](conformance.md).

## ADR-003-006 — Dependency-level startup

**Decision:** Before startup, the resolved service dependencies form a graph.
The graph is validated and partitioned into dependency levels. Levels start
sequentially in dependency order. Services within the same level may start
concurrently.

**Rationale:** Level ordering guarantees that providers are running before
their dependents while preserving optional concurrency where the graph proves
there is no ordering dependency.

**Consequences:**

- startup does not begin until graph construction, resolution, validation, and
  Runtime Freeze complete;
- every level completes before the next level begins;
- same-level concurrency is optional, not required; and
- externally observable startup results remain deterministic even when an
  implementation uses permitted concurrency.

**Reference:** ADR-003-006;
[dependency levels](diagrams/dependency-levels.mmd);
[startup sequence](diagrams/startup-sequence.mmd);
[requirements](requirements.yaml); [conformance criteria](conformance.md).

## ADR-003-007 — Reverse dependency shutdown

**Decision:** Shutdown follows reverse dependency order. Dependents stop
before the providers on which they depend.

**Rationale:** A dependent may need its providers while completing shutdown.
Reversing the validated startup dependency order preserves those providers
until their dependents have stopped.

**Consequences:**

- shutdown uses the resolved graph rather than registration or lexical order;
- a provider is not stopped before its dependents;
- the rule does not assert concurrent shutdown; and
- the Lifecycle Manager advances to Stopped only after ordered shutdown
  completes.

**Reference:** ADR-003-007;
[shutdown sequence](diagrams/shutdown-sequence.mmd);
[requirements](requirements.yaml); [conformance criteria](conformance.md).

## ADR-003-008 — Failure, rollback, and deterministic cleanup

**Decision:** A lifecycle failure enters `Failed`, proceeds through
`Rollback`, and terminates at `Destroyed`. Rollback performs deterministic
cleanup for the failed Runtime instance.

**Rationale:** A single failure path makes partial initialization and startup
observable and recoverable. Deterministic rollback prevents failed instances
from retaining ambiguous registrations, dependencies, or resources.

**Consequences:**

- a failed instance cannot transition to Running;
- deterministic cleanup includes work completed before the failure;
- cleanup outcomes are observable; and
- rollback ends in Destroyed, after which the instance is not reused.

**Reference:** ADR-003-008;
[runtime lifecycle](diagrams/runtime-lifecycle.mmd);
[shutdown sequence](diagrams/shutdown-sequence.mmd);
[requirements](requirements.yaml); [conformance criteria](conformance.md).

## Navigation

- [CCA-RF-1.0](README.md)
- [Normative requirements](requirements.yaml)
- [Conformance criteria](conformance.md)
- [Specification repository](../../README.md)
- [ADR index](../../adrs/README.md)
