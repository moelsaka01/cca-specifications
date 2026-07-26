# {STANDARD-ID}: {Standard title}

## Document metadata

| Field | Value |
|---|---|
| Identifier | `{STANDARD-ID}` |
| Version | `0.1` |
| Status | Draft |
| Stability | Experimental |
| Scope | {Concise scope statement} |
| Owners | {Project-defined maintaining role or group} |
| Approval record | {Link to approval evidence when available} |
| Supersedes | None |

## 1. Purpose

State why this standard exists, the outcomes it governs, and the audience that
must apply it.

## 2. Scope

### 2.1 In scope

- {Included concern}

### 2.2 Out of scope

- {Excluded concern}

An excluded concern is not authorized for implementation by this standard.

## 3. Normative language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**,
and **MAY** are to be interpreted as normative requirement levels. Each
normative statement must have a stable identifier and corresponding
verification evidence.

## 4. Terms and definitions

| Term | Definition |
|---|---|
| {Term} | {Unambiguous definition} |

## 5. Normative statements

### {STANDARD-ID}-N001: {Statement title}

{The subject} **MUST** {observable obligation}.

Rationale: {Why the obligation exists.}

Verification: {How conformance can be demonstrated.}

### {STANDARD-ID}-N002: {Statement title}

{The subject} **MUST NOT** {prohibited behavior}.

Rationale: {Why the prohibition exists.}

Verification: {How the absence of prohibited behavior can be demonstrated.}

## 6. Architecture

### 6.1 Context and boundaries

Describe the system boundary, external actors, trusted inputs, and explicit
exclusions.

### 6.2 Components and responsibilities

| Component | Responsibility | Must not own |
|---|---|---|
| {Component} | {Single responsibility} | {Excluded responsibility} |

### 6.3 Dependency direction

Document permitted dependency edges and prohibited reverse dependencies.

### 6.4 Data and control flow

Describe ordered processing, failure propagation, ownership, and environmental
dependencies.

### 6.5 Invariants

- `{STANDARD-ID}-INV-001`: {Invariant}

Architecture changes require an accepted ADR linked through the
[ADR template](adr-template.md) before implementation.

## 7. Requirements

Author requirements using the
[requirement template](requirement-template.yaml). Every requirement must
trace to at least one normative statement and verification method.

| Requirement ID | Normative source | ADR | Milestone | Verification |
|---|---|---|---|---|
| `{STANDARD-ID}-REQ-001` | `{STANDARD-ID}-N001` | {ADR or None} | {Milestone or Unassigned} | {Evidence} |

## 8. Conformance

### 8.1 Conformance target

Define the product, library, document, or process that can claim conformance.

### 8.2 Required evidence

- {Automated test, inspection, analysis, or review evidence}

### 8.3 Exceptions

An exception must be documented, time-bounded when appropriate, approved
through the project-defined process, and traceable to affected requirements.
An exception must not silently redefine the standard.

## 9. Security, privacy, and operational considerations

Record applicable concerns or state explicitly why they are outside this
standard's scope.

## 10. Compatibility and change control

Describe compatibility promises, deprecation policy, and migration
requirements. Apply the versioning rules in
[governance](../governance/README.md).

## 11. References

- [Specification suite](../README.md)
- [CCA Engineering Handbook 1.0](../specifications/CCA-ENG-1.0/README.md)
- [CCA Runtime Foundation Standard 1.0](../specifications/CCA-RF-1.0/README.md)
- [Governance](../governance/README.md)
- [ADR index](../adrs/README.md)
- [Requirement template](requirement-template.yaml)

## 12. Version history

| Version | Date | Status | Change summary | Approval record |
|---|---|---|---|---|
| `0.1` | {YYYY-MM-DD} | Draft | Initial proposal | Pending |
