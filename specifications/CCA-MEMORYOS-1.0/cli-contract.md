# MemoryOS CLI Contract

**Standard:** CCA-MEMORYOS-1.0  
**CLI version:** 1.0.0  
**JSON envelope version:** 1.0  
**Status:** Normative

## Purpose

The `memoryos` CLI is the scriptable automation client of the public SDK. It
does not own investigation, MIP, Regression, or Explorer behavior.

## Command surface

The exact top-level commands are:

```text
version  help  observe  trace  replay  compare  regression
investigate  verify  import  export  inspect  session
```

`checkpoint` and `restore` are stateful records inside `session`; they are not
top-level commands.

An empty invocation, or an invocation whose first token is `--help` or `-h`,
dispatches the unscoped `help` command. An invocation whose first token is
`--version` or `-V` dispatches `version`. These are first-token aliases of the
released parser; they do not create additional commands. A first-token alias
dispatches immediately, so later tokens do not alter its result.

## Complete grammar and command results

In this grammar, a bare upper-case token is one required operand; brackets
enclose an optional element; and an ellipsis permits the immediately preceding
option to repeat in source order. Options follow the command.

Every value-bearing long option accepts either `--name VALUE` or
`--name=VALUE`; the value must be non-empty. A flag accepts only its bare
`--name` spelling. A non-repeating option and a flag may occur at most once;
`--action` may repeat and retains source order. The standalone token `--`
ends option recognition, and all later tokens are positional. Before that
delimiter, an unknown `--name`, missing value, value supplied to a flag,
duplicate non-repeating option, or wrong positional cardinality is invalid
arguments. The command rows show the space-separated spelling for readability
but include both accepted value spellings through this general rule.

| Command | Exact grammar | Required behavior and success result |
|---|---|---|
| `version` | `memoryos version [--json]` | Returns CLI and SDK versions. |
| `help` | `memoryos help [COMMAND] [--json]` | Returns a concise command overview or concise syntax synopsis for one known command; an unknown topic is invalid arguments. This table, not help prose, is the complete grammar. |
| `observe` | `memoryos observe --workspace FILE --snapshot FILE [--id ID] [--operation NAME] [--query FILE] [--result-code CODE] [--json]` | Reads a Workspace JSON object with non-empty `identifier` and a complete snapshot JSON object, opens that exact Workspace, and returns Investigation metadata. Optional values are forwarded unchanged as explicit source metadata. |
| `trace` | `memoryos trace PACKAGE --reflection ID [--id ID] [--json]` | Imports one package, forwards the exact Trace selector, and returns Investigation metadata. |
| `replay` | `memoryos replay PACKAGE --trace ID [--action ACTION ...] [--id ID] [--json]` | Imports one package, forwards the exact Trace selector, opens its Replay, applies zero or more actions in argument order, and returns Investigation metadata plus Replay identity, status, and cursor. `ACTION` is exactly `play`, `pause`, `restart`, `previous`, `next`, or `advance`. |
| `compare` | `memoryos compare PACKAGE --trace ID --evolution ID [--id ID] [--json]` | Imports one package, applies the Trace, advances Replay through SDK actions until complete, and enters the exact authored Evolution. Returns Investigation metadata, Evolution identifier, deterministic Replay step count, and comparison stage. It accepts one package, not two. |
| `regression` | `memoryos regression BASELINE CANDIDATE [--json]` | Imports exactly two MIPs through one SDK client and returns the complete Regression Report. Each operand is a file or `-`; at most one operand may be `-`. Regression detected is success. |
| `investigate` | `memoryos investigate REPORT [--category CATEGORY] [--reflection ID] [--transition TRANSITION] [--json]` | Reads one raw Regression Report or successful Regression JSON envelope and returns the complete Explorer result. `REPORT` may be `-`. Category uses the fixed Regression category tokens. Reflection and transition are mutually exclusive and may only accompany their corresponding category as defined by the Explorer contract. Empty result is success. |
| `verify` | `memoryos verify PACKAGE [--json]` | Verifies one package through the SDK and returns validity, status, count, and ordered diagnostics. `PACKAGE` may be `-`. |
| `import` | `memoryos import PACKAGE [--id ID] [--json]` | Imports one package through the SDK and returns Investigation metadata. `PACKAGE` may be `-`. |
| `export` | `memoryos export PACKAGE --output FILE|- [--id ID] [--json]` | Imports one package, obtains its exact SDK-exported bytes, and writes them to the required destination. Raw standard output (`--output -`) is incompatible with `--json`. JSON mode returns byte length, destination, package kind, and package version. |
| `inspect` | `memoryos inspect PACKAGE [--id ID] [--json]` | Imports one package and returns Investigation metadata. |
| `session` | `memoryos session [WORKFLOW.memoryos] [--json]` | Processes one JSON object per non-empty input line from the optional file or standard input through one SDK client, emits results in input order, and stops at the first error. |

Investigation metadata contains exactly the observable fields `availability`,
`identifier`, `lifecycle`, `phase`, `sourceKind`, `transitionCount`,
`transitionLogDigest`, and `workspaceIdentifier`.

### Closed JSON success results

The following result member sets are exact. A string is non-empty unless the
table states otherwise. Every count is a non-negative safe integer. No
command-specific foreign member is permitted.

| Command or session record | Exact result contract |
|---|---|
| `version` | `cliVersion` and `sdkVersion`, both version strings |
| `help` | `topic` string (`memoryos` for unscoped help, otherwise the exact known command) and non-empty `usage` string |
| `observe`, `trace`, `import`, `inspect`, session `restore` | The complete Investigation metadata set above |
| `replay` | Investigation metadata plus integer `cursor` greater than or equal to -1, `replayIdentifier`, and `replayStatus` in `ready`, `playing`, `paused`, or `completed` |
| `compare` | Investigation metadata plus `evolutionIdentifier`, `replaySteps`, and `stage`; top-level `compare` reports the number of SDK Replay steps it applied, while a session comparison has no `replaySteps` member |
| `regression` | Complete immutable Regression Report defined in [cognitive-regression.md](cognitive-regression.md) |
| `investigate` | Complete immutable Explorer result defined in [investigation-explorer.md](investigation-explorer.md) |
| `verify` | `diagnosticCount`, ordered `diagnostics` array, `status`, and boolean `valid`; the count equals the array length |
| `export` | `byteLength`, exact caller-provided `output` destination, `packageKind`, and `packageVersion` |
| session `checkpoint` | Exact caller-assigned `name` and `stored` equal to true |

Investigation `availability` contains exactly the seven boolean keys defined
in [investigation-core.md](investigation-core.md). `transitionCount` is a
non-negative safe integer. `lifecycle`, `phase`, and `sourceKind` use their
closed Core values; the remaining Investigation metadata members are
non-empty strings. A successful `verify` result uses only the verifier's
deterministic status and diagnostics; a verification failure is represented
by the error path and exit code 3, not a successful false result.

### Session records

A session input record contains a `command` and the fields below. A path field
denotes a caller-provided file and is never inferred.

| Record command | Required fields | Contract |
|---|---|---|
| `version` | none | Returns CLI and SDK versions. |
| `observe` | `workspace`, `snapshot` | Opens and observes the explicit files in the live SDK binding. |
| `import` | `package` | Imports the explicit package as current Investigation. |
| `trace` | `reflection` | Applies the exact Trace selector. |
| `replay` | `action` | `action` is `open` or one of the six Replay actions; `open` acquires the prepared session without a transition. |
| `compare` | `evolution` | Enters the exact Evolution after Replay completion. |
| `checkpoint` | `name` | Stores the opaque SDK checkpoint under a process-local non-empty key. |
| `restore` | `name` | Restores the exact object stored under that key through the same SDK binding. |
| `verify` | none, or `package` | Verifies current Investigation or explicit package. |
| `export` | `output` | Exports the current package-backed Investigation to the explicit destination. |
| `inspect` | none | Returns current Investigation metadata. |

`regression` and `investigate` are standalone commands, not session records.
A session never serializes a checkpoint. A checkpoint name is only a key in
the process-local object map and is not a restoration credential.

## Human and JSON output

Human output MUST present the same command result facts in fixed field and
record order with no color, timestamp, progress, random, or locale-dependent
content. The exact whitespace of human mode is not a cross-implementation
compatibility surface.

JSON mode emits a success envelope:

```json
{"command":"verify","ok":true,"result":{},"schemaVersion":"1.0"}
```

or an error envelope:

```json
{"command":"verify","error":{"code":"VERIFICATION_FAILED","details":[],"exitCode":3,"message":"Memory Investigation Package verification failed."},"ok":false,"schemaVersion":"1.0"}
```

The success envelope contains exactly `command`, `ok` true, command-specific
`result`, and `schemaVersion` `1.0`. The error envelope contains exactly
`command`, `error`, `ok` false, and `schemaVersion` `1.0`; `error` contains
`code`, ordered `details`, `exitCode`, and `message`.

One standalone success envelope is written only to standard output. One
standalone error envelope is written only to standard error. In JSON session
mode, standard output is an ordered JSON Lines stream containing one envelope
for each successful record followed by at most one error envelope. No other
bytes may share an envelope stream. Raw package output is the sole non-JSON
success body and MUST preserve exact SDK bytes.

## Exit codes

| Code | Outcome |
|---:|---|
| 0 | Success |
| 1 | Invalid arguments |
| 2 | Validation failure |
| 3 | Verification failure |
| 4 | Package error |
| 5 | SDK failure |

Detection of a Regression and an Explorer result with status `empty` are
successful outcomes and use exit code `0`.

### Failure classification precedence

The following rows are evaluated in order; the first matching row determines
the exit code. No implementation may reclassify a later row ahead of an
earlier row.

| Priority | Observable failure | Exit code |
|---:|---|---:|
| 1 | CLI command, option, positional-argument, arity, or option-conflict grammar is invalid | 1 |
| 2 | CLI validation rejects structured input, JSON, Workspace input, Replay action, session record, session checkpoint key, or non-package input read | 2 |
| 3 | The explicit `verify` operation completes verification and reports the package invalid using `VERIFICATION_FAILED` | 3 |
| 4 | Reading or writing caller-designated package bytes fails | 4 |
| 5 | The CLI Replay completion guard reaches 10,000 SDK steps without completion, using `REPLAY_DID_NOT_COMPLETE` | 5 |
| 6 | An otherwise unclassified lower-layer failure is a supplied-value type or range violation | 2 |
| 7 | An otherwise unclassified lower-layer failure has code `ENOENT`, `EACCES`, or `EISDIR` | 4 |
| 8 | An otherwise unclassified failure originates from a MIP Producer, Consumer, or Verifier error; originates from `import`, `importPackage`, `export`, or `exportPackage`; or has code `CAPABILITY_UNAVAILABLE` | 4 |
| 9 | An otherwise unclassified lower-layer failure has code `INVALID_INPUT`, `INVALID_QUERY`, `INVALID_REGRESSION_REPORT`, `INVALID_SELECTION`, `TRACE_NOT_FOUND`, or `WORKSPACE_MISMATCH` | 2 |
| 10 | Every other lower-layer SDK or Core failure, including `NOT_FOUND` and `INVALID_TRANSITION` | 5 |

An error already classified by priorities 1 through 5 retains that
classification. Error-envelope `exitCode` and process exit status MUST be the
same selected value. The classification depends only on the observable
failure, operation, and code, never on message text or host exception type.

## Normative requirements

### CCA-MOS-CLI-001 — Executable and command identity

A conforming baseline CLI **MUST** identify as `memoryos` version `1.0.0` and
**MUST** expose exactly the top-level command set and first-token aliases in
this document.

### CCA-MOS-CLI-002 — SDK-only dependency

Every cognitive CLI operation **MUST** delegate through the public MemoryOS SDK
and **MUST NOT** directly invoke or implement Runtime, Investigation Core, MIP,
Trace, Replay, Evolution, Comparative Reconstruction, Regression, Explorer, or
renderer behavior.

### CCA-MOS-CLI-003 — Explicit noninteractive input

Commands **MUST** require their documented deterministic files, identifiers,
selectors, actions, and destinations; ordinary execution **MUST NOT** prompt,
sample ambient state, infer a missing selection, or retain state between
processes.

### CCA-MOS-CLI-004 — Human and JSON modes

Every command **MUST** support deterministic human-readable output and, except
raw package output, deterministic JSON output selected explicitly by `--json`.

### CCA-MOS-CLI-005 — Deterministic JSON framing

JSON output **MUST** be one compact UTF-8 object followed by one line-feed,
with every object recursively ordered by ascending UTF-16 code-unit member
name, proper prefix first, and SDK array order preserved. Command-specific
success results **MUST** use the exact member and value contracts in this
document. Output **MUST NOT** include ANSI color, an implicit timestamp,
randomness, progress output, or locale-dependent formatting.

### CCA-MOS-CLI-006 — Exit-code mapping

The CLI **MUST** use exactly the exit-code table and ordered failure
classification precedence in this document in both human and JSON modes, and
JSON errors **MUST** include the selected code without converting a successful
Regression or empty Explorer result into failure.

### CCA-MOS-CLI-007 — Investigation command fidelity

`observe`, `trace`, `replay`, `compare`, `verify`, `import`, `export`, and
`inspect` **MUST** pass exact explicit inputs to the SDK and preserve the SDK's
state, ordering, identifiers, diagnostics, and package bytes without semantic
reinterpretation.

### CCA-MOS-CLI-008 — Regression and Explorer fidelity

`regression` **MUST** pass two imported Investigation handles to the SDK, and
`investigate` **MUST** pass one existing Regression Report and closed query to
the SDK; neither command may compare, filter, resolve, replay, summarize,
explain, rank, or generate evidence locally.

### CCA-MOS-CLI-009 — Session isolation

`session` **MUST** process ordered JSON Lines through one isolated SDK client,
stop on the first failure, retain checkpoints only as opaque process-local SDK
objects, and **MUST NOT** serialize or restore a checkpoint across processes.

### CCA-MOS-CLI-010 — Output stream integrity

A standalone success **MUST** write its result only to standard output and a
standalone error only to standard error; raw package export **MUST** preserve
exact bytes and **MUST NOT** be combined with JSON output.

## Conformance

Evidence group `MOS-EVID-CLI-001` covers the complete command grammar, SDK-only
dependency inspection, empty and first-token aliases, concise help versus the
authoritative grammar, every exact success-result member set and value type,
all command examples, deterministic repeat output, JSON Lines framing,
stdout/stderr separation, exit codes, invalid arguments, Regression, Explorer,
package round-trip, session failure, and checkpoint isolation.
