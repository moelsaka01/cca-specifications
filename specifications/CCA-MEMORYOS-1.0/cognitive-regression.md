# Cognitive Regression Behavior

**Standard:** CCA-MEMORYOS-1.0  
**Report kind:** MemoryOSCognitiveRegressionReport  
**Report version:** 1.0.0  
**Status:** Normative

## Purpose

Cognitive Regression answers whether deterministic cognition differs between
two investigations. It reports observed facts and never explains, predicts,
scores, ranks, summarizes, or infers meaning.

## Closed report contract

Categories occur exactly in this order:

1. `replay`
2. `reflection`
3. `evidence`
4. `retrieval`
5. `evolution`
6. `verification`
7. `transition`
8. `lifecycle`

Category status is `identical` or `changed`. Difference change is `added`,
`removed`, or `modified`. Overall status is `identical` or
`regressionDetected`.

The report contains exactly `kind`, `version`, `identifier`, `baseline`,
`candidate`, `categories`, `regressionDetected`, and `overall`. Each source
descriptor contains exactly `sourceIdentifier`, `sourceKind`, and
`workspaceIdentifier`. `sourceKind` is `native` or `mip`; source identity is
the native Investigation identifier or MIP manifest package identifier.
The complete standardized source descriptor is part of operand truth. A
native Investigation identifier is therefore not an excluded local alias. A
local handle used to import a MIP is excluded because the MIP descriptor uses
the package identifier instead.

Each category contains exactly `category`, `status`, and `differences`. Each
difference contains exactly `change`, `subject`, nullable `beforeDigest`, and
nullable `afterDigest`. Added has only an after digest, removed has only a
before digest, and modified has two unequal digests.

## Exact fact projection

Regression projects facts from the current immutable Investigation state as
follows. "Current MIP Observation" means the Core-selected source-authored
Observation; no package section is recomputed.

| Category | MIP facts | Native facts |
|---|---|---|
| `replay` | Every source-authored package Replay | The active Replay, when present |
| `reflection` | Current MIP Observation records whose role is `reflection` | Nonaggregate, nondetail world nodes with family `Reflection` and kind `reflection` |
| `evidence` | Current MIP Observation records whose role is `evidence` | Nonaggregate, nondetail world nodes with family `LongTermMemory` and kind `long-term` |
| `retrieval` | Current MIP Observation records whose role is `retrieval` | Nonaggregate, nondetail world nodes with family `Retrieval session` and kind `retrieval` |
| `evolution` | Semantic-transformation records, relationships, and source-authored Evolutions | Semantic/Episodic/Procedural transformation nodes, noncontainment relationships, and the active Evolution |
| `verification` | Core verification state and exact package verification | Core verification state and null package verification |
| `transition` | Every authoritative Core transition | Every authoritative Core transition |
| `lifecycle` | Current lifecycle | Current lifecycle |

Replay subject is `{identifier}`. Replay value is `{artifact, state}`, where
state is the active Replay state only when identifiers match, otherwise null.

For an MIP Reflection, Evidence, or Retrieval record, subject is the exact
record `reference` and value is the complete record. For a corresponding
native node, subject is `{family, identifier, key, kind}` and value adds
`revision`, using null when absent.

An MIP semantic-transformation fact has subject
`{artifact:"semanticTransformation", reference}` and the complete record as
value. A native transformation is a nonaggregate, nondetail node whose family
is `SemanticMemory`, `EpisodicMemory`, or `ProceduralMemory` and whose kind is
respectively one of `semantic`, `episodic`, or `procedural`; its subject is
`{artifact:"semanticTransformation", key}` and its value contains `family`,
`identifier`, `key`, `kind`, and nullable `revision`.

An MIP relationship has subject `{artifact:"relationship", reference}` and
the complete relationship as value. A native relationship excludes relation
`contains`; its subject is `{artifact:"relationship", key}` and its value is
the complete relationship with `key` and presentation-only `flowKind`
removed.

An Evolution fact has subject `{artifact:"evolution", identifier}` and value
`{active, artifact}`. `active` is true exactly when it is the active Evolution.
For MIP input, artifact is the complete source-authored Evolution. For native
input, artifact contains exactly `differences`, `from`, `identifier`,
`sessionIdentifier`, `to`, and `workspaceIdentifier`.

A verification fact has subject `{kind:"verification"}` and value
`{core, package}`. Core is null when no Core verification exists; otherwise it
contains exactly `checks` and `status`. Package is exact MIP verification for
MIP source and null for native source.

A transition fact has subject `{index}` and value `{kind, payload}`. Created
payload is normalized to `{sourceKind}`; Observed payload to `{operation,
query, resultCode, snapshot}`; Package Imported payload to
`{cognitionDigest}` using the package integrity value; every other payload is
used exactly. A reported transition difference augments its subject with
`transition:{afterAction, afterKind, beforeAction, beforeKind}`, using null for
an absent value. Lifecycle has subject `{kind:"lifecycle"}`, value `{state}`,
and a reported difference augments its subject with
`lifecycle:{afterState,beforeState}`.

## Comparison and digest construction

Facts are uniquely indexed by the RFC 8785 canonical JSON text of their
subject. The union of subject texts is ordered by ascending UTF-16 code-unit
sequence, with a proper prefix before the longer string. Equal subject texts
are invalid within one fact set.

For category `C` and fact value `V`, the digest is:

```text
"sha256:" + lowercase_hex(SHA-256(
  UTF8("MIP-1") || NUL ||
  UTF8("INVESTIGATION-CORE-REGRESSION-FACT-1.0") || NUL ||
  UTF8(C) || NUL || UTF8(JCS(V))
))
```

For a matched subject, equal digests produce no difference. Otherwise the
presence of each side selects added, removed, or modified. Category order and
subject order define final difference order.

Report identity material contains, in member-name canonical order,
`baseline`, `candidate`, `categories`, `overall`, and `regressionDetected`.
The identifier is `regression:` followed by the lower-case SHA-256 hex from
the same construction using domain
`INVESTIGATION-CORE-REGRESSION-1.0` and `JCS(identity material)` as its sole
post-domain part. Validation recomputes every status and this identifier.

Any published Cognitive Regression Report JSON Schema is a nonnormative
structural projection. It cannot replace the semantic comparison,
source-binding, digest, ordering, and validation rules in this document.

## Normative requirements

### CCA-MOS-REG-001 — Read-only compatible operands

Regression **MUST** compare two immutable investigations from the same
Workspace and the same source kind and **MUST** reject a Workspace or source
kind mismatch without changing either Investigation, transition log, or
lifecycle.

### CCA-MOS-REG-002 — Exact report identity

A successful report **MUST** have kind
`MemoryOSCognitiveRegressionReport`, version `1.0.0`, and a deterministic
content-derived identifier.

### CCA-MOS-REG-003 — Closed ordered categories

Every report **MUST** contain exactly the eight categories in the fixed order
listed in this document, with each category classified only as `identical` or
`changed`.

### CCA-MOS-REG-004 — Closed ordered differences

Differences **MUST** use only `added`, `removed`, or `modified`, retain exact
canonical subject identity, and be ordered by canonical subject identity
within their fixed category.

### CCA-MOS-REG-005 — Digest-only facts

Before and after values **MUST** be domain-separated SHA-256 fact digests, with
the absent side represented as null, and the report **MUST NOT** duplicate
semantic payload or include an explanation, score, severity, recommendation,
layout, or renderer value.

### CCA-MOS-REG-006 — Authoritative fact basis

Regression **MUST** compare only Investigation Core truth: verified cognition
digest for imported MIP cognition and accepted Observation plus deterministic
artifact, verification, transition, and lifecycle facts for native cognition;
local MIP import aliases, package metadata, noncritical extensions, and
presentation state **MUST NOT** affect the result.

### CCA-MOS-REG-007 — Overall result

`regressionDetected` **MUST** be true exactly when at least one category is
`changed`, and `overall` **MUST** be `regressionDetected` exactly in that case
and `identical` otherwise.

### CCA-MOS-REG-008 — Determinism and atomic failure

Equivalent operand truth, including equal standardized source descriptors,
**MUST** produce byte-equivalent canonical report content, and unknown,
incompatible, or invalid operands **MUST** fail without a partial report using
the applicable stable Core error.

## Conformance

Evidence group `MOS-EVID-REG-001` covers identical, added, removed, modified,
all-category, ordering, digest-domain, Workspace/source-kind, immutable,
package-neutrality, presentation-exclusion, deterministic repeat, and atomic
failure cases.
