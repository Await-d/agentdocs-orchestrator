# State Machine: Task and Workflow Status Contract

This document defines the canonical status model for `agentdocs-orchestrator`.
All workflow TODO markers, runtime `master_plan.md` rows, and final reports must use
these states consistently.

## Status Values

| State | Icon | Meaning | Owner |
|-------|------|---------|-------|
| `pending` | 🟡 | Task is ready but not started | Coordinator |
| `waiting` | ⏸️ | Task is blocked by dependencies | Coordinator |
| `running` | 🔵 | Task is actively executing | Coordinator |
| `completed` | ✅ | Task output was verified and synced | Coordinator |
| `failed` | ❌ | Task attempt failed and may be retried | Coordinator |
| `abandoned` | ⚫ | Task will not be retried | Coordinator |
| `archived` | 📦 | Workflow is complete and moved to `done/` | Coordinator |

Workers may report progress in result files, but only the coordinator writes status
changes to workflow docs or `master_plan.md`.

## Allowed Transitions

```text
pending   -> running
pending   -> waiting
waiting   -> running
running   -> completed
running   -> failed
failed    -> running      # retry
failed    -> abandoned
completed -> archived     # workflow-level only
```

Any other transition must be explained in the workflow notes before it is applied.

## Completion Contract

A task is not `completed` until all of the following are true:

- Required output exists in the expected location.
- Output was reviewed or otherwise verified against the task acceptance criteria.
- Workflow TODO marker is checked.
- In full orchestration mode, the matching `master_plan.md` row is also updated.
- The implementation plan has been maintained according to
  [plan-maintenance-policy.md](plan-maintenance-policy.md), including drift notes
  for failed, retried, abandoned, newly discovered, obsolete, blocked, or
  unplanned work.
- Dependent tasks were unblocked if their prerequisites are now satisfied.

## Failure and Retry Contract

Failed tasks must record:

- failing task id
- attempt number
- observed failure
- retry decision
- downstream impact

Retry files must not overwrite earlier evidence. Use attempt-specific names when
needed, for example `agent-03-attempt-2.md`.

## Dependency Rules

- A task with incomplete prerequisites must be `waiting`, not `pending`.
- A downstream task may move from `waiting` to `pending` only after all required
  upstream tasks are `completed`.
- If an upstream task becomes `abandoned`, downstream tasks must be reassessed and
  either replanned, abandoned, or explicitly marked as no longer dependent.

## Workflow Archival

A workflow may become `archived` only after:

- all planned tasks are `completed` or intentionally `abandoned`
- memory sync has been considered and recorded
- final output has been generated for full orchestration mode
- [cleanup-policy.md](cleanup-policy.md) archive readiness gates pass

Never archive a workflow while any task remains `running`. Runtime deletion is a
separate protected cleanup step and may be blocked after archival if evidence must
be retained.
