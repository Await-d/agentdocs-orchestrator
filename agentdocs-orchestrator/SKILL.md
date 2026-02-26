---
name: agentdocs-orchestrator
description: Advanced task orchestration system integrated with agentdocs knowledge management. Decomposes complex requests into atomic tasks, auto-creates workflow planning documents, manages multi-agent parallel execution, and syncs task status. Use when handling complex multi-step tasks, parallel execution needs, or sub-agent orchestration.
---

# Agentdocs Orchestrator

## When to Activate (Read First)

**Activate** when:
- Complex multi-step tasks (3+ steps with parallelizable subtasks)
- User mentions "parallel", "concurrent", "subtasks", "agents", "sub-agent"
- Task requires decomposition and cross-agent coordination

**Do NOT activate** for:
- Sequential tasks with no parallel potential (just execute directly)
- Tasks with ≤2 clear steps (unnecessary overhead)
- Single-file changes or simple refactors

---

## Task Complexity Decision Tree

Before creating any documents, classify the task:

```
Task received
    │
    ├── ≤2 steps, strictly sequential
    │   → Execute directly. No .agentdocs needed.
    │
    ├── 3–5 steps, some parallelizable
    │   → Lightweight mode:
    │     1. Create workflow doc only (no runtime dir)
    │     2. Execute via task() or sequentially in context
    │
    └── Complex, 5+ steps, multiple independent streams
        → Full orchestration:
          workflow doc + runtime dir + master_plan + agent files
```

---

## Directory Structure

```
.agentdocs/
├── index.md                      # Knowledge entry point
├── workflow/                     # Task planning (persistent — commit to git)
│   ├── YYMMDD-task-name.md      # Task analysis, design, TODOs
│   └── done/                     # Completed task archive
└── runtime/                      # Execution coordination (temporary — gitignore)
    └── YYMMDD-task-name/
        ├── master_plan.md
        ├── agent_tasks/
        │   ├── agent-01.md
        │   └── ...
        └── results/
            ├── agent-01-result.md
            └── ...
```

**Key distinction:**
- `workflow/` — persistent planning & decisions (commit to git)
- `runtime/<task-id>/` — temporary execution artifacts (add to `.gitignore`)

**Git setup:**
```bash
# Add to .gitignore
echo ".agentdocs/runtime/" >> .gitignore
```

---

## Execution Modes

### Mode A: OpenCode Native ✅ (Preferred in OpenCode environment)

Use the native `task()` system for true parallel execution. No need to create `agent_tasks/*.md` files — dispatch directly.

```typescript
// Launch parallel agents
const t1 = task(category="quick", run_in_background=true, load_skills=[], prompt="...")
const t2 = task(subagent_type="explore", run_in_background=true, load_skills=[], prompt="...")

// Collect results when needed
const r1 = background_output(task_id=t1.task_id)
const r2 = background_output(task_id=t2.task_id)
```

### Mode B: Sequential Execution (in-context)

Execute subtasks one by one in the current context. Write results to
`runtime/<task-id>/results/`. This is **real execution**, not a simulation.

Use when: tasks must share context, or the orchestration itself is the valuable output.

### Mode C: Claude CLI (for external/automated pipelines)

```bash
# Linux/Mac
TASK_ID="YYMMDD-task-name"
RUNTIME_PATH=".agentdocs/runtime/$TASK_ID"
parallel claude -p "$(cat {})" ::: $RUNTIME_PATH/agent_tasks/*.md
```

See [cli-integration.md](cli-integration.md) for full reference.

---

## Core Five-Phase Workflow

### Phase 1️⃣ Task Analysis and Planning

1. Detect user language — all documents use the same language as the user
2. Read `.agentdocs/index.md` for existing relevant context
3. Analyze intent, identify dependencies (build DAG)
4. Create workflow document for 3+ step tasks
5. Break down into atomic tasks (target: 1–5 min each)

### Phase 2️⃣ Agent Assignment

- Create task-specific runtime directory: `runtime/<task-id>/`
- Create `master_plan.md` with task decomposition and status table
- Generate agent task files (Mode B/C) OR dispatch via `task()` (Mode A)

**Status icons:** 🟡 Pending | 🔵 Running | ✅ Completed | ❌ Failed | ⏸️ Waiting

### Phase 3️⃣ Parallel Execution

- Mode A: `task(run_in_background=true)` + `background_output()`
- Mode B: Sequential execution in context, write results to `results/`
- Mode C: `claude -p` CLI scripts (see [cli-integration.md](cli-integration.md))

Track all status changes in `master_plan.md`.

### Phase 4️⃣ Result Aggregation

- Collect results from `runtime/<task-id>/results/` (or via `background_output()`)
- Merge in dependency order
- Generate `final_output.md`

### Phase 5️⃣ Status Sync and Cleanup

1. Update workflow TODOs — mark completed items (`- [x] T-01 ✅`)
2. If all TODOs done: move workflow doc to `done/`, update `index.md`
3. Cleanup: `rm -rf .agentdocs/runtime/YYMMDD-task-name/`

---

## Workflow Document Template

```markdown
# .agentdocs/workflow/YYMMDD-task-name.md

## Task Overview
[Brief description]

## Current Analysis
[Problem analysis, constraints, considerations]

## Solution Design
[High-level approach and key decisions]

## Implementation Plan

### Phase 1: [Phase Name]
- [ ] T-01: [Task description]
- [ ] T-02: [Task description]

### Phase 2: [Phase Name]
- [ ] T-03: [Task description]

## Notes
[Important observations, blockers, decisions]
```

---

## Mandatory Gates (When Writing Code)

### Gate 1: Plan First
Before any code changes, provide a plan (goal, impacted files, risks, verification strategy).
Wait for **explicit user approval** before starting.

Approval signals include:
- English: "approved", "go ahead", "start", "ok", "yes", "proceed", "sounds good", "let's do it"
- Chinese: "好的", "开始", "开始吧", "可以", "没问题", "同意", "确认", "去做"

If approval is not explicit, stop at planning.

### Gate 2: Segmented Changes (>3 files)
Split into stages of ≤3 files each. Verify each stage before the next.

### Gate 3: TDD for Bug Fixes
Red (failing repro/test) → Green (minimal fix) → Refactor.
No repro evidence = no fix implementation.

### Gate 4: Defensive Programming Output
After code changes, always attach:
1. **Potential Bug Checklist** — regression risks, edge cases, error paths, performance side effects
2. **Test Case Checklist** — happy path, edge case, failure path

---

## Language Adaptation

All documents use the user's language:
- User speaks Chinese → `workflow/260112-修复音频播放器.md` with Chinese content
- User speaks English → `workflow/260112-fix-audio-player.md` with English content

---

## Best Practices

1. **Lightweight first**: Only create runtime dirs when you need agent isolation or CLI execution
2. **Mode A preferred**: In OpenCode environments, use `task()` over CLI scripts
3. **Status sync**: Update workflow TODOs after each subtask completion
4. **Index maintenance**: Register new workflows in `index.md`; remove when moved to `done/`
5. **Clean runtime**: Delete runtime dir after task completion (it's temporary)
6. **Chunked file creation**: Max 150 lines per write operation

---

## Context Window Management

When agents pass results to downstream agents, large payloads can exceed context limits.
Apply these rules when constructing inter-agent prompts:

| Result size | Strategy |
|-------------|----------|
| < 200 lines | Pass full content inline |
| 200–500 lines | Pass full content with a summary header |
| > 500 lines | **Summarize first**: extract key findings (max 100 lines), pass summary + file path reference |
| > 1000 lines | Write to `results/agent-XX-result.md`, pass only the file path to downstream agents |

**Summarization trigger** — before injecting a previous agent's result, check:
```
If len(result) > 500 lines:
    summary = extract_key_findings(result)   # decisions, errors, metrics, file list
    inject = summary + "\n\nFull result: runtime/<task-id>/results/agent-XX-result.md"
Else:
    inject = result
```

**Never** dump raw multi-thousand-line file contents into an agent prompt.
Always extract only what the downstream agent needs to proceed.

---

## Error Handling

When an agent/task fails:
1. Log in `master_plan.md` error table
2. Update workflow doc with blocker notes
3. Retry up to 3 times with exponential backoff
4. Mark as `❌ Failed` if all retries exhausted

Task isolation rule: only handle errors from the current task's runtime. Do not modify other concurrent tasks' runtimes.

---

## Related Files

- [RULES.md](RULES.md) — Mandatory guardrails (project-configurable defaults)
- [workflow.md](workflow.md) — Detailed workflow reference with scheduling algorithm
- [templates.md](templates.md) — Complete template collection (master_plan, agent task, result, final output)
- [cli-integration.md](cli-integration.md) — Claude CLI Mode C integration
- [examples.md](examples.md) — Practical usage examples (code review, translation, API testing, bug fix with TDD)
