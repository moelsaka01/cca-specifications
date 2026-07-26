# CCA Engineering Handbook

**Identifier:** CCA-ENG-1.0  
**Version:** 1.0  
**Status:** Published  
**Publication date:** 2026-07-26  
**Scope:** Engineering governance for CCA specifications and their implementations

CCA-ENG-1.0 defines how engineering work is proposed, decided, traced, reviewed,
verified, and published. It is a process standard: it does not define CCA
architecture, runtime semantics, interfaces, or implementation behavior.

Companion material:

- [CCA Specifications](../../README.md)
- [CCA Runtime Foundation 1.0](../CCA-RF-1.0/README.md)
- [ADR template](../../adrs/ADR-TEMPLATE.md)
- [Requirement template](../../templates/requirement-template.yaml)

The terms **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** express
the strength of an engineering rule. A local process may add controls, but it
must not weaken a rule in this handbook.

## Mission

CCA engineering exists to turn reviewed architecture into trustworthy,
traceable, and reproducible releases. Engineering work must preserve a clear
line from intent to decision, from decision to requirement, and from
requirement to verification evidence.

This handbook has four objectives:

1. Keep architecture authoritative over implementation convenience.
2. Make decisions and requirements reviewable before they become commitments.
3. Make every release independently verifiable from versioned sources.
4. Keep human accountability explicit, including when AI tools contribute.

Speed is valuable only when the result remains understandable, reviewable, and
safe to change. No tool, prototype, generated artifact, or implementation
shortcut may silently establish architecture.

## Architecture First Engineering

Architecture precedes implementation. Before work changes a system boundary,
contract, dependency, quality attribute, or externally observable obligation,
the governing architecture must state the intended result.

Architecture-first work follows this chain:

1. State the problem and its constraints without presupposing a solution.
2. Identify the affected specifications, requirements, and decisions.
3. Record material alternatives and trade-offs in an ADR.
4. Update or add requirements that make the decision verifiable.
5. Obtain the required review before implementation begins.
6. Implement only the approved scope.
7. Verify the result against the approved requirements.

Exploration is permitted, but it must be labeled non-normative and must not be
treated as an accepted design. If implementation reveals that an approved
architecture is incomplete or impractical, work pauses at that boundary. The
architecture, ADR, or requirement is revised and reviewed; the implementation
does not become the de facto specification.

Generated documentation and reports are evidence derived from authoritative
sources. They do not replace those sources.

## Engineering Lifecycle

Every change moves through the following lifecycle. Phases may overlap for
small changes, but their evidence must remain distinguishable.

| Phase | Purpose | Exit evidence |
|---|---|---|
| Frame | Define the problem, users, constraints, and explicit exclusions. | Reviewed scope and affected assets |
| Architect | Evaluate material alternatives and decide system-level direction. | Approved or proposed ADRs as required |
| Specify | Express obligations as versioned, testable requirements. | Valid requirement records with traceability |
| Plan | Group approved work into a bounded milestone. | Milestone scope, dependencies, risks, and gates |
| Implement | Change only the approved sources and supporting artifacts. | Reviewable, traceable change set |
| Verify | Demonstrate conformance and engineering quality. | Passing quality gates and retained evidence |
| Publish | Freeze, identify, document, and distribute the release. | Versioned release and changelog |
| Maintain | Evaluate feedback, defects, and proposed evolution. | New scoped change or documented disposition |

A phase is not complete merely because a document or code change exists. Its
exit evidence must be reviewable, internally consistent, and linked to the
work it governs. Failed evidence returns the change to the earliest affected
phase.

## Repository Rules

The repository is a governed engineering record. Contributors MUST:

- Preserve the documented repository structure and place each artifact in its
  designated area.
- Keep authoritative sources separate from generated outputs and label both
  clearly.
- Use stable identifiers for specifications, requirements, and ADRs.
- Use relative links for repository-local references and keep them valid.
- Update related indexes and changelogs in the same change when applicable.
- Keep changes narrowly scoped; unrelated cleanup belongs in a separate change.
- Never commit credentials, private keys, access tokens, or confidential input.
- Preserve attribution, license notices, and provenance for incorporated work.
- Avoid editing generated files by hand; regenerate them from their identified
  sources.
- Keep published artifacts reproducible from versioned inputs and documented
  tools.

A specification directory owns its publication document, machine-readable
requirement registry, changelog, and supporting diagrams. A blank directory is
retained only when its purpose is explicit. Files must not be added merely to
suggest future behavior.

Repository history is part of the engineering evidence. Destructive history
rewrites on shared branches and silent replacement of published artifacts are
not acceptable.

## Architecture Governance

Architecture governance protects coherence without making decisions invisible.
A proposed architecture change MUST identify:

- the problem and decision boundary;
- affected specifications, requirements, and ADRs;
- compatibility and migration impact;
- considered alternatives and material trade-offs;
- risks, assumptions, and unresolved questions; and
- the evidence that will verify the outcome.

Editorial clarification may use normal document review when it does not alter
meaning. A change that alters boundaries, contracts, dependencies, required
qualities, compatibility, or normative behavior requires an ADR and the
appropriate architecture review.

Review authority must be explicit in the hosting project. An author may not
self-approve a material architecture decision. Reviewers evaluate consistency
with existing decisions, completeness of impacts, reversibility, and the
quality of the proposed evidence. Approval records the decision; it does not
erase dissent, alternatives, or known consequences.

When two normative sources conflict, implementation stops at the conflict. The
governing sources are reconciled through review and versioning before work
continues.

## Requirement Process

Requirements are the verifiable bridge between architecture and delivery. New
records MUST use the [requirement template](../../templates/requirement-template.yaml)
and satisfy every field that the template marks as required.

A requirement must be:

- uniquely identified and scoped to one obligation;
- stated unambiguously and without unverifiable adjectives;
- traceable to its source, rationale, and governing decision;
- testable by a named verification method or reviewable evidence;
- independent of a particular implementation unless the architecture requires
  that constraint; and
- versioned when its normative meaning changes.

The requirement author checks for overlap, contradiction, and unintended
coupling. Reviewers check necessity, clarity, feasibility, traceability, and
verifiability. A requirement is not a baseline obligation until it has passed
the review state defined by the governing process.

Changes must link to the requirements they implement or affect. Verification
evidence must identify the requirement it satisfies. Retirement or replacement
preserves history and records the successor where one exists.

The registry in this handbook package is intentionally empty. Runtime
Foundation requirements belong to
[CCA-RF-1.0](../CCA-RF-1.0/README.md), not to this process handbook.

## ADR Process

An Architecture Decision Record captures a material decision and its context.
Use the [ADR template](../../adrs/ADR-TEMPLATE.md) when a choice changes or
constrains architecture, interfaces, dependencies, compatibility, security,
operational qualities, or long-lived engineering policy.

The ADR process is:

1. Assign a stable identifier and state the decision question.
2. Record context, constraints, and decision drivers.
3. Describe viable alternatives fairly, including doing nothing.
4. Record the selected option and its consequences.
5. Link affected specifications and requirements.
6. Obtain review from the designated decision authority.
7. Record the outcome and follow-up obligations.

Rejected alternatives remain in the record because they explain the decision.
An accepted ADR is not rewritten to make history appear cleaner. Substantive
change is made by a new ADR that supersedes the earlier record; the links in
both records are updated. Minor editorial corrections must not change the
decision's meaning.

## Git Workflow

Git changes must be small enough to understand and complete enough to verify.
The authoritative branch receives changes through review.

- Create a short-lived branch from the current authoritative baseline.
- Keep commits cohesive, buildable where applicable, and written in imperative,
  explanatory language.
- Reconcile upstream changes before final review without discarding another
  contributor's work.
- Open a review that states purpose, scope, exclusions, linked requirements and
  ADRs, risks, verification performed, and publication impact.
- Resolve review findings in the change, not only in discussion.
- Do not merge while required gates fail or required decisions remain open.
- Remove the branch after integration according to repository policy.

Large changes should be split by independently reviewable outcomes rather than
by arbitrary file counts. Mechanical changes should be isolated from semantic
changes so reviewers can assess meaning.

Emergency work is not exempt from traceability. If an abbreviated review is
authorized, the missing records and evidence become explicit follow-up work
with an owner and deadline.

## Milestone Workflow

A milestone is a bounded, reviewable delivery commitment. It MUST declare:

- one measurable objective;
- included and excluded scope;
- dependencies and assumptions;
- linked requirements and ADRs;
- expected artifacts;
- risks and unresolved decisions;
- required quality gates; and
- completion criteria.

At entry, architecture and requirements are sufficiently reviewed to support
the planned work. During execution, discoveries are recorded; they do not
silently broaden scope. A scope change is re-planned and reviewed with its
impact on dependencies, evidence, and schedule.

At exit, every included item is complete or explicitly deferred, required
evidence is retained, generated artifacts are reproducible, documentation and
changelog are current, and exclusions remain excluded. A milestone is not
complete because a date arrived or because most tasks closed.

Specifications such as
[CCA Runtime Foundation 1.0](../CCA-RF-1.0/README.md) retain their own scope and
requirements. A milestone may deliver against them but may not redefine them.

## AI Engineering Policy

AI tools may assist engineering, but accountability remains human. An AI system
is neither an architecture authority nor an approver.

Contributors using AI MUST:

- provide only data authorized for the selected tool and environment;
- exclude secrets, personal data, restricted source, and confidential context
  unless explicitly approved;
- review generated content for correctness, security, licensing, provenance,
  and scope;
- verify all generated code, tests, analysis, citations, and commands;
- disclose material AI contribution when required by repository or
  organizational policy;
- preserve human-readable reasoning for architecture and governance decisions;
  and
- treat model output as untrusted input until the normal quality gates pass.

AI must not fabricate requirements, evidence, approvals, test results,
citations, or compatibility claims. It must not make unreviewed changes to
published specifications or execute destructive or externally consequential
actions without the required authority.

Human reviewers remain responsible for understanding the change. "Generated by
AI" is neither evidence of quality nor a reason to waive a gate.

## Versioning Policy

Published CCA specifications use an explicit `MAJOR.MINOR` identity, as in
`CCA-ENG-1.0`.

- **MAJOR** changes identify an incompatible change to normative meaning,
  governance, or conformance expectations.
- **MINOR** changes identify compatible additions or normative refinements that
  do not invalidate conforming use of the prior minor version.
- Editorial errata that do not change normative meaning retain the version but
  are recorded in the changelog; previously published artifacts remain
  recoverable.

Every normative change receives a new published version. References between
specifications use explicit versions rather than an ambiguous "latest."
Dependencies, schemas, examples, generated reports, and changelogs must agree
with the published identity.

A release candidate or draft must be visibly distinguished from a publication
and must not replace it in place. Deprecation identifies the supported
successor and does not erase the superseded version.

## Quality Gates

A change is eligible for publication only when every applicable gate passes:

1. **Scope:** The change satisfies its stated objective and does not introduce
   excluded behavior.
2. **Architecture:** Material decisions are approved and the change is
   consistent with governing specifications and ADRs.
3. **Requirements:** Records are valid, unique, traceable, and verifiable.
4. **Validation:** Applicable schemas and standards-compiler checks pass with
   no unresolved errors.
5. **Build and tests:** Affected build and test suites pass; coverage meets the
   approved threshold when executable code is in scope.
6. **Static quality:** Required warnings, analysis, formatting, and lint checks
   pass without suppressed failures.
7. **Documentation:** Links, examples, indexes, diagrams, and changelog are
   accurate and synchronized.
8. **Security and provenance:** Secrets, dependencies, licenses, generated
   content, and supply-chain inputs have the required review.
9. **Reproducibility:** Published outputs can be regenerated from identified,
   versioned inputs.

Gate evidence must be attributable to the reviewed change. A skipped gate
requires explicit authority, rationale, risk acceptance, and follow-up; silence
is not a waiver.

## Engineering Manifesto

We put architecture before convenience.

We make decisions visible before they become expensive.

We write requirements that can be understood and verified.

We prefer small, complete, reversible changes.

We preserve history, provenance, and the reasons behind choices.

We automate repeatable checks and keep human judgment accountable.

We treat generated output as untrusted until verified.

We surface ambiguity instead of encoding guesses.

We publish evidence, not confidence.

We leave the system easier to reason about than we found it.
