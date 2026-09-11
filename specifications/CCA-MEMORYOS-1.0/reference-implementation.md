# MemoryOS Reference Implementation Guide

**Standard:** CCA-MEMORYOS-1.0  
**Status:** Informative

## Purpose

This guide identifies the implementation used to demonstrate the Standard. It
is evidence and an interoperability reference, not normative architecture.
Another implementation may use different languages, algorithms, storage, or
repository structure while producing the same required observable behavior.

## Reference baseline

The official initial Reference Implementation is MemoryOS 1.2.0 through
MO-1208. Its component contract baselines are:

| Boundary | Version | Reference responsibility |
|---|---|---|
| Runtime Foundation | CCA-RF-1.0 | Underlying deterministic Runtime obligations |
| Native MemoryOS investigation artifacts | 1.1 | Trace, Replay, Evolution, and Comparative Reconstruction |
| CCA Studio native Observation profile | 1.1.0 | Exact snapshot-to-Frame projection and Frame-to-Trace bindings defined by [native-observation-profile.md](native-observation-profile.md) |
| MIP implementation | 1.0.0 | Canonical Producer, Consumer, and Verifier |
| AI Runtime Adapter contract | 1.0.0 | Settled provider-neutral translation |
| Investigation Core | 1.0.0 | Single investigation execution authority |
| SDK | 1.0.0 | JavaScript, Python, and C++ public facades |
| CLI | 1.0.0 | SDK-backed shell and JSON Lines automation |
| Regression report | 1.0.0 | Deterministic factual difference report |
| Explorer result | 1.0.0 | Deterministic report-evidence navigation |

For native Observation and Frame-to-Trace conformance, the Reference
Implementation declares projection profile identifier
`cca-studio-native-observation`, version `1.1.0`. This identity belongs to
assessment metadata; it does not add a member to the Observation Frame and is
not a portable input schema. Its exact conditional contract is published in
[native-observation-profile.md](native-observation-profile.md).

## Implementation map

The current reference workspace locates Runtime and released memory
capabilities in `cca-core`; MIP, adapters, Investigation Core, and presentation
integration in `cca-studio`; public language facades in `cca-sdk`; automation
in `memoryos-cli`; and the official assessment harness in
`cca-conformance`.

These names help an assessor locate evidence. They are not required names for
another implementation.

## Evidence flow

```text
versioned input vectors
        |
        v
Reference Implementation test suites
        |
        v
versioned conformance manifest
        |
        v
CCA-MEMORYOS-1.0 conformance report
```

The Reference Implementation demonstrates one way to satisfy the Standard.
The Standard remains authoritative when implementation documentation,
behavior, or tests disagree with it.

## Independent implementation guidance

An independent implementation can begin with the normative document map in
[README.md](README.md#5-publication-set), implement one authority for
investigation behavior, consume the exact incorporated MIP publication, expose
SDK and CLI behavior without duplicating semantics, and run an equivalent
evidence set under [conformance.md](conformance.md).

Implementation-specific optimization is permitted only when externally
observable identity, ordering, lifecycle, error, atomicity, compatibility, and
determinism remain equivalent.
