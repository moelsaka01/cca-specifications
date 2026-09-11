# Investigation Core Behavior

**Standard:** CCA-MEMORYOS-1.0  
**Core contract version:** 1.0.0  
**Status:** Normative

## Purpose

This document defines the renderer-independent Investigation Core. The Core is
the single authority for deterministic investigation execution. It consumes
accepted native observations or verified MIPs and projects immutable state to
clients.

## Authority and dependency direction

```text
Runtime truth -> Investigation Core -> SDK -> clients
                      |
                      +-> CCA-MIP-1.0 Producer / Consumer / Verifier
```

The diagram is informative. The normative authority is expressed by the
requirements below.

## Observable operations

The complete Core operation set is `create`, `load`, `observe`, `trace`,
`replay`, `compare`, `regression`, `investigate`, `checkpoint`, `restore`,
`verify`, `archive`, `export`, `import`, and `returnToWorld`.

Replay actions are `play`, `pause`, `restart`, `previous`, `next`, and
`advance`. Compare actions are `enter`, `previous`, `next`, `start`, `play`,
`pause`, `reset`, `previousStep`, `nextStep`, `advance`, and `back`.

Core `create` has two existing forms. Workspace/source-only creation appends
one `CREATED` transition and returns `Created`. Creation with an accepted
native snapshot atomically appends `CREATED` and `OBSERVED` and returns
`Observed`. Import similarly publishes `CREATED` and `PACKAGE_IMPORTED` as one
atomic command. No client observes an intermediate state from a composite
command.

Core `load` accepts one non-empty Investigation identifier and resolves only
the Investigation owned by that Core instance. It returns the complete,
immutable current Investigation projection re-derived from the exact retained
transition log, including an Archived Investigation. It appends no transition,
changes no state, and performs no Runtime, Workspace, or other-Core discovery.
An invalid identifier fails with `INVALID_INPUT`; an identifier not owned by
that Core fails with `NOT_FOUND`.

Direct Core MIP selection has these deterministic fallbacks. A `trace`
payload with no non-null Trace identifier, target reference, or target
identifier selects the last authored Trace that has a bound authored Replay.
A comparison `enter` payload with
no non-null Evolution identifier selects the last authored Evolution. A
comparative `start` payload with no non-null Comparative identifier selects
the first authored Comparative Reconstruction whose Evolution identifier is
the active one. An explicit selector that has no match is capability
unavailable. For native comparative start, a null target uses the exact target
saved in the comparison checkpoint; an explicit target overrides it. These
Core fallbacks preserve source authorship and package order. They do not relax
the SDK's stricter explicit-selection contract.

## Investigation identity

An explicit non-empty local Investigation identifier is preserved. Otherwise
the Core derives:

```text
"investigation:" + suffix(D(
  "INVESTIGATION-CORE-IDENTIFIER-1.0",
  workspaceIdentifier,
  sourceIdentifier))
```

`D` is the exact digest construction in
[investigation-lifecycle.md](investigation-lifecycle.md), and `suffix` removes
the leading `sha256:`. For native creation, `sourceIdentifier` is the accepted
declared-profile input's `observationIdentifier`. Without native input it is
an explicit creation `sourceIdentifier` or the literal `created`. For import
it is the MIP manifest's `packageIdentifier`. An existing derived or explicit local
identifier fails with `DUPLICATE_IDENTIFIER`; the local identifier never
changes package or Runtime source identity.

## Source boundaries

| Source kind | Authority | Core behavior |
|---|---|---|
| `native` | Accepted detached MemoryOS observation | Deterministically derives the Trace, Replay, Evolution, and Comparative Reconstruction values defined by [native-artifacts.md](native-artifacts.md). |
| `mip` | Verified CCA-MIP-1.0 package | Selects only validated artifacts already authored in that package. |

For `native`, a product-specific source snapshot MAY enter through a declared
native projection profile at or within the Core boundary. The Core's published
accepted state is the resulting validated `MemoryOSObservationFrame` version
`1.1`. The profile input schema is not portable; its output and all subsequent
artifacts obey [native-artifacts.md](native-artifacts.md). For `mip`, the
verified package itself is the portable input and no native Frame is invented.

The native cognitive artifact kinds `MemoryOSCognitiveTrace`,
`MemoryOSCognitiveReplay`, `MemoryOSCognitiveEvolution`, and
`MemoryOSComparativeReconstruction` use version `1.1`. MIP-backed values
retain their exact package shapes and versions.

## Phase and availability projection

Phase uses this exact precedence:

```text
Archived lifecycle
  > active Comparative Reconstruction
  > active Evolution
  > active Trace with Replay cursor >= 0 or Replay status != ready
  > active Trace
  > observe
```

The corresponding phase values are `archived`, `compare`, `evolution`,
`replay`, `trace`, and `observe`.

Availability contains exactly `trace`, `replay`, `compare`, `comparative`,
`verify`, `checkpoint`, and `export`, all booleans. Let Observation count mean
native Frame count for `native` and package Observation count for `mip`.

| Key | Exact true condition |
|---|---|
| `trace` | Not Archived, and native has a current Frame or MIP has at least one authored Trace with a bound authored Replay |
| `replay` | Not Archived and an active Replay exists |
| `compare` | Not Archived, and Evolution is active or Replay is completed with at least two Observations and the source is native or the MIP has at least one authored Evolution |
| `comparative` | Not Archived, Evolution is active, and the source is native or the MIP has at least one authored Comparative Reconstruction bound to that active Evolution |
| `verify` | Not Archived |
| `checkpoint` | Always true, including Archived |
| `export` | Source is MIP and a verified backing package exists |

These values report command availability; they do not execute, infer, or
preselect a command.

## MIP-backed controller state and views

MIP Replay, Evolution, and Comparative Reconstruction artifacts remain the
exact source-authored CCA-MIP-1.0 values. The Core owns only the following
ephemeral controller state and projections; none is exported into MIP.

For a MIP Replay with `N` authored steps, Replay controller state contains
exactly `replayIdentifier`, `status`, and `cursor`. Status is one of `ready`,
`playing`, `paused`, or `completed`; initial state is `ready` with cursor `-1`.
The six Replay actions have these exact effects:

| Action | MIP Replay state transition |
|---|---|
| `play` | From `ready`, enters `playing` at cursor 0; from `paused`, enters `playing` at the same cursor; otherwise no-op. |
| `pause` | From `playing`, enters `paused` at the same cursor; otherwise no-op. |
| `restart` | Enters `ready` at cursor `-1`. |
| `previous` | At cursor 0 or `-1`, enters `ready` at `-1`; otherwise enters `paused` at the preceding cursor. |
| `next` | Moves at most one step; the final step is `completed`, otherwise the result is `paused`. |
| `advance` | Only from `playing`, moves at most one step and becomes `completed` on the final step; otherwise no-op. |

The immutable MIP Replay view contains exactly `kind`, `version`, `identifier`,
`status`, `cursor`, `total`, `completedNodeKeys`, `completedEdgeKeys`,
`currentNodeKey`, `currentEdgeKey`, `futureNodeKeys`, `futureEdgeKeys`,
`currentRole`, and `currentType`. Kind is
`MemoryOSInvestigationPackageReplayView`, version is Core version `1.0.0`, and
total is `N`. A reference key is the exact JCS text of its MIP Reference.
Visible steps are the ordered prefix through the cursor. If status is
`completed`, that complete prefix is completed and there is no current step;
otherwise its last step is current and its preceding prefix is completed.
Future values are the ordered suffix after the cursor. Node and relationship
keys retain their respective step order. Current key, role, and type are null
when no current step exists; current type is the authored MIP `elementType`.

MIP Evolution navigation retains an index into the package's canonical
`evolutions` array. `previous` and `next` move exactly one array position and
select that authored Evolution; an array boundary is a no-op. They do not
recompute an Observation pair. Comparative selection is restricted to authored
reconstructions whose `evolutionIdentifier` equals the active Evolution's
identifier. An explicit identifier outside that subset is capability
unavailable; the null default selects the first value in that subset.

For a selected MIP Comparative Reconstruction with `N` authored moments,
controller state contains exactly `reconstructionIdentifier`, `status`,
`cursor`, and `pauseReason`. Status uses the Replay status set; pause reason is
null, `divergence`, or `engineer`; initial state is `ready`, cursor `-1`, and
null reason. A moment is divergent exactly when its authored `state` is not
`shared`. The six Comparative actions have these exact effects:

| Action | MIP Comparative state transition |
|---|---|
| `play` | From `ready`, enters moment 0, using `playing` except that a sole shared moment is `completed`; a divergent moment pauses with reason `divergence`. From `paused`, enters `playing` at the same cursor with null reason. From `playing` or `completed`, no-op. |
| `pause` | From `playing`, enters `paused` at the same cursor with reason `engineer`; otherwise no-op. |
| `reset` | Enters `ready` at cursor `-1` with null reason. |
| `previousStep` | At cursor 0 or `-1`, enters initial state; otherwise moves one moment backward and pauses with reason `divergence` for a divergent moment or `engineer` for a shared moment. |
| `nextStep` | From `completed`, no-op. At the final cursor, enters `completed`. Otherwise moves one moment forward and pauses with reason `divergence` for a divergent moment or `engineer` for a shared moment. |
| `advance` | Only from `playing`. At the final cursor, enters `completed`; otherwise moves one moment forward, remaining `playing` for a shared moment or pausing with reason `divergence`. |

The immutable MIP Comparative view contains exactly `kind`, `version`,
`identifier`, `active`, `status`, `pauseReason`, `cursor`, `total`,
`firstDivergenceIndex`, `divergenceIndices`, `divergenceCount`, `currentMoment`,
`atDivergence`, and `moments`. Kind is
`MemoryOSInvestigationPackageComparativeView`, version is Core version `1.0.0`,
active is true, and total is `N`. Divergence indices and moments are the exact
authored arrays; first divergence is their first index or null and count is
their length. Current moment is null in `ready` and `completed`, otherwise it
is the authored moment at the cursor. `atDivergence` is true exactly when that
current moment exists and its state is not `shared`.

The complete immutable Investigation projection has exactly these 22 members:

```text
identifier, lifecycle, phase, workspaceIdentifier, sourceKind,
observationFrames, packageObservations, currentFrame,
currentPackageObservation, trace, replay, replayState, replayView,
evolutionController, evolution, comparisonFrames,
comparativeReconstruction, comparativeReplayState, comparativeView,
traceDiagnostic, verification, availability
```

Native Observation Frames and MIP Observations retain their accepted order;
their current members are the last accepted value or null. Active artifact,
controller, state, view, diagnostic, and verification members retain the
derived immutable value or null. For active native Evolution,
`comparisonFrames` has `from` and `to` containing the controller-selected
Frames; for active MIP Evolution those members are the package Observations
whose identifiers match its `fromObservationIdentifier` and
`toObservationIdentifier`, or null when absent; without active Evolution the
member is null. `phase` and `availability` are the exact projections above.

## Verification Session projection

A successful `verify` command appends `VERIFIED` and publishes a recursively
immutable Verification Session with exactly these eight members:

```text
kind, version, identifier, investigationIdentifier,
transitionLogDigest, lifecycle, status, checks
```

`kind` is `MemoryOSInvestigationVerificationSession`; `version` is `1.0.0`;
`investigationIdentifier` is the verified Investigation; `transitionLogDigest`
is the digest of the authoritative log through the `VERIFIED` transition;
`lifecycle` is `Verified`; and `status` is `passed`. The identifier is:

```text
D("INVESTIGATION-CORE-VERIFICATION-1.0",
  investigationIdentifier,
  transitionLogDigest,
  "Verified")
```

Each check has exactly `code` and `status`, with status `passed`. Checks are
ordered `TRANSITION_LOG`, `LIFECYCLE`, followed for a native source by each
applicable active artifact in this order: `TRACE`, `REPLAY`, `EVOLUTION`,
`COMPARATIVE`. For a MIP source, `MIP` follows the first two checks. An absent
optional artifact contributes no check. Any failed check raises
`VERIFICATION_FAILED` and publishes neither a Verification Session nor the
transition. The point-in-time session remains valid after terminal archival
and remains bound to the log digest through `VERIFIED`; any other subsequent
mutation invalidates it.

## Checkpoint projection

The Core checkpoint is an integrity-bound value with exactly these nine
members:

```text
kind, version, identifier, investigationIdentifier, workspaceIdentifier,
transitionLog, transitionLogDigest, transitionCount, stateDigest
```

`kind` is `MemoryOSInvestigationCheckpoint`, version is `1.0.0`,
`transitionLog` is the complete authoritative immutable log at capture,
`transitionLogDigest` is its digest, and `transitionCount` is its transition
count. `stateDigest` is:

```text
D("INVESTIGATION-CORE-STATE-1.0", JCS({
  activeReplayIdentifier,
  activeTraceIdentifier,
  comparativeIdentifier,
  comparativeReplayState,
  currentObservationIdentifier,
  evolutionIdentifier,
  investigationIdentifier,
  lifecycle,
  observationIdentifiers,
  replayState,
  sourceKind,
  supportedExtensions,
  traceDiagnostic,
  workspaceIdentifier
}))
```

Nullable active identifiers and states retain null. The current Observation
identifier is the current native world-frame identifier, otherwise the
current MIP Observation identifier, otherwise null. Observation identifiers
are the native world-frame identifiers or MIP Observation identifiers in
their accepted order. `JCS` and `D` are the canonicalization and digest
construction defined by the incorporated CCA-MIP-1.0 contract and
[investigation-lifecycle.md](investigation-lifecycle.md).

The checkpoint identifier is
`D("INVESTIGATION-CORE-CHECKPOINT-1.0", investigationIdentifier,
transitionLogDigest, stateDigest)`. Restoration requires an intact Checkpoint
value of the same contract implementation, exact Investigation and Workspace
identity, intact log digest and count, and an exact state digest after
re-derivation. A fresh Core instance may restore it; a Core that already owns
a different history under that Investigation identifier MUST reject its
replacement. The SDK may impose its same-client handle rule and may keep the
checkpoint opaque, but any public Core checkpoint projection has this exact
value.

## Normative requirements

### CCA-MOS-CORE-001 — Single execution authority

The Investigation Core **MUST** be the only implementation of investigation
lifecycle, Trace, Replay, Evolution, Comparative Reconstruction, Cognitive
Regression, and Explorer execution used by platform clients.

### CCA-MOS-CORE-002 — Isolated ownership

One Core instance **MUST** own isolated investigation state, and each
investigation **MUST** retain one exact Workspace and one exact source kind,
`native` or `mip`, for its lifetime.

### CCA-MOS-CORE-003 — Closed operation contract

A conforming Core **MUST** implement the complete operation and action sets
listed in this document, both create forms, exact default Investigation
identity, exact direct-Core source-selection fallbacks, and lifecycle behavior defined in
[investigation-lifecycle.md](investigation-lifecycle.md).

### CCA-MOS-CORE-004 — Immutable canonical values

The Core **MUST** canonical-clone accepted data, publish recursively immutable
investigations, states, sessions, reports, results, logs, and checkpoints, and
**MUST NOT** expose mutable internal state as authority. Its successful
Verification Session **MUST** use the exact public projection, content-derived
identity, and check order defined above. A public Core checkpoint **MUST** use
the exact integrity-bound projection and identities defined above.

### CCA-MOS-CORE-005 — Native derivation fidelity

For native cognition, the Core **MUST** derive and validate Trace, Replay,
Evolution, Comparative Reconstruction, and their control states exactly as
defined by [native-artifacts.md](native-artifacts.md), and **MUST** retain the
specified native kind and version identities.

### CCA-MOS-CORE-006 — MIP authorship fidelity

For MIP cognition, the Core **MUST** verify the package and expose only its
source-authored Observations and derived artifacts; it **MUST NOT** project an
Observation-only package into missing investigation artifacts or relabel a
partial package record as a native artifact.

### CCA-MOS-CORE-007 — Package operation boundary

Core import, export, and package verification **MUST** delegate canonical
package meaning to CCA-MIP-1.0; native export without an authored MIP **MUST**
fail with capability unavailable rather than synthesizing package cognition.

### CCA-MOS-CORE-008 — Read-only Regression and Explorer

Regression and Explorer operations **MUST** be read-only, append no transition,
change no lifecycle, and preserve the exact contracts in
[cognitive-regression.md](cognitive-regression.md) and
[investigation-explorer.md](investigation-explorer.md).

### CCA-MOS-CORE-009 — Renderer-independent projection

The Core **MUST** project complete immutable investigation state for consumers
with the exact phase precedence and closed availability projection in this
document, publish the exact 22-member complete Investigation projection, and
**MUST NOT** depend on a DOM, renderer, camera, layout, pixels,
user interface route, or presentation cache to derive that state.

### CCA-MOS-CORE-010 — Deterministic execution environment

Core execution **MUST NOT** use randomness, a wall clock, timer, polling,
network request, Provider SDK, filesystem enumeration, or mutable global state
to determine semantic identity, order, transitions, reports, or results.

### CCA-MOS-CORE-011 — Resource bounds

The Core **MUST** reject an investigation that would exceed 10,000 transitions
or whose transition payload has a UTF-8 byte length greater than
`16 * 1024 * 1024` after exact `JCS(payload)` canonicalization with
`RESOURCE_LIMIT_EXCEEDED`, before publishing the candidate transition.

### CCA-MOS-CORE-012 — Stable atomic failure

Core failure **MUST** publish no partial result or transition. A failure
originating in the Core **MUST** preserve a stable code, operation, message,
and ordered immutable diagnostics and use only the applicable codes
`CAPABILITY_UNAVAILABLE`, `CHECKPOINT_MISMATCH`, `CORE_BUSY`,
`DUPLICATE_IDENTIFIER`, `INVALID_COMMAND`, `INVALID_INPUT`,
`INVALID_TRANSITION`, `NOT_FOUND`, `RESOURCE_LIMIT_EXCEEDED`,
`SOURCE_KIND_MISMATCH`, `VERIFICATION_FAILED`, `WORKSPACE_MISMATCH`,
`INVALID_QUERY`, and `INVALID_REGRESSION_REPORT`. A delegated native Trace
failure instead preserves its artifact code and message from this closed set:
`INVALID_QUERY`, `WORKSPACE_MISMATCH`, `SESSION_MISMATCH`, `FRAME_MISMATCH`,
`NOT_FOUND`, `INVALID_TARGET`, `MISSING_EVIDENCE`, `AMBIGUOUS_EVIDENCE`,
`INCONSISTENT_EVIDENCE`, `INCONSISTENT_RELATIONSHIP`, and `INVALID_TRACE`.
Such an artifact failure is atomic but is not required to acquire the Core
error object's operation or diagnostics members.

## Conformance

Evidence group `MOS-EVID-CORE-001` covers all operations; both create forms;
explicit and all default-identifier inputs; exact state, phase, availability,
and value versions; Verification Session identity, members, and check order;
Checkpoint members, digests, restoration negatives; native and MIP source
paths; transition-log re-derivation;
immutability; isolation; resource limits; deterministic repeats; renderer
separation; and atomic negative cases.
