# Schemas: Structured Artifact Contracts

This document defines the minimum structured fields expected inside Markdown
artifacts. The files remain human-readable Markdown, but tools and agents should
treat these sections as required contracts.

## Workflow Document

Path: `.agentdocs/workflow/<task-id>.md`

Required sections:

- `## Task Overview`
- `## Current Analysis`
- `## Solution Design`
- `## Complexity Assessment`
- `## Implementation Plan`
- `## Notes`

Required complexity semantics:

```yaml
atomic_steps: number
parallel_streams: yes | no
modules_or_systems: number
long_step_over_5_min: yes | no
persisted_review_artifacts: yes | no
opencode_available: yes | no
total_score: number
chosen_mode: Direct | Lightweight | Full orchestration
routing_rationale: string
```

These are semantic fields, not a required literal YAML block. A workflow may
express them as a Markdown table, bullet list, or YAML-like block as long as each
field can be mapped unambiguously. For example, `Atomic steps`, `atomic_steps`,
and `| Atomic steps | ... |` all satisfy the same semantic field.

Required task marker format:

```markdown
- [ ] T-01: Task description
- [x] T-02 ✅: Completed task description
- [ ] T-03 ⏸️: Waiting task description
```

## Runtime Master Plan

Path: `.agentdocs/runtime/<task-id>/master_plan.md`

Required sections:

- `## Workflow Reference`
- `## Original Request`
- `## Goal Definition`
- `## Complexity Assessment`
- `## Task Decomposition`
- `## Agent Assignment`
- `## Execution Progress`
- `## Execution Log`
- `## Completion Sync Gate`
- `## Error Log`
- `## Final Output`

Required task table columns:

| Column | Required | Notes |
|--------|----------|-------|
| Task ID | Yes | Stable id such as `T-01` |
| Agent | Yes | Agent id or `coordinator` |
| Status | Yes | Must match `state-machine.md` |
| Dependencies | Yes | `none` or task id list |
| Output | Yes | Result path or inline summary |

## Agent Task File

Path: `.agentdocs/runtime/<task-id>/agent_tasks/agent-XX.md`

Required sections:

- `## Task Information`
- `## Input`
- `## Task Description`
- `## Expected Output`
- `## Constraints`

Required metadata:

```yaml
task_id: T-XX
agent_id: agent-XX
dependencies: []
expected_output_path: .agentdocs/runtime/<task-id>/results/agent-XX-result.md
allowed_files: []
status_owner: coordinator
```

Workers must not edit workflow docs or `master_plan.md` unless the coordinator
explicitly assigns that responsibility.

## Agent Result File

Path: `.agentdocs/runtime/<task-id>/results/agent-XX-result.md`

Required sections:

- `## Execution Summary`
- `## Output Result`
- `## Verification`
- `## Warnings and Errors`
- `## Plan Sync Confirmation`

`Plan Sync Confirmation` must say whether the coordinator has synced status.
Workers may request sync, but they must not claim sync if they did not perform it.

## Final Output File

Path: `.agentdocs/runtime/<task-id>/final_output.md`

Required sections:

- `## Execution Summary`
- `## Original Goal`
- `## Completion Status`
- `## Integrated Results`
- `## Verification`
- `## Notes`

The final output is the aggregation source for memory sync and cleanup decisions.
