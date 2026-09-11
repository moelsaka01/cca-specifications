# CCA Studio Native Observation Profile

**Standard:** CCA-MEMORYOS-1.0  
**Profile identifier:** `cca-studio-native-observation`  
**Profile version:** `1.1.0`  
**Status:** Normative for an assessment that declares this profile

## Scope

This document publishes the existing CCA Studio native snapshot-to-Frame
profile used by the Reference Implementation. It defines the source material
and exact projection needed to reconstruct a Cognitive Trace from a CCA
Studio Observation. It does not make a CCA Studio snapshot portable. A
conforming implementation MAY declare another native profile; CCA-MIP-1.0
remains the only portable cross-implementation cognition package.

The profile-specific projection clauses in `CCA-MOS-ART-001`,
`CCA-MOS-ART-003`, and `CCA-MOS-ART-004` apply whenever an assessment declares
the profile identifier and version above. `CCA-MOS-SDK-005` applies to this
profile only when the assessment also selects the SDK scoped profile or the
complete-platform scope. A Native Investigation Artifacts assessment that
tests Trace reconstruction or Trace validation from an already published
Frame MUST still declare the profile whose authored-order, path, endpoint,
kind, and relation bindings make that reconstruction independently evaluable;
it need not repeat raw snapshot-to-Frame vectors unless it also claims that
part of the profile.

## Accepted snapshot envelope

The input is detached canonical JSON and has exactly these top-level members:

```text
consolidationSessions  contract            episodicMemory
longTermMemory        memory              observationIdentifier
proceduralMemory      providerSessions    reflectionSessions
reflections           result              retrievalSessions
semanticMemory        session             source
validation            workingMemory       workspaceIdentifier
```

`contract` is exactly `CCA-STUDIO-1.0`.
`workspaceIdentifier`, `observationIdentifier`, `source`, and
`session.identifier` are non-empty strings. Foreign top-level members are
invalid. The six memory aggregates are objects containing their released
arrays; `retrievalSessions`, `consolidationSessions`, `reflections`,
`reflectionSessions`, `providerSessions`, and `validation` are arrays.
Retrieval sessions contain `candidates`; provider sessions contain
`descriptors`; Semantic concepts contain `categories`, `sourceEntries`, and
`linkedConceptIdentifiers`; Episodes contain `sourceEntries` and
`linkedEpisodeIdentifiers`; Procedures contain `steps`, `sourceEntries`, and
`linkedProcedureIdentifiers`. These values retain authored array order.

Members not consumed by the Frame-to-Trace rules remain exact source truth:
they participate in snapshot canonicalization, Observation fingerprinting,
and published node revisions, but this profile does not reinterpret them.
Specifically, nested `result` content and `session.state` are preserved source
data and are not acceptance gates. Only the explicit outer Frame operation's
`resultCode` governs publication, and it must be `OK`.

Acceptance canonical-clones the snapshot, rejects unsupported or
non-canonical JSON data, detaches it from the caller, and recursively freezes
the accepted copy. Frame publication then applies the canonical native value,
fingerprint, world, and timeline rules in
[native-artifacts.md](native-artifacts.md).

## Reflection forms

A traceable Reflection uses one of exactly two source paths. In the grammar
below, `I`, `J`, and `K` are canonical decimal non-negative safe integers with
no leading zero unless the value is zero.

| Form | Snapshot location | Target `observationPath` |
|---|---|---|
| Standalone | `reflections[I]` | `Reflection.values[I]` |
| Derived session result | `reflectionSessions[I].reflection` | `Reflection.sessions[I].reflection` |

A traceable Reflection has a non-empty `identifier`, the exact snapshot
`workspaceIdentifier`, non-empty `knowledge`, and a non-empty authored
`sources` array.

For a derived session result, `reflectionSessions[I]` has the exact Workspace,
state `Derived`, a non-null `query`, an authored `sources` array, and a
non-null `reflection`. The query has the exact Workspace and non-empty
`identifier` and `knowledge`. Query identifier and knowledge equal the
Reflection identifier and knowledge. Session sources and Reflection sources
are canonically equal arrays in the same authored order.

## Reflection source variants

Each `sources[J]` value has:

- `kind` exactly `Semantic`, `Episodic`, or `Procedural`;
- the exact snapshot `workspaceIdentifier`;
- a non-empty `sourceIdentifier` distinct from the Reflection identifier;
- `rankScore` as a safe integer from 0 through `4294967295`;
- a non-empty `chain` array containing only non-empty strings; and
- exactly one populated typed value selected by this table; the other two
  typed members are null or absent.

| `kind` | Populated member | Source node kind | Source node family |
|---|---|---|---|
| `Semantic` | `semanticConcept` | `semantic` | `Semantic concept` |
| `Episodic` | `episode` | `episodic` | `Episode` |
| `Procedural` | `procedure` | `procedural` | `Procedure` |

The populated typed value is an object whose non-empty `identifier` equals
`sourceIdentifier`. It contains a non-empty authored `sourceEntries` array.
Every entry is an object with a non-empty `identifier`. Multiple candidates
with the same `(kind, sourceIdentifier)` pair are ambiguous and invalid.

Candidate order is the authored `Reflection.sources` order. Evidence order is
the selected typed value's authored `sourceEntries` order. `rankScore` and
`chain` are validated source truth; they do not reorder either sequence.

## Path grammar

For target path `T` and source index `J`, the exact paths are:

```text
sourceReferencePath = T + ".sources[" + J + "]"

Semantic source value   = sourceReferencePath + ".semanticConcept"
Episodic source value   = sourceReferencePath + ".episode"
Procedural source value = sourceReferencePath + ".procedure"

evidence K = source value path + ".sourceEntries[" + K + "]"
```

The Trace branch `sourceReferencePath` is exactly the first line above.
Paths are case-sensitive and contain no escaping, aliases, or inferred
segments.

## Stable key construction

`E(x)` below is ECMAScript 2023 `encodeURIComponent(String(x))`. It encodes
the UTF-8 bytes of the input with upper-case hexadecimal percent triplets and
leaves only ASCII letters, ASCII digits, and `-_.!~*'()` unescaped. A lone
UTF-16 surrogate is invalid and causes projection failure.

For an ordered input sequence and an identity, occurrence is the zero-based
count of earlier values in that sequence with the same identity.

| Value | Exact stable key |
|---|---|
| Standalone Reflection | `reflection:E(identifier):occurrence` |
| Reflection session | `reflection-session:E(identifier):occurrence` |
| Session Reflection | `<session-key>:reflection:E(reflection.identifier):0` |
| Source candidate | `<target-key>:source-candidate:E(kind + ":" + sourceIdentifier):occurrence` |
| Typed source | `<candidate-key>:source:E(source.identifier):0` |
| Evidence entry | `<source-key>:evidence:E(entry.identifier):occurrence` |

Other released domain members use the same
`<domain-prefix>:E(identity):occurrence` construction. A Retrieval candidate's
identity is its complete canonical value. A Semantic category or Procedure
step uses its authored scalar value. Stable-key collisions are invalid.

In the complete projection below, `C(value)` is the native canonical text from
[native-artifacts.md](native-artifacts.md), not a second JSON value. Unless a
row says `aggregate` or `detail` is true, that member is absent and its
effective flag is false.

## Complete graph projection

The graph identity is exactly `Memory intelligence graph`; its description is
exactly `One deterministic topology of MemoryOS state, provenance, retrieval,
reflection, providers, and validation.`. It begins with this Workspace node:

```text
{key: "workspace", identifier: snapshot.workspaceIdentifier,
 label: "Workspace", kind: "workspace", family: "Workspace", size: 21,
 aggregate: true,
 revision: C({workspaceIdentifier: snapshot.workspaceIdentifier})}
```

It then creates the following aggregate nodes. Each key is
`aggregate:<kind>`, identifier is `capability-<kind>`, size is 14,
`aggregate` is true, and revision is `C(source value)`. Each receives one
`workspace -> aggregate` relationship with relation `contains`.

| Kind | Label | Family | Source value |
|---|---|---|---|
| `memory` | `Memory` | `Memory aggregate` | `snapshot.memory` |
| `working` | `Working` | `WorkingMemory aggregate` | `snapshot.workingMemory` |
| `consolidation` | `Consolidation` | `Consolidation aggregate` | `snapshot.consolidationSessions` |
| `long-term` | `Long-Term` | `LongTermMemory aggregate` | `snapshot.longTermMemory` |
| `semantic` | `Semantic` | `SemanticMemory aggregate` | `snapshot.semanticMemory` |
| `episodic` | `Episodic` | `EpisodicMemory aggregate` | `snapshot.episodicMemory` |
| `procedural` | `Procedural` | `ProceduralMemory aggregate` | `snapshot.proceduralMemory` |
| `retrieval` | `Retrieval` | `Retrieval aggregate` | `snapshot.retrievalSessions` |
| `reflection` | `Reflection` | `Reflection aggregate` | `[snapshot.reflections, snapshot.reflectionSessions]` |
| `providers` | `Providers` | `Providers aggregate` | `snapshot.providerSessions` |
| `validation` | `Validation` | `Validation aggregate` | `snapshot.validation` |

Every released member row below is projected in authored array order. Its key
uses the row's prefix and the zero-based occurrence of the source
`identifier`; label and identifier are that source identifier; revision is
`C(complete source value)`; and its aggregate receives one `contains`
relationship to it.

| Source array | Key prefix | Kind | Family | Size | `observationPath` |
|---|---|---|---|---:|---|
| `memory.entries` | `memory` | `memory` | `Memory` | 5.5 | `Memory.entries[I]` |
| `workingMemory.entries` | `working` | `working` | `WorkingMemory` | 6 | `WorkingMemory.entries[I]` |
| `consolidationSessions` | `consolidation` | `consolidation` | `Consolidation session` | 7 | `Consolidation.sessions[I]` |
| `longTermMemory.entries` | `long-term` | `long-term` | `LongTermMemory` | 6 | `LongTermMemory.entries[I]` |
| `semanticMemory.concepts` | `semantic` | `semantic` | `SemanticMemory` | 8 | `SemanticMemory.concepts[I]` |
| `episodicMemory.episodes` | `episodic` | `episodic` | `EpisodicMemory` | 8 | `EpisodicMemory.episodes[I]` |
| `proceduralMemory.procedures` | `procedural` | `procedural` | `ProceduralMemory` | 8 | `ProceduralMemory.procedures[I]` |
| `retrievalSessions` | `retrieval` | `retrieval` | `Retrieval session` | 7 | `Retrieval.sessions[I]` |
| `reflections` | `reflection` | `reflection` | `Reflection` | 11 | `Reflection.values[I]` |
| `reflectionSessions` | `reflection-session` | `reflection` | `Reflection session` | 7 | `Reflection.sessions[I]` |
| `providerSessions` | `providers` | `providers` | `Provider session` | 7 | `Providers.sessions[I]` |

The remaining detail nodes are exact:

- When `workingMemory.activeTaskIdentifier` is truthy, create key
  `working-task:E(activeTaskIdentifier):0`, source identifier and label,
  kind `working`, family `Working task`, size 4, `detail` true, revision
  `C({identifier: activeTaskIdentifier, active: workingMemory.active})`, and
  one Working-aggregate `contains` relationship. It has no observation path.
- Each `validation[I]` creates key
  `validation:E(check.identifier):occurrence`, identifier `check.identifier`,
  label `check.label`, kind `validation`, family `Validation check`, size 5,
  revision `C(check)`, and one Validation-aggregate `contains` relationship.
  It has no observation path.
- Each `semanticMemory.concepts[I].categories[J]` creates key
  `<concept-key>:category:E(category):occurrence`, concept identifier, category
  label, kind `semantic`, family `SemanticMemory`, size 3, `detail` true,
  path `SemanticMemory.concepts[I].categories[J]`, revision `C(category)`, and
  `contains` relationships from both the Semantic aggregate and concept.
- Each `proceduralMemory.procedures[I].steps[J]` is the analogous
  `<procedure-key>:step:E(step):occurrence` node, with procedure identifier,
  step label, kind `procedural`, family `ProceduralMemory`, size 3, `detail`
  true, path `ProceduralMemory.procedures[I].steps[J]`, revision `C(step)`,
  and `contains` relationships from the Procedural aggregate and procedure.
- Each `retrievalSessions[I].candidates[J]` creates key
  `<session-key>:candidate:E(C(candidate)):occurrence`, identifier and label
  `candidate.sourceIdentifier`, kind `retrieval`, family
  `Retrieval candidate`, size 4, `detail` true, path
  `Retrieval.sessions[I].candidates[J]`, revision `C(candidate)`, and
  `contains` relationships from both the Retrieval aggregate and session.
  It also links to the first released source whose identifier equals
  `sourceIdentifier` in the domain selected by kind `Memory`, `WorkingMemory`,
  `LongTermMemory`, `Semantic`, `Episodic`, or `Procedural`, when one exists.
- Each `providerSessions[I].descriptors[J]` creates key
  `<session-key>:descriptor:E(descriptor.identifier):occurrence`, descriptor
  identifier and label, kind `providers`, family `Provider descriptor`, size
  4, `detail` true, path `Providers.sessions[I].descriptors[J]`, revision
  `C(descriptor)`, and `contains` relationships from both the Providers
  aggregate and session.

Domain evidence and association relationships preserve authored source order:

- each Semantic concept, Episode, or Procedure receives `evidence` from the
  first Long-Term entry with the identifier of each authored `sourceEntries`
  member, when one exists;
- every occurrence of a linked concept, Episode, or Procedure identifier
  receives `links` from the source artifact; and
- each Consolidation session receives `contributes` from the first Working
  entry matching `request.entryIdentifier` and emits `links` to the first
  Long-Term entry matching `candidate.longTermMemoryIdentifier`, when the
  respective nested object and match exist.

The Reflection branch adds the nodes and relationships in the next section.
Every Reflection source-candidate, typed-source, and evidence-detail node
receives `contains` from the Reflection aggregate. A session Reflection target
receives `contains` from both the Reflection aggregate and its nonaggregate
Reflection-session node.

After graph construction, nodes are ordered by native lexical key. Edges are
ordered by `from`, `to`, `relation`, then `C(complete pre-key edge)`. Edge keys
and occurrence are assigned as defined below. `flowKind` is then exactly:
`evidence` for relation `evidence`; `reflection` for `contributes` into a
Reflection node; `retrieval` when either endpoint is Retrieval and the edge is
not aggregate containment; `consolidation` for `contributes` involving
Consolidation; `association` for `links`; otherwise null.

## Exact semantic layout

The public layout is exactly:

```text
identifier = "memoryos-semantic-world-v1"
strategy   = "deterministic-semantic-anchors"
viewBox    = {width: 960, height: 540, centerX: 480, centerY: 270}
```

Anchors are: Workspace `(470,265)`, Memory `(310,295)`, Validation
`(300,225)`, Long-Term `(395,195)`, Semantic `(500,175)`, Episodic
`(600,205)`, Procedural `(650,280)`, Retrieval `(565,275)`, Reflection
`(535,340)`, Providers `(485,390)`, Consolidation `(420,335)`, and Working
`(350,345)`. The layout's `anchors` object uses the corresponding lower-case
kind keys, with `long-term` hyphenated.

Workspace and aggregate nodes use their anchors. Every other node not
contained by a nonaggregate parent is positioned around its kind anchor with
radius range 32 through 47 for detail nodes or 40 through 61 otherwise, and
vertical scale 0.78. A node contained by a nonaggregate parent is positioned
around that parent with radius 23 through 36 and vertical scale 0.82. An
unresolvable nested node falls back around world center with radius 170
through 215 and vertical scale 0.72.

For anchor `(ax, ay)`, key `k`, radius bounds `lo, hi`, and vertical scale
`v`, ECMAScript 2023 Number and `Math` semantics determine:

```text
outward = atan2(ay - 270, ax - 480)
angle   = outward + (F(k + ":angle") / 0xffffffff) * 2 * PI
radius  = lo + (hi - lo) * (F(k + ":radius") / 0xffffffff)
x       = clamp(ax + cos(angle) * radius, 110, 850)
y       = clamp(ay + sin(angle) * radius * v, 145, 435)
```

`F` is the unsigned 32-bit FNV-1a operation defined in
[native-artifacts.md](native-artifacts.md). Each coordinate is converted by
ECMAScript `Number(value.toFixed(2))`. Pending nonaggregate-parent children
are first sorted in ascending native lexical key order, scanned from the last
entry to the first in each pass, and removed when placed; passes repeat until
all resolvable descendants are placed or a pass places none.

## Trace subgraph projection

The profile publishes these exact nodes for each traceable Reflection branch:

| Snapshot value | `kind` | `family` | `size` | `detail` | Revision |
|---|---|---|---:|---|---|
| Reflection target | `reflection` | `Reflection` | 11 | false | canonical complete Reflection |
| Source candidate | `retrieval` | `Reflection source` | 4 | true | canonical complete candidate |
| Typed source | Table above | Table above | 7 | true | canonical complete typed value |
| Evidence entry | `long-term` | `Provenance snapshot` | 5 | true | canonical complete entry |

Each node uses the exact stable key and `observationPath` defined above. A
standalone target is a member of aggregate `reflection`; a session target is
contained by its `Reflection session` node. The profile publishes these exact
Trace relationships:

| From | To | `relation` | Relationship `observationPath` |
|---|---|---|---|
| Source candidate | Reflection target | `contributes` | source reference path |
| Source candidate | Typed source | `links` | typed source path |
| Evidence entry | Typed source | `evidence` | evidence path |

The semantic world orders relationships by `from`, `to`, `relation`, then
complete canonical edge value. It assigns each edge key
`<from>|<relation>|<to>|<occurrence>`, where occurrence counts earlier equal
semantic triples. Trace construction requires exactly one matching node for
each path and exactly one matching relationship for each required endpoint
and relation.

## Frame production and Trace resolution

For accepted snapshot `S` at timeline index `N`, the profile:

1. builds the complete graph and relationships defined above in authored
   domain order;
2. canonicalizes and places it using the exact layout above;
3. publishes one Frame with `sequence` `N`, the exact operation and query,
   result code `OK`, detached snapshot `S`, the projected world, and the
   deterministic delta from Frame `N-1`; `world.frame.source` is exactly
   `S.source`, and that value is identity-bound indirectly through the
   complete-snapshot `observationFingerprint`; and
4. resolves a Trace only from an exact nonaggregate node with kind
   `reflection`, family `Reflection`, and one of the two target paths above.

A selected Reflection aggregate or Reflection-session node may resolve to a
target only when exactly one directly contained node satisfies the applicable
target path. Resolution never selects among multiple candidate targets.

The Trace then creates one branch per authored source and one
`origin-evidence` step per authored source entry, followed by the exact
transformation, retrieval, and current-Reflection steps defined in
[native-artifacts.md](native-artifacts.md).

## Failure boundary

Snapshot-envelope, canonicalization, graph, Frame, or profile projection
failure is `INVALID_INPUT` at the Core `create` or `observe` boundary and
publishes no transition. A non-`OK` outer Frame `resultCode` cannot be
published; similarly named values inside the preserved snapshot do not alter
that gate.
Timeline Workspace or session changes are rejected atomically.

Trace querying uses the closed outcomes `INVALID_QUERY`,
`WORKSPACE_MISMATCH`, `SESSION_MISMATCH`, `FRAME_MISMATCH`, `NOT_FOUND`,
`INVALID_TARGET`, `MISSING_EVIDENCE`, `AMBIGUOUS_EVIDENCE`,
`INCONSISTENT_EVIDENCE`, `INCONSISTENT_RELATIONSHIP`, and `INVALID_TRACE`.
The Core preserves the applicable artifact code and message exactly and never
publishes a partial Trace or Replay. A language binding that receives the
delegated artifact error applies its documented deterministic profile/binding
mapping; cross-binding diagnostic equality is not required for those
binding-level error objects, but atomicity is unchanged.

## Conformance evidence

Complete profile conformance evidence supplies raw snapshot-to-Frame vectors,
both Reflection forms, all three source variants, repeated source-entry
identities, path/key/edge vectors, authored-order vectors, canonicalization
and `encodeURIComponent` negatives, Workspace/session mismatches, and every
failure class above. Evidence that begins with a complete accepted Frame may
omit the raw reprojection vectors, but it remains bound to this profile for
the exact Frame-to-Trace semantics it assesses.
