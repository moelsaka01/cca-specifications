# MemoryOS Runtime Behavior

**Standard:** CCA-MEMORYOS-1.0  
**Status:** Normative

## Purpose

This document standardizes only the MemoryOS-observable cognition boundary.
CCA-RF-1.0 remains authoritative for Runtime Foundation components, host
behavior, lifecycle, services, failure, and cleanup. Nothing here redefines
that architecture.

## Informative model

The source Runtime owns cognition. MemoryOS receives an accepted, detached
observation of settled source truth and investigates that value without
acquiring ownership of the source Runtime or its implementation state.

```text
source Runtime truth -> settled result -> detached Observation -> investigation
```

## Standardized native boundary

The implementation-independent native publication and conformance seam is the
immutable `MemoryOSObservationFrame` version `1.1` defined by
[native-artifacts.md](native-artifacts.md). It contains the accepted detached
source snapshot and its complete semantic-world projection.

For this boundary, successful completion is the Frame's explicit
`resultCode` equal to `OK`. Product-profile fields nested inside the preserved
snapshot are source data, not additional portable lifecycle gates unless that
profile explicitly defines them as such. In particular, the published CCA
Studio profile preserves its nested `result` and `session.state` but does not
use either field to override the Frame result code.

A product MAY expose a product-specific adapter, at or within its Core
boundary, that converts a settled Runtime result or product snapshot into that
Frame. The adapter's input schema and private projection procedure are not a
portable MemoryOS data contract. They MUST be deterministic, preserve all
source truth used by the Frame, and validate the resulting Frame before it is
published as accepted state. The Reference Implementation's CCA Studio
snapshot adapter is such a nonportable adapter; another implementation does
not need to reproduce its input schema.

CCA-MIP-1.0 is the only standardized portable interchange representation. A
native adapter output does not become a MIP and MUST NOT be exported as one
unless an actual conforming MIP was authored and verified.

## Normative requirements

### CCA-MOS-RT-001 — Runtime Foundation incorporation

A conforming MemoryOS platform **MUST** satisfy the applicable
`CCA-RF-001` through `CCA-RF-040` requirements of the exact CCA-RF-1.0
publication incorporated by CCA-MEMORYOS-1.0.

### CCA-MOS-RT-002 — Source authority

MemoryOS **MUST** treat the source Runtime as the authority for the cognition
it observes and **MUST NOT** replace, reinterpret, or acquire ownership of that
source truth.

### CCA-MOS-RT-003 — Observation acceptance boundary

MemoryOS **MUST** publish as accepted cognition only a coherent, detached,
immutable native Observation Frame whose outer `resultCode` is `OK`, or a
verified MIP Observation. It **MUST NOT** publish an outer non-`OK`, incomplete,
partially acquired, or invalidly projected source result. Under the published
CCA Studio native profile, nested snapshot `result` and `session.state` values
are preserved source data and are not acceptance gates; they do not override
the outer Frame result code.

### CCA-MOS-RT-004 — Deterministic boundary outcome

Equivalent accepted source cognition, explicit input, Workspace identity, and
initial investigation state **MUST** produce equivalent acceptance, rejection,
identity, ordering, and diagnostic outcomes.

### CCA-MOS-RT-005 — Workspace isolation

Every accepted Observation **MUST** retain its exact Workspace identity, and
an operation **MUST** reject a cross-Workspace value without changing either
Workspace or any previously accepted investigation state.

### CCA-MOS-RT-006 — Runtime and presentation independence

After Observation acceptance, deterministic investigation, verification, and
portable package processing **MUST NOT** require a live source Runtime,
renderer, camera, layout, timer, polling loop, network request, or Provider;
none of those concerns may alter accepted cognition.

## Conformance

Evidence group `MOS-EVID-RT-001` covers this document. Required evidence
includes source-ownership review, settled-result negative cases,
cross-Workspace rejection, equivalent-input vectors, and investigation with
the source Runtime and presentation absent.
