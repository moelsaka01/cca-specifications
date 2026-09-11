---
id: CCA-MIP-CONFORMANCE-1.0
title: Memory Investigation Package Conformance
version: "1.0"
status: Draft
specification: CCA-MIP-1.0
---

# Memory Investigation Package Conformance

## Purpose

This document defines the evidence required to claim conformance with
MIP-001. It does not add to or reinterpret the normative contract in
[MIP-001.md](MIP-001.md).

## Conformance targets

A claim identifies one or more targets:

| Target | Required capability |
|---|---|
| Producer | Exports valid, canonical, deterministic `.mip` bytes atomically. |
| Consumer | Imports valid packages, rejects invalid packages atomically, and preserves unknown noncritical extensions. |
| Verifier | Produces deterministic validation results and independently reconstructs every supplied derived artifact. |
| Round-trip implementation | Imports and re-exports without semantic or byte changes when metadata is unchanged. |

A target may claim only the capabilities for which it supplies all mandatory
evidence.

## Required evidence

### Producer

A conforming Producer demonstrates:

1. exact UTF-8 RFC 8785 serialization;
2. all required closed core sections, including empty optional arrays;
3. exact Workspace and source-identifier preservation;
4. canonical array ordering and contiguous semantic indices;
5. reference-only derived artifacts;
6. exact section, cognition, and package digests;
7. byte-identical output for equivalent input and identical explicit metadata;
8. prohibited-content rejection; and
9. no partial destination on any export failure.

### Consumer

A conforming Consumer demonstrates:

1. strict byte, JSON, version, schema, and checksum validation;
2. deterministic reference and Workspace validation;
3. independent Trace, Replay, Evolution, and Comparative reconstruction;
4. rejection of unknown critical extensions and preservation of unknown
   noncritical extensions;
5. no normalization, repair, silent downgrade, or silent data loss; and
6. no published state after any import failure.

### Verifier

A conforming Verifier demonstrates:

1. all validation phases in MIP-001 section 17 in the specified order;
2. stable failure codes and RFC 6901 paths;
3. deterministic diagnostic ordering;
4. semantic validation in addition to JSON Schema validation; and
5. equivalent results independent of locale, timezone, renderer, Runtime,
   Provider, filesystem enumeration, and map iteration order.

The first five fixed verification checks and mechanically decidable prohibited
content are Consumer-verifiable. Evidence that allowed cognitive text came from
the accepted source, rather than being added by the Producer, is demonstrated
by Producer source-fixture tests and independent review; it cannot be inferred
from package bytes alone.

## Golden vectors

The following valid vectors are mandatory:

| Vector | Expected behavior |
|---|---|
| [`minimal/minimal-observation.mip`](examples/minimal/minimal-observation.mip) | Accept an Observation-only package. |
| [`complete/complete-investigation.mip`](examples/complete/complete-investigation.mip) | Accept and independently reconstruct every artifact. |
| [`extended/noncritical-extension.mip`](examples/extended/noncritical-extension.mip) | Accept, ignore semantically, and preserve the unknown noncritical extension. |

For each vector, a conforming Producer emits exact reference bytes and a
conforming Consumer confirms every published digest in
[examples/README.md](examples/README.md).

## Negative vectors

The invalid fixture catalog is in [examples/invalid/README.md](examples/invalid/README.md).
A conforming Consumer rejects every listed condition atomically with the
specified primary code. At minimum, automated evidence covers:

- malformed UTF-8 and JSON;
- duplicate member names;
- valid JSON that is not canonical bytes;
- unsupported major format version;
- missing and unknown core members;
- bad section, cognition, and package digests;
- wrong Workspace and duplicate typed identity;
- invalid provenance role, cardinality, relationship, or cycle;
- dangling relationship and artifact references;
- invalid Trace branch or step order;
- Replay not equal to deterministic Trace projection;
- Evolution not equal to semantic Observation comparison;
- Comparative alignment or reason mismatch;
- unknown critical extension;
- prohibited UI, media, Runtime, Provider, executable, or generated content;
  and
- configured resource-limit failure.

## Compatibility matrix

Implementations test this minimum matrix:

| Package | Consumer | Expected result |
|---|---|---|
| `1.0.x` | `1.0.x` | Accept when otherwise valid. |
| older `1.x` | newer `1.x` | Accept when otherwise valid. |
| newer `1.x`, unknown noncritical extension only | older `1.x` | Accept and preserve extension. |
| newer `1.x`, unknown critical extension | older `1.x` | Reject atomically. |
| unknown major | `1.x` | Reject atomically. |
| unknown top-level/core member | `1.x` | Reject atomically. |

## Round-trip evidence

For a package without caller-requested metadata changes:

```text
input bytes
  -> strict import
  -> in-memory representation
  -> canonical export
  -> byte-for-byte identical output
```

Unknown noncritical extension payloads are included in this guarantee. A
semantic-only comparison is insufficient.

## Requirement coverage

Every mandatory entry in [requirements.yaml](requirements.yaml) maps to at
least one automated test, golden vector, negative vector, or independent
review. Evidence records identify requirement IDs directly.

Schema validation alone cannot satisfy a requirement involving order,
reference closure, deterministic reconstruction, compatibility, integrity,
prohibited content, or atomic failure.

## Conformance claim

A claim states:

- MIP specification version and format version;
- target type or types;
- requirement IDs covered;
- test and vector versions;
- platform-independent evidence location;
- known exceptions; and
- review or release record.

The phrase **CCA-MIP-1.0 conforming** may be used only when every applicable
mandatory requirement passes and no exception changes package meaning,
canonical bytes, or compatibility.
