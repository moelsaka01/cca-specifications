# CCA Platform Constitution Conformance

## Purpose

Define the evidence required to assess conformance with CCA Platform Constitution 1.0.

## Conformance criteria

| ID | Criterion | Evidence |
| --- | --- | --- |
| AR-001 | Workspace is the constitutional root. | CCA-ARCH-1.0, ADR-001 |
| AR-002 | Domains organize semantic concerns. | CCA-ARCH-1.0, CCA-LEX-1.0 |
| AR-003 | Assets represent durable information. | CCA-ARCH-1.0, CCA-LEX-1.0 |
| AR-004 | Services perform behavior and never own persistent state. | CCA-ARCH-1.0, CCA-LEX-1.0, ADR-003 |
| AR-005 | Policies constrain behavior. | CCA-ARCH-1.0, CCA-LEX-1.0 |
| AR-006 | Providers implement infrastructure and never define architecture. | CCA-ARCH-1.0, CCA-LEX-1.0 |
| AR-007 | Contracts define public behavior. | CCA-ARCH-1.0, CCA-LEX-1.0 |
| AR-008 | Dependencies remain acyclic. | CCA-ARCH-1.0 |
| AR-009 | Workspace is the boundary of consistency. | ADR-002 |
| AR-010 | Runtime is disposable. | ADR-003 |
| AR-011 | Persistence preserves Workspace state. | ADR-004 |
| AR-012 | Structural, behavioral, and evolution views are used together. | ADR-005 |
| GOV-001 | Proposals, reviews, ADRs, versions, compatibility, releases, and revisions follow constitutional governance. | CCA-GOV-1.0 |
| ENG-001 | Work follows Architecture → Specification → Review → Freeze → Implementation → Verification → Release. | CCA-ENG-2.0 |

## Assessment

An independent review SHALL verify each criterion against the cited document and record any deviation before release.
