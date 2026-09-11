# MemoryOS Versioning Guide

**Standard:** CCA-MEMORYOS-1.0  
**Status:** Normative

## Purpose

This document separates specification identity from product and component
identity and defines how a published MemoryOS Standard evolves.

## Baseline identities

| Concern | Baseline identity |
|---|---|
| MemoryOS Standard | `CCA-MEMORYOS-1.0` |
| Reference Implementation product release | MemoryOS `1.2.0` |
| Conformance specification | `1.0.0` |
| Investigation Core | `1.0.0` |
| AI Runtime Adapter contract | `1.0.0` |
| SDK | `1.0.0` |
| CLI | `1.0.0` |
| Native Observation Frame, Trace, Replay, Evolution, and Comparative Reconstruction artifacts | `1.1` |
| CCA Studio native Observation profile | `1.1.0` |
| Regression report | `1.0.0` |
| Explorer result | `1.0.0` |
| MIP specification / format | `CCA-MIP-1.0` / `1.0.0` |

The Reference Implementation product version records the MemoryOS 1.2
milestone baseline. It is not a claim that every independently versioned
component has version 1.2.0.

## Standard change classification

- a Standard **major** version changes or removes normative behavior, accepted
  input, required output, ownership, identity, ordering, lifecycle,
  compatibility, or conformance meaning;
- a Standard **minor** version makes only backward-compatible normative
  additions; and
- an erratum may correct wording, links, or publication metadata without
  changing any valid input, output, requirement meaning, or evidence outcome.

## Normative requirements

### CCA-MOS-VER-001 — Independent declarations

Every conformance claim and public component **MUST** declare its own applicable
version identity and **MUST NOT** substitute the product release, Standard,
MIP, SDK, CLI, Core, report, or artifact version for another.

### CCA-MOS-VER-002 — Baseline version fidelity

An implementation claiming the CCA-MEMORYOS-1.0 baseline **MUST** identify and
honor the baseline contract versions in this document or explicitly identify a
later version whose compatibility has been verified.

### CCA-MOS-VER-003 — Incompatible Standard change

A change that removes an accepted input, changes a required deterministic
outcome, alters ownership, identity, ordering, lifecycle, or compatibility, or
invalidates passing conformance evidence **MUST** publish a new Standard major
version.

### CCA-MOS-VER-004 — Compatible Standard change

A new Standard minor version **MUST** preserve all prior valid inputs and
required outcomes and **MUST** add complete requirements, compatibility impact,
conformance evidence, and publication history for every addition.

### CCA-MOS-VER-005 — Conformance and Reference Implementation versioning

The conformance suite, conformance report, and Reference Implementation
**MUST** identify their versions and the exact Standard and incorporated
publications assessed; a new implementation release **MUST NOT** rewrite an
earlier assessment or imply that version equality is conformance evidence.

## Conformance

Evidence group `MOS-EVID-VER-001` covers published identities, unsupported
version negatives, change-classification review, and assessment pinning.
