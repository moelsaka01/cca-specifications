# Cognitive Computing Architecture specifications

The Cognitive Computing Architecture (CCA) is a standards-governed
architecture for cognitive computing systems. This repository is the
authoritative publication location for approved CCA standards, requirements,
architectural decisions, templates, and governance.

This is a specification repository. It contains no runtime, compiler, MemoryOS,
application, test, CI, or build implementation.

## Engineering philosophy

CCA applies the following principles:

- architecture precedes implementation;
- standards govern implementation;
- requirements drive development and verification;
- explicit inputs and controlled processes support deterministic engineering;
- decisions, requirements, evidence, and releases remain traceable;
- the smallest sufficient architecture is preferred;
- standards use precise, professional technical language.

An unspecified topic is not an implicit design decision. It remains unresolved
until an authorized decision is recorded.

## Repository layout

| Path | Purpose |
|---|---|
| [`specifications/CCA-ENG-1.0/`](specifications/CCA-ENG-1.0/) | Engineering Handbook and lifecycle governance |
| [`specifications/CCA-RF-1.0/`](specifications/CCA-RF-1.0/) | Runtime Foundation Standard and machine-readable requirements |
| [`specifications/CCA-MIP-1.0/`](specifications/CCA-MIP-1.0/) | Memory Investigation Package format, validation, examples, and conformance |
| [`specifications/CCA-MEMORYOS-1.0/`](specifications/CCA-MEMORYOS-1.0/) | Published MemoryOS platform behavior, compatibility, versioning, and conformance Standard |
| [`adrs/`](adrs/) | Architectural Decision Record process and repository template |
| [`templates/`](templates/) | Reusable standard, ADR, and requirement templates |
| [`governance/`](governance/) | Authority, change control, traceability, and publication policy |

## Standards

- [CCA-ENG-1.0 Engineering Handbook](specifications/CCA-ENG-1.0/README.md)
- [CCA-RF-1.0 Runtime Foundation Standard](specifications/CCA-RF-1.0/README.md)
- [CCA-RF-1.0 machine-readable requirements](specifications/CCA-RF-1.0/requirements.yaml)
- [CCA-RF-1.0 approved decisions](specifications/CCA-RF-1.0/decisions.md)
- [CCA-RF-1.0 conformance standard](specifications/CCA-RF-1.0/conformance.md)
- [CCA-MIP-1.0 Memory Investigation Package](specifications/CCA-MIP-1.0/MIP-001.md)
- [CCA-MIP-1.0 machine-readable schema](specifications/CCA-MIP-1.0/schema/memory-investigation-package-1.0.schema.json)
- [CCA-MIP-1.0 reference examples](specifications/CCA-MIP-1.0/examples/README.md)
- [CCA-MEMORYOS-1.0 MemoryOS Standard](specifications/CCA-MEMORYOS-1.0/README.md)
- [CCA-MEMORYOS-1.0 normative requirements](specifications/CCA-MEMORYOS-1.0/requirements.yaml)
- [CCA-MEMORYOS-1.0 conformance specification](specifications/CCA-MEMORYOS-1.0/conformance.md)

The informative
[Reference Implementation guide](specifications/CCA-MEMORYOS-1.0/reference-implementation.md)
identifies MemoryOS 1.2.0 as the initial implementation assessed against
CCA-MEMORYOS-1.0. That designation does not make implementation source,
documentation, or tests normative.

Normative statements use **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**,
and **MAY**. `MUST` and `MUST NOT` state requirements for conformance.

## Related repositories

CCA implementation repositories consume, but do not redefine, these standards:

- `cca-core` provides shared engineering foundations;
- `cca-compiler` validates canonical specifications and produces engineering
  artifacts;
- `memoryos`, `cca-studio`, `cca-sdk`, `cca-conformance`, and `cca-atlas` are
  separate architecture and implementation concerns.

Repository locations and release relationships are assigned through governance.
Their names do not authorize implementation work absent approved requirements
and architecture.

## Contributing

Read the [Engineering Handbook](specifications/CCA-ENG-1.0/README.md),
[governance policy](governance/README.md), and
[contribution guide](CONTRIBUTING.md) before proposing a change. Architectural
changes require an ADR. Requirement changes must preserve stable identifiers,
traceability, and version history.

The licensing and inbound-contribution decision remains pending. See
[LICENSE](LICENSE) before using or contributing content.
