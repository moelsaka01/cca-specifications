# Architecture Decision Records

Architecture Decision Records (ADRs) capture reviewed decisions that shape the
CCA specifications or their implementation boundaries. They preserve why a
choice was made, which alternatives were considered, and which requirements
and standards depend on it.

An ADR records a decision; it does not replace a normative standard or
requirement. Normative obligations remain in the applicable standard and
requirement catalog.

## Authoring an ADR

1. Copy [ADR-TEMPLATE.md](ADR-TEMPLATE.md) to a new file in this directory.
2. Name the file `ADR-NNNN-short-title.md`, using the next available
   zero-padded number.
3. Complete every required section without removing alternatives or
   consequences.
4. Link affected standards and requirement identifiers.
5. Keep the status `Proposed` until the project-defined approval process is
   complete.
6. After approval, update the affected standards, requirements, milestones,
   and verification plans in the same governed change.

The reusable [ADR authoring template](../templates/adr-template.md) contains
the same decision structure with additional writing prompts.

## Status lifecycle

| Status | Meaning |
|---|---|
| Proposed | Under review; implementation is not authorized |
| Accepted | Approved through the project-defined process |
| Rejected | Considered and not approved |
| Deprecated | Retained for history but no longer recommended |
| Superseded | Replaced by a later ADR that links back to this record |

Changing an accepted ADR requires a new ADR. Do not rewrite the decision
history in place. Editorial corrections may be made without changing its
meaning.

## Traceability

Every architecture-affecting ADR should identify:

- affected normative statements;
- affected requirement identifiers;
- implementation milestones, if assigned;
- verification evidence that must change;
- ADRs it supersedes or depends on.

Requirements reference accepted ADRs through the `adr` list defined by the
[requirement template](../templates/requirement-template.yaml).

## Navigation

- [Specification suite](../README.md)
- [CCA Engineering Handbook 1.0](../specifications/CCA-ENG-1.0/README.md)
- [CCA Runtime Foundation Standard 1.0](../specifications/CCA-RF-1.0/README.md)
- [Governance](../governance/README.md)
- [Standard template](../templates/standard-template.md)
- [Repository ADR template](ADR-TEMPLATE.md)
- [Reusable ADR template](../templates/adr-template.md)
- [Requirement template](../templates/requirement-template.yaml)
