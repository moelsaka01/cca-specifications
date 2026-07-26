# Contributing to CCA specifications

CCA specifications are architecture records, not implementation notes.
Contributions must preserve authority, precision, traceability, and scope.

## Before proposing a change

1. Read the [repository overview](README.md).
2. Read [CCA-ENG-1.0](specifications/CCA-ENG-1.0/README.md).
3. Read the [governance policy](governance/README.md).
4. Identify the affected standard, requirement identifiers, ADRs, and version.
5. Confirm that the proposal does not rely on an unapproved architecture
   assumption.

The licensing and inbound-contribution terms are unresolved. Submission does
not imply acceptance or a license grant. See [LICENSE](LICENSE).

## Permitted content

This repository accepts specification text, requirement YAML, ADRs, approved
diagrams, templates, governance records, and changelogs. Do not add:

- runtime, compiler, MemoryOS, application, or tooling source code;
- tests, CI workflows, build systems, binaries, or generated build output;
- implementation-specific behavior presented as architecture;
- unapproved product, protocol, persistence, network, AI, or plugin designs.

## Change workflow

1. Create a focused branch from the current approved baseline.
2. Make one coherent standards change.
3. Add or update an ADR when the change selects architecture.
4. Update every affected normative statement and requirement record.
5. Update cross-references and the specification changelog.
6. State compatibility and conformance consequences.
7. Request review from the authority defined by
   [governance](governance/README.md).
8. Merge only after the required approval is recorded.

No branch-name, commit-message, hosting-platform, or merge-strategy convention
is imposed until governance approves one.

## Architecture and ADRs

Use [`adrs/ADR-TEMPLATE.md`](adrs/ADR-TEMPLATE.md). An ADR proposal must state
context, the proposed decision, credible alternatives, consequences, and
references. A proposal is not architecture until its status is accepted by the
authorized decision authority.

Do not silently resolve an ambiguity through wording changes.

## Requirements

Use [`templates/requirement-template.yaml`](templates/requirement-template.yaml).
Requirement identifiers are durable. Do not reuse or silently renumber an
identifier. A changed requirement must retain traceability to its standard,
ADRs, verification method, implementation milestone, status, and release
history.

## Review quality

A reviewable proposal:

- uses consistent normative terminology;
- distinguishes normative requirements from rationale and examples;
- contains no broken relative links;
- contains valid, consistently structured YAML;
- records affected versions and changelogs;
- avoids decorative or promotional language;
- marks unresolved decisions without choosing them implicitly.

