# ADR-NNNN: {Decision title}

Use this template for a proposed architecture decision, then store the
completed record under [`adrs/`](../adrs/README.md) using the filename
`ADR-NNNN-short-title.md`. Do not treat a draft or proposed ADR as approval to
implement.

## Metadata

| Field | Value |
|---|---|
| Status | Proposed |
| Date | {YYYY-MM-DD} |
| Deciders | {Project-defined approving role or group} |
| Affected standards | {Identifiers and links} |
| Affected requirements | {Identifiers and links} |
| Supersedes | {ADR identifiers or None} |

## Status

**Proposed**

Record only a status supported by the
[ADR lifecycle](../adrs/README.md#status-lifecycle). Include a link to approval
evidence when the status becomes Accepted.

## Context

Explain the problem, constraints, evidence, assumptions, and scope boundary.
Identify the normative statements and requirements that make the decision
necessary.

## Decision

State the proposed decision precisely, including dependency direction,
ownership, interfaces, invariants, failure behavior, and explicit exclusions
where applicable.

## Alternatives

For every viable alternative, record:

- description;
- advantages;
- disadvantages;
- compatibility and migration impact;
- reason it was not selected.

Include retaining the current architecture when that is viable.

## Consequences

### Positive

- {Expected benefit}

### Negative

- {Accepted cost or limitation}

### Risks and mitigations

| Risk | Mitigation | Verification evidence |
|---|---|---|
| {Risk} | {Mitigation} | {Evidence} |

### Required follow-up

- [ ] Update normative standards.
- [ ] Update affected requirement records and ADR links.
- [ ] Update implementation milestones.
- [ ] Update verification and conformance evidence.

## References

- [Specification suite](../README.md)
- [CCA Engineering Handbook 1.0](../specifications/CCA-ENG-1.0/README.md)
- [CCA Runtime Foundation Standard 1.0](../specifications/CCA-RF-1.0/README.md)
- [Governance](../governance/README.md)
- [Repository ADR template](../adrs/ADR-TEMPLATE.md)
- [Requirement template](requirement-template.yaml)
- {Additional evidence, issue, or specification link}
