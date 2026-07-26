# CCA Runtime Foundation Standard 1.0

**Identifier:** CCA-RF-1.0  
**Version:** 1.0  
**Status:** Published  
**Publication date:** 2026-07-26  
**Boundary:** Runtime Foundation

CCA-RF-1.0 is the authoritative architecture standard for the CCA Runtime
Foundation. It defines the official headless runtime host, the canonical layer
model, Runtime instance isolation, the six foundation components, lifecycle
and freeze semantics, typed service composition, startup and shutdown
ordering, failure recovery, the programming model, and conformance.

This standard defines architectural obligations rather than a source-language
API. An implementation MAY select concrete types and mechanisms only within
those obligations.

## Navigation

Normative and governance material:

- [Specification repository](../../README.md)
- [CCA Engineering Handbook 1.0](../CCA-ENG-1.0/README.md)
- [Normative requirements](requirements.yaml)
- [Approved architecture decisions](decisions.md)
- [Conformance requirements](conformance.md)
- [Version history](CHANGELOG.md)

Normative diagrams:

- [Canonical layer model](diagrams/layer-model.mmd)
- [Runtime lifecycle](diagrams/runtime-lifecycle.mmd)
- [Dependency levels](diagrams/dependency-levels.mmd)
- [Service Registry](diagrams/service-registry.mmd)
- [Runtime communication](diagrams/runtime-communication.mmd)
- [Startup sequence](diagrams/startup-sequence.mmd)
- [Shutdown sequence](diagrams/shutdown-sequence.mmd)

## 1. Normative language

The terms **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY**
express requirement strength. `MUST` and `MUST NOT` define conditions for
CCA-RF-1.0 conformance.

The prose, [requirements registry](requirements.yaml), [decision
record](decisions.md), [conformance standard](conformance.md), and linked
diagrams form one publication. A conformance claim MUST satisfy their
consistent interpretation.

## 2. Purpose and scope

The Runtime Foundation provides the deterministic foundation on which higher
CCA layers execute. It owns Runtime lifecycle, service composition, dependency
injection, asynchronous notification, configuration, and observability as one
coherent boundary.

Architecture is authoritative over implementation. Runtime responsibilities
and externally observable contracts MUST be documented independently of the
mechanisms selected to implement them. Implementation structure MUST NOT
silently redefine the boundary, layer model, component set, or lifecycle.

For equivalent declared inputs and initial conditions, the Runtime Foundation
MUST produce equivalent documented acceptance, rejection, diagnostic, and
cleanup outcomes. Permitted concurrency does not weaken this determinism
obligation.

CCA-RF-1.0 defines:

- the official headless `cca-runtime` host;
- multiple isolated Runtime instances without a Runtime singleton;
- the five-layer CCA model and its dependency direction;
- exactly six Runtime Foundation components;
- the complete Runtime lifecycle, Runtime Freeze, failure, and rollback paths;
- compile-time type-safe service registration and resolution;
- dependency graph validation and dependency-level startup;
- reverse-dependency shutdown and deterministic cleanup;
- communication, configuration, observability, and thread-safety obligations;
  and
- the evidence required for conformance.

CCA-RF-1.0 does not prescribe source-level classes, function signatures,
command-line syntax, configuration syntax, event payload schemas, wire
formats, storage models, or scheduling algorithms. Such details are valid only
when they preserve every architectural obligation in this standard.

## 3. Canonical layer model

CCA uses the following five layers, numbered from the operating foundation to
applications:

| Layer | Canonical name |
|---|---|
| L5 | Applications |
| L4 | Domain Engines: Representation, Process, and Persistence |
| L3 | Runtime Foundation |
| L2 | Platform Abstraction |
| L1 | Operating System |

L4 contains exactly the three named Domain Engines: **Representation**,
**Process**, and **Persistence**. This statement defines the layer membership;
it does not make those engines Runtime Foundation components.

An inter-layer dependency MUST originate at a higher-numbered layer and target
a lower-numbered layer. An inter-layer dependency from a lower-numbered layer
to a higher-numbered layer is prohibited. A dependency MAY cross more than one
layer. This rule does not prohibit an explicit dependency whose source and
target are within the same layer.

Callbacks, events, and returned results do not reverse the direction of the
dependency that established an interaction. Code placement or naming MUST NOT
be used to conceal an upward dependency.

The [canonical layer diagram](diagrams/layer-model.mmd) is the normative visual
representation of this model.

## 4. Official runtime host

`cca-runtime` is the official headless host for the CCA Runtime Foundation. It
owns Runtime instance creation, lifecycle execution, and destruction without
introducing a graphical or interactive application layer.

The host MUST support multiple Runtime instances. Each Runtime instance MUST:

- have its own execution context;
- own its own six Runtime Foundation components;
- own its own configuration, registrations, provider bindings, dependency
  graph, events, lifecycle state, and observability context;
- execute and fail independently of other Runtime instances; and
- release its owned resources through its own shutdown or rollback path.

There MUST be no process-wide Runtime singleton. Runtime behavior, dependency
access, or lifecycle control MUST NOT rely on mutable global Runtime state.
Creating, freezing, starting, stopping, failing, or destroying one Runtime
instance MUST NOT implicitly perform the same operation on another instance.

## 5. Runtime Foundation components

Each Runtime instance contains exactly these six Runtime Foundation components:

| Component | Normative responsibility |
|---|---|
| Lifecycle Manager | Enforces the Runtime state model and coordinates startup, shutdown, failure, rollback, and destruction |
| Service Registry | Records typed Service Contracts and their internal Providers, validates cardinality, and exposes the frozen service composition |
| Dependency Injector | Resolves and supplies required collaborators according to Service Contracts and the validated dependency graph |
| Event Bus | Carries asynchronous notifications within one Runtime instance |
| Configuration Manager | Establishes and validates the Runtime instance configuration before Runtime Freeze |
| Observability | Makes lifecycle, composition, communication, startup, shutdown, failure, and cleanup outcomes inspectable |

These responsibilities form one Runtime Foundation. An implementation MUST
NOT add another peer component to this component set or rename a component in
a conformance claim.

Required collaboration between components and services MUST use the
Dependency Injector. Asynchronous notification MUST use the Event Bus. The two
mechanisms have distinct roles and MUST NOT be substituted in a way that
hides a required dependency.

The [Runtime communication diagram](diagrams/runtime-communication.mmd) shows
these collaboration paths.

## 6. Runtime lifecycle

The Lifecycle Manager MUST enforce this exact successful lifecycle:

```text
Constructed
  -> Initializing
  -> Configuring
  -> Registering Services
  -> Resolving Dependencies
  -> Validating
  -> Runtime Freeze
  -> Starting
  -> Running
  -> Stopping
  -> Stopped
  -> Destroyed
```

The failure path is exactly:

```text
Failed -> Rollback -> Destroyed
```

The lifecycle states have the following architectural meaning:

| State | Required outcome |
|---|---|
| Constructed | The Runtime instance and its isolated execution context exist |
| Initializing | The six Runtime Foundation components are initialized for the instance |
| Configuring | Configuration Manager establishes and validates instance configuration inputs |
| Registering Services | Service Contracts and internal Providers are registered |
| Resolving Dependencies | Dependency Injector resolves required collaborators and the startup dependency graph is computed |
| Validating | Configuration, registrations, cardinalities, provider bindings, and the dependency graph are validated |
| Runtime Freeze | Runtime composition becomes immutable |
| Starting | Services start by validated dependency level |
| Running | All required startup levels completed successfully |
| Stopping | Services stop in reverse dependency order |
| Stopped | Required service shutdown completed |
| Destroyed | Instance-owned Runtime resources have completed deterministic cleanup |
| Failed | A lifecycle operation failed and normal progression ceased |
| Rollback | Deterministic cleanup executes for the failed instance |

Only the Lifecycle Manager MUST advance the Runtime lifecycle. A lifecycle
transition that is not part of the successful path or failure path MUST be
rejected deterministically and made observable.

`Runtime Freeze` occurs after successful validation and before any service
startup. No lifecycle path MUST bypass it on the way to `Starting`.

The [Runtime lifecycle diagram](diagrams/runtime-lifecycle.mmd) is the
normative state visualization.

## 7. Configuration and Runtime Freeze

Configuration Manager MUST establish all configuration that affects Runtime
composition during `Configuring`. Configuration required to validate a
Service Contract, Provider, dependency, or startup level MUST be available
before `Validating`.

During `Registering Services`, the Service Registry receives the complete
Service Contract and Provider composition. During `Resolving Dependencies`,
the Dependency Injector resolves required collaborations and the Runtime
computes its complete startup dependency graph. During `Validating`, the
Runtime verifies the resulting composition.

Entry into `Runtime Freeze` MUST make the Runtime topology and composition
immutable. After Runtime Freeze:

- configuration that affects Runtime composition MUST NOT be added, removed,
  or replaced;
- Service Contracts and Providers MUST NOT be added, removed, or replaced;
- provider bindings and declared cardinalities MUST NOT change; and
- dependency edges and computed startup levels MUST NOT change.

Operational lifecycle state and observable outcomes continue to advance after
Runtime Freeze; the composition on which those operations act does not.

If validation fails, the Runtime MUST enter `Failed`; it MUST NOT enter
`Runtime Freeze` or start services.

## 8. Service Registry

The Service Registry MUST be compile-time type-safe. A Service Contract is
identified by its program type, and registration and resolution MUST preserve
that type identity. Runtime string lookup MUST NOT be used for service
identity or service resolution.

A **Service Contract** is the dependency surface available to a consumer. A
**Provider** is the internal implementation bound to a Service Contract within
the Runtime instance. Consumers MUST depend on Service Contracts rather than
internal Provider types. Provider identity and construction remain internal to
the Runtime composition.

Every Service Contract dependency MUST declare exactly one of these
cardinalities:

| Cardinality | Valid provider count |
|---|---|
| `ExactlyOne` | Exactly one Provider |
| `ZeroOrOne` | Zero or one Provider |
| `OneOrMore` | One or more Providers |

The Runtime MUST validate every declared cardinality before Runtime Freeze. A
provider count outside the declared cardinality is a validation failure and
MUST follow the failure path.

The Service Registry belongs to one Runtime instance. A registration in one
instance MUST NOT satisfy a Service Contract in another instance. After
Runtime Freeze, the registry is read-only for the remainder of that instance's
lifecycle.

The [Service Registry diagram](diagrams/service-registry.mmd) defines the
contract, Provider, and cardinality relationships.

## 9. Dependency injection and communication

Dependency Injector is the only Runtime Foundation mechanism for supplying a
required collaborator. Required dependencies MUST be declared as typed
Service Contracts with one of the three approved cardinalities. Resolution
MUST use the frozen Service Registry composition and MUST NOT use string
lookup, mutable global state, or a Provider from another Runtime instance.

The Event Bus carries asynchronous notifications. Publishing a notification
does not establish a required collaborator relationship and MUST NOT be used
to conceal one. A service that cannot perform its responsibility without
another service MUST declare that collaboration for dependency injection.

The Event Bus belongs to one Runtime instance. Notification publication and
observation MUST remain within that instance. Event Bus activity MUST be
observable through the instance's Observability component.

Asynchronous notification does not change the canonical layer dependency rule.
The [Runtime communication diagram](diagrams/runtime-communication.mmd)
illustrates the distinct dependency-injection and notification paths.

## 10. Startup dependency graph

The Runtime MUST compute the complete service startup dependency graph during
`Resolving Dependencies`, validate it during `Validating`, and freeze it during
`Runtime Freeze`. Service startup MUST NOT begin before the graph is complete,
valid, and immutable.

Each graph node represents a Provider startup responsibility. Each dependency
edge records that the Provider at the dependent end requires the Provider at
the dependency end. The graph MUST satisfy all typed Service Contracts and
their cardinalities. A missing required Provider, invalid cardinality, or
dependency cycle is a validation failure.

The Runtime MUST derive ordered dependency levels from the graph:

- a Provider's dependencies MUST be in earlier levels;
- levels MUST execute sequentially;
- every Provider in one level MUST complete startup successfully before the
  next level begins; and
- Providers within the same level MAY start concurrently because no dependency
  edge orders them relative to one another.

The choice to execute one level concurrently does not weaken instance
isolation, thread safety, deterministic failure handling, or observability.
Externally observable startup results MUST remain deterministic when this
concurrency is used.

The Runtime enters `Running` only after every Provider in every level starts
successfully.

The [dependency-level diagram](diagrams/dependency-levels.mmd) defines level
ordering. The [startup sequence](diagrams/startup-sequence.mmd) shows the
required lifecycle coordination.

## 11. Shutdown

Normal shutdown MUST follow:

```text
Running -> Stopping -> Stopped -> Destroyed
```

During `Stopping`, the Runtime MUST traverse the frozen dependency graph in
reverse dependency order. A Provider MUST stop only after every started
dependent that requires it has stopped. This guarantees that a required
collaborator remains available while its dependents shut down.

Reverse dependency edges are the complete shutdown ordering rule. Providers
that are unrelated by a dependency edge have no relative shutdown order under
CCA-RF-1.0. The selected execution schedule MUST preserve every reverse
dependency edge and deterministic cleanup.

After all required Provider shutdown completes, the Runtime enters `Stopped`
and then `Destroyed`. Destruction completes deterministic cleanup for the
instance and MUST NOT alter another Runtime instance.

The [shutdown sequence](diagrams/shutdown-sequence.mmd) is the normative visual
ordering.

## 12. Failure, rollback, and deterministic cleanup

A failure in lifecycle execution MUST move the affected Runtime instance to
`Failed`. The Runtime MUST cease normal lifecycle progression and execute:

```text
Failed -> Rollback -> Destroyed
```

Rollback MUST perform deterministic cleanup for the failed Runtime instance,
including work completed before the failure. It MUST terminate in
`Destroyed`.

For equivalent Runtime composition, completed-work set, and failure condition,
rollback MUST select equivalent cleanup responsibilities and produce
equivalent observable outcomes. The original failure and rollback outcomes
MUST be available through Observability.

Rollback ends in `Destroyed`; a failed Runtime does not transition to
`Running`, `Stopping`, or `Stopped`. Failure and rollback are isolated to the
affected Runtime instance.

## 13. Observability

Each Runtime instance MUST provide an Observability component that makes the
following outcomes attributable to that instance:

- lifecycle state changes and rejected transitions;
- configuration and composition validation;
- Service Contract registration, resolution, and cardinality failures;
- dependency graph and startup-level execution;
- Event Bus notification failures;
- normal shutdown ordering; and
- failure, rollback, and deterministic cleanup.

Observability MUST preserve Runtime instance identity so evidence from
concurrent instances cannot be conflated. Observability records outcomes; it
does not become a seventh coordination component or a substitute for the
Lifecycle Manager, Dependency Injector, or Event Bus.

## 14. Programming model

A conforming implementation MUST publish a programming-model document that
explains how its concrete implementation realizes this standard without
presenting implementation-specific syntax as CCA-RF-1.0 architecture.

The programming-model document MUST cover:

1. how the headless `cca-runtime` host creates, owns, and destroys isolated
   Runtime instances;
2. how callers distinguish and retain Runtime instance identity without a
   global singleton;
3. how configuration is supplied and finalized before Runtime Freeze;
4. how typed Service Contracts, internal Providers, and the `ExactlyOne`,
   `ZeroOrOne`, and `OneOrMore` cardinalities are declared and registered;
5. how required dependencies are declared and their collaborators are supplied
   through Dependency Injector;
6. how asynchronous notifications are published and observed through Event
   Bus;
7. how lifecycle progress, validation failure, startup, shutdown, rollback,
   destruction, and other Runtime errors are reported and observed;
8. how ownership and lifetime align with reverse dependency shutdown and
   deterministic cleanup; and
9. the thread-safety guarantees of the host, Runtime instances, all six
   components, Service Contracts, Providers, and notification handlers.

Thread-safety documentation MUST distinguish instance confinement from shared
state. It MUST identify which operations may execute concurrently and how the
implementation prevents data races. If Providers in one dependency level start
concurrently, their startup operations and supplied collaborators MUST be
safe for that execution. Asynchronous Event Bus delivery MUST honor the
documented thread-safety guarantees.

No implementation MUST rely on accidental serialization to satisfy a declared
thread-safety guarantee.

## 15. Conformance overview

Conformance is assessed against this standard, the
[machine-readable requirements](requirements.yaml), the
[architecture decisions](decisions.md), and the detailed
[conformance standard](conformance.md).

A CCA-RF-1.0 conformance claim MUST identify the implementation and version
under assessment and provide evidence that:

- `cca-runtime` is the official headless host;
- multiple isolated Runtime instances operate without a Runtime singleton;
- each Runtime contains exactly the six named foundation components;
- the five canonical layers and downward inter-layer dependency rule are
  preserved;
- the exact successful and failure lifecycle paths are enforced;
- Runtime Freeze occurs after validation and before service startup;
- configuration, service composition, provider bindings, and dependency levels
  are immutable after Runtime Freeze;
- Service Registry is compile-time type-safe, uses no string lookup, separates
  Service Contracts from internal Providers, and enforces the three approved
  cardinalities;
- required collaboration uses Dependency Injector and asynchronous
  notification uses Event Bus;
- startup follows sequential dependency levels with optional concurrency only
  within a level;
- shutdown respects reverse dependency order;
- rollback performs deterministic cleanup and terminates in `Destroyed`;
- failure cleanup is observable;
- instance isolation and thread-safety obligations are documented and
  demonstrated; and
- every applicable requirement has attributable verification evidence.

A conformance claim MUST disclose every unmet requirement. Similar naming or
partial lifecycle behavior is not conformance.

## 16. Version history

| Version | Summary |
|---|---|
| 1.0 | Initial authoritative Runtime Foundation architecture, lifecycle, composition, ordering, failure, programming-model, and conformance standard |

See [CHANGELOG.md](CHANGELOG.md) for the publication record.
