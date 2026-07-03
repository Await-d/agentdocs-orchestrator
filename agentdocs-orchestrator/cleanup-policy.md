# Cleanup Policy: Protected Lifecycle Operations

Cleanup is a coordinator-only lifecycle operation. It is not a normal worker task,
because it can delete evidence, hide failed attempts, or make workflow state
impossible to audit.

## Cleanup Types

| Type | Target | When |
|------|--------|------|
| Runtime cleanup | `.agentdocs/runtime/<task-id>/` | After completion or archival gates pass |
| Workflow archive cleanup | `.agentdocs/workflow/<task-id>.md` and `index.md` | After all task states are resolved |
| Temporary artifact cleanup | locks, partial outputs, scratch files | After no task is running |
| Orphan runtime cleanup | runtime without active workflow | Report via doctor first |
| Stale lock cleanup | expired or ownerless lock files | After coordinator verifies no active writer |

## Gate Types

Cleanup uses different gates depending on the cleanup type. Do not apply final
runtime cleanup gates to temporary recovery cleanup.

### Archive Readiness Gates

Archival is allowed only when all conditions are true:

- no task in the current workflow is `running`
- all tasks are `completed` or intentionally `abandoned`
- workflow TODOs and runtime `master_plan.md` rows are synced
- full orchestration mode has a generated `final_output.md`
- memory sync is recorded as `completed`, `skipped`, or `not applicable`
- retry and failure evidence has been preserved

If any archive gate fails, archival must stop and the blocker must be recorded in
workflow notes.

### Final Runtime Cleanup Gates

Runtime deletion is allowed only when all archive readiness gates are true and:

- the cleanup target is exactly the current task runtime path
- no result, retry, error, or final-output evidence is still needed from runtime
- the workflow has been archived or is being archived in the same coordinator step

If runtime cleanup is blocked after archival, keep the runtime and record the
blocker in the archived workflow notes or `.agentdocs/index.md`.

### Temporary Recovery Cleanup Gates

Temporary artifacts may be cleaned during active workflows when all conditions are
true:

- the coordinator has verified the artifact is stale, partial, or no longer owned
  by an active writer
- deleting it cannot hide a failure cause
- any useful failure detail has been copied to the error log
- the cleanup target is an exact file path, not a broad runtime glob

## Runtime Cleanup

Allowed:

```bash
rm -rf ".agentdocs/runtime/<task-id>"
```

Forbidden:

```bash
rm -rf ".agentdocs/runtime/*"
rm -rf ".agentdocs/runtime/"
find .agentdocs/runtime -type d -delete
```

The coordinator must resolve `<task-id>` from the active workflow being archived
or completed. Never infer cleanup targets from broad glob patterns.

## Evidence Retention

Do not delete or overwrite:

- failed attempt outputs
- retry attempt files
- error logs
- final output used for memory sync
- abandoned task rationale

When retrying a task, write attempt-specific files if the default result path
would overwrite prior evidence, for example:

```text
.agentdocs/runtime/<task-id>/results/agent-03-attempt-1.md
.agentdocs/runtime/<task-id>/results/agent-03-attempt-2.md
.agentdocs/runtime/<task-id>/results/agent-03-result.md
```

## Temporary Artifact Cleanup

Temporary artifacts may be cleaned after:

- all writers are inactive
- partial files are not needed to explain a failure

Examples:

- stale lock files
- `.tmp` aggregation files
- partial result drafts

If a temporary file explains a failure, move its key details into the error log
before deleting it. Temporary cleanup does not require all workflow tasks to be
completed.

## Orphan Runtime Handling

An orphan runtime is a `.agentdocs/runtime/<task-id>/` directory without a matching
active workflow.

Default behavior:

1. Report it via `agentdocs doctor`.
2. Determine whether the matching workflow was archived, deleted, or renamed.
3. Preserve the runtime until the coordinator confirms it is not needed.
4. Clean only the confirmed orphan runtime path.

Doctor may recommend cleanup, but it must not silently delete orphan runtimes.

## Archive Cleanup

Archival must happen before runtime deletion:

1. Verify lint, state-machine gates, and archive readiness gates.
2. Record memory sync decision.
3. Move workflow to `.agentdocs/workflow/done/<task-id>.md`.
4. Remove the current-task entry from `.agentdocs/index.md`.
5. Delete `.agentdocs/runtime/<task-id>/` only if final runtime cleanup gates pass.

If archive succeeds but runtime cleanup is blocked, keep the runtime and record the
blocker in the archived workflow notes or index.

## Worker Restrictions

Workers must not:

- delete runtime directories
- archive workflow docs
- edit `.agentdocs/index.md`
- remove lock files owned by other agents
- claim cleanup completion without coordinator confirmation

Workers may recommend cleanup in result files.
