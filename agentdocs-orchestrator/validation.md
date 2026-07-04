# Validation: Lint and Doctor Design

This document defines the validation surface for future automation. Until a CLI
exists, agents should use these checks manually before reporting completion.

## Commands

Recommended future commands:

```bash
agentdocs init
agentdocs lint [--task <task-id>]
agentdocs doctor
agentdocs sync [--task <task-id>]
agentdocs archive <task-id>
```

## `agentdocs init`

Creates the required project structure:

- `.agentdocs/index.md`
- `.agentdocs/workflow/done/`
- `.agentdocs/runtime/`
- `.gitignore` entry for `.agentdocs/runtime/`

It must not overwrite existing project memory.

## `agentdocs lint`

Validates one active task or all active workflows.

Required checks:

- workflow has all required sections from `schemas.md`
- complexity assessment has every score field
- chosen mode matches total score
- task ids are unique
- dependency references point to existing task ids
- status values match `state-machine.md`
- implementation plan maintenance follows `plan-maintenance-policy.md`
- completed workflow TODOs have matching runtime rows in full orchestration mode
- completed work is not absent from the implementation plan
- pending tasks are not actually completed, obsolete, or blocked without notes
- worker result files do not claim unsynced completion as final
- cleanup readiness follows `cleanup-policy.md`

Lint failures should be grouped by file and task id.

## `agentdocs doctor`

Checks repository-level health:

- `.agentdocs/index.md` exists
- `.agentdocs/runtime/` is ignored by git
- archived workflows with active runtime dirs have an explicit cleanup blocker note
- no runtime dir exists without a matching active workflow
- no workflow is missing from the index current-task list
- no completed workflow remains outside `workflow/done/` without explanation
- stale locks and orphan runtimes are reported without silent deletion

Doctor should report warnings separately from hard failures. An archived workflow
with an active runtime dir is a warning when a cleanup blocker is recorded, and a
failure when no blocker explains why runtime evidence remains.

## `agentdocs sync`

Synchronizes status from runtime artifacts into workflow docs.

Rules:

- Coordinator is the only writer.
- Never mark a task completed without a verified result.
- Never overwrite workflow notes; append a sync note instead.
- If runtime and workflow disagree, prefer the artifact with stronger evidence and
  record the decision in `## Notes`.

## `agentdocs archive`

Archives a workflow only when:

- lint passes for the task
- all tasks are completed or abandoned
- final output exists for full orchestration mode
- memory sync is recorded as completed, skipped, or not applicable
- archive readiness gates in `cleanup-policy.md` pass
- any runtime cleanup blocker is recorded before completion is reported

The archive command should move the workflow to `.agentdocs/workflow/done/` and
remove the matching current-task entry from `.agentdocs/index.md`.

## Manual Completion Checklist

Before reporting completion without a CLI:

- Run the equivalent lint checks by reading workflow and runtime files.
- Confirm state transitions are valid.
- Confirm workflow and master plan statuses agree.
- Confirm the implementation plan reflects completed, failed, retried,
  abandoned, newly discovered, obsolete, blocked, and unplanned work.
- Confirm memory sync decision is recorded.
- Confirm cleanup follows `cleanup-policy.md` and targets only the current runtime path.
