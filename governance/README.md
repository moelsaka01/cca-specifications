# Specification governance

This governance model keeps architecture, normative standards, requirements,
implementation, and verification synchronized. It defines process and
evidence, not named decision makers. Projects applying this repository must
identify approving roles or groups through their established governance
without rewriting technical history.

## Governing principles

### Architecture first

Architecture is approved before dependent implementation begins. A change that
alters boundaries, dependency direction, ownership, data contracts, control
flow, compatibility, or a normative invariant requires:

1. an explicit proposal;
2. an ADR when an architecture choice is involved;
3. updates to affected standards and requirements;
4. project-defined review and approval;
5. only then, implementation.

Exploratory work must not be represented as approved architecture or merged as
production behavior before these conditions are met.

### Traceability

Every implementation obligation traces through:

```text
Normative statement -> Requirement -> Accepted ADR when applicable
                    -> Implementation milestone -> Verification evidence
```

A requirement without a normative source is incomplete. An implementation
change without a requirement is out of scope. A completed requirement without
verification evidence is not verified.

### Approval before implementation

`Draft` and `Proposed` artifacts invite review; they do not authorize
implementation. Approval is recorded using the status and evidence conventions
of the project applying this repository. Approval records must be durable,
reviewable, and linked from the affected artifact.

No document in this repository assigns approval authority to an unnamed person
or assumes that authors may approve their own architecture.

## Artifact responsibilities

| Artifact | Responsibility |
|---|---|
| [CCA Engineering Handbook 1.0](../specifications/CCA-ENG-1.0/README.md) | Normative engineering process within its declared scope |
| [CCA Runtime Foundation Standard 1.0](../specifications/CCA-RF-1.0/README.md) | Normative Runtime Foundation contract within its declared scope |
| [Requirement catalog](../templates/requirement-template.yaml) | Atomic, prioritized, verifiable obligations |
| [ADR](../adrs/ADR-TEMPLATE.md) | Context and rationale for an architecture choice |
| Implementation milestone | Authorized delivery boundary for approved requirements |
| Verification evidence | Durable proof that an implemented requirement conforms |

One artifact must not silently redefine another. Conflicts are resolved through
the change process before implementation continues.

## Change process

### 1. Propose

Identify the motivation, affected scope, compatibility impact, security or
operational considerations, and intended outcome.

### 2. Establish traceability

List affected normative statement identifiers, requirement identifiers, ADRs,
milestones, and verification evidence.

### 3. Record architecture decisions

Use the [ADR template](../adrs/ADR-TEMPLATE.md) when the proposal changes an
architecture boundary or selects among meaningful alternatives. ADRs remain
Proposed until approved.

### 4. Update governed artifacts

Update standards, requirement catalogs, diagrams, examples, and conformance
material together. Use the
[standard template](../templates/standard-template.md) and
[requirement template](../templates/requirement-template.yaml) for new
artifacts.

### 5. Review and approve

Apply the project's defined reviewers, approval rules, and segregation of
duties. Record the outcome and supporting evidence. Rejection or deferral is a
valid outcome and must not be represented as approval.

### 6. Implement

Implementation begins only after all blocking architecture and normative
changes are approved. The implementation references requirement identifiers
and accepted ADRs.

### 7. Verify and publish

Attach reproducible evidence, update requirement status, update version
history, and publish all mutually dependent artifacts as one coherent change.

## Requirement lifecycle

Recommended statuses are:

| Status | Meaning |
|---|---|
| Proposed | Authored but not approved |
| Approved | Authorized for an implementation milestone |
| Implemented | Implementation exists; verification may still be pending |
| Verified | Required evidence has passed and is linked |
| Deprecated | Still present for compatibility but scheduled for removal |
| Superseded | Replaced by another requirement with explicit traceability |
| Rejected | Reviewed and not approved |

Use the requirement's `stability` independently from delivery `status`.
Stability describes expected change risk; status describes governance and
implementation progress.

## Version and change control

Published CCA specifications use `MAJOR.MINOR` versions:

- **MAJOR**: incompatible normative or structural change;
- **MINOR**: backward-compatible normative addition or extension.

Editorial errata that do not change normative meaning retain the published
version and are recorded in its changelog. A change that alters what a
conforming implementation must or must not do requires a new published
version, even when the textual edit is small.

Every published standard records:

- version and status;
- change summary;
- affected requirement and ADR identifiers;
- compatibility or migration impact;
- approval evidence;
- publication date.

Published artifacts are immutable. An erratum is additive and must not silently
replace the original artifact. Superseded material remains available with a
link to its successor.

## Exceptions

An exception must identify scope, rationale, risk, affected requirements,
mitigations, approval evidence, and an expiry or review condition when
appropriate. An exception does not amend a standard and cannot be used to
bypass an unresolved architecture decision.

## Navigation

- [Specification suite](../README.md)
- [CCA Engineering Handbook 1.0](../specifications/CCA-ENG-1.0/README.md)
- [CCA Runtime Foundation Standard 1.0](../specifications/CCA-RF-1.0/README.md)
- [ADR index](../adrs/README.md)
- [ADR template](../adrs/ADR-TEMPLATE.md)
- [Standard template](../templates/standard-template.md)
- [Requirement template](../templates/requirement-template.yaml)
