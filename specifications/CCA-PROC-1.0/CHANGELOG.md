# CCA Process Foundation Changelog

All notable changes to CCA-PROC are recorded here. Entries describe
specification content, not implementation activity.

## 1.0 - Draft package for independent review

### Architecture

- Added SP-003 Process Foundation Standard.
- Established Process as an L4 Domain Engine that consumes CCA-REP-1.0
  documents and depends on CCA-RF-1.0 Runtime Foundation services.
- Defined deterministic, synchronous, side-effect-free structural execution.
- Preserved the frozen Runtime and Representation boundaries and explicit
  Process exclusions.

### Public API

- Added API-002 Process Public API.
- Added API-002-HPP as the sole public C++ declaration authority.
- Defined ProcessDefinition, ExecutionContext, ExecutionState,
  ExecutionResult, and ProcessEngine.
- Defined the required direct execution example and Runtime-hosted service
  integration.

### Requirements and implementation

- Added thirty-eight traceable IM-005 requirements.
- Added IS-005 Process Foundation Implementation Specification.
- Defined required test, build, documentation, and evidence obligations.

### Conformance

- Added the CCA-PROC-1.0 conformance target, checklists, compliance matrix, and
  verification guidance.

No C++ implementation is included in this package.
