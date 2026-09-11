# Cognitive Investigation Explorer Behavior

**Standard:** CCA-MEMORYOS-1.0  
**Result kind:** MemoryOSCognitiveInvestigationResult  
**Result version:** 1.0.0  
**Status:** Normative

## Purpose

The Explorer navigates deterministic evidence already represented in one
valid Cognitive Regression Report. It does not recompute Regression, replay an
Investigation, or generate explanation.

## Closed query

A normalized query contains exactly:

| Member | Value |
|---|---|
| `category` | One fixed Regression category or null |
| `reflectionIdentifier` | One nonempty exact identifier or null |
| `transition` | One nonempty exact transition token or null |

Reflection and transition selectors are mutually exclusive. A Reflection may
be combined only with category `reflection`; a transition may be combined only
with category `transition` or `lifecycle`.

Transition normalization inserts `-` between an ASCII lower-case letter or
digit and a following ASCII upper-case letter, replaces each non-empty run of
underscore or ECMAScript whitespace with one `-`, then applies ECMAScript
locale-independent lower-casing. It does not trim, expand, or otherwise infer
a token.

Reflection selection tests the subject's non-empty `identifier` first, then
its nested `reference.identifier`, and otherwise has no identifier. Transition
selection tests only the normalized non-empty before/after kind and action
tokens in a transition subject, or before/after state tokens in a lifecycle
subject.

## Exact result construction

The result contains exactly `kind`, `version`, `identifier`,
`regressionIdentifier`, `workspaceIdentifier`, `query`, `status`,
`matchCount`, and `matches`. The normalized query retains all three members,
using null for every absent selector.

Traversal uses the Regression report's fixed category order and each
category's existing difference order. A selected difference yields one match
containing exactly `index`, `category`, `change`, `subject`, `baseline`, and
`candidate`. Index is zero-based in the result, not in the source report.

For source category index `C` and difference index `D`, the base RFC 6901
pointer is `/categories/C/differences/D`. A non-null baseline endpoint points
to `<base>/beforeDigest`; a non-null candidate endpoint points to
`<base>/afterDigest`. Each endpoint contains exactly `digest`, `pointer`,
`sourceIdentifier`, and `sourceKind`, copied from the validated report. Added
has candidate only, removed baseline only, and modified both.

The navigator constructs those endpoints from the validated source report and
therefore guarantees that every emitted pointer resolves to the exact digest
in that report. Detached result validation has no report operand: it validates
the closed endpoint member set, digest and pointer syntax, source descriptor,
change-side rules, ordering, and content identity, but it does not dereference
the pointer or independently prove source-report membership.

Result identity material contains `matchCount`, `matches`, `query`,
`regressionIdentifier`, `status`, and `workspaceIdentifier`. Its identifier is
`explorer:` followed by the lower-case SHA-256 hex of:

```text
UTF8("MIP-1") || NUL ||
UTF8("INVESTIGATION-CORE-EXPLORER-1.0") || NUL ||
UTF8(JCS(identity material))
```

Detached validation requires the exact member sets, immutable canonical
values, normalized query, contiguous indices, selected matches, deterministic
category/subject order, endpoint shape and side rules, status/count agreement,
and recomputed identity. Source-report pointer resolution is a navigator
construction guarantee and is tested with the report and result together.

Any published Cognitive Investigation Result JSON Schema is a nonnormative
structural projection. It cannot replace the semantic construction and
validation rules in this document.

## Normative requirements

### CCA-MOS-EXPL-001 — Existing-report navigation

Explorer navigation **MUST** accept one detached Cognitive Regression Report
and one closed query, validate the report before traversal, and **MUST NOT**
load, execute, replay, compare, or modify either source Investigation.

### CCA-MOS-EXPL-002 — Closed query contract

A query **MUST** normalize to exactly `category`, `reflectionIdentifier`, and
`transition` and **MUST** reject unknown members, unknown categories, empty
selectors, mutually exclusive selectors used together, or a selector combined
with an incompatible category.

### CCA-MOS-EXPL-003 — Lexical transition normalization

Transition selection **MUST** normalize ASCII case, spaces, underscores, and
camel-case boundaries to one hyphenated token and **MUST NOT** infer or expand a
transition absent from the report.

### CCA-MOS-EXPL-004 — Exact result contract

A successful result **MUST** have kind
`MemoryOSCognitiveInvestigationResult`, version `1.0.0`, a canonical
content-derived identifier, the exact Regression and Workspace identities,
status `matched` or `empty`, and a match count equal to the match sequence.

### CCA-MOS-EXPL-005 — Deterministic match order

Matches **MUST** retain fixed Regression category order followed by each
category's existing difference order, with contiguous zero-based result
indices; an exact valid selection with no match **MUST** return `empty` rather
than fail.

### CCA-MOS-EXPL-006 — Evidence endpoints

Each match **MUST** contain the unchanged category, change, and subject plus
nullable baseline and candidate endpoints that identify the report source,
source kind, existing fact digest, and RFC 6901-compatible pointer to that
digest; added has only candidate, removed only baseline, and modified both.

### CCA-MOS-EXPL-007 — No alternate cognition

An Explorer result **MUST NOT** copy semantic payload, construct a new evidence
fact, rank, summarize, explain, infer causality, or become an alternate source
of cognition; navigation **MUST** terminate at a digest already present in the
validated report.

### CCA-MOS-EXPL-008 — Deterministic atomic failure

Equivalent report and query values **MUST** produce byte-equivalent immutable
results; malformed or tampered reports **MUST** fail with
`INVALID_REGRESSION_REPORT`, invalid queries with `INVALID_QUERY`, and failure
**MUST** publish no partial result or lifecycle transition.

## Conformance

Evidence group `MOS-EVID-EXPL-001` covers complete and category navigation,
Reflection, Replay, Evidence, Retrieval, Evolution, Verification, transition,
lifecycle, empty results, transition normalization, endpoint pointer
resolution, tampering, query negatives, immutability, performance, and
deterministic repeats.
