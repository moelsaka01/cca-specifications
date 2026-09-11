# MemoryOS Conformance Specification

**Standard:** CCA-MEMORYOS-1.0  
**Conformance specification version:** 1.0.0  
**Status:** Normative

## Purpose

This document defines the evidence and report rules for a MemoryOS conformance
claim. It does not add platform behavior.

## Conformance targets

The complete target is one identified MemoryOS implementation and version,
including its Runtime observation boundary, Investigation Core, MIP
integration, Adapter contract, SDK, CLI, Cognitive Regression, and Explorer.

A component assessment may identify one of these scoped profiles:

- Runtime Observation Boundary;
- Investigation Core;
- Native Investigation Artifacts;
- MIP Integration;
- AI Runtime Adapter;
- SDK;
- CLI;
- Cognitive Regression; or
- Cognitive Investigation Explorer.

A profile assessment is not complete MemoryOS conformance.

## Required requirement set

A complete conformance claim covers:

1. every `CCA-MOS-*` requirement in [requirements.yaml](requirements.yaml);
2. incorporated `CCA-RF-001` through `CCA-RF-040`; and
3. incorporated `CCA-MIP-001` through `CCA-MIP-064`.

An implementation that does not expose an optional scoped component may mark
that component's requirements `NOT APPLICABLE` only for a scoped assessment.
No component in the complete target is optional.

## Evidence groups

| Group | Scope |
|---|---|
| `MOS-EVID-RT-001` | Runtime observation and incorporated Runtime Foundation behavior |
| `MOS-EVID-LIFE-001` | Investigation lifecycle and transition integrity |
| `MOS-EVID-MIP-001` | Incorporated MIP behavior and integration |
| `MOS-EVID-ADAPT-001` | AI Runtime Adapter contract |
| `MOS-EVID-CORE-001` | Investigation Core behavior |
| `MOS-EVID-ART-001` | Native artifact reconstruction, identity, ordering, and control state |
| `MOS-EVID-SDK-001` | SDK contract and binding parity |
| `MOS-EVID-CLI-001` | CLI contract and automation |
| `MOS-EVID-REG-001` | Cognitive Regression behavior |
| `MOS-EVID-EXPL-001` | Explorer behavior |
| `MOS-EVID-CONF-001` | Report completeness and evidence integrity |
| `MOS-EVID-COMP-001` | Compatibility verification |
| `MOS-EVID-VER-001` | Version identity and publication checks |

### Scoped-profile applicability

The following applicability registry is exhaustive:

| Scoped profile | Profile evidence groups |
|---|---|
| AI Runtime Adapter | `MOS-EVID-ADAPT-001` |
| CLI | `MOS-EVID-CLI-001` |
| Cognitive Investigation Explorer | `MOS-EVID-EXPL-001` |
| Cognitive Regression | `MOS-EVID-REG-001` |
| Investigation Core | `MOS-EVID-CORE-001`, `MOS-EVID-LIFE-001` |
| MIP Integration | `MOS-EVID-MIP-001` |
| Native Investigation Artifacts | `MOS-EVID-ART-001` |
| Runtime Observation Boundary | `MOS-EVID-RT-001` |
| SDK | `MOS-EVID-SDK-001` |

`MOS-EVID-COMP-001`, `MOS-EVID-CONF-001`, and `MOS-EVID-VER-001`
are common evidence groups and apply to every scoped assessment. For an
assessment selecting one or more profiles, the applicable evidence-group set
is the union of those common groups and every group listed for each selected
profile. A MemoryOS-specific requirement is applicable exactly when at least
one of its `evidence_groups` in
[requirements.yaml](requirements.yaml) belongs to that set. Every requirement
in an incorporated range inherits the `evidence_groups` on its corresponding
`incorporated_requirements` entry and uses the same test. Every other
requirement is `NOT
APPLICABLE` and requires a factual rationale. A complete-platform assessment
remains subject to every requirement, irrespective of evidence-group mapping.

The machine-readable registry publishes this profile map and maps each
requirement to at least one group.
An evidence manifest resolves a group to concrete commands, tests, vectors,
reviews, expected outcomes, and retained results for the assessed version.
The retained evidence artifact **MUST** use canonical bytes at an immutable
content-addressed location. The report **MUST** record the SHA-256 digest of
those exact evidence bytes in addition to the durable evidence root; a different
valid artifact at the same root is not the evidence assessed by that report.

## Required evidence properties

Automated evidence must be deterministic and reproducible from versioned
inputs. Every evidence record identifies the assessed implementation and
version, applicable requirement IDs, input or fixture, expected outcome,
observed outcome, execution environment where material, and durable result.

Review evidence identifies the reviewed artifact, revision, criterion,
reviewer, date, and result. A passing test unrelated to a requirement is not
evidence for that requirement.

An assessment that includes an implementation-specific raw Observation
snapshot-to-Frame projection, Trace reconstruction, or Trace validation MUST
record `nativeProjectionProfile` with that profile's non-empty `identifier`
and semantic `version`. The initial Reference Implementation value is
`{"identifier":"cca-studio-native-observation","version":"1.1.0"}`. An
assessment that begins with an already published Frame still records the
profile when it assesses Frame-to-Trace semantics; it may omit the field only
when every assessed operation is independent of native profile semantics.

## Result vocabulary

Every requirement row has exactly one result:

| Result | Meaning |
|---|---|
| `PASS` | Complete attributable evidence satisfies the requirement. |
| `FAIL` | Evidence demonstrates nonconformance or required evidence is incomplete. |
| `NOT APPLICABLE` | The requirement is outside a correctly named scoped profile, with rationale. |

`NOT APPLICABLE` cannot be used in a complete platform claim. Skipped,
unknown, not assessed, waived, and planned are not passing results.

## Normative requirements

### CCA-MOS-CONF-001 — Complete claim coverage

A complete MemoryOS conformance claim **MUST** report every MemoryOS-specific
and incorporated requirement listed in this document and **MUST NOT** omit,
waive, or mark any row `NOT APPLICABLE`.

### CCA-MOS-CONF-002 — Scoped claim labeling

A scoped assessment **MUST** identify exactly one or more named profiles and
contain exactly one row for every requirement in the resolved registry.
Applicable rows MUST be `PASS` or `FAIL`; every non-applicable row MUST be
`NOT APPLICABLE` with a factual rationale. A scoped assessment **MUST NOT** use
the phrase `CCA-MEMORYOS-1.0 Conformant`.

### CCA-MOS-CONF-003 — Deterministic evidence

Automated conformance evidence **MUST** be reproducible from versioned inputs
and **MUST** produce equivalent outcomes for equivalent inputs independent of
locale, timezone, renderer, layout, filesystem enumeration, map iteration,
clock, and randomness.

### CCA-MOS-CONF-004 — Attributable evidence manifest

Every reported result **MUST** reference an evidence manifest entry that
identifies the assessed implementation, version, requirement IDs, verification
method, inputs, expected outcome, observed outcome, and durable evidence.

### CCA-MOS-CONF-005 — Closed result vocabulary

A conformance report **MUST** use exactly `PASS`, `FAIL`, or
`NOT APPLICABLE` for each row and **MUST** treat missing or incomplete evidence
as `FAIL`, not as a pass.

### CCA-MOS-CONF-006 — No schema-only substitution

Schema validation alone **MUST NOT** satisfy a requirement involving semantic
derivation, ordering, lifecycle, identity, ownership, reference closure,
atomicity, compatibility, deterministic output, or prohibited behavior.

### CCA-MOS-CONF-007 — Claim identity

A conformance claim **MUST** identify the MemoryOS Standard version,
conformance specification and suite versions, assessed implementation and
version, applicable native projection profile, MIP and incorporated Runtime
publication identities, profile or complete scope, evidence root, assessor,
exact evidence digest, date, and every known failure.

## Conformance report

The report format and certification procedure are defined in
[certification.md](certification.md). A complete claim is permitted only when
every row is `PASS`. Independent verification additionally requires an
independent assessor to reproduce or review the complete evidence set.
