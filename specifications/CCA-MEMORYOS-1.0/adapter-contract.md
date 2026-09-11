# AI Runtime Adapter Contract

**Standard:** CCA-MEMORYOS-1.0  
**Contract version:** 1.0.0  
**Status:** Normative

## Purpose

This document defines the provider-independent boundary that translates a
successfully settled external AI Runtime result into a deterministic,
Observation-only MIP. It does not standardize a provider SDK, transport,
authentication method, model, agent, tool, or network operation.

## Informative model

```text
provider events -> lifecycle validation -> settled source projection
                -> CCA-MIP-1.0 Producer -> independent verification
```

Provider event streams are private translation input. The portable output is
the verified package, not the transport transcript.

## Public adapter value

A conforming adapter publishes contract version `1.0.0`, producer identity
`{name: "MemoryOS AI Runtime Adapters", version: "1.0.0"}`, an immutable
descriptor, and the literal source-authorship attestation `true`.

The descriptor is a closed value with exactly these non-empty string members:

| Member | Contract |
|---|---|
| `identifier` | Matches `^[a-z0-9](?:[a-z0-9.-]*[a-z0-9])?$` |
| `name` | Provider-neutral display identity |
| `version` | Adapter implementation version |
| `sourceName` | Source Runtime identity written to MIP metadata |
| `eventKind` | Exact kind required on every completed projection record |

An adapter definition contains exactly `descriptor`, optional `initialize`,
required `observeEvent`, required `finalize`, and
`sourceAuthorshipAttested`. The three hooks are private translation mechanics,
not portable Runtime-event semantics. If present, `initialize` receives the
normalized request and source stream. `observeEvent` receives one source value
and immutable context containing its zero-based `eventIndex`, normalized
request, and adapter-owned state. `finalize` receives immutable context
containing `eventCount`, normalized request, original source stream, and that
state. Provider event members and state shape remain outside this Standard.

The adapter value exposes semantic operations equivalent to:

| Operation | Result |
|---|---|
| `createInvestigation(events, request)` | Complete immutable verified MIP value |
| `exportInvestigation(events, request)` | Exact canonical UTF-8 MIP bytes |
| `createArtifact(events, request, fileName?)` | Immutable artifact envelope defined below |

Language bindings MAY spell or group these operations idiomatically, but all
three results MUST be produced through the same adapter projection and
CCA-MIP-1.0 construction path.

## Closed request

The request contains only the following members. The first five are identity
or source facts; omitted policy members receive exactly the listed defaults.

| Member | Contract | Default |
|---|---|---|
| `packageIdentifier` | Required non-empty string | none |
| `workspaceIdentifier` | Required non-empty string | none |
| `observationIdentifier` | Required non-empty string | none |
| `observationSequence` | Non-negative safe integer | `0` |
| `sourceVersion` | Required non-empty string | none |
| `maxEvents` | Positive safe integer | `10000` |
| `maxPackageBytes` | Positive safe integer | `16777216` |
| `maxSourceBytes` | Positive safe integer | `8388608` |
| `maxSourceValues` | Positive safe integer | `1000000` |

Unknown members are invalid. Limits are caller-declared acceptance policy and
do not authorize truncation. Identifiers, sequence, and source version are
never generated, inferred, or repaired.

## Runtime-event snapshot boundary

Before a completed projection can enter package cognition, it is detached as
canonical JSON. Accepted values are null, booleans, strings, finite numbers
other than negative zero, arrays without holes or foreign properties, and
plain objects whose own properties are enumerable string-keyed data
properties. Integer numbers MUST be safe integers. Accessors, symbols,
functions, `undefined`, `BigInt`, custom prototypes, cyclic structures,
non-enumerable data, sparse arrays, and non-finite numbers are invalid.

The fixed maximum nesting depth is 128 with the root at depth zero. The
projection contains at most `maxSourceValues` JSON values, its object-member
names and string values contain at most `maxSourceBytes` UTF-16 code units in
total, and its canonical UTF-8 representation contains at most
`maxSourceBytes` bytes. Breaching any bound is
`RUNTIME_EVENT_RESOURCE_LIMIT`; the value is never truncated.

The public snapshot helper, when exposed by a binding, accepts only
`maxBytes` and `maxValues`, both positive safe integers. Their defaults are
`8388608` and `1000000`. It applies the same detached-value rules, fixed depth
limit, UTF-16-unit policy, and canonical UTF-8 byte policy, and returns the
recursively immutable detached value.

## Completed projection

`finalize` returns a closed projection with exactly `accepted`, `records`, and
`relationships`. `accepted` is exactly `true`; `relationships` is exactly an
empty array; and `records` is a non-empty authored-order array.

Each record contains only optional `identifier` plus required `kind`,
`sourceOrder`, and `revision`. Identifier defaults to the request's
`observationIdentifier` and otherwise is a non-empty string. Kind is exactly
the descriptor's `eventKind`. Source order is a non-negative safe integer and
is strictly increasing across records. Revision is a plain canonical JSON
object. A record maps to one MIP Observation record as follows:

```text
role       = "context"
provenance = []
reference  = {identifier, kind, occurrence}
revision   = exact detached source revision
```

`occurrence` is the zero-based count of earlier records with the same exact
`[kind, identifier]` pair. No other relationship or semantic role is
constructed.

## Package and artifact result

The result contains exactly one Observation with the explicit identifier,
sequence, and Workspace; its records are the completed projection and its
relationships are empty. Trace, Replay, Evolution, and Comparative
Reconstruction arrays are empty and extensions are empty. Source acceptance
and source-authorship attestation are both true. Metadata contains the fixed
producer identity and `{name: descriptor.sourceName, version: sourceVersion}`.
Package identity, integrity, media type, and canonical bytes are governed by
CCA-MIP-1.0.

An artifact envelope contains exactly `bytes`, `mediaType`, `name`, and
`package`. `bytes` is the canonical package byte sequence represented as
immutable octets; `mediaType` is
`application/vnd.memoryos.mip+json`; and `package` is the same complete MIP
value. A supplied name MUST be a safe `.mip` basename: 5 through 255
characters, no control character or `<>:\"|?*\\/`, no space or period directly
before `.mip`, stem other than `.` or `..`, and no case-insensitive reserved
Windows device stem `con`, `prn`, `aux`, `nul`, `com1` through `com9`, or
`lpt1` through `lpt9` before a period or end.

When no name is supplied, `<packageIdentifier>.mip` is used if safe;
otherwise the name is
`memory-investigation-<digest-prefix>.mip`, where `digest-prefix` is the first
16 lower-case hexadecimal characters after `sha256:` in the package digest.

## Normative requirements

### CCA-MOS-ADAPT-001 — Provider-independent contract

An adapter **MUST** implement contract version `1.0.0` through a
provider-independent boundary with the exact descriptor, definition,
producer identity, attestation, and three-operation semantics in this
document. It **MUST NOT** require provider-specific semantics from consumers
or make a provider SDK part of the MemoryOS Standard.

### CCA-MOS-ADAPT-002 — External Runtime ownership

The external Runtime **MUST** retain ownership of execution and cognitive
truth; the adapter **MUST NOT** start, control, retry, modify, or claim ownership
of that Runtime.

### CCA-MOS-ADAPT-003 — Settled successful source

An adapter **MUST** validate its complete source lifecycle and publish only
after explicit successful settlement; empty, failed, cancelled, interrupted,
incomplete, ambiguous, or unsupported source outcomes **MUST** fail closed.

### CCA-MOS-ADAPT-004 — Explicit deterministic request

An adapter **MUST** accept only the closed request, exact defaults, and numeric
constraints in this document. Package, Workspace, Observation, and
source-version inputs **MUST** be explicit. It **MUST NOT** generate an
identifier or read a clock, random source, locale, renderer, filesystem order,
or network state to complete those inputs.

### CCA-MOS-ADAPT-005 — Source-authorship attestation

An adapter definition **MUST** include explicit source-authorship attestation
before any source value may cross into package cognition.

### CCA-MOS-ADAPT-006 — Truth-preserving projection

The completed projection **MUST** use the exact closed projection and record
shapes, source-order rule, occurrence rule, and context-record mapping in this
document. Each accepted settled semantic item **MUST** retain its exact source
identity and source-faithful canonical revision with empty provenance. An
adapter **MUST NOT** infer Evidence, Retrieval, Semantic Transformation,
Reflection, causality, explanation, or another role from provider event names
or transport shape.

### CCA-MOS-ADAPT-007 — Observation-only MIP output

Adapter output **MUST** be the exact verified CCA-MIP-1.0 Observation-only
package and, when requested, exact byte or artifact result defined here.
Trace, Replay, Evolution, and Comparative Reconstruction sections **MUST** be
empty; transport deltas, timing, usage, provider configuration, raw envelopes,
and SDK objects **MUST NOT** enter package cognition.

### CCA-MOS-ADAPT-008 — Deterministic atomic publication

Equivalent successfully settled cognition and explicit request values **MUST**
produce byte-identical package output independent of transport chunking, and
the exact snapshot and resource rules in this document **MUST** be enforced.
Any adapter, MIP construction, resource, or verification failure **MUST**
publish no partial investigation, byte sequence, or artifact.

### CCA-MOS-ADAPT-009 — Stable adapter failures

Adapter-specific failure codes **MUST** be limited to
`EMPTY_RUNTIME_STREAM`, `INVALID_RUNTIME_EVENT`, `EVENT_ORDER_VIOLATION`,
`INCOMPLETE_RUNTIME_STREAM`, `RUNTIME_STREAM_FAILURE`, and
`RUNTIME_EVENT_RESOURCE_LIMIT`; MIP failures **MUST** retain their CCA-MIP
diagnostics rather than being rewritten as adapter success.

## Informative reference adapters

The Reference Implementation includes adapters for OpenAI Agents SDK,
Anthropic SDK, and LangGraph. Those provider names and event shapes demonstrate
the contract; they are not normative dependencies or privileged providers.

## Conformance

Evidence group `MOS-EVID-ADAPT-001` covers settled and failed lifecycle
vectors; descriptor, definition, request, default, projection, record,
occurrence, package, byte, artifact, and filename vectors; generic snapshot
value, depth, value-count, UTF-16-unit, and UTF-8-byte vectors; chunk-boundary
equivalence; source-order and identifier preservation; provider-dependency
inspection; source-authorship attestation; MIP verification; and atomic
failure.
