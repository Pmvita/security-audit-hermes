# Vulnerability Hunting

### Phase 2: Run coverage-led hunting waves

The parent assigns `planned` ledger units to `general` agents. Use enough focused hunters to cover the units without combining unrelated boundaries. One hunter may own closely related units in one subsystem; no unit may be silently unassigned because of an agent-count limit — a unit the budget cannot reach is explicitly `deferred` with reason `budget_cannot_reserve_critics_and_validation`.

When a budget or profile caps hunter count, assign units in priority order and record the ordering rationale in the ledger. Rank by: (1) unauthenticated or lowest-trust entry surfaces before authenticated ones; (2) boundaries protecting the most valuable resources (credentials, cross-tenant data, code execution, release authority); (3) prior-run gaps, revalidation targets, and changed source before same-source re-passes; (4) units whose class historically yields confirmed findings for this target type over speculative ones. Ties break lexicographically by `coverage_id` so runs stay deterministic.

Before launch, the parent changes assigned units to `in_progress`, sets a canonical lowercase `agent_id`, and creates that agent's `scratch/` and parent-owned `artifacts/`. Hunters read source and parent-provided context, write only to their unique `scratch/`, and return one structured result through `delegate_task`. They never write retained artifacts or edit target source, `architecture.md`, `coverage-ledger.json`, `findings.json`, or another agent's files.

## Required hunter prompt

Every hunter prompt contains these parts in this order:

1. A two-sentence role preamble: the hunter's goal is to find source-grounded security invariant failures in its assigned units, and it must return exactly one JSON object matching the structured-result contract at the end of this prompt.
2. `architecture.md` verbatim.
3. Assigned coverage IDs, subsystem, boundary, repository-relative starting paths, and each unit's assignment block map from `coverage-ledger.json`.
4. The exact selected blocks, copied verbatim: each selected ordinary attack-class block from `ATTACK-CLASSES.md`, and from each selected companion its `Core discipline`, each chosen attack-class subsection, `Universal moves`, and `Validation rules`. Ordinary blocks are self-contained and carry no companion-style `Core discipline`, `Universal moves`, or `Validation rules` sections. Do not send block or companion names alone.
5. Explicit excluded ordinary and companion blocks with a reason for each exclusion.
6. The core hunting method below, followed by the promotion procedure block.
7. The core validation rules below.
8. Carried same-source prior confirmed exclusions, each limited to fingerprint, title, and root cause, plus peer-owned current coverage IDs that this hunter must not duplicate.
9. The unique scratch/artifact paths, safe agent ID, predeclared promotion allowlist and byte limits, and the structured-result contract, including the Structured hunter result block below and the `confirmed` and `needs_validation` branches of `report-schema.json` copied verbatim.

A prompt may select several companion blocks when the same path crosses several domains. Keep their constraints together. Scope is the hunter's coverage obligation, not permission to duplicate excluded work. If an unexpected different boundary appears, return it under `uncovered` so the parent creates a stable ledger unit and assigns it in the next wave.

#### Core hunting method — include in every hunter prompt

```
## Defensive vulnerability-finding method

Your goal is to find source-grounded security invariant failures and the smallest fix,
not to expand harm beyond the boundary result. Stay within source review and bounded local execution.
Do not contact deployed endpoints, provider APIs, registries, identity systems,
message brokers, shared services, or other users. Use local dummy data only.

READ THE CODE AT DEPTH. Follow each assigned input through parsing, identity,
authorization, normalization, state, derived copies, and the final sink. Read sibling,
legacy, batch, retry, cancellation, migration, and error paths that produce the same
effect. Compare sibling controls for equivalence, not only presence, and compare what
one component guarantees with what the next component assumes.

WORK FROM A CONCRETE INVARIANT:
1. Name the lower-trust principal and starting capability.
2. Name the accepted value, action, state transition, or resource selector.
3. Locate the control that should reject, bind, isolate, limit, or revoke it.
4. Trace the exact source path after that decision.
5. Stop at the smallest affected dummy record, wrong return value, process-integrity
   effect, or locally observable shared-resource effect.
6. State a source-level change and regression case that enforce the invariant.

DEPTH BOUND: trace only paths that can reach your assigned boundary or whose
guarantees that boundary relies on. Stop a line of investigation as soon as the
invariant is settled either way, and record the result in your structured output —
a covered, candidate, or blocked disposition, or an `uncovered` entry — instead of
continuing to search.

TEST SAD PATHS AND DISAGREEMENTS. Check absent, empty, zero, negative, maximum,
over-limit, duplicate, mixed encoding, stale, revoked, reordered, concurrent,
partially migrated, failed dependency, and rollback state only where the interface
accepts them. Compare canonicalization and units at every parser or policy handoff.
For multi-step issues, treat each output as a prerequisite and do not assume a later
boundary. If any prerequisite is not established, record a blocker.

When a proposed high or critical candidate reveals a reusable root cause, search paths
owned by the assigned coverage IDs for lexical, structural, and logical variants.
Consolidate the same root cause, but establish each variant's conditions and impact
independently. Do not investigate peer-owned units. Return a variant with no current
coverage unit as `uncovered`.

USE THE NARROWEST LOCAL CHECK THAT SETTLES THE CLAIM. Target-controlled builds,
tests, processes, browsers, emulators, fuzzers, and fixture processing may run only
inside the parent-approved OS-enforced sandbox. It must disable external networking,
start from an empty allowlisted environment, expose target and tools read-only,
permit writes only to your scratch directory, and apply low CPU, memory, process,
file-size, disk, and wall-clock limits. Isolated loopback is allowed only for a
local fixture. If any control is unavailable, do not execute: return needs_validation
with that exact blocker. Prefer an existing unit test, minimal function harness,
dummy-tenant service call, small malformed fixture, deterministic race schedule,
or locally rendered policy. Do not install or fetch tools.

Record the exact input, command, limits, and minimum result. For the environment,
record only allowlisted variable names and safe non-secret values needed to reproduce
the check. Never capture the ambient environment, inherited variables, credentials,
authentication state, or unrelated host paths. The target-controlled process writes
only in scratch. After the sandbox and all its processes terminate, only trusted
parent-side code may promote predeclared scratch-relative files, following the
promotion procedure block included verbatim in this prompt. You and target code never
write retained artifacts. If promotion is unavailable or fails for decisive evidence,
return needs_validation with the exact promotion blocker.
Never stress availability, invoke a live target, use a real credential, publish an
artifact, or continue past the minimum observed effect.

A deployment, browser, provider, broker, OS, proxy, package, secret, or identity fact
outside source is not proof either way. If one such fact is decisive, return a
needs_validation record with the exact missing observation and safe owner-observed check.
```

#### Promotion procedure — copy this promotion procedure verbatim into every hunter prompt

```
Artifact promotion procedure (trusted parent-side code only):

Before execution, the parent opens and retains trusted, non-inheritable directory
descriptors for the agent's scratch/ and artifacts/ roots, and records an allowlist
of expected scratch-relative artifact files plus explicit per-file and cumulative
byte limits. Never pass those descriptors to the agent or sandbox. After the sandbox
and all its processes terminate, trusted parent-side code promotes each allowlisted
file separately:

1. Validate the declared relative path: reject absolute, empty, `.`, `..`, or
   symlinked components.
2. Walk each parent component from the retained scratch-root descriptor with
   no-follow directory-relative operations; never reopen by path.
3. Open the leaf no-follow and nonblocking.
4. Verify with `fstat` that it is a regular file with link count exactly one and
   within the recorded per-file and cumulative byte limits.
5. Enforce those limits again while reading from that descriptor.
6. Copy exactly the verified size, repeat `fstat`, and reject a changed identity,
   type, link count, or size.
7. For the destination, walk every parent component from the retained
   artifacts-root descriptor with no-follow directory-relative operations; require
   each existing component to be a real directory, and create any missing directory
   exclusively before reopening and verifying it no-follow.
8. Create the leaf exclusively without following links, verify that the opened
   destination is a regular file with link count exactly one, and copy from the
   verified source descriptor without reopening either path.
9. Use equivalent race-safe APIs on non-POSIX systems.
10. Never recursively copy or glob scratch, extract an archive into artifacts, or
    open or promote a symlink, FIFO, socket, device, directory, hard-linked file,
    changing file, or file that exceeds its bound.
11. If any check is unavailable, cannot be enforced, or fails, discard the scratch
    entry; if it is decisive evidence, retain `needs_validation` with the exact
    promotion blocker.
```

#### Coverage-critic waves

The parent uses a coverage critic after each wave to find gaps and reassignments. The critic returns `missing_units` (new coverage units) and `reassign_ids` (units needing reassignment). For each accepted `missing_unit`, the parent creates a stable ledger unit and adds it to the current work queue. For each legitimate `reassign_id` with live `blocked`, `covered`, or `candidate` evidence, append that exact terminal record to the unit's `attempts` with the critic's source-backed `reassignment_reason`, increment `wave`, and set the live state to unassigned `deferred` with empty evidence and reason `quick_profile_final_critic` (for quick profile) or reassign to a fresh owner.

The run profile bounds this loop. A `quick` run has exactly one hunter wave followed by exactly one final critic pass. Add each accepted `missing_unit` to the current ledger and mark it `deferred` with reason `quick_profile_final_critic`. For each legitimate evidence-bearing `reassign_id`, archive the live terminal state in `attempts`, increment `wave`, and set the live state to unassigned `deferred` with empty evidence and reason `quick_profile_final_critic`. Do not launch a second hunter wave or another critic. In a scoped run, the critic still reports out-of-scope gaps it notices, but the parent records them as `out_of_scope` with the critic's reason instead of assigning them. The early-stop rule above is the same mechanism: `quick` is a pre-declared early stop, not evidence of complete coverage.

A budget bounds it the same way. Before assigning each wave, compare remaining budget against its hunter count, the validation reserve, the immediate post-wave critic, and the retained final-clean critic (`quick` reserves only its single final post-wave critic). Shrink the hunter wave to fit, taking units in priority order. If those mandatory reserves do not fit, launch no hunter from that wave and mark its planned units `deferred` with reason `budget_cannot_reserve_critics_and_validation`. Critic-proposed units enter the same ranked queue rather than extending the budget. If surviving candidates exceed the validation reserve, follow the incomplete-run rule in `SKILL.md`: stop hunting, validate in fingerprint order, retain unvalidated units as unresolved candidates, and never present them as findings.

### Structured hunter result

Each hunter returns one JSON object with:

- `coverage_ids`: array of coverage IDs assigned
- `status`: `covered` | `candidate` | `blocked` | `deferred`
- `reviewed_paths`: array of repository-relative paths reviewed
- `local_checks`: array of checks performed (each with `agent_id`, `reviewed_paths`, `method`, `artifact`)
- `result_fingerprints`: array of candidate fingerprints (nonempty only for `candidate` status)
- `uncovered`: array of unexpected boundaries found (each becomes a new ledger unit)
- `unresolved`: array of unresolved items (nonempty for `blocked`/`deferred`)

## Hermes Agent notes

- Use `delegate_task` to spawn hunter agents (instead of "Task tool")
- Hunters write to unique `scratch/` directories; use `terminal` for local checks within sandbox
- Use `write_file` for parent-side promotion of artifacts (trusted parent-side code only)
- Use `read_file`, `search_files` for source inspection during hunting
- Output directory: `<output-dir>/agents/<agent-id>/scratch/` and `<output-dir>/agents/<agent-id>/artifacts/`
