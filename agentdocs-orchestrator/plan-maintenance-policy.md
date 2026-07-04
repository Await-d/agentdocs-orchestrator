# Plan Maintenance Policy: Keep Implementation Plans Alive

Agents often finish the work but forget to maintain the implementation plan. This
policy makes plan maintenance a required completion gate, not an optional
documentation cleanup.

## Principle

The workflow implementation plan is the source of coordination truth. It must
match the actual work before any task, phase, or workflow is reported complete.

## Required Maintenance Moments

The coordinator must update the implementation plan:

- after every atomic task finishes
- after any task fails, is retried, or is abandoned
- when discovery changes the task graph
- before reporting partial progress
- before final output, archive, memory sync, or cleanup

Workers may report suggested plan changes in result files, but the coordinator
owns shared workflow and `master_plan.md` edits.

## Completion Gate

Before saying a task is done, perform this sequence:

1. Locate the matching task id in `## Implementation Plan`.
2. Update the checkbox/status marker to the real final state.
3. In full orchestration mode, update the matching `master_plan.md` row.
4. Append a short note when the actual work differs from the original plan.
5. Add newly discovered follow-up tasks if they are required for the user's goal.
6. Mark obsolete tasks as abandoned or removed with a reason instead of leaving
   them pending.

If no matching task id exists, create one or add a note explaining why the work
was unplanned. Do not report completion while the implementation plan omits the
completed work.

## Drift Handling

Plan drift is expected. Silent drift is not allowed.

| Drift | Required action |
|-------|-----------------|
| New required work discovered | Add a new pending task with dependencies |
| Planned work no longer needed | Mark abandoned/removed and explain why |
| Scope changed by user | Record the new scope in notes and update tasks |
| Task completed differently than planned | Keep the task id, update notes with actual approach |
| Partial completion | Leave task unchecked or split completed and remaining work |

## Maintenance Note Format

Use concise notes:

```markdown
## Notes
- Plan maintenance: T-03 completed; added T-05 for discovered cleanup blocker.
- Plan maintenance: T-04 abandoned because T-02 made it obsolete.
```

## Manual Self-Check

Before final response:

- every completed unit of work has a checked workflow marker
- no completed work is absent from the implementation plan
- no pending task is actually done, obsolete, or blocked without a note
- full orchestration rows match workflow markers
- final response matches the maintained plan state
