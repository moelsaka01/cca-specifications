# MemoryOS SDK Contract

**Standard:** CCA-MEMORYOS-1.0  
**SDK version:** 1.0.0  
**Status:** Normative

## Purpose

The SDK is the single public programming facade over the Investigation Core.
It provides explicit, immutable, Workspace-scoped values without copying
investigation behavior into a language binding.

This Standard defines behavior rather than source-language spelling. A binding
may follow the idioms of its language only while preserving the same ownership,
operations, outcomes, ordering, errors, and canonical data.

## Common object model

| Value | Contract |
|---|---|
| MemoryOS client | Owns one isolated Core lifetime. |
| Workspace | Exact identity owned by one client. |
| Investigation | Immutable current projection and command handle. |
| Replay session | Handle for an already prepared Replay. |
| Comparison session | Configured or active Evolution and Comparative handle. |
| Checkpoint | Opaque same-client restoration handle. |
| Memory Investigation Package | Detached immutable canonical bytes. |
| Verification result | Immutable Core or MIP evidence. |
| Regression report | Detached immutable Core report. |
| Investigation query/result | Closed detached Explorer input and immutable Core output. |

The common cross-binding semantic value is the complete immutable
Investigation projection returned by the Core, together with its immutable
transition log. A binding MAY additionally expose `refresh`, `ReplaySession`,
`ComparisonSession`, named field accessors, or other command-handle
conveniences. Those wrapper names and wrapper-only projections are not a
required common surface and do not replace the complete Investigation value.

## Public operation contract

Names in the tables are logical names. A language binding MAY use its normal
case convention, but it MUST preserve operation identity, required arguments,
result type, ownership, and effect.

When a caller supplies a non-empty local Investigation identifier, the SDK
preserves it. Otherwise the Core derives the identifier exactly as specified
in [investigation-core.md](investigation-core.md): native initial observation
uses the accepted snapshot's Observation identifier; package import uses the
verified manifest's package identifier. This default is a deterministic local
handle identity, not a generated cognitive or package identifier.

### Client operations

| Operation | Required input | Result | Observable contract |
|---|---|---|---|
| `openWorkspace` | One non-empty Workspace identifier | `Workspace` | Creates an immutable Workspace handle owned by this client; it does not inspect a Runtime. |
| `observe` | Owned `Workspace`; input accepted by the implementation's declared native projection profile | Complete `Investigation` projection | Creates one native Investigation by atomically committing `CREATED` and `OBSERVED`, then publishes the exact validated Frame. Optional source metadata is limited to explicit Investigation identifier, operation, query, and result code; omitted local identity follows the exact Core default. |
| `importPackage` | Exact MIP bytes | `Investigation` | Verifies and imports the bytes through the Core. Options may declare supported extensions and an explicit local Investigation identifier; omitted local identity follows the exact Core default. |
| `exportPackage` | Owned MIP-backed `Investigation` | `Memory Investigation Package` | Returns exact canonical bytes of that backing package; a native Investigation is capability unavailable. |
| `verifyPackage` | Exact MIP bytes | `Verification result` | Returns MIP verification status and ordered diagnostics without importing or mutating an Investigation. |
| `regression` | Owned baseline and candidate Investigations | `Regression report` | Returns the Core's immutable factual report; operands MUST share this client and one Workspace. |
| `investigate` | Valid Regression report; closed Explorer query or empty query | `Investigation result` | Returns the Core's ordered evidence navigation result without loading or replaying either source Investigation. |
| `restore` | Same-client opaque `Checkpoint` | `Investigation` | Restores only while the checkpoint's bound Investigation history is intact. |

### Investigation operations

An `Investigation` makes the complete immutable Core projection and transition
log available, regardless of whether a binding exposes individual field
accessors. The projection has exactly the 22 named members and value rules in
[investigation-core.md](investigation-core.md#phase-and-availability-projection).
Null or empty values preserve that Core projection; a binding does not omit
them to create a different semantic shape.

| Operation | Required input | Result | Observable contract |
|---|---|---|---|
| `refresh` | none | Complete `Investigation` projection | Optional binding convenience that loads the current Core projection and appends no transition. |
| `observe` | Declared-profile input | `Investigation` | Appends an explicit Observation and publishes its validated Frame; profile adaptation and optional source metadata have the same limits as client `observe`. |
| `trace` | Exact Reflection selector for native input or exact authored Trace selector for MIP input | `Investigation` | Selects one Trace and atomically prepares Replay. |
| `replay` | none | Complete `Investigation` projection, optionally wrapped by a Replay command handle | Opens the already prepared Replay; it performs no selection or transition. |
| `comparisonSession` | Explicit Evolution identifier for MIP input, or explicit null for the native current pair | Complete `Investigation` projection plus an immutable configured request, optionally wrapped | Configures a request and appends no transition. |
| `compare` | Configured owned comparison request | Updated complete `Investigation` projection, optionally wrapped | Forwards Core action `enter`; Replay MUST already be complete. |
| `verify` | none | `Verification result` | Performs point-in-time Core investigation verification and records the defined verification transition. |
| `checkpoint` | none | `Checkpoint` | Captures an opaque, same-client, same-Investigation integrity-bound handle. |
| `restore` | Bound `Checkpoint` | `Investigation` | Equivalent to client restore with the same ownership checks. |
| `returnToWorld` | none | `Investigation` | Clears active artifact projections and returns according to the lifecycle contract. |
| `archive` | none | `Investigation` | Enters the terminal Archived state. |

### Replay operations

Replay supports exactly `play`, `pause`, `restart`, `previous`, `next`, and
`advance`. Each operation requires the owned prepared Replay, forwards the
same-named Core action, and returns the updated complete Investigation
projection. A binding MAY wrap that projection with immutable `replay`,
`state`, and `view` accessors. It MUST preserve the no-op rules in
[native-artifacts.md](native-artifacts.md) or the exact authored MIP Replay
state, as applicable.

### Comparison operations

Comparison supports the operations below. Each returns the updated complete
Investigation projection. A binding MAY expose an immutable configured or
active session wrapper with `evolution`, `reconstruction`, `state`, and `view`
accessors.

| Operation | Required input | Observable contract |
|---|---|---|
| `previousObservation`, `nextObservation` | active Evolution | Moves the Core's source-kind-specific selection: one adjacent native Observation pair or one position in the MIP package's canonical authored Evolution array. |
| `start` | Native exact Reflection target, or MIP exact Comparative Reconstruction identifier | Starts the selected reconstruction; it MUST NOT infer a target. |
| `play`, `pause`, `reset`, `previous`, `next`, `advance` | active Comparative Reconstruction | Forwards the corresponding comparative action and returns the new immutable projection. |
| `back` | active comparison stage | Forwards the defined Core comparison-return action. |

### Query and result values

`InvestigationQuery` contains only nullable `category`,
`reflectionIdentifier`, and `transition` selectors and obeys the closed
Explorer query rules. `RegressionReport` and `InvestigationResult` preserve
the exact kinds, versions, values, and array order defined by their normative
contracts. Verification results contain validity or status and ordered
diagnostics. A package value exposes a detached copy of its exact bytes.

### Errors

A Core-originated `InvestigationCoreError` is observable as an SDK error
containing the exact Core `code`, `operation`, `message`, and ordered immutable
diagnostics. Wrong-client or wrong-Investigation handles are rejected before
forwarding as the matching ownership or checkpoint error. Invalid package
verification is returned as an invalid Verification result rather than an
imported Investigation. A binding transport or conversion failure remains
distinguishable as a binding error and MUST NOT be relabeled as a Core
semantic rejection.

The optional CCA Studio native projection profile also exposes delegated
Trace artifact failures that are not Core error objects. A binding MUST map
such a failure deterministically and document the mapping, but that mapping is
a binding/profile surface and is not subject to cross-binding code,
operation, or diagnostics equality. The Reference JavaScript binding
preserves the Trace artifact code and message; the Reference C++ and Python
bridge deterministically exposes an otherwise unclassified delegated failure
as `BINDING_FAILURE`. All bindings retain the common atomicity guarantee and
MUST NOT convert a failed selection into accepted cognition.

## Normative requirements

### CCA-MOS-SDK-001 — Facade and Core ownership

One SDK client **MUST** own one isolated Investigation Core lifetime and
**MUST** expose all deterministic investigation execution through that Core;
the Core **MUST NOT** depend on the SDK.

### CCA-MOS-SDK-002 — Version identity

A conforming baseline SDK **MUST** identify its public SDK contract as version
`1.0.0` independently of the MemoryOS Standard, Core, CLI, report, artifact,
and MIP format versions.

### CCA-MOS-SDK-003 — Immutable owned handles

State-bearing SDK values **MUST** be immutable handles or immutable
projections, and a Workspace, Investigation, Replay session, Comparison
session, or checkpoint owned by one client **MUST NOT** mutate another client
or a different Investigation.

### CCA-MOS-SDK-004 — Complete operation parity

Every conforming language binding **MUST** expose the complete immutable
22-member Investigation projection and equivalent behavior for every common semantic
operation, required input, ownership rule, and effect in the public operation
contract, including Replay and comparison actions, Regression, Explorer
navigation, verification, and restoration. A binding **MAY** omit `refresh`
and named session-wrapper projections when equivalent commands and the
complete Investigation result remain available.

### CCA-MOS-SDK-005 — Explicit selection and input

The SDK **MUST** require explicit Workspace identity, declared-profile native
Observation input, Reflection or authored Trace selection, and
comparison selection as applicable; it **MUST NOT** sample a Runtime, discover
missing truth, auto-select cognition, or infer an unavailable artifact. An
omitted local Investigation identifier **MUST** use the exact Core default and
**MUST NOT** be replaced by random, clock-derived, or binding-specific identity.

### CCA-MOS-SDK-006 — Exact package transport

Package import and verification **MUST** accept exact bytes, package export
**MUST** return exact canonical MIP bytes only for a MIP-backed Investigation,
and native export **MUST** fail without synthesized package content.

### CCA-MOS-SDK-007 — Exact Core forwarding

The SDK **MUST** forward investigation commands, Regression operands, and
Explorer reports and queries to the Core without implementing lifecycle,
Trace, Replay, comparison, Regression, navigation, ordering, identity, or
semantic logic of its own.

### CCA-MOS-SDK-008 — Cross-binding equivalence

Equivalent explicit inputs and Core state **MUST** produce semantically
equivalent successful complete immutable Investigation projections, canonical
package bytes, report content, navigation content, ordering, and
Core-originated error outcomes across all conforming SDK language bindings.
Binding-specific convenience wrappers need not have identical source-language
shapes. Delegated native-profile artifact diagnostics have the explicitly
documented binding scope above and are excluded from cross-binding diagnostic
equality.

### CCA-MOS-SDK-009 — Error preservation

A binding **MUST** preserve a Core-originated `InvestigationCoreError` code,
operation, message, and ordered diagnostics. It **MUST** map a delegated native
artifact failure deterministically according to its documented binding/profile
contract. A package-verification rejection **MUST** be a returned invalid
Verification result where that binding contract specifies one, while binding
transport failures **MUST** remain distinguishable from Core-originated
failures.

### CCA-MOS-SDK-010 — No implicit retry

A binding failure **MUST NOT** trigger an implicit retry that could duplicate a
transition, and SDK transport or value conversion **MUST NOT** add cognitive
semantics.

## Conformance

Evidence group `MOS-EVID-SDK-001` covers client isolation, ownership rejection,
all common semantic operation families, optional-wrapper boundaries,
complete-Investigation projection parity, explicit-input and declared-profile
negatives, default and explicit Investigation identifiers, exact package
bytes, Regression and Explorer parity, equivalent cross-binding vectors,
stable errors, and duplicate-transition prevention.
