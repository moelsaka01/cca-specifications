# Native Investigation Artifact Contract

**Standard:** CCA-MEMORYOS-1.0  
**Artifact contract version:** 1.1  
**Status:** Normative

## Purpose

This document defines the renderer-independent native Observation, Cognitive
Trace, Cognitive Replay, Cognitive Evolution, and Comparative Reconstruction
values consumed and produced by the Investigation Core. It freezes the
observable MemoryOS 1.1 artifact behavior used by the MemoryOS 1.2 platform.
It does not standardize a renderer, camera, layout algorithm, or user
interface.

## Canonical value rules

A native artifact is a recursively immutable finite data value. Object member
order has no semantic meaning. Array order is semantic where this document
defines an order. An implementation may use any internal representation, but
its public projection MUST contain the fields and values defined here.

For identity material in this contract, canonical text is produced
recursively as follows: `null` is `null`; arrays retain order; object members
are ordered by ascending UTF-16 code-unit member name; and strings, booleans,
and finite numbers use the exact primitive serialization produced by
ECMAScript 2023 `JSON.stringify`. This includes its escaping, negative-zero,
decimal, and exponent rules. Undefined, non-finite, `BigInt`, function,
symbol, cyclic, or otherwise non-JSON values are rejected rather than omitted
or converted. The native fingerprint is the eight lower-case
hexadecimal digits of unsigned 32-bit FNV-1a over the Unicode scalar values of
that canonical text, initialized to `2166136261` and multiplied modulo
2^32 by `16777619` after each exclusive-or.

"Native lexical order" in this publication means ascending comparison of
UTF-16 code-unit sequences, with a proper prefix ordered before the longer
sequence. Every use of lexical order in this document uses that comparator.

## Observation Frame

### Published Frame envelope

A native Observation Frame has kind `MemoryOSObservationFrame`, version
`1.1`, and these members:

| Member | Contract |
|---|---|
| `sequence` | Zero-based non-negative safe integer in one Investigation timeline |
| `operation` | Non-empty explicit source operation |
| `query` | Null or detached canonical source query |
| `resultCode` | Exact `OK`; another result cannot be published as an accepted Frame |
| `snapshot` | Detached, recursively immutable source truth with the common identity envelope below and all remaining content defined by the declared native projection profile |
| `world` | Complete `MemoryOSSemanticWorld` projection described below |
| `activity` | Deterministic delta from the immediately preceding Frame, or the defined empty delta for sequence zero |

Every snapshot is an object with non-empty string root members
`workspaceIdentifier` and `observationIdentifier`. Its optional root `session`
member is absent or null for no session, or is an object with a non-empty
string `identifier`; the effective session identity is
`snapshot.session?.identifier ?? null`. All remaining snapshot members are
profile-defined source truth. The world frame's `workspaceIdentifier`,
`observationIdentifier`, and nullable `sessionIdentifier` MUST equal those
three effective snapshot identities. The Frame sequence MUST equal the world
frame index. The world frame Observation fingerprint MUST equal the native
fingerprint of the complete snapshot.

### Semantic World envelope

`MemoryOSSemanticWorld` version `1.1` contains a non-empty `identity`, a
description, an opaque stable `layout`, exact `frame` metadata, and ordered
`nodes` and `edges`. Layout and coordinates are orientation data, not cognitive
meaning; a native timeline MUST retain one exact non-empty layout identifier.

World frame metadata contains `identifier`, `index`, `fingerprint`,
`topologyFingerprint`, `observationFingerprint`, `observationIdentifier`,
`workspaceIdentifier`, nullable `sessionIdentifier`, and nullable `source`.
The exact frame-fingerprint input is:

```text
{
  frameIndex: world.frame.index,
  topologyFingerprint: world.frame.topologyFingerprint,
  observationFingerprint: world.frame.observationFingerprint,
  observationIdentifier: world.frame.observationIdentifier,
  workspaceIdentifier: world.frame.workspaceIdentifier,
  sessionIdentifier: world.frame.sessionIdentifier
}
```

It is canonicalized and fingerprinted by the native rules above. Member names
are literal. The three identity members are their exact published string or
null values; no member is omitted. `world.frame.fingerprint` is the result,
and `world.frame.identifier` is exactly
`<observationIdentifier>:<index>:<frame fingerprint>`.

Every node contains a unique non-empty stable `key`, non-empty `identifier`,
`kind`, and `family` strings, a positive finite `size`, and its producer-owned fields. A
node participating in Trace or Evolution additionally contains an exact
`observationPath` and canonical `revision`. Its effective `aggregate` and
`detail` flags are `Boolean(node.aggregate)` and `Boolean(node.detail)`; an
omitted flag therefore means false and is not repaired into a public member.
Nodes are ordered in native lexical order by key. Every edge contains known
`from` and `to` node keys and a non-empty `relation`. Edges are ordered by `from`, then `to`,
then `relation`, then complete canonical edge value. Duplicate semantic
triples receive zero-based occurrence keys
`<from>|<relation>|<to>|<occurrence>` in that order. A nullable `flowKind` may
be projected for presentation but has no relationship revision meaning.

The exact topology-fingerprint input is:

```text
{
  nodes: world.nodes.map(node => ({
    key: node.key,
    identifier: node.identifier,
    kind: node.kind,
    family: node.family,
    size: node.size,
    aggregate: Boolean(node.aggregate),
    detail: Boolean(node.detail),
    observationPath: node.observationPath ?? null,
    revision: node.revision ?? null,
    x: node.x,
    y: node.y
  })),
  edges: world.edges
}
```

The notation describes an exact data value, not executable code. `world.nodes`
and `world.edges` use their already-defined canonical order. Every node has
finite profile-published stable `x` and `y` coordinates. Each edge is its
complete published edge value, including its assigned `key`, nullable
`flowKind`, and any producer-owned members. The value is canonicalized and
fingerprinted by the native rules above; the result is
`world.frame.topologyFingerprint`. These exact member choices are retained for
artifact contract version `1.1`.

Activity contains `nodes` and `edges`, each with ordered `added`, `changed`,
and `removed` key arrays, plus `nodeKeys` and `edgeKeys`. It is the exact
canonical-value difference between consecutive published worlds by stable key;
the first Frame uses empty arrays. Each component array retains the key order
encountered in the corresponding canonical current world for `added` and
`changed`, and canonical preceding world for `removed`. `nodeKeys` is exactly
`nodes.added` followed by `nodes.changed`; `edgeKeys` is exactly `edges.added`
followed by `edges.changed`. Removed keys do not appear in either combined
array.

### Native projection profiles

A native projection profile defines the product-specific input snapshot shape,
its deterministic conversion to the published Frame and Semantic World, and
the target, authored-order, path, node-kind, endpoint, and relation bindings
needed to reconstruct an expected Trace from that Frame. Its identity and
version are assessment and producer metadata, not additional Frame members.
A conformance report MUST identify that profile when the assessment includes
raw snapshot-to-Frame projection vectors or assesses Trace reconstruction or
Trace validation from a Frame.

The Reference Implementation profile `cca-studio-native-observation` version
`1.1.0` is specified in
[native-observation-profile.md](native-observation-profile.md). Its raw
snapshot reprojection requirements apply only when that profile is declared;
the accepted Frame remains the generic native-artifact conformance seam.

Producer-side validation MUST apply the declared profile to the accepted
snapshot and require exact equality with the Frame it publishes. Core-side
self-contained validation MUST verify the envelope, canonical fingerprints,
ordering, reference closure, Workspace/session/Observation bindings, result
code, and timeline sequence. Trace construction and validation MUST apply the
declared profile's exact Frame-to-Trace bindings. A Core that also owns the
profile adapter MAY perform producer-side reprojection internally, as the
Reference Implementation does.

An independent implementation is not required to accept another product's
native projection profile. It MUST publish the profile it uses for any
Frame-to-Trace claim so that authored ordering and expected bindings are
independently evaluable. Standard conformance vectors begin either at the
declared profile input or at a complete Frame whose Trace semantics are
defined by that same declared profile. CCA-MIP-1.0 is the only portable
cross-implementation cognition package.

## Cognitive Trace

### Value shape

`MemoryOSCognitiveTrace` version `1.1` contains:

| Member | Meaning |
|---|---|
| `identifier` | `trace:` followed by percent-encoded Workspace identifier, frame identifier, and target node key, separated by `:` |
| `workspaceIdentifier` | Exact observed Workspace identity |
| `sessionIdentifier` | Exact observed session identity or null |
| `frameIdentifier` | Exact bound Observation Frame identity |
| `frameSequence` | Bound Observation Frame sequence |
| `targetNodeKey` | Exact nonaggregate Reflection node key |
| `branches` | Ordered, non-empty evidence branches |

Each branch contains its zero-based contiguous `sequence`, exact
`sourceReferencePath`, and ordered non-empty `steps`. Each step contains its
zero-based contiguous `sequence`, `role`, `nodeKey`, nullable `edgeKey`, and
nullable `direction`.

Each Trace-identifier component is encoded by ECMAScript 2023
`encodeURIComponent`: the component is encoded as UTF-8 with upper-case
hexadecimal percent triplets, while ASCII letters, ASCII digits, and
`-_.!~*'()` remain unescaped. A component containing a lone UTF-16 surrogate
is invalid and MUST be rejected rather than repaired or replaced.

### Deterministic reconstruction

The target MUST resolve to exactly one complete Reflection owned by the bound
Workspace. The Trace MUST preserve the Reflection's authored source-candidate
order. Duplicate typed source candidates are invalid. Each candidate MUST own
exactly one supported Semantic, Episodic, or Procedural source value and that
source MUST preserve one or more ordered Long-Term Memory evidence entries.

For each source candidate, the branch steps are constructed in this exact
order:

1. one `origin-evidence` step per ordered source entry, using its evidence
   relationship and `forward` direction;
2. one `semantic-transformation` step for the derived source and its source
   link, using `reverse` direction;
3. one `retrieval` step for the candidate and its contribution relationship,
   using `forward` direction; and
4. one `reflection-current` step for the Trace target, with null relationship
   and direction.

Every referenced node and relationship MUST exist exactly once in the bound
world and MUST match its expected kind, path, endpoints, and relation. Trace
validation MUST reconstruct the expected Trace from the bound frame and
require exact equality; it MUST report ordered deterministic issues without
repairing the Trace.

## Cognitive Replay

### Replay value

`MemoryOSCognitiveReplay` version `1.1` contains `identifier`,
`traceIdentifier`, `frameIdentifier`, `targetNodeKey`, and contiguous `steps`.
Its identifier is `replay:` followed by the exact Trace identifier.

The exact Replay root member set is `kind`, `version`, `identifier`,
`traceIdentifier`, `frameIdentifier`, `targetNodeKey`, and `steps`. Each step
has exactly `index`, `type`, `role`,
`nodeKey`, `edgeKey`, `direction`, and `memberships`. `index` is its zero-based
contiguous position. Each membership has exactly `branch` and `step`, both
one-based positive integers.

Replay is constructed by visiting roles in this order:

```text
origin-evidence
semantic-transformation
retrieval
reflection-current
```

For each role, Replay emits first-occurrence unique node steps in Trace branch
and step order, followed by first-occurrence unique relationship steps in the
same order. A node step has `type` `node`, its `nodeKey`, null `edgeKey`, and
`forward` direction. A relationship step has `type` `relationship`, null
`nodeKey`, its `edgeKey`, and the Trace direction. Every step preserves all
one-based `{branch, step}` memberships in Trace order. The last Replay step
MUST be the target Reflection node.

### Replay state and actions

Replay state contains `replayIdentifier`, `status`, and `cursor`. Status is
one of `ready`, `playing`, `paused`, or `completed`; initial state is
`ready` with cursor `-1`.

| Action | Deterministic result |
|---|---|
| `play` | From ready or paused, enters playing at the current step, or step 0 if cursor is -1; playing and completed are no-ops. |
| `pause` | Playing becomes paused at the same cursor; otherwise no-op. |
| `restart` | Becomes ready at cursor -1. |
| `previous` | Cursor 0 or -1 becomes ready at -1; otherwise becomes paused at cursor - 1. |
| `next` | Advances at most one step; a final cursor is completed, otherwise paused. |
| `advance` | Advances at most one step only while playing; a final cursor is completed. |

The Replay projection has exactly these thirteen members:

```text
active, identifier, status, cursor, total,
completedNodeKeys, completedEdgeKeys,
currentNodeKey, currentEdgeKey,
futureNodeKeys, futureEdgeKeys,
currentRole, currentType
```

`active` is true, `identifier` is the Replay identifier, and `total` is the
step count. Visible steps are the ordered prefix through the cursor. Unless
status is `completed`, the last visible step is current and the preceding
visible steps are completed; when status is `completed`, every visible step
is completed and no step is current. Future steps are the ordered suffix
after the cursor. The four key arrays filter their corresponding ordered
step partition without reordering or deduplication. The current node or edge
key, role, and type are taken from the current step and are null when no
current step exists. State snapshots restore only when bound to the same
Replay.

## Cognitive Evolution

### Inputs and categories

Evolution accepts two exact immutable Observation Frames from the same
Workspace, session, and stable semantic-world layout. Frame A MUST precede
Frame B. `MemoryOSCognitiveEvolution` version `1.1` represents differences in
the following closed classes:

```text
addedEvidence, removedEvidence,
addedRelationships, removedRelationships, modifiedRelationships,
addedSemanticTransformations, removedSemanticTransformations,
addedRetrievals, removedRetrievals,
addedReflections, removedReflections
```

Nonaggregate, nondetail Long-Term nodes are Evidence. Semantic, Episodic, and
Procedural family nodes are Semantic Transformations. Retrieval-session nodes
are Retrievals. Reflection nodes are Reflections. A semantic relationship is
any relationship except `contains`.

### Difference rules

Elements are matched by stable semantic key. A key present only in B is
added; a key present only in A is removed. A node whose stable key persists
but whose exact `revision ?? null` value changes appears as removal of the A
revision and addition of the B revision. Native profiles MUST publish a node
revision as null or canonical text, making this an exact value comparison. A
relationship's Evolution revision material is the complete edge after
removing `key` and `flowKind`; its canonical text is the exact revision. A
relationship whose key persists but whose revision text changes appears once
in `modifiedRelationships` with both revision fingerprints. `flowKind` is
presentation-only and excluded from relationship semantics.

Every difference array MUST be ordered in native lexical order by stable key.
Unchanged node and relationship key arrays MUST use the same order. References
preserve exact identity, kind, family, frame identity, and revision fingerprint.
The exact Evolution-identifier fingerprint input is:

```text
{
  workspaceIdentifier: A.snapshot.workspaceIdentifier,
  fromFrameIdentifier: A.world.frame.identifier,
  toFrameIdentifier: B.world.frame.identifier,
  differences: evolution.differences
}
```

The `differences` value has exactly the eleven arrays and exact reference
shapes defined below; its arrays already have their defined native lexical
order. The value is canonicalized and fingerprinted by the native rules
above. The Evolution identifier is exactly
`evolution:<A sequence>:<B sequence>:<fingerprint>`.

An added or removed node reference has exactly `key`, `identifier`, `kind`,
`family`, `frameIdentifier`, and `revisionFingerprint`. The revision
fingerprint is the native fingerprint of the node's exact `revision ?? null`
value. An added or removed relationship reference has exactly `key`, `from`,
`to`, `relation`, `frameIdentifier`, and `revisionFingerprint`; that
fingerprint is the native fingerprint of the canonical-text string defined as
the relationship's Evolution revision above. A modified relationship has
exactly `key`, `from`, `to`, `relation`, `fromFrameIdentifier`,
`toFrameIdentifier`, `beforeRevisionFingerprint`, and
`afterRevisionFingerprint`; its endpoint and relation values are B's values
and both fingerprints use the same canonical-text-string rule.

The Evolution root has exactly these members:

```text
kind, version, identifier, workspaceIdentifier, sessionIdentifier,
from, to, differences, summary, unchanged, view, world
```

`from` and `to` each have exactly `sequence`, `frameIdentifier`, and
`observationIdentifier`. `differences` has exactly the eleven closed arrays
listed above. `summary` has exactly the same eleven member names, with each
value equal to the corresponding array length. `unchanged` has exactly
`nodeKeys` and `relationshipKeys`.

The immutable Evolution view has exactly these eleven members:

```text
kind, version, identifier,
addedNodeKeys, removedNodeKeys, evolvedNodeKeys, unchangedNodeKeys,
addedRelationshipKeys, removedRelationshipKeys,
modifiedRelationshipKeys, unchangedRelationshipKeys
```

Its kind is `MemoryOSCognitiveEvolutionView`, version is `1.1`, and identifier
is the Evolution identifier. `evolvedNodeKeys` contains a shared stable key
whose node revision changed; the remaining arrays are the exact key
projections of the comparison described above.

The stable union world has kind `MemoryOSCognitiveEvolutionWorld`, version
`1.1`, identity `Cognitive evolution`, and description
`A deterministic semantic union of two observed cognitive states.` Its
`layout` is exactly B's layout. Its frame preserves all of B's frame members,
replaces `identifier` with the Evolution identifier, and adds `comparison`
with exactly `fromFrameIdentifier` and `toFrameIdentifier`. Its ordered nodes
and edges are the respective stable-key unions, with B's complete value
winning for a shared key. That world preserves the existing stable semantic
geography and is not a second semantic comparison.

## Comparative Reconstruction

### Construction and alignment

Comparative Reconstruction requires two valid bound Traces over ordered
Observation Frames from the same Workspace, session, and stable layout. It
builds each Trace's canonical Replay, resolves every Replay node or
relationship to its exact frame revision, and aligns the two step sequences
by identity `node:<key>` or `relationship:<key>`. For a node, the exact frame
revision is the canonical text of `node.revision ?? null`. For a relationship,
it is the canonical text of the complete edge after removing only `flowKind`;
the edge `key` therefore remains. The step-reference `revisionFingerprint` is
the native fingerprint of that exact canonical-text string. These rules are
distinct from Evolution relationship revision material and are retained for
artifact contract version `1.1`.

Alignment uses a longest common subsequence, matches equal identities
immediately, and consumes the A-only step first on equal-cost ties. The
resulting moments are zero-based and contiguous. Each moment has state
`shared`, `a-only`, `b-only`, or `modified`; `divergent` is false exactly for
`shared` and true otherwise.

A shared identity is `modified` when its exact semantic revision differs or
its Trace role or traversal direction differs. Closed reason codes are
`A_ONLY`, `B_ONLY`, `RELATIONSHIP_REVISION_CHANGED`,
`SEMANTIC_REVISION_CHANGED`, and `TRACE_BINDING_CHANGED`, emitted in the
order just implied by those checks. A shared moment has no reason code.

Each nullable A/B step reference has exactly `replayIndex`, `elementType`,
`role`, `key`, `nodeKey`, `edgeKey`, `direction`, and
`revisionFingerprint`. Every moment has exactly `index`, `state`,
`divergent`, `role`, `elementType`, `reasonCodes`, `reason`, `from`, and `to`.
Its `role` and `elementType` come from the present side, and from B when both
sides are present. The exact reason strings are:

| Classification | `reason` |
|---|---|
| `b-only` | `Observation B contains this exact semantic step; Observation A does not.` |
| `a-only` | `Observation A contains this exact semantic step; Observation B does not.` |
| `shared` | `Both observations contain the same exact semantic step.` |
| `RELATIONSHIP_REVISION_CHANGED` | `The same relationship identity has a different exact runtime revision.` |
| `SEMANTIC_REVISION_CHANGED` | `The same semantic identity has a different exact runtime revision.` |
| `TRACE_BINDING_CHANGED` | `The same semantic identity has a different trace role or traversal direction.` |

For a modified moment, the reason-code phrases are concatenated in reason-code
order with one ASCII space. A-only and B-only moments carry their one matching
reason code; shared moments carry an empty reason-code array.

### Value and identity

`MemoryOSComparativeReconstruction` version `1.1` has exactly these members:

```text
kind, version, identifier, workspaceIdentifier, sessionIdentifier,
evolutionIdentifier, worldIdentifier, from, to, moments,
divergenceIndices, firstDivergenceIndex, summary, world
```

`from` and `to` each have exactly `frameSequence`, `frameIdentifier`,
`traceIdentifier`, `replayIdentifier`, and `targetNodeKey`. `moments` contains
the exact moment shape above. `divergenceIndices` is the ordered list of
divergent moment indices and `firstDivergenceIndex` is its first value or
null. `evolutionIdentifier`, `worldIdentifier`, and `world.frame.identifier`
are equal, and `world` is the single Evolution world.

The summary has exactly these eleven members:

```text
moments, shared, divergent, aOnly, bOnly, modified,
evidence, semanticTransformations, retrievals, reflections, relationships
```

The first six count all moments and their states. `relationships` counts only
divergent relationship moments. The other four category counts include only
divergent node moments with roles `origin-evidence`,
`semantic-transformation`, `retrieval`, and `reflection-current`,
respectively.

The exact Comparative Reconstruction identifier-fingerprint input is:

```text
{
  workspaceIdentifier: A.trace.workspaceIdentifier,
  fromFrameIdentifier: A.frame.world.frame.identifier,
  fromTraceIdentifier: A.trace.identifier,
  toFrameIdentifier: B.frame.world.frame.identifier,
  toTraceIdentifier: B.trace.identifier,
  moments: reconstruction.moments.map(moment => ({
    state: moment.state,
    reasonCodes: moment.reasonCodes,
    from: moment.from,
    to: moment.to
  }))
}
```

The notation describes an exact data value, not executable code. Each nullable
`from` or `to` value is null or the exact eight-member step reference defined
above; no moment index, reason string, role, element type, or divergence flag
participates independently. The ordered `reasonCodes`, ordered moments, and
all nulls are retained. The value is canonicalized and fingerprinted by the
native rules above. The identifier is exactly
`comparative-reconstruction:<A sequence>:<B sequence>:<fingerprint>`. The
construction operation guarantees the exact identifier, aligned moments,
source-bound revisions, summary, and world described above.

The detached Comparative validator has a narrower structural scope. It
requires the published kind and version; requires the reconstruction root,
its `moments` array, and its `world` value themselves to be frozen; and checks
a non-empty contiguous moment sequence, state/divergence agreement, one bound
world, reference closure into that world, shared-side identity rules, and
exact divergence indices and first-divergence value. It does not recursively
test every nested member for immutability. Without the source Frames and
Traces, that detached validator does not independently reconstruct alignment,
recompute content identity or summaries, or prove canonical construction.

### Comparative Replay

Comparative Replay state contains `reconstructionIdentifier`, status, cursor,
and nullable `pauseReason`. Status is `ready`, `playing`, `paused`, or
`completed`; pause reason is null, `divergence`, or `engineer`; initial state
is ready at cursor -1.

The six actions have these exact results:

| Action | Deterministic result |
|---|---|
| `play` | Playing and completed are no-ops. From ready, visits moment 0: a divergent moment becomes paused with reason `divergence`; otherwise a one-moment reconstruction completes and a longer reconstruction plays. From paused, becomes playing at the same cursor and clears the pause reason, including at the final cursor. |
| `pause` | Playing becomes paused at the same cursor with reason `engineer`; otherwise no-op. |
| `reset` | Becomes ready at cursor -1 with null reason. |
| `previousStep` | Cursor 0 or -1 becomes ready at -1. Otherwise visits cursor - 1: a divergent moment pauses with reason `divergence`; a nondivergent moment pauses with reason `engineer`. |
| `nextStep` | Completed is a no-op. At the final cursor, becomes completed there. Otherwise visits cursor + 1: a divergent moment pauses with reason `divergence`; a nondivergent moment pauses with reason `engineer`. |
| `advance` | Non-playing is a no-op. At the final cursor, becomes completed there. Otherwise visits cursor + 1: a divergent moment pauses with reason `divergence`; a nondivergent moment remains playing. |

Arrival at a divergent moment therefore pauses deterministically. Playback
proceeds beyond a divergence only after an explicit engineer action. Reaching
the final moment and completing are distinct steps except for `play` on a
single nondivergent moment.
The immutable `MemoryOSComparativeReconstructionView` version `1.1` projects
the reconstruction and has exactly these eighteen members:

```text
kind, version, identifier, worldIdentifier, active, status, pauseReason,
cursor, total, firstDivergenceIndex, divergenceIndices, divergenceCount,
currentMoment, atDivergence, moments, records, nodeRecords,
relationshipRecords
```

`active` is true. `currentMoment` is null while ready and when completed;
otherwise it is the moment at the cursor. `atDivergence` is true exactly when
that current moment is divergent. `moments`, divergence values, total, and
world binding are the exact Reconstruction values.

For every non-null moment side, projection creates one occurrence with
exactly `key`, `elementType`, `role`, `side`, `semanticState`, `phase`, and
`momentIndex`. Side is `a` for `from` and `b` for `to`. Phase is `completed`
when Replay is completed or the moment precedes the cursor, `current` at the
cursor, and `future` otherwise. Occurrences are visited by moment index, A
before B, and grouped by `<elementType>:<key>`.

Each record has exactly `key`, `elementType`, `roles`, `sides`,
`semanticState`, `phase`, `momentIndices`, `occurrences`, and `sideRecords`.
Records are ordered by native lexical order of `<elementType>:<key>`.
`roles` is the unique native-lexically ordered set across occurrences.
`momentIndices` is the unique numerically ascending set. For `sides`,
`semanticState`, and `phase`, current occurrences take precedence when any
exist; otherwise all occurrences are used. Sides are unique and
native-lexically ordered; a single shared value is retained, while multiple
semantic states or phases yield `split`. Each side record has exactly `side`,
`roles`, `sides`, `semanticState`, `phase`, `momentIndices`, and
`occurrences`, applying the same rules to that side. Side records are emitted
in A-then-B order when present. `nodeRecords` and `relationshipRecords`
preserve record order while filtering by element type. The view does not
compute or alter the reconstruction.

## Normative requirements

### CCA-MOS-ART-001 — Observation Frame publication and binding

A native producer **MUST** publish only an immutable canonical
`MemoryOSObservationFrame` version `1.1` with the defined envelope and exact
sequence, Workspace, session, Observation, fingerprint, snapshot, world, and
profile bindings. When an assessment includes an implementation-specific raw
Observation projection or Frame-to-Trace semantics, its conformance evidence
**MUST** identify that native projection profile.

### CCA-MOS-ART-002 — Trace identity and shape

A native Cognitive Trace **MUST** use kind `MemoryOSCognitiveTrace`, version
`1.1`, the defined identity, exact frame and Workspace binding, and ordered
non-empty branch and step shapes.

### CCA-MOS-ART-003 — Trace reconstruction

Trace reconstruction **MUST** apply the declared native projection profile,
resolve one exact complete Reflection, and preserve authored candidate and
evidence order using the defined four-role branch sequence and exact world
relationships.

### CCA-MOS-ART-004 — Trace validation

Trace validation **MUST** apply the declared native projection profile to
deterministically reconstruct the expected Trace, require exact equality and
reference closure, and reject incomplete, ambiguous, cross-Workspace,
inconsistent, or mutable input without repair.

### CCA-MOS-ART-005 — Replay construction

A native Replay **MUST** use kind `MemoryOSCognitiveReplay`, version `1.1`,
the defined Trace-derived identity, role order, first-occurrence node-then-edge
ordering, exact root and step members, memberships, and terminal target.

### CCA-MOS-ART-006 — Replay state machine

Replay **MUST** expose only the defined statuses, cursor, six actions, no-op
behavior, and exact thirteen-member deterministic completed/current/future
projection.

### CCA-MOS-ART-007 — Evolution input binding

Evolution **MUST** compare only two ordered immutable Observation Frames from
one Workspace, session, and stable semantic geography.

### CCA-MOS-ART-008 — Evolution difference classes

Evolution **MUST** expose exactly the defined eleven difference classes and
classify only nonaggregate, nondetail Long-Term nodes as Evidence and the
defined Semantic Transformation, Retrieval, Reflection, and non-`contains`
relationship values.

### CCA-MOS-ART-009 — Evolution comparison and order

Evolution **MUST** compare stable semantic keys and exact revisions as
defined, exclude `flowKind` from relationship semantics, and order every
difference and unchanged-key array in native lexical order.

### CCA-MOS-ART-010 — Evolution identity and world

A native Evolution **MUST** use kind `MemoryOSCognitiveEvolution`, version
`1.1`, its exact published member and reference shapes, defined content
identity, exact frame references, difference counts and view, and one
deterministic union world in which the later revision wins.

### CCA-MOS-ART-011 — Comparative input binding

Comparative Reconstruction **MUST** accept only two valid bound Traces over
ordered frames from one Workspace, session, and stable semantic geography.

### CCA-MOS-ART-012 — Comparative alignment

Comparative Reconstruction **MUST** align canonical Replay steps by exact
element identity using deterministic longest-common-subsequence alignment and
the A-first tie rule.

### CCA-MOS-ART-013 — Comparative divergence

Comparative moments **MUST** use only the defined states and reason codes,
preserve the exact moment, A/B reference, and reason-string shapes, and
classify changed revision, role, and direction exactly as defined.

### CCA-MOS-ART-014 — Comparative value and validation

A native Comparative Reconstruction **MUST** use kind
`MemoryOSComparativeReconstruction`, version `1.1`, its defined content
identity, exact published members, one stable world, exact summaries, and
exact ordered divergence indices when constructed from source Frames and
Traces. Its detached validator
**MUST** enforce the structural, exact frozen-surface, world-binding,
reference, shared-moment, and divergence-index checks defined above without
claiming to reconstruct source-dependent guarantees from a detached value
alone.

### CCA-MOS-ART-015 — Comparative Replay

Comparative Replay **MUST** expose only the defined state and action contract,
pause deterministically on divergence, and expose the exact immutable View
and record projection rather than compute the underlying reconstruction.

### CCA-MOS-ART-016 — Artifact independence and atomicity

Native artifact derivation and validation **MUST** be independent of renderer,
camera, pixels, clock, randomness, and mutable presentation state, and failure
**MUST** publish no partial artifact or state change.

## Conformance

Evidence group `MOS-EVID-ART-001` covers canonical frame binding; declared
native-profile reprojection where applicable; exact Trace, Replay, Evolution,
Comparative Reconstruction, and Comparative Replay vectors; identity and
ordering; construction guarantees separately from detached validation scope;
divergence behavior; immutability; renderer independence; and atomic negative
cases.
