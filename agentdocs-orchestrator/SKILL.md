---
name: agentdocs-orchestrator
description: Advanced task orchestration system integrated with agentdocs knowledge management. Decomposes complex requests into atomic tasks, auto-scores task difficulty before choosing direct/lightweight/full execution, creates workflow planning documents, manages multi-agent parallel execution, and syncs task status. Use this skill whenever the user asks to create or update a plan for non-trivial work, break down a multi-step task, assess task difficulty, choose between direct/lightweight/full execution, coordinate sub-agents, maintain `.agentdocs` workflow files, or keep plan/TODO markers in sync — even if they never say "orchestration" explicitly.
---

# Agentdocs Orchestrator

## When to Activate (Read First)

**Activate** when:
- Complex multi-step tasks (3+ steps, even if parallelism is not obvious yet)
- User mentions "parallel", "concurrent", "subtasks", "agents", "sub-agent"
- Task requires decomposition, cross-agent coordination, or explicit plan/routing assessment
- User asks for a workflow, implementation plan, roadmap, task breakdown, TODO list, or difficulty grading for a piece of work
- The task needs `.agentdocs` workflow docs, runtime plans, progress tracking, or completion-status syncing

**Do NOT activate** for:
- Tiny sequential tasks that clearly score `Direct` after assessment (for example, ≤2 atomic steps with no orchestration benefit)
- Tasks with ≤2 clear steps (unnecessary overhead)
- Single-file changes or simple refactors

### High-confidence trigger cues

Treat these requests as strong signals to load this skill:
- "帮我拆一下任务 / 给我一个执行方案 / 做个工作流"
- "评估这个任务难度 / 该走简单方案还是复杂方案"
- "把这些子任务并行起来 / 帮我协调几个 agent"
- "更新一下计划里的进度 / 把完成的任务勾掉"
- "在 `.agentdocs` 里建 workflow / runtime / master_plan"

---

## Task Complexity Assessment

Before creating any documents, evaluate complexity across **two dimensions** and classify the task:

### Step 1: Score the task

| Signal | Score |
|--------|-------|
| ≤2 atomic steps | –2 |
| 3–4 atomic steps | 0 |
| 5+ atomic steps | +2 |
| Multiple independent parallel streams (agents can run simultaneously) | +2 |
| Involves 3+ modules, systems, or services | +1 |
| Any single step estimated >5 min to complete | +1 |
| Results must be persisted for review by others | +1 |
| Running in OpenCode (Mode A available) | –1 |

### Step 2: Choose mode based on total score

```
Score ≤ –1  → Execute directly. No .agentdocs needed.
Score 0–2   → Lightweight mode:
                 1. Create workflow doc only (no runtime dir)
                 2. Execute via task() or sequentially in context
Score 3+    → Full orchestration:
                 workflow doc + runtime dir + master_plan
                 execution method chosen separately:
                 - Mode A: direct `task()` dispatch
                 - Mode B/C: agent task files
```

> **Authority rule:** The **score result is the final routing decision**. Step count is only one input signal inside the score, not a separate override.

### Step 2.5: Record the routing decision (mandatory)

Before creating a workflow doc, runtime dir, or task list, write a visible `## Complexity Assessment` block that records:

- Each scoring signal that was considered
- The numeric total
- The selected mode (`Direct`, `Lightweight`, or `Full orchestration`)
- One short justification for the selected mode

If the selected mode is `Direct` and no workflow doc will be created, place the same `## Complexity Assessment` block in the response or task notes before executing.

Never skip the written score. Never default to `Direct`, `Lightweight`, or `Full orchestration` because the task "feels simple" or because the mode seems convenient.

### Step 3: Decomposition check (before starting implementation)

Before writing any code or running any tasks, ask:
1. **Can this be split?** — Are there 2+ independent subtasks that can run in parallel without waiting for each other?
2. **Should it be split?** — Does splitting reduce total time OR reduce failure blast radius?
3. **What is the dependency order?** — Build a DAG: which tasks block which others?

If the answer to both #1 and #2 is YES → decompose and assign to agents.
If #1 is YES but #2 is NO (overhead exceeds benefit) → execute sequentially in context.
If #1 is NO → execute directly without decomposition.

> **Routing discipline:** When evidence is incomplete, inspect enough context to finish the score. Do **not** default to lightweight. Only choose a mode after the written score is recorded, and re-score if scope expands.

### Quick classification examples

| Task | Score | Mode |
|------|-------|------|
| Fix a typo in one file | –2 | Direct |
| Refactor 2 functions with tests | –1 | Direct |
| Add auth to 3 API endpoints | 1 | Lightweight |
| Code review across 5 modules | 3 | Full orchestration |
| Multi-service migration (DB + API + frontend) | 6 | Full orchestration |
| Translate 10 docs in parallel | 4 | Full orchestration |

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
- `.agentdocs/workflow/` — persistent planning & decisions (commit to git)
- `.agentdocs/runtime/<task-id>/` — temporary execution artifacts (add to `.gitignore`)

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

Execute subtasks one by one in the current context. This is **real execution**, not a simulation.

- **Full orchestration**: write results to `.agentdocs/runtime/<task-id>/results/`
- **Lightweight mode**: synthesize results directly in the current context — no result files needed

Use when: tasks must share context, or the orchestration itself is the valuable output.

### Mode C: Claude CLI (for external/automated pipelines)

```bash
# Linux/Mac
TASK_ID="YYMMDD-task-name"
RUNTIME_PATH=".agentdocs/runtime/$TASK_ID"
mkdir -p "$RUNTIME_PATH/results"
export RUNTIME_PATH
parallel 'agent_id=$(basename {} .md); claude -p "$(cat {})" > "$RUNTIME_PATH/results/${agent_id}-result.md" 2>&1' ::: "$RUNTIME_PATH"/agent_tasks/*.md
```

See [cli-integration.md](cli-integration.md) for full reference.

---

## Core Five-Phase Workflow

### Phase 1️⃣ Task Analysis and Planning

1. Detect user language — all documents use the same language as the user
2. Read `.agentdocs/index.md` for existing relevant context
3. Do a minimal intent parse (goal, scope, major modules/systems involved)
4. Record a `## Complexity Assessment` section with score breakdown, total, and chosen mode
5. If routing selects `Direct`, keep that written assessment in the response or task notes and execute without `.agentdocs`
6. If routing selects `Lightweight` or `Full orchestration`, create a workflow document and copy the same complexity assessment into it
7. For routed tasks, analyze dependencies (build DAG)
8. Break down into atomic tasks (target: 1–5 min each)

### Phase 2️⃣ Agent Assignment

> **Lightweight mode (when score = 0–2; typically 3–4 step tasks):** Skip runtime dir. Track tasks in the workflow doc. Inline/context notes are optional summaries only. Dispatch directly via `task()` or execute sequentially in context — no `master_plan.md` or `agent_tasks/` files needed.
>
> **Full orchestration (when score ≥ 3; commonly 5+ step or high-coordination tasks):** Create task-specific runtime directory: `.agentdocs/runtime/<task-id>/`, create `master_plan.md` with task decomposition and status table, then choose execution method: generate agent task files for Mode B/C, or dispatch directly via `task()` for Mode A.

**Status icons:** 🟡 Pending | 🔵 Running | ✅ Completed | ❌ Failed | ⏸️ Waiting

### Phase 3️⃣ Parallel Execution

**Lightweight mode:** Dispatch tasks inline via `task()` (Mode A) or run sequentially in context (Mode B). No runtime result files are needed, but progress must still be tracked in the workflow doc. Current-context notes are optional summaries, not a substitute for workflow markers.

**Full orchestration only:**
- Mode A: `task(run_in_background=true)` + `background_output()`
- Mode B: Sequential execution in context, write results to `.agentdocs/runtime/<task-id>/results/`
- Mode C: `claude -p` CLI scripts (see [cli-integration.md](cli-integration.md))
- Track all status changes in `.agentdocs/runtime/<task-id>/master_plan.md`

### Phase 4️⃣ Result Aggregation

**Lightweight mode:** Synthesize results directly in context; no file aggregation needed.

**Full orchestration only:**
- Collect results from `.agentdocs/runtime/<task-id>/results/` (or via `background_output()`)
- Merge in dependency order
- Generate `.agentdocs/runtime/<task-id>/final_output.md`

### Phase 5️⃣ Status Sync, Memory Sync, and Cleanup

1. Immediately update plan markers for every completed task:
   - Workflow TODOs (`- [x] T-01 ✅`)
   - Full orchestration only: matching `.agentdocs/runtime/<task-id>/master_plan.md` status rows
2. In parallel runs, a coordinator may serialize shared file writes, but the workflow/master-plan sync must still happen before completion is reported
3. Do not report a subtask or task as complete until those markers are synced
4. Extract durable memory and append to `.agentdocs/index.md` (see Memory Protocol) — **do this before archiving**
5. If all TODOs done: move workflow doc to `.agentdocs/workflow/done/`, update `index.md`
6. **Full orchestration only:** Cleanup runtime dir: `rm -rf .agentdocs/runtime/YYMMDD-task-name/`

---

## Memory Protocol (Durable Knowledge)

Use `.agentdocs/index.md` as the durable memory entry point. After each completed complex task,
extract only reusable knowledge (not task noise) and write it into structured sections.

### Memory Sections in `index.md`

```markdown
## Architecture Decisions
- [YYYY-MM-DD] [Decision] — [Why] — [Scope/Impact]

## Coding Conventions
- [Rule] — [Example path]

## Known Pitfalls
- [Symptom] → [Root cause] → [Fix]

## Global Important Memory
- [Long-lived constraint or preference]
```

### What to Write

Write memory only if at least one condition is true:
- A non-obvious architecture decision was made
- A recurring bug/pitfall was identified and fixed
- A stable coding convention was established or corrected
- User clarified a persistent preference or process rule

### What NOT to Write

Do NOT write:
- One-off implementation details
- Temporary runtime logs
- Raw command output dumps
- Duplicates of existing memory entries

### Memory Update Rules

1. **Deduplicate first**: if similar memory exists, update existing line instead of appending a new duplicate
2. **Keep concise**: each memory item should be 1–2 lines max
3. **Timestamp decisions**: include date on architecture decisions and major pitfalls
4. **Project-local precedence**: if `.agentdocs/local-rules.md` exists, memory updates must respect it

### Automatic Memory Sync (Recommended)

At task completion, run a small memory-sync pass before cleanup:
1. Read current memory sections in `.agentdocs/index.md`
2. Extract candidate entries from `.agentdocs/workflow/<task-id>.md` and `.agentdocs/runtime/<task-id>/final_output.md` (or `.agentdocs/workflow/done/<task-id>.md` as fallback)
3. Apply dedup/update rules (update existing entries when semantically similar)
4. Write only net-new durable memory back to `index.md`
5. Add one line in workflow notes: `Memory sync: completed`

Use the reusable prompt in `templates.md` under:
`## 7. Durable Memory Templates` → `### Memory Sync Prompt Template`

### Read Path Before Execution

Before implementing complex tasks, read in this order:
1. `.agentdocs/local-rules.md` (if exists)
2. `.agentdocs/index.md` memory sections
3. Relevant active workflow docs

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

## Complexity Assessment
- Atomic steps: [count] → [score]
- Parallel streams: [yes/no] → [score]
- Modules/systems/services: [count] → [score]
- Long step (>5 min): [yes/no] → [score]
- Persisted review artifacts: [yes/no] → [score]
- OpenCode available: [yes/no] → [score]
- **Total score**: [number]
- **Chosen mode**: [Direct / Lightweight / Full orchestration]
- **Routing rationale**: [1-2 sentences]

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
- User speaks Chinese → `.agentdocs/workflow/260112-修复音频播放器.md` with Chinese content
- User speaks English → `.agentdocs/workflow/260112-fix-audio-player.md` with English content

---

## Best Practices

1. **Score before routing**: Record the written difficulty score before choosing any mode
2. **Mode A preferred**: In OpenCode environments, use `task()` over CLI scripts
3. **Status sync is part of completion**: Update workflow TODOs after each subtask completion; for full orchestration, sync `master_plan.md` in the same step
4. **No unchecked completed tasks**: Never leave a finished task marked pending in the workflow doc or master plan
5. **Index maintenance**: Register new workflows in `index.md`; remove when moved to `done/`
6. **Clean runtime**: Delete runtime dir after task completion (it's temporary)
7. **Chunked file creation**: Max 150 lines per write operation

---

## Context Window Management

When agents pass results to downstream agents, large payloads can exceed context limits.
Apply these rules when constructing inter-agent prompts:

| Result size | Strategy |
|-------------|----------|
| < 200 lines | Pass full content inline |
| 200–500 lines | Pass full content with a summary header |
| > 500 lines | **Summarize first**: extract key findings (max 100 lines), pass summary + file path reference |
| > 1000 lines | Write to `.agentdocs/runtime/<task-id>/results/agent-XX-result.md`, pass only the file path to downstream agents |

**Summarization trigger** — before injecting a previous agent's result, check:
```
If len(result) > 500 lines:
    summary = extract_key_findings(result)   # decisions, errors, metrics, file list
    inject = summary + "\n\nFull result: .agentdocs/runtime/<task-id>/results/agent-XX-result.md"
Else:
    inject = result
```

**Never** dump raw multi-thousand-line file contents into an agent prompt.
Always extract only what the downstream agent needs to proceed.

---

## Error Handling

**Full orchestration mode:**
1. Log in `.agentdocs/runtime/<task-id>/master_plan.md` error table
2. Update workflow doc with blocker notes
3. Retry up to 3 times with exponential backoff
4. Mark as `❌ Failed` if all retries exhausted

Task isolation rule: only handle errors from the current task's runtime. Do not modify other concurrent tasks' runtimes.

**Lightweight mode:**
1. Note the blocker in the workflow doc (current-context summary optional)
2. Retry inline or re-dispatch the failed subtask via `task()`
3. If unresolvable: mark the subtask failed in the workflow doc and proceed or stop

---

## Related Files

- [RULES.md](RULES.md) — Mandatory guardrails (project-configurable defaults)
- [workflow.md](workflow.md) — Detailed workflow reference with scheduling algorithm
- [templates.md](templates.md) — Complete template collection (master_plan, agent task, result, final output)
- [cli-integration.md](cli-integration.md) — Claude CLI Mode C integration
- [examples.md](examples.md) — Practical usage examples (code review, translation, API testing, bug fix with TDD)
