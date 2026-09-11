# MemoryOS Compatibility Guide

**Standard:** CCA-MEMORYOS-1.0  
**Status:** Normative

## Purpose

This document defines compatibility without requiring implementation identity.
Compatibility concerns observable behavior, accepted data, deterministic
outcomes, and public contracts; it does not compare source code, layout, or
internal algorithms.

## Compatibility dimensions

- **Behavioral compatibility:** equivalent standard-defined inputs and initial
  state have equivalent acceptance, rejection, identity, order, lifecycle,
  report, result, diagnostic, and atomicity outcomes.
- **Backward compatibility:** a newer contract implementation continues to
  accept every valid older-contract input it claims to support with its older
  meaning preserved.
- **Forward compatibility:** only behavior explicitly defined by the governing
  contract is guaranteed. Unknown closed-contract members or versions are not
  silently accepted.
- **MIP compatibility:** governed exclusively by CCA-MIP-1.0 section 20 and
  `CCA-MIP-053` through `CCA-MIP-058`.

## Normative requirements

### CCA-MOS-COMP-001 — Behavioral compatibility

Two implementations or releases claiming behavioral compatibility **MUST**
produce equivalent standard-defined observable outcomes for the same explicit
inputs and initial state; implementation, renderer, layout, timing, and pixel
similarity **MUST NOT** establish behavioral compatibility.

### CCA-MOS-COMP-002 — Backward-compatible evolution

A backward-compatible revision **MUST** continue to accept and preserve the
meaning and deterministic outcomes of every input valid under the earlier
claimed contract; removing an accepted input or changing its observable
meaning requires an incompatible version.

### CCA-MOS-COMP-003 — Closed-contract forward behavior

A consumer **MUST** reject or report unsupported an unknown major version,
unknown required feature, or unknown member of a closed Core operation, SDK
public value or projection, CLI JSON, Regression, or Explorer contract and
**MUST NOT** guess its meaning.

### CCA-MOS-COMP-004 — MIP compatibility authority

MIP backward, forward, extension, and unknown-section behavior **MUST** follow
the exact incorporated CCA-MIP-1.0 rules, including same-major acceptance,
preservation of unknown noncritical extensions, and atomic rejection of
unknown critical extensions and unknown majors.

### CCA-MOS-COMP-005 — Independent version identities

Compatibility **MUST** be evaluated against each component's declared contract
version; equality or inequality among Standard, Reference Implementation, SDK,
CLI, Core, Adapter, artifact, report, Explorer-result, and MIP versions
**MUST NOT** imply compatibility by itself.

### CCA-MOS-COMP-006 — No silent conversion

An implementation **MUST NOT** claim compatibility by silently repairing,
normalizing, downgrading, discarding, synthesizing, or reinterpreting cognition,
package data, a report, a query, a transition, or a diagnostic.

## Informative compatibility matrix

| Boundary | Baseline | Guaranteed handling |
|---|---|---|
| Standard | 1.0 | Exact CCA-MEMORYOS-1.0 behavior |
| MIP format | 1.0.0 / major 1 | CCA-MIP same-major rules |
| Core | 1.0.0 | Exact closed contract |
| Adapter | 1.0.0 | Exact closed contract |
| SDK | 1.0.0 | Exact public facade behavior |
| CLI | 1.0.0 | Exact command and output behavior |
| Regression report | 1.0.0 | Exact closed report |
| Explorer result | 1.0.0 | Exact closed result |

## Conformance

Evidence group `MOS-EVID-COMP-001` covers prior valid vectors, unsupported
versions and members, MIP same-major and extension cases, behavioral parity,
and inspection for silent conversion.
