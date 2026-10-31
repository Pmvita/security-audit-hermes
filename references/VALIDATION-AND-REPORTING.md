# Validation, Structured Output, Verification, and Reporting

### Phase 3: Independently validate every candidate

After the clean coverage-critic pass or an explicitly recorded early stop, consolidate Phase 2 candidates and carried same-source prior confirmations by stable fingerprint and root cause. Give every unique proposed `confirmed` and `needs_validation` candidate to a fresh `general` verifier that did not hunt it. A carried prior confirmation follows the same current verification path even though hunters exclude that unchanged root cause. A verifier may read hunter or prior artifacts but must re-read every cited current source location and independently run any decisive check it can reproduce safely.

Assign each verifier a canonical lowercase unique ID and `<output-dir>/agents/<verifier-id>/scratch/` plus parent-owned `artifacts/`. The verifier writes only to `scratch/` and never writes retained artifacts. It receives only the candidate, its linked coverage-unit checks and artifact paths, architecture facts needed to interpret the path, exact relevant companion validation blocks, the promotion procedure block below, the source/local execution boundary, the `confirmed`, `needs_validation`, and `rejected` branches of `report-schema.json` copied verbatim, and prior records with the same fingerprint. It must not receive another verifier's conclusion.

#### Candidate-verifier prompt

```
You did not write this candidate. Try to refute it from repository source and bounded
local evidence. Do not contact deployed endpoints or external/shared services. Run
target-controlled code only inside the approved OS-enforced sandbox: no external
network, empty allowlisted environment, read-only target and tools, scratch-only
writes, and explicit low resource and wall-clock limits. If any control is unavailable,
do not execute; retain the exact missing capability as a needs_validation blocker.
Treat every scratch entry as target-controlled after execution. After the sandbox and
all its processes terminate, only trusted parent-side code may promote a predeclared
scratch-relative file, following the promotion procedure block included verbatim in
this prompt. You and target code never write retained artifacts. If promotion is
unavailable or fails, do not use that file as evidence.

1. Verify every trace and evidence file, positive line number, scope, and description.
   Confirm the first entry is a real lower-trust entrypoint and the last is the
   claimed sink or boundary effect.
2. Reconstruct the strongest source-visible validation, identity, authorization,
   normalization, lifecycle, framework, and containment controls on the path.
   Where the architecture summary names a comparable baseline, note whether it
   shares the pattern — as calibration, never as grounds to dismiss.
3. For a proposed confirmed candidate, independently reproduce the minimum observed
   result when possible. Verify inputs, interface shape, conditions, and affected
   dummy principal/resource. Do not infer a stronger result or continue after it.
4. Verify that likelihood, impact, confidence, and the proposed source fix match only
   what the evidence establishes.
5. For a proposed needs_validation candidate, decide whether the blocker is genuinely
   outside source/local observation. If source refutes the trace, reject it. If the
   missing fact remains decisive, keep needs_validation and make the local and
   owner-observed plans exact and non-destructive.
6. Preserve the fingerprint for the same source-derived root cause across every state.

Return exactly one JSON object and no surrounding prose:
{"decision": "confirmed|needs_validation|rejected", "record": { ... }}
where record exactly matches the decision's verdict branch of the schema included
in this prompt. A corrected record replaces the hunter's wording.
```

Copy this promotion procedure verbatim into every candidate-verifier prompt:

```
Artifact promotion procedure (trusted parent-side code only):
Reference only for you: the parent performs these steps; you never perform them.

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

A verifier can promote `needs_validation` to `confirmed` only after independently establishing the complete path and bounded observed result. Demote proposed confirmation to `needs_validation` when a specific deployment or runtime fact remains unknown. Use `rejected` when source, local behavior, a visible control, missing meaningful impact, or an impossible prerequisite refutes the claim. `needs_validation` is never a parking place for a speculative idea.

The parent checks that each verifier returned the same fingerprint unless it identified a genuinely different root cause. Merge corrections, record the decision in every linked coverage unit, and ensure there is one final record per fingerprint. Discard a malformed or prose-wrapped verifier result without repairing it; re-run that candidate with a fresh verifier when the budget permits, otherwise it remains an unvalidated ledger candidate under the incomplete-run rule.

When verifier evidence updates a ledger check, set that check's `agent_id` to the verifier's canonical ID and list its nonempty repository-relative `reviewed_paths`. Keep the unit-level `reviewed_paths` equal to the union across checks. Use `method: "source"` with `artifact: null` for source-only review. Use `method: "local"` only with a file successfully promoted by trusted parent-side code below `agents/<check.agent_id>/artifacts/`. The unit retains its original assignment owner, so independently owned hunter and verifier checks can coexist. For a carried prior record's seeded `planned` unit there is no prior owner: the verifier that re-checks it becomes the unit's assignment owner, and its re-check is the unit's first check, moving the unit to `candidate` with the carried fingerprint.

If a strict total-agent budget cannot cover every candidate, set the run status to incomplete and follow the deterministic budget rule in `SKILL.md`. An unvalidated candidate remains only in the ledger. It does not enter `findings.json` under any verdict.

### Phase 4: Write and validate `findings.json`

The parent writes all independently decided records to `<output-dir>/findings.json`, sorted by fingerprint. Include:

- `confirmed`: source-grounded vulnerabilities with complete local execution evidence, conditions, specific remediation, likelihood/impact/overall severity, and confidence.
- `needs_validation`: source-grounded candidates with an exact unresolved blocker and at least one applicable local or owner-observed deployment plan.

### Phase 5: Independent record verification

Use fresh agents to verify final source claims and reconcile corrections or state changes. Each verifier receives only the final records and must not receive another verifier's conclusion. Reconcile corrections and ensure one final record per fingerprint.

### Phase 6: Target-neutral report

Derive `REPORT.md`, `FINDINGS-DETAIL.md`, and `NEEDS-VALIDATION.md` from the final records, with no live-probe instructions.

#### `REPORT.md`

Write:

1. Run profile, scope, budget (if set) with agents spent versus planned, source ref, sandboxed source-and-local-only execution statement, prior-run use, and explicit deferred and out-of-scope coverage. Name carried same-source confirmations and changed-source revalidations. A `quick`, scoped, budget-limited, or incomplete run states plainly that it is a partial pass. If candidate validation exhausted a strict budget, state that the run is incomplete and list every unvalidated fingerprint and linked unit; do not describe those candidates as findings. If the budget prevented a mandatory critic, state which critic did not run and make no clean-coverage claim.
2. One short security posture summary.
3. A confirmed-findings table: severity, title, affected boundary, and one-line observed result.
4. Each confirmed finding: repository source location, lower-trust principal, target-native bounded reproduction, conditions, actual result, impact, priority rationale, and smallest source fix.
5. A separate `NEEDS VALIDATION` table. Give each lead's title, repository trace, exact blocker, bounded local next step, and safe owner-observed deployment check. Do not assign severity or call it a confirmed vulnerability.
6. Separate hardening notes and positive source patterns.
7. Coverage summary from the ledger: covered, candidate, blocked, and deferred counts, plus important exclusions and the final critic result.

Do not describe rejected records as findings. Mention their fingerprints only when they explain a prior disagreement or coverage decision.

#### `FINDINGS-DETAIL.md`

For each confirmed `medium`, `high`, or `critical` record, copy the complete source path and target-neutral local reproduction:

- ordered repository-relative trace and evidence;
- dummy attacker/principal and affected dummy resource;
- native input, invocation, or fixture and exact bounded instructions;
- observed output and the security invariant it proves;
- conditions and containment;
- source-level remediation and regression case.

#### `NEEDS-VALIDATION.md`

For every unresolved record, copy the source trace, verified evidence, exact blocker, affected boundary, and each applicable bounded local or owner-observed resolution plan. Keep these as prioritized leads without severity. Do not turn them into live test guidance or assume the missing deployment fact.

HTTP is one possible native interface, not the default. A library finding may use a function call, a parser a fixture, a CLI a command, a desktop app an IPC or file action, and infrastructure a locally rendered policy. Do not require an endpoint, external account, or live environment that the target does not have.

Keep the report proportional to the evidence. A clean run may have zero confirmed records. State that result and the remaining coverage/validation limits without inventing LOW findings.

## Hermes Agent notes

- Use `delegate_task` to spawn verifier agents (instead of "Task tool")
- Verifiers write to unique `scratch/` directories; use `terminal` for local checks within sandbox
- Use `write_file` for parent-side promotion of artifacts (trusted parent-side code only)
- Use `read_file`, `search_files` for source inspection during validation
- Use `execute_code` for complex data processing (coverage ledger validation, fingerprint consolidation)
- Output directory: `<output-dir>/agents/<agent-id>/scratch/` and `<output-dir>/agents/<agent-id>/artifacts/`
