# Memory Investigation Package Contract

**Standard:** CCA-MEMORYOS-1.0  
**Status:** Normative

## Purpose

This document defines how MemoryOS uses the canonical Memory Investigation
Package. It incorporates CCA-MIP-1.0 and does not copy, amend, or reinterpret
its format.

## Incorporated publication

The exact incorporated artifacts and SHA-256 digests are recorded in
[README.md](README.md#62-memory-investigation-package). The frozen source
metadata remains historical; MO-1208 gives the digest-pinned requirement range
normative force for a CCA-MEMORYOS-1.0 conformance claim.

## Normative requirements

### CCA-MOS-MIP-001 — Complete MIP incorporation

A conforming MemoryOS platform that produces, consumes, verifies, imports, or
exports a `.mip` value **MUST** satisfy every applicable `CCA-MIP-001` through
`CCA-MIP-064` requirement from the exact incorporated publication.

### CCA-MOS-MIP-002 — Baseline export format identity

MemoryOS Standard 1.0 baseline package export **MUST** use CCA-MIP format
`1.0.0`, schema identifier `urn:memoryos:mip:schema:1.0.0`, media type
`application/vnd.memoryos.mip+json`, canonicalization profile `RFC8785`, digest
profile `SHA-256`, and validation profile `MIP-CORE-1.0` exactly as
incorporated; import and round-trip **MUST** retain the incorporated CCA-MIP
same-major compatibility behavior.

### CCA-MOS-MIP-003 — Strict import and verification

Import and verification **MUST** treat package bytes as untrusted, execute the
CCA-MIP-1.0 validation contract, publish only a completely valid immutable
package, and leave existing state unchanged on failure.

### CCA-MOS-MIP-004 — Exact export boundary

Export **MUST** return exact canonical bytes for an existing verified
MIP-backed investigation and **MUST NOT** manufacture a native-to-MIP
translation, missing artifact, identifier, semantic payload, or extension.

### CCA-MOS-MIP-005 — No competing package semantics

An implementation **MUST NOT** introduce another portable MemoryOS cognition
package contract or apply implementation-specific normalization, repair,
ordering, hashing, compatibility, or presentation behavior to a `.mip` value.

## Conformance

Evidence group `MOS-EVID-MIP-001` covers this integration contract. The
CCA-MIP-1.0 golden vectors, negative vectors, checksums, round-trip tests, and
all sixty-four incorporated requirement results remain mandatory.
