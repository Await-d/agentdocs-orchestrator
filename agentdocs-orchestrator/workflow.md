# Workflow: Detailed Distributed Task Orchestration Workflow

## Complete Execution Flow Diagram

> **Note:** Phases 2–4 below describe the **full orchestration path** (when routing score ≥ 3; commonly 5+ steps or high-coordination tasks). For lightweight mode (score 0–2; typically 3–4 steps), skip Phases 2–4 runtime artifacts and keep task tracking in the workflow doc.

```
┌─────────────────────────────────────────────────────────────────┐
│                    User Submits Complex Request                  │
└─────────────────────────────────┬───────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│ Phase 1: Task Analysis and Decomposition                         │
│ ┌───────────────────────────────────────────────────────────┐   │
│ │ 1.1 Minimal intent parse                                   │   │
│ │ 1.2 Record complexity assessment + route                   │   │
│ │ 1.3 Identify dependencies (build DAG)                      │   │
│ │ 1.4 Break down into atomic tasks                           │   │
│ │ 1.5 Define Input/Output for each task                      │   │
│ └───────────────────────────────────────────────────────────┘   │
│ 📄 Output: .agentdocs/workflow/<task-id>.md                               │
└─────────────────────────────────┬───────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│ Phase 2: Agent Assignment and Status Marking                     │
│ ┌───────────────────────────────────────────────────────────┐   │
│ │ 2.1 Assign Agent ID for each task                          │   │
│ │ 2.2 Create task status table                               │   │
│ │ 2.3 Generate Agent task files                              │   │
│ │ 2.4 Initialize status as "Pending"                         │   │
│ └───────────────────────────────────────────────────────────┘   │
│ 📄 Output: .agentdocs/runtime/<task-id>/agent_tasks/agent-XX.md          │
└─────────────────────────────────┬───────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│ Phase 3: Parallel Execution                                      │
│ ┌───────────────────────────────────────────────────────────┐   │
│ │ 3.1 Identify parallelizable task groups                    │   │
│ │ 3.2 Choose execution method (sequential / OpenCode task() / CLI) │   │
│ │ 3.3 Execute tasks with no dependencies                     │   │
│ │ 3.4 Execute subsequent tasks after dependencies complete   │   │
│ │ 3.5 Record execution logs                                  │   │
│ └───────────────────────────────────────────────────────────┘   │
│ 📄 Output: .agentdocs/runtime/<task-id>/results/agent-XX-result.md       │
└─────────────────────────────────┬───────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│ Phase 4: Result Aggregation and Integration                      │
│ ┌───────────────────────────────────────────────────────────┐   │
│ │ 4.1 Collect all Agent results                              │   │
│ │ 4.2 Verify result completeness                             │   │
│ │ 4.3 Merge results according to dependency order            │   │
│ │ 4.4 Generate final output                                  │   │
│ └───────────────────────────────────────────────────────────┘   │
│ 📄 Output: .agentdocs/runtime/<task-id>/final_output.md                  │
└─────────────────────────────────────────────────────────────────┘
```

## Mandatory Gates Before Implementation (New)

### Gate A: Plan and User Approval First
- Before any implementation code changes, produce a concrete plan with:
  - Goal and scope
  - Impacted files
  - Risks and rollback idea
  - Verification strategy
- Wait for explicit user approval (for example: "approved", "go ahead", "start").
- If approval is not explicit, stop at planning.

### Gate B: Segmented File Scope (>3 Files)
- If expected changes touch more than **3 files**, split into stages.
- Each stage should include at most 3 files and end with verification before continuing.
- Do not batch large multi-file edits into a single pass.

### Gate C: TDD Entry for Bug Fixes
- Bug fix tasks must start with a failing repro script/test (Red).
- Only after stable reproduction can implementation begin (Green).
- Refactor is allowed only after tests pass and behavior is stable.

### Gate D: No-Approval No-Code Rule
- Do not generate implementation code when any mandatory gate is missing.
- Treat missing repro evidence for bug fixes as a blocking condition.

## Phase 1: Task Analysis and Decomposition (Detailed)

### 1.0 Complexity Assessment (Required Before Detailed Decomposition)

After a minimal intent parse (goal, scope, major modules/systems), but before building a DAG or decomposing into tasks, score the task using the two-dimension model:

| Signal | Score |
|--------|-------|
| ≤2 atomic steps | –2 |
| 3–4 atomic steps | 0 |
| 5+ atomic steps | +2 |
| Multiple independent parallel streams | +2 |
| Involves 3+ modules, systems, or services | +1 |
| Any single step estimated >5 min | +1 |
| Results must be persisted for review by others | +1 |
| Running in OpenCode (Mode A available) | –1 |

**Routing:**
- Score ≤ –1 → Execute directly (no `.agentdocs` needed)
- Score 0–2 → Lightweight mode (workflow doc only)
- Score 3+ → Full orchestration (workflow + runtime + master_plan; use agent task files only for Mode B/C)

> **Authority rule:** Use the score result as the final routing decision. Step count alone does not override the score.

**Scoring output is mandatory:**
- Write the score into a visible `## Complexity Assessment` section before creating tasks or runtime artifacts.
- Include each signal considered, the total score, and the chosen mode.
- Do not default to `Direct`, `Lightweight`, or `Full orchestration` without a written score.
- If the chosen mode is `Direct`, keep the block in the response or task notes because no `.agentdocs` artifact is created.

```markdown
## Complexity Assessment
- Atomic steps: 4 → 0
- Parallel streams: yes → +2
- Modules/systems/services: 3 → +1
- Long step (>5 min): no → 0
- Persisted review artifacts: no → 0
- OpenCode available: yes → -1
- **Total score**: 2
- **Chosen mode**: Lightweight
- **Why**: Multi-file review benefits from decomposition, but does not justify runtime artifacts.
```

**Decomposition check (if score ≥ 0):**
1. **Can it be split?** — 2+ independent subtasks that can run in parallel?
2. **Should it be split?** — Splitting reduces total time OR failure blast radius?
3. **Dependency order?** — Build a DAG before assigning agents.

If both #1 and #2 are YES → decompose. If #1 YES but #2 NO → execute sequentially in context.

---

### 1.1 Parse User Intent

```markdown
## Intent Analysis Checklist

### Core Objectives
- [ ] What is the primary goal?
- [ ] What are the secondary goals?
- [ ] What are the success criteria?

### Constraints
- [ ] Time constraints?
- [ ] Resource constraints?
- [ ] Technical constraints?

### Implicit Requirements
- [ ] What does the user expect but didn't explicitly state?
- [ ] Industry best practices?
```

### 1.2 Dependency Analysis

**Dependency Types:**

| Type | Description | Example |
|------|-------------|---------|
| Data Dependency | B needs A's output as input | Analyze code → Generate report |
| Sequential Dependency | B must execute after A | Create file → Write content |
| Resource Dependency | A and B compete for same resource | Write to same file simultaneously |
| No Dependency | Completely independent | Process different files separately |

**Building Dependency Graph:**

```
Example: Code Review Task

          ┌─→ [T-02: Check code style] ─┐
[T-01] ──┤                              ├──→ [T-05: Generate report]
Read code ├─→ [T-03: Security scan] ────┤
          └─→ [T-04: Performance analysis] ─┘
```

### 1.3 Atomic Task Definition

**Atomic Task Criteria:**
- ✅ Single Responsibility: Does only one thing
- ✅ Independently Executable: Doesn't depend on runtime context
- ✅ Verifiable Output: Has clear success/failure criteria
- ✅ Retriable: Can be safely retried after failure

```markdown
## Task: T-03

### Definition
- **Task Name**: Security vulnerability scan
- **Input**: Source code file list
- **Output**: Vulnerability report JSON
- **Estimated Time**: 2 minutes

### Atomicity Check
- [x] Single responsibility
- [x] Independently executable
- [x] Verifiable output
- [x] Safe to retry
```

## Phase 2: Agent Assignment (Detailed)

### 2.1 Agent ID Assignment Rules

```
Display label: Agent-{sequence}
Filename stem: agent-{sequence}
Sequence: 01, 02, 03, ... (two-digit zero-padded)
```

### 2.2 Complete Task Status Table Fields

```markdown
| Task ID | Task Description | Agent | Status | Priority | Deps | Start | End | Retries |
|---------|------------------|-------|--------|----------|------|-------|-----|---------|
| T-01 | Read code | Agent-01 | ✅ | P0 | None | 10:00 | 10:01 | 0 |
| T-02 | Style check | Agent-02 | 🔵 | P1 | T-01 | 10:01 | - | 0 |
| T-03 | Security scan | Agent-03 | 🔵 | P1 | T-01 | 10:01 | - | 0 |
| T-04 | Performance analysis | Agent-04 | 🟡 | P1 | T-01 | - | - | 0 |
| T-05 | Generate report | Agent-05 | ⏸️ | P2 | T-02,T-03,T-04 | - | - | 0 |
```

### 2.3 Priority Definitions

| Priority | Meaning | Description |
|----------|---------|-------------|
| P0 | Critical Path | Blocks other tasks, execute first |
| P1 | High Priority | Has downstream dependencies |
| P2 | Normal | Standard priority |
| P3 | Low Priority | Can be delayed |

## Phase 3: Parallel Execution (Detailed)

### 3.1 Execution Scheduling Algorithm

```python
# Pseudocode: Topological Sort + Parallel Scheduling
def schedule_tasks(tasks, dependencies):
    ready_queue = [t for t in tasks if no_dependencies(t)]
    running = set()
    completed = set()
    
    while ready_queue or running:
        # Start all ready tasks
        for task in ready_queue:
            start_agent(task)
            running.add(task)
        ready_queue.clear()
        
        # Wait for any task to complete
        finished = wait_any(running)
        completed.add(finished)
        running.remove(finished)
        
        # Check for newly ready tasks
        for task in tasks:
            if task not in completed and task not in running:
                if all_deps_complete(task, completed):
                    ready_queue.append(task)
    
    # Deadlock / circular dependency detection
    remaining = [t for t in tasks if t not in completed]
    if remaining:
        for t in remaining:
            mark_failed(t, reason="Circular dependency or unresolvable blocker")
        raise RuntimeError(f"Deadlock detected: {len(remaining)} tasks could not complete: {remaining}")
```

### 3.2 Sequential Execution Output Format

```
══════════════════════════════════════════════════════════════════
                    🚀 Parallel Execution Batch #1
══════════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────────────
│ 🤖 Agent-01 [T-01: Read Code]
├──────────────────────────────────────────────────────────────────
│ 📥 Instruction: Read all .ts files in src/ directory
│ ⚙️ Execution:
│    → Scan directory structure
│    → Read 15 TypeScript files
│    → Calculate file statistics
│ 📤 Output: 
│    - File count: 15
│    - Total lines: 2,847
│    - Saved to: .agentdocs/runtime/<task-id>/results/agent-01-result.md
│ ⏱️ Duration: 1.2s
│ ✅ Status: Completed
└──────────────────────────────────────────────────────────────────

══════════════════════════════════════════════════════════════════
                    🚀 Parallel Execution Batch #2
══════════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────────────
│ 🤖 Agent-02 [T-02: Style Check] ║ Agent-03 [T-03: Security Scan]
├──────────────────────────────────────────────────────────────────
│ [Executing in parallel...]
│ 
│ Agent-02 completed ✅ (2.1s)
│ Agent-03 completed ✅ (3.4s)
└──────────────────────────────────────────────────────────────────
```

### 3.3 CLI Execution Mode

**Windows PowerShell Parallel Execution:**

```powershell
# Method 1: Using Jobs (with task-specific runtime path)
$taskId = "YYMMDD-task-name"
$runtimePath = ".agentdocs/runtime/$taskId"

$taskFiles = Get-ChildItem "$runtimePath/agent_tasks/*.md"
if ($taskFiles.Count -eq 0) {
    Write-Error "No task files found in $runtimePath/agent_tasks"
    exit 1
}

$jobs = foreach ($file in $taskFiles) {
    $agentId = $file.BaseName
    Start-Job -Name $agentId -ScriptBlock {
        param($taskPath, $resultPath)
        $task = Get-Content $taskPath -Raw
        $result = claude -p $task 2>&1
        $exitCode = $LASTEXITCODE
        $result | Out-File $resultPath -Encoding UTF8
        return @{ Agent = [System.IO.Path]::GetFileNameWithoutExtension($taskPath); Success = ($exitCode -eq 0); ExitCode = $exitCode; Output = $result }
    } -ArgumentList $file.FullName, "$runtimePath/results/$agentId-result.md"
}

# Wait for all to complete
$results = $jobs | Wait-Job | Receive-Job

# Collect results
$results | ForEach-Object {
    if ($_.Success) {
        Write-Host "[$($_.Agent)] Status: ✅ Success"
    } else {
        Write-Host "[$($_.Agent)] Status: ❌ Failed (exit code: $($_.ExitCode))"
    }
}

if ($results | Where-Object { -not $_.Success }) {
    Write-Error "One or more agent jobs failed. Stop and inspect result files before continuing."
    $jobs | Remove-Job
    exit 1
}

# Cleanup
$jobs | Remove-Job
```

**Method 2: Using Runspace Pool (More Efficient)**

```powershell
# Create Runspace Pool (with task-specific runtime path)
$taskId = "YYMMDD-task-name"
$runtimePath = ".agentdocs/runtime/$taskId"

$pool = [RunspaceFactory]::CreateRunspacePool(1, 5)  # Max 5 parallel
$pool.Open()

$tasks = Get-ChildItem "$runtimePath/agent_tasks/*.md"
if ($tasks.Count -eq 0) {
    $pool.Close()
    $pool.Dispose()
    throw "No task files found in $runtimePath/agent_tasks"
}

$runspaces = @()

foreach ($task in $tasks) {
    $ps = [PowerShell]::Create()
    $ps.RunspacePool = $pool
    $ps.AddScript({
        param($taskPath)
        $content = Get-Content $taskPath -Raw
        $result = claude -p $content 2>&1
        $exitCode = $LASTEXITCODE
        if ($exitCode -ne 0) {
            throw "claude exited with code $exitCode"
        }
        return $result
    }).AddArgument($task.FullName) | Out-Null

    $runspaces += [PSCustomObject]@{
        PowerShell = $ps
        Handle = $ps.BeginInvoke()
        Task = $task.BaseName
    }
}

# Wait and collect results
foreach ($rs in $runspaces) {
    try {
        $result = $rs.PowerShell.EndInvoke($rs.Handle)
        $result | Out-File "$runtimePath/results/$($rs.Task)-result.md"
    }
    catch {
        $_.Exception.Message | Out-File "$runtimePath/results/$($rs.Task)-result.md"
        $rs.PowerShell.Dispose()
        $pool.Close()
        $pool.Dispose()
        throw
    }
    $rs.PowerShell.Dispose()
}

$pool.Close()
$pool.Dispose()
```

## Phase 4: Result Aggregation (Detailed)

### 4.1 Result Collection Check

```markdown
## Result Collection Checklist

### Expected Results
| Agent | Result File | Status |
|-------|-------------|--------|
| Agent-01 | agent-01-result.md | ✅ Exists |
| Agent-02 | agent-02-result.md | ✅ Exists |
| Agent-03 | agent-03-result.md | ❌ Missing |

### Missing Result Handling
- Agent-03: Re-execute / Mark as failed
```

### 4.2 Result Merging Strategies

**Strategy A: Simple Concatenation**
```markdown
# Final Report

## Agent-01 Output
[Content]

## Agent-02 Output
[Content]
```

**Strategy B: Structured Merge**
```markdown
# Code Review Report

## Overview
- Code style issues: 12 (from Agent-02)
- Security vulnerabilities: 3 (from Agent-03)
- Performance issues: 5 (from Agent-04)

## Detailed Findings
### Code Style
[Merge Agent-02 detailed content]

### Security
[Merge Agent-03 detailed content]
```

**Strategy C: Intelligent Merge (Requires AI Processing)**
```powershell
# Use Claude to merge multiple results (with task-specific runtime path)
$taskId = "YYMMDD-task-name"
$runtimePath = ".agentdocs/runtime/$taskId"

$results = Get-Content "$runtimePath/results/*.md" -Raw
$mergePrompt = @"
Please merge the following multiple subtask results into a complete report:

$results

Requirements:
1. Preserve all key information
2. Eliminate duplicate content
3. Organize in logical order
4. Generate executive summary
"@

$finalReport = claude -p $mergePrompt 2>&1
$mergeExitCode = $LASTEXITCODE
if ($mergeExitCode -ne 0) {
    throw "Merge failed with exit code $mergeExitCode"
}

$finalReport | Out-File "$runtimePath/final_output.md"
```

### 4.3 Defensive Completion Artifacts (Required)

Before closing an implementation task, attach both checklists:

1. `Potential Bug Checklist`
   - Regression risks in existing behavior
   - Edge cases not fully covered
   - Error-path handling gaps
   - Performance or memory side effects

2. `Test Case Checklist`
   - Happy path
   - Edge case
   - Failure path

For bug-fix tasks, also include TDD evidence:
- Red evidence: failing output before fix
- Green evidence: passing output after fix

## State Persistence Specification

**Full orchestration only:** every state change must update `.agentdocs/runtime/<task-id>/master_plan.md`.

**All routed tasks:** every completed task must also update the matching workflow checkbox before you report completion.

**Lightweight mode:** record state directly in the workflow doc. Current-context updates are optional summaries only.

No task may be reported as complete while its workflow row/checkbox still shows pending.

```markdown
## Execution Log

### [2025-01-12 14:30:00] Initialization
- Created task plan
- Assigned 5 Agents

### [2025-01-12 14:30:15] Batch #1 Started
- Agent-01 started executing T-01

### [2025-01-12 14:30:22] Agent-01 Completed
- T-01 completed, duration 7s
- Output saved

### [2025-01-12 14:30:22] Batch #2 Started
- Agent-02, Agent-03, Agent-04 launched in parallel

### [2025-01-12 14:30:45] Batch #2 Completed
- All parallel tasks completed

### [2025-01-12 14:30:50] Result Aggregation Completed
- Final output: .agentdocs/runtime/<task-id>/final_output.md
```

## Phase 5: Status Sync and Cleanup (Detailed)

### 5.1 Update Workflow Document

After each task completion, synchronize status to workflow document immediately. This is a required completion step, not an optional cleanup pass.

```markdown
# .agentdocs/workflow/YYMMDD-task-name.md

## Implementation Plan

### Phase 1: Initial Setup
- [x] T-01: Setup project structure ✅
- [x] T-02: Configure dependencies ✅

### Phase 2: Core Implementation
- [ ] T-03: Implement feature X
- [ ] T-04: Add tests
```

**Required sync rule:**
- If a task finished, flip its workflow marker in the same step.
- For full orchestration, update the matching `master_plan.md` status row to `✅ Completed` at the same time.
- In parallel runs, use a coordinator if needed to serialize shared file writes, but do not skip or delay the sync beyond task completion.
- Do not batch these updates until the end.

### 5.2 Durable Memory Extraction (Required — do BEFORE archiving)

Extract reusable knowledge into `.agentdocs/index.md` **before** moving the workflow doc to `done/`.

**Write to these sections only:**
- `Architecture Decisions`
- `Coding Conventions`
- `Known Pitfalls`
- `Global Important Memory`

**Extraction rules:**
1. Keep each entry to 1-2 lines
2. Skip one-off details and runtime noise
3. Deduplicate against existing entries before append
4. Include date for decisions and pitfalls

**Automatic sync sequence:**
1. Read `.agentdocs/index.md` memory sections
2. Read task artifacts (`.agentdocs/workflow/<task-id>.md` + `.agentdocs/runtime/<task-id>/final_output.md` if it exists, or `.agentdocs/workflow/done/<task-id>.md` as fallback)
3. Use `templates.md` → `### Memory Sync Prompt Template` to generate update/add/skip lists
4. Apply only validated updates to `index.md`
5. Record in workflow notes: `Memory sync: completed` (or blocker reason)

### 5.3 Check Task Completion

After memory sync, when all TODOs in workflow document are marked as done, and full-orchestration `master_plan.md` rows are also synced to their final states:

1. **Move to archive**:
   ```bash
   mv .agentdocs/workflow/YYMMDD-task-name.md .agentdocs/workflow/done/
   ```

2. **Update index.md**:
   - Remove from "Current Task Documents" section
   - Optionally add to "Completed Tasks" section with brief summary

Example:

```markdown
## Known Pitfalls
- [2026-02-26] Symptom: flaky parallel write failures → Root cause: shared output path in non-isolated runtime → Fix: always use `.agentdocs/runtime/<task-id>/results/`.
```

### 5.4 Cleanup Task Runtime

**Critical**: Only clean up the specific task's runtime directory (full orchestration only — lightweight mode has no runtime dir to clean):

```bash
# Clean up this task's runtime
rm -rf .agentdocs/runtime/YYMMDD-task-name/

# Do NOT clean up other concurrent tasks
# Each task has isolated runtime
```

### 5.5 Documentation Output Restrictions

**Important**: Maintain minimal documentation footprint:

- ✅ **DO (full orchestration)**: Consolidate all outputs in `.agentdocs/runtime/<task-id>/final_output.md`
- ✅ **DO (lightweight mode)**: Synthesize results in context; optionally record key outcome notes in the workflow doc
- ✅ **DO**: Update workflow document TODOs
- ✅ **DO (full orchestration only)**: Clean up runtime after task completion
- ❌ **DO NOT**: Create additional summary documents in project root
- ❌ **DO NOT**: Generate separate README or documentation files
- ❌ **DO NOT**: Create feature summaries unless explicitly requested
- ❌ **DO NOT**: Leave temporary files or logs in project directories

**Rationale**:
- Workflow document captures planning and decisions (persistent)
- Runtime final_output.md captures full-orchestration execution results (temporary)
- Lightweight mode outputs stay in context; workflow doc captures durable notes
- Additional documentation creates clutter and confusion
- Keep the project clean and focused on actual code/content

### 5.6 Self-Evolution Rule Sync (Required)

When the user corrects process rules:

- Write or update the rule in **the target project's** `.agentdocs/local-rules.md` (do NOT modify the skill's own `RULES.md`, `SKILL.md`, or `workflow.md` — those live in the skill directory and are not project-editable).
- Mark the new rule as default for subsequent tasks in this project.
- Note the rule update in the current task's execution log.

### 5.7 Final Checklist

Before considering task complete:

- [ ] Plan provided and explicitly approved before implementation
- [ ] For bug fixes: failing repro/test captured before coding
- [ ] If >3 files, work executed in segmented stages
- [ ] Potential Bug Checklist attached
- [ ] Test Case Checklist attached
- [ ] Complexity Assessment recorded before routing
- [ ] All workflow TODOs marked as done
- [ ] No completed task left unchecked in workflow doc
- [ ] Full-orchestration `master_plan.md` statuses synced with workflow doc
- [ ] Workflow document moved to `done/` directory
- [ ] Index.md updated (removed from current tasks)
- [ ] Task runtime directory cleaned up (full orchestration only — skip for lightweight mode)
- [ ] No additional documentation files created
- [ ] Project directory remains clean
