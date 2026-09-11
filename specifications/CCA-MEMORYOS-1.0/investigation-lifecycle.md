# MemoryOS Investigation Lifecycle

**Standard:** CCA-MEMORYOS-1.0  
**Status:** Normative

## Purpose

This document defines the observable lifecycle of one deterministic MemoryOS
investigation. Private state representation is outside the Standard.

## Informative lifecycle view

The normative transition table below is the authority. The accompanying
[Mermaid source](diagrams/investigation-lifecycle.mmd) is informative.

## Lifecycle states

The exact lifecycle states are `Created`, `Observed`, `Traced`, `ReplayReady`,
`Replaying`, `ReplayComplete`, `ComparisonReady`, `Comparing`, `Verified`, and
`Archived`.

The observable phases are `observe`, `trace`, `replay`, `evolution`, `compare`,
and `archived`. A phase is a projection of lifecycle and active artifacts; it
is not an additional lifecycle state.

## Transition contract

| Trigger | Preconditions | Result |
|---|---|---|
| create without native snapshot | Identifier is unused; Workspace and source kind are exact | One `CREATED` transition and `Created` |
| create with accepted native input | Identifier is unused; declared-profile input Workspace matches | Atomically appends `CREATED`, then `OBSERVED`, and returns `Observed` |
| initial or untraced native observe | Live native investigation and valid matching input | `Observed` |
| native observe with an active Trace target | Live native investigation and valid matching input | Rebuilds the exact target Trace and Replay against the new Frame and returns `ReplayReady`; if that target cannot be rebuilt, accepts the Frame, returns `Observed`, and publishes the deterministic `traceDiagnostic` described below |
| import | Unused MIP-backed identifier and verified matching package | Atomically appends `CREATED`, then `PACKAGE_IMPORTED`, and returns `Observed` |
| trace | Accepted Observation and exact Reflection or authored Trace | `Traced`, then atomically `ReplayReady` after Replay preparation |
| Replay action | Prepared Replay | `ReplayReady`, `Replaying`, or `ReplayComplete` according to Replay state |
| compare enter | Completed Replay and at least two Observations | `ComparisonReady` |
| comparison move | Evolution active and the source-kind-specific preceding or following selection exists | `ComparisonReady` |
| comparative start | Evolution active and exact target or authored reconstruction exists | `Comparing` |
| comparative action | Comparative Reconstruction active | `Comparing` |
| back | Comparative active, or Evolution active | `ComparisonReady`, or restored Replay state |
| verify | Live investigation and complete validation success | `Verified` |
| return to world | Non-observe phase | `Observed`, or `Created` when no Observation exists |
| archive | Any live state | `Archived` |

Invalid transitions fail without publishing a replacement state. A bounded
navigation action already at its boundary is a no-op.

When native observe rebuilds a prior target, success preserves no old artifact
bytes: it reconstructs Trace and Replay from the newly accepted Frame. A
rebuild failure publishes no partial artifact. Its immutable
`traceDiagnostic` contains exactly `code`, `message`, and `targetNodeKey`; the
accepted Observation remains current and lifecycle is `Observed`.

## Transition and log identity

Each transition has exactly `kind`, `version`, `investigationIdentifier`,
`index`, `payload`, `previousLogDigest`, and `identifier`. Version is the Core
version `1.0.0`; index is zero-based and contiguous; payload is detached
canonical data. The containing log has exactly `kind`
`MemoryOSInvestigationTransitionLog`, version `1.0.0`,
`investigationIdentifier`, ordered `transitions`, and `digest`.

Let `D(domain, parts...)` be lower-case SHA-256 with prefix `sha256:` over:

```text
UTF8("MIP-1") || NUL || UTF8(domain)
  || (NUL || exact part bytes) for each part in order
```

A textual part is UTF-8. `JCS(x)` is the exact CCA-MIP-1.0 canonical JSON text.
Transition identity is:

```text
D("INVESTIGATION-CORE-TRANSITION-1.0",
  JCS({investigationIdentifier, index, kind, payload, previousLogDigest}))
```

The empty-log digest is:

```text
D("INVESTIGATION-CORE-LOG-1.0", investigationIdentifier, "[]")
```

For a non-empty prefix, the log digest is:

```text
D("INVESTIGATION-CORE-LOG-1.0", investigationIdentifier,
  JCS(transitions.map(t => ({
    identifier: t.identifier,
    index: t.index,
    investigationIdentifier: t.investigationIdentifier,
    kind: t.kind,
    payload: t.payload,
    previousLogDigest: t.previousLogDigest
  }))))
```

Transition zero binds the empty-log digest. Every later transition binds the
digest of the complete accepted prefix preceding it. The published log digest
is the digest of the complete sequence. Deserialization recomputes every
transition identity, preceding-prefix digest, and final digest; mismatch is
`INVALID_TRANSITION`.

## Generated transition payloads

The Core-generated payload for each transition kind is the exact closed value
below. A nullable selector remains present with value null; it is not omitted.
An "empty" payload is exactly `{}`. These generated shapes are identity-bound
by the preceding digest construction. This table defines publication by Core
operations; the detached Transition constructor's canonical-data check does
not by itself claim semantic validation of a payload without re-deriving its
complete log.

| Transition kind | Exact payload |
|---|---|
| `CREATED` | `{sourceKind, workspaceIdentifier}`, where source kind is `native` or `mip` |
| `OBSERVED` | `{operation, query, resultCode, snapshot}`; operation is explicit or defaults to `InitialObservation` in composite create and `Observe` in later observe, query is explicit canonical data or null, result code is explicit or `OK`, and `snapshot` is the complete accepted native input described below |
| `PACKAGE_IMPORTED` | `{package, supportedExtensions}`; package is the complete verified MIP value and extensions are the accepted tokens in native lexical order |
| `TRACE_SELECTED` for native | `{selectedNodeKey}` containing the exact selected key |
| `TRACE_SELECTED` for MIP | `{targetIdentifier, targetReference, traceIdentifier}` containing the exact selector members, each string/reference or null; selection precedence is Trace identifier, target reference, target identifier, otherwise the last authored Trace that has a bound authored Replay |
| `REPLAY_PREPARED` | empty |
| `REPLAY_ACTION` | `{action}`, where action is one of the six Replay actions |
| `EVOLUTION_ENTERED` | `{evolutionIdentifier}`; exact caller value or null by default. MIP derivation uses the exact matching authored Evolution, or the last authored Evolution when null; native derivation ignores this selector member but retains it in the transition identity |
| `EVOLUTION_MOVED` | `{direction}`, exactly `previous` or `next` |
| `COMPARATIVE_ENTERED` | `{comparativeIdentifier, targetNodeKey}`; each is the exact caller value or null by default. Native derivation uses an explicit `targetNodeKey`, otherwise the comparison-checkpoint target. MIP derivation uses an exact matching `comparativeIdentifier`, otherwise the first authored Reconstruction bound to the active Evolution. The source-kind-irrelevant member is ignored by derivation but remains identity-bound in the payload |
| `COMPARATIVE_ACTION` | `{action}`, exactly `play`, `pause`, `reset`, `previousStep`, `nextStep`, or `advance` |
| `COMPARATIVE_LEFT` | empty |
| `EVOLUTION_LEFT` | empty |
| `RETURNED_TO_WORLD` | empty |
| `VERIFIED` | empty |
| `ARCHIVED` | empty |

The `snapshot` member name is retained for compatibility. Its value is exactly
the complete detached input accepted by the implementation's declared native
projection profile. It is retained without loss and deterministically
projected again whenever state is re-derived. No other form is accepted. The
profile projection uses the exact retained outer `operation`, `query`, and
`resultCode` values. Thus the retained `OBSERVED` transition payload remains
sufficient to reconstruct the same published Frame on every re-derivation.

Explicit string and selector inputs are retained exactly. Canonical cloning
orders object members for identity but does not normalize string content.
No-op Replay, Evolution movement, or Comparative action publishes no payload
because it publishes no transition.

For native cognition, comparison movement advances the defined adjacent
Observation pair. For MIP cognition, it advances one position in the
package's canonical authored Evolution array and does not recompute an
Observation pair. The exact MIP controller states, actions, views, and
active-Evolution binding are defined by
[investigation-core.md](investigation-core.md).

## Normative requirements

### CCA-MOS-LIFE-001 — Closed lifecycle state set

A conforming investigation **MUST** expose exactly the ten lifecycle states
listed in this document and **MUST NOT** expose a private controller or
presentation condition as another lifecycle state.

### CCA-MOS-LIFE-002 — Transition table

Lifecycle changes **MUST** follow the transition contract in this document;
an undefined or precondition-failing transition **MUST** be rejected
deterministically.

### CCA-MOS-LIFE-003 — Authoritative transition log

Investigation history **MUST** be an append-only log beginning at index zero,
with the exact closed transition and log shapes, contiguous indices, exact
ordered transition kinds, exact Core-generated payloads, the published
prefix-digest chain, and content-derived identities using the exact
domain-separated constructions in this document.

The closed transition-kind set is `CREATED`, `OBSERVED`, `PACKAGE_IMPORTED`,
`TRACE_SELECTED`, `REPLAY_PREPARED`, `REPLAY_ACTION`, `EVOLUTION_ENTERED`,
`EVOLUTION_MOVED`, `COMPARATIVE_ENTERED`, `COMPARATIVE_ACTION`,
`COMPARATIVE_LEFT`, `EVOLUTION_LEFT`, `RETURNED_TO_WORLD`, `VERIFIED`, and
`ARCHIVED`.

### CCA-MOS-LIFE-004 — Atomic derivation

Every state-changing command **MUST** build a candidate transition log,
re-derive its complete state from transition zero, and publish the candidate
only after complete derivation succeeds; failure **MUST** leave the prior log
and state unchanged.

### CCA-MOS-LIFE-005 — Trace and Replay preparation

Trace selection **MUST** resolve one exact Reflection for native cognition or
one exact source-authored Trace for MIP cognition, and successful selection
**MUST** atomically prepare the corresponding existing deterministic Replay.
A later accepted native Observation **MUST** rebuild that target against the
new Frame or publish `Observed` with the defined deterministic diagnostic; it
**MUST NOT** retain stale Trace or Replay content.

### CCA-MOS-LIFE-006 — Replay actions

Replay **MUST** consume the prepared Replay and support only `play`, `pause`,
`restart`, `previous`, `next`, and `advance`; a Replay action that produces no
state change **MUST NOT** append a transition.

### CCA-MOS-LIFE-007 — Comparison flow

Comparison **MUST** begin only after Replay completion and at least two
accepted Observations, enter Cognitive Evolution before Comparative
Reconstruction, and restore the integrity-bound pre-comparison Replay state
when leaving Evolution.

### CCA-MOS-LIFE-008 — Verification lifetime

Verification **MUST** validate the complete current investigation before
publishing `Verified`; a later mutation **MUST** invalidate that point-in-time
verification, while immediate archival **MUST** preserve it.

### CCA-MOS-LIFE-009 — Terminal archive

`Archived` **MUST** be terminal for mutation. An archived investigation
**MUST** remain loadable and checkpointable and **MUST NOT** accept another
state-changing command.

### CCA-MOS-LIFE-010 — Return to world

Return-to-world **MUST** clear active Trace, Replay, Evolution, Comparative
Reconstruction, and comparison checkpoint state without changing accepted
Observations, then expose `Observed` or `Created` as specified in the
transition table.

### CCA-MOS-LIFE-011 — Checkpoint restoration

A checkpoint **MUST** bind its Investigation, Workspace, intact transition
log, log digest, transition count, and derived-state digest; restoration
**MUST** re-derive state and **MUST** reject cross-owner, tampered, stale, or
conflicting checkpoints atomically.

### CCA-MOS-LIFE-012 — Source-authored capability availability

A MIP-backed investigation **MUST** expose Trace, Replay, Evolution, and
Comparative Reconstruction only when the imported package contains the
corresponding validated source-authored artifact; an Observation-only package
**MUST** remain Observation-only.

## Conformance

Evidence group `MOS-EVID-LIFE-001` covers both create forms; exact-state and
transition matrices; transition/log member, every exact payload form,
identifier, empty-log, prefix-digest, and tamper vectors; initial and
Trace-preserving Observation; rebuild diagnostics; Replay; comparison;
verification; archive; checkpoint; no-op; MIP availability; and atomic
failure.
