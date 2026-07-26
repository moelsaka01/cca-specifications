# CCA specifications roadmap

This roadmap governs specification maturity. It does not authorize software
implementation or supply missing architecture.

## Version 1.0 baseline

Status: complete specification baseline for implementation milestone IM-003.

The baseline contains:

- [CCA-ENG-1.0](specifications/CCA-ENG-1.0/README.md), the Engineering
  Handbook;
- [CCA-RF-1.0](specifications/CCA-RF-1.0/README.md), the Runtime Foundation
  Standard;
- forty machine-readable Runtime Foundation requirements assigned to IM-003;
- summaries of ADR-003-001 through ADR-003-008;
- Runtime Foundation conformance levels, checklists, and compliance matrix;
- normative Mermaid sources for the layer, lifecycle, service, communication,
  startup, and shutdown models;
- ADR, standard, and requirement templates;
- repository governance and contribution procedures.

## Governance completion

The following decisions must be recorded before the repository is presented as
an open public standards project:

1. copyright ownership and content licensing;
2. inbound contribution terms;
3. named approval and publication authorities;
4. repository locations and cross-repository release coordination;
5. support and compatibility policy for published standard versions.

These are governance decisions, not implementation milestones.

## Future specification work

Future work may proceed only from approved architecture and accepted ADRs.
Candidate work includes:

- publishing later accepted architectural decisions;
- maintaining implementation evidence against the CCA-RF-1.0 compliance
  matrix;
- defining compatibility and deprecation rules for minor and major releases;
- adding future standards only after their architecture is approved.

This list is planning context, not approval of any architecture, protocol,
runtime behavior, MemoryOS behavior, or implementation.

## Release rule

Every release must identify its governing decisions, changed normative
statements, requirement identifier changes, compatibility impact, and
conformance impact. A release must not claim approval or implementation status
without recorded evidence.
