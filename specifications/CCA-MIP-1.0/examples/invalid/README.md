# Invalid MIP-001 Vectors

## Purpose

This catalog defines deterministic negative cases. A conforming importer
applies each mutation to a valid reference fixture, rejects the complete
package, reports the listed primary code, and publishes no partial state.

| Vector | Mutation | Primary code |
|---|---|---|
| `invalid-encoding` | Encode `minimal-observation.mip` with malformed UTF-8. | `INVALID_ENCODING` |
| `invalid-json` | Remove the final `}`. | `INVALID_JSON` |
| `duplicate-member` | Add a second top-level `kind` member before parsing. | `DUPLICATE_MEMBER` |
| `noncanonical-whitespace` | Insert one insignificant space into otherwise valid JSON. | `NON_CANONICAL` |
| `unsupported-major` | Change `formatVersion` to `2.0.0`, then recompute syntactic hashes. | `UNSUPPORTED_VERSION` |
| `missing-section` | Remove `replays`. | `MISSING_REQUIRED_SECTION` |
| `unknown-core-member` | Add top-level `camera`. | `UNKNOWN_CORE_MEMBER` |
| `checksum-mismatch` | Change one semantic scalar without updating integrity. | `CHECKSUM_MISMATCH` |
| `workspace-mismatch` | Change one Observation Workspace and recompute integrity. | `WORKSPACE_MISMATCH` |
| `duplicate-identity` | Duplicate one record typed reference. | `DUPLICATE_IDENTIFIER` |
| `dangling-reference` | Point a relationship endpoint at an absent record. | `DANGLING_REFERENCE` |
| `invalid-provenance` | Give an evidence record a source, use a wrong-role source, reuse a provenance relationship, or create a provenance cycle while keeping all references resolvable. | `INVALID_PROVENANCE` |
| `trace-order` | Swap two Trace steps and retain their indices. | `INVALID_TRACE` |
| `replay-mismatch` | Remove one Replay relationship step and close the indices. | `INVALID_REPLAY` |
| `evolution-mismatch` | Remove a real added-evidence difference. | `INVALID_EVOLUTION` |
| `comparative-mismatch` | Mark a divergent moment as `shared`. | `INVALID_COMPARATIVE_RECONSTRUCTION` |
| `critical-extension` | Add an unknown `critical: true` extension, declare it in `features.required`, and recompute all digests. | `UNSUPPORTED_CRITICAL_EXTENSION` |
| `prohibited-ui` | Add camera coordinates in a noncritical extension payload, declare it in `features.optional`, and recompute all digests. | `PROHIBITED_CONTENT` |
| `prohibited-media` | Add base64 screenshot data in a semantic revision. | `PROHIBITED_CONTENT` |
| `prohibited-runtime` | Add an event-bus handle in a declared noncritical extension payload and recompute all digests. | `PROHIBITED_CONTENT` |
| `resource-limit` | Declare or supply input larger than configured preflight policy. | `RESOURCE_LIMIT_EXCEEDED` |

For semantic mutations, recompute cryptographic digests unless the vector is
specifically testing checksum failure. This ensures the intended semantic
validation phase, not an earlier checksum phase, supplies the primary code.
