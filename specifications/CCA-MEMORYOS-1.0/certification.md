# MemoryOS Certification Guide

**Standard:** CCA-MEMORYOS-1.0  
**Certification procedure version:** 1.0.0  
**Status:** Normative

## Purpose

This document defines how a conformance assessment becomes a report and how a
qualifying report may support a certification claim. Certification records
evidence; it does not change platform behavior.

## Assessment levels

| Level | Meaning | Permitted statement |
|---|---|---|
| C0 — Identified | A pre-assessment registration identifies the target and versions; it is not a conformance report. | No conformance claim |
| C1 — Assessed | Every row has a result and evidence reference. | Assessment only |
| C2 — Conformant | Every required row is `PASS`. | `CCA-MEMORYOS-1.0 Conformant` |
| C3 — Independently Verified | C2 evidence is reproduced or reviewed by an independent assessor. | `CCA-MEMORYOS-1.0 Independently Verified` |

### C0 registration

A C0 artifact is a pre-assessment registration record, not a report and not a
conformance claim. It identifies the Standard and conformance-specification
versions, intended complete or named-profile scope, implementation name and
version, immutable implementation revision when used, applicable native
projection profile, incorporated CCA-RF and CCA-MIP publication identities,
registrant, and registration date. It may identify a planned conformance-suite
version. It has no requirement matrix, result rows, assessment result, or
evidence-completeness implication. The report schema and the assessment
procedure below apply only to C1, C2, and C3.

## Report header

A report records:

| Field | Value |
|---|---|
| Standard | `CCA-MEMORYOS-1.0` |
| Standard publication date | Exact published date |
| Conformance specification | `1.0.0` |
| Conformance suite | Identified version |
| Scope | Complete platform or named profile |
| Implementation | Identified name |
| Implementation version | Semantic Version in `MAJOR.MINOR.PATCH` form |
| Implementation revision | Immutable revision identifier, when used by the assessed release |
| Native projection profile | Required when the assessment includes implementation-specific raw Observation snapshot projection or Frame-to-Trace semantics; otherwise optional |
| Incorporated CCA-RF publication | Version and pinned digest set |
| Incorporated CCA-MIP publication | Version and pinned digest set |
| Evidence root | Durable reference to immutable evidence bytes |
| Evidence digest | SHA-256 digest of the exact canonical evidence artifact |
| Assessor | Identified person or accountable body |
| Assessment date | Explicit date |
| Assessment level | C1, C2, or C3 |

## Requirement matrix

A C1, C2, or C3 report contains exactly one row for every requirement in the
complete resolved registry. An applicable row is `PASS` or `FAIL`; an
out-of-scope row is `NOT APPLICABLE` with a factual rationale:

| Requirement | Result | Evidence group | Evidence reference | Notes |
|---|---|---|---|---|
| `CCA-MOS-...` | `PASS` / `FAIL` / `NOT APPLICABLE` | `MOS-EVID-...` | Durable reference | Factual qualification only |

The resolved registry includes every `CCA-MOS` requirement plus
`CCA-RF-001` through `CCA-RF-040` and `CCA-MIP-001` through `CCA-MIP-064`.
A complete report cannot use `NOT APPLICABLE`; a scoped report retains the
full registry and uses it only for requirements outside its resolved profile
scope.

## Procedure

1. Freeze the target identity and all assessed inputs.
2. Resolve the requirement registry, Standard publication date, and incorporated publication digests.
3. Execute every automated evidence group from clean versioned inputs.
4. Complete architecture, dependency, and prohibited-behavior reviews.
5. Record each requirement result using the closed vocabulary.
6. Treat missing, stale, non-attributable, or irreproducible evidence as
   `FAIL`.
7. Determine C1 through C3 from the completed matrix.
8. Publish the evidence artifact at an immutable content-addressed location and
   bind its exact canonical SHA-256 digest in the report.
9. Publish the immutable report beside that retained evidence.

This procedure is governed by `CCA-MOS-CONF-001` through
`CCA-MOS-CONF-007`.

## Certification rules

Only a complete C2 or C3 report may use a complete conformance statement. A
profile report names the profile in every displayed claim. An assessment date
cannot precede the Standard publication date. A C3 assessment is based on a C2
report, is dated no earlier than its source assessment, and names an assessor
different from the source evidence assessor. A later assessment supersedes an
earlier report by reference and never rewrites it. A report with one or more
`FAIL` rows is not certified.

Certification does not grant compatibility with an unassessed version and
does not transfer authority from the Standard to the Reference Implementation
or assessor.

## Publication record

The report is accompanied by its content-addressed evidence artifact and exact
evidence digest, machine-readable result matrix, failing diagnostics,
environment description where material, and independent-assessor identity for
C3. Human-readable summaries may accompany the report but cannot replace its
requirement rows.
