# Templates: Distributed Task Orchestration Template Collection

## 1. Master Plan File Template (master_plan.md)

```markdown
# 🎯 Distributed Task Plan

## Original Request
> [Complete content of user's original request]

## Goal Definition
**Primary Goal**: [One sentence describing the final result to achieve]
**Success Criteria**: [How to determine task completion]

## Complexity Assessment

| Signal | Observation | Score |
|--------|-------------|-------|
| Atomic steps | [count / note] | [score] |
| Parallel streams | [yes/no + note] | [score] |
| Modules/systems/services | [count / note] | [score] |
| Long step (>5 min) | [yes/no] | [score] |
| Persisted review artifacts | [yes/no] | [score] |
| OpenCode available | [yes/no] | [score] |
| **Total** |  | **[score]** |

- **Chosen mode**: [Direct / Lightweight / Full orchestration]
- **Routing rationale**: [Why this mode fits]
- **Discipline**: Never leave this section blank; do not choose a mode before writing the score.

## ✅ Approval Gate (Must Pass Before Coding)
- [ ] Plan includes scope, impacted files, risk analysis, and verification strategy
- [ ] User explicitly approved implementation (e.g., "approved", "start")
- [ ] Verification baseline is defined before code changes

## 🧩 Segmented Change Plan (Required if >3 files)

| Stage | File Scope (<=3 files) | Objective | Verification |
|-------|-------------------------|-----------|--------------|
| S1 | [file-a, file-b, file-c] | [Goal] | [Checks/tests] |
| S2 | [Next <=3 files] | [Goal] | [Checks/tests] |
| S3 | [Next <=3 files] | [Goal] | [Checks/tests] |

---

## 📋 Task Decomposition

### Dependency Graph
```
[Use ASCII diagram or describe dependencies between tasks]
```

### Task List

| Task ID | Task Name | Description | Dependencies | Priority | Est. Time |
|---------|-----------|-------------|--------------|----------|-----------|
| T-01 | [Name] | [Brief description] | None | P0 | 1min |
| T-02 | [Name] | [Brief description] | T-01 | P1 | 2min |
| T-03 | [Name] | [Brief description] | T-01 | P1 | 3min |
| T-04 | [Name] | [Brief description] | T-02,T-03 | P2 | 2min |

---

## 🤖 Agent Assignment

| Task ID | Agent | Status | Start Time | End Time | Retries |
|---------|-------|--------|------------|----------|---------|
| T-01 | Agent-01 | 🟡 Pending | - | - | 0 |
| T-02 | Agent-02 | 🟡 Pending | - | - | 0 |
| T-03 | Agent-03 | 🟡 Pending | - | - | 0 |
| T-04 | Agent-04 | ⏸️ Waiting | - | - | 0 |

### Status Legend
- 🟡 Pending - Awaiting execution
- 🔵 Running - Currently executing
- ✅ Completed - Execution successful
- ❌ Failed - Execution failed
- ⏸️ Waiting - Dependencies not satisfied
- 🔄 Retrying - Retrying after failure

---

## 📊 Execution Progress

### Current Batch: #0
**Status**: Initializing

### Completion Statistics
- Total tasks: 4
- Completed: 0
- In progress: 0
- Waiting: 4
- Failed: 0

---

## 📝 Execution Log

### [YYYY-MM-DD HH:MM:SS] Initialization
- Task plan created
- Assigned N Agents

## 🔄 Completion Sync Gate

- [ ] Workflow TODOs updated for each completed task
- [ ] Full orchestration only: `master_plan.md` status rows synced to actual task state
- [ ] Implementation plan reflects failed, retried, abandoned, newly discovered, obsolete, blocked, or unplanned work
- [ ] No completed task remains pending in any plan artifact
- [ ] If tasks ran in parallel, a coordinator serialized shared plan-file updates

---

## ⚠️ Error Log

| Time | Agent | Task ID | Error Type | Description | Resolution |
|------|-------|---------|------------|-------------|------------|
| - | - | - | - | - | - |

---

## 📦 Final Output

**Output Location**: `.agentdocs/runtime/<task-id>/final_output.md`
**Status**: Pending generation
```

---

## 2. Agent Task File Template (agent-XX.md)

```markdown
# 🤖 Agent-XX Task Assignment

## Task Information
- **Task ID**: T-XX
- **Task Name**: [Task name]
- **Priority**: P1
- **Estimated Time**: 3 minutes

---

## 📥 Input

### Parameter List
| Parameter | Type | Source | Value/Description |
|-----------|------|--------|-------------------|
| param1 | string | User input | [Value] |
| param2 | file | T-01 output | .agentdocs/runtime/<task-id>/results/agent-01-result.md |

### Context Information
[Any background information helpful for completing the task]

---

## 🎯 Task Description

[Detailed description of task to complete]

### Specific Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

### TDD Bug Reproduction (Required for bug fixes)
- Repro script/test path: [path/to/failing-test-or-script]
- Expected failing output: [error message / failing assertion]
- Reproduced on: [environment]
- [ ] Failure reproduced before fix (Red)

---

## 📤 Expected Output

### Output Format
[Describe expected output format: text/JSON/Markdown etc.]

### Output Example
```
[Provide example of output format]
```

### Output Location
`.agentdocs/runtime/<task-id>/results/agent-XX-result.md`

---

## ⚠️ Constraints

- [Constraint 1: e.g., cannot modify original files]
- [Constraint 2: e.g., must use specific format]
- [Constraint 3: e.g., time limit]
- [Constraint 4: do not implement code before approval gate is passed]
- [Constraint 5: if expected changes >3 files, execute by segmented stages]
- [Constraint 6: before reporting success, update the workflow checkbox for this task]
- [Constraint 7: if full orchestration is active, also update the matching `master_plan.md` status row]
- [Constraint 8: if direct writes to shared plan files are unsafe, emit the exact marker change for the coordinator to apply immediately]

---

## 💡 Execution Hints

[Any hints or suggestions to help Agent complete the task better]
```

---

## 3. Agent Result File Template (agent-XX-result.md)

```markdown
# 📤 Agent-XX Execution Result

## Execution Summary
- **Task ID**: T-XX
- **Status**: ✅ Success / ❌ Failed
- **Start Time**: YYYY-MM-DD HH:MM:SS
- **End Time**: YYYY-MM-DD HH:MM:SS
- **Duration**: X.Xs

---

## 📋 Execution Process

### Step 1: [Step name]
- Action: [Action performed]
- Result: [Action result]

### Step 2: [Step name]
- Action: [Action performed]
- Result: [Action result]

---

## 📦 Output Result

[Actual output content]

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| Items processed | X |
| Successful | X |
| Warnings | X |
| Errors | X |

---

## ⚠️ Warnings and Errors

### Warnings
- [Warning information]

### Errors
- [Error information and how it was handled]

---

## 🐞 Potential Bug Checklist (Defensive Programming)

- [ ] Regression risk in existing behavior
- [ ] Unhandled edge cases
- [ ] Error-path handling gaps
- [ ] Performance/memory side effects

---

## ✅ Test Case Checklist

| Case Type | Scenario | Expected Result | Actual Result |
|-----------|----------|-----------------|---------------|
| Happy path | [Scenario] | [Expected] | [Actual] |
| Edge case | [Scenario] | [Expected] | [Actual] |
| Failure path | [Scenario] | [Expected] | [Actual] |

### TDD Evidence (for bug fixes)
- Red evidence (before fix): [failed test or script output]
- Green evidence (after fix): [passed test output]

---

## 🔄 Plan Sync Confirmation (Required)

- Workflow marker updated: [path + `- [x] T-XX` entry]
- Full orchestration only: `master_plan.md` row updated: [path + status]
- Plan drift handled: [none / added T-XX / abandoned T-XX / split T-XX / noted unplanned work]
- Verified no stale pending marker remains for this task: [yes/no]
- If a coordinator applied the write: [who applied it + when]

---

## 📎 Additional Information

[Any extra information useful for subsequent tasks]
```

---

## 4. Final Output File Template (final_output.md)

```markdown
# 📊 Distributed Task Execution Report

## Execution Summary

| Metric | Value |
|--------|-------|
| Total tasks | N |
| Successful tasks | X |
| Failed tasks | Y |
| Total duration | Zs |
| Parallel efficiency | XX% |

---

## 🎯 Original Goal

> [User's original request]

---

## ✅ Completion Status

### Task Completion Details

| Task ID | Task Name | Agent | Status | Duration |
|---------|-----------|-------|--------|----------|
| T-01 | [Name] | Agent-01 | ✅ | 1.2s |
| T-02 | [Name] | Agent-02 | ✅ | 2.3s |
| T-03 | [Name] | Agent-03 | ✅ | 1.8s |
| T-04 | [Name] | Agent-04 | ✅ | 0.9s |

---

## 📦 Integrated Results

[Final result logically integrated from each Agent's output]

### Part One: [Title]
[Processing result from Agent-01]

### Part Two: [Title]
[Merged results from Agent-02 and Agent-03]

### Part Three: [Title]
[Processing result from Agent-04]

---

## 📈 Key Findings/Recommendations

1. [Finding/Recommendation 1]
2. [Finding/Recommendation 2]
3. [Finding/Recommendation 3]

---

## ⚠️ Notes

- [Items requiring user attention]
- [Suggested follow-up actions]

---

## 📝 Execution Timeline

```
[Timeline visualization]
T-01: ████████ (1.2s)
T-02:          ████████████ (2.3s)
T-03:          █████████ (1.8s)  
T-04:                         █████ (0.9s)
─────────────────────────────────────────→ Time
0s                                      4.2s
```

---

## 📎 Appendix

### A. Detailed Agent Outputs
- [Agent-01 Result](./results/agent-01-result.md)
- [Agent-02 Result](./results/agent-02-result.md)
- [Agent-03 Result](./results/agent-03-result.md)
- [Agent-04 Result](./results/agent-04-result.md)

### B. Error Log
[List any errors here]
```

---

## 5. CLI Launch Script Templates

### Windows PowerShell (run-agents.ps1)

```powershell
# Distributed Task Orchestration - Agent Launch Script
# Usage: .\run-agents.ps1 -TaskId "YYMMDD-task-name" [-Parallel] [-MaxJobs 5]

param(
    [Parameter(Mandatory=$true)]
    [string]$TaskId,
    [switch]$Parallel = $false,
    [int]$MaxJobs = 4,
    [string]$TaskDir = ".agentdocs/runtime/$TaskId/agent_tasks",
    [string]$ResultDir = ".agentdocs/runtime/$TaskId/results"
)

# Ensure result directory exists
if (-not (Test-Path $ResultDir)) {
    New-Item -ItemType Directory -Path $ResultDir -Force | Out-Null
}

# Get all task files
$taskFiles = Get-ChildItem "$TaskDir/*.md" | Sort-Object Name

if ($taskFiles.Count -eq 0) {
    Write-Error "No task files found in $TaskDir"
    exit 1
}

$anyFailed = $false

Write-Host "═══════════════════════════════════════════════════" -ForegroundColor Cyan
Write-Host "       🚀 Distributed Task Orchestration - Agent Executor" -ForegroundColor Cyan
Write-Host "═══════════════════════════════════════════════════" -ForegroundColor Cyan
Write-Host ""
Write-Host "Found $($taskFiles.Count) tasks" -ForegroundColor Yellow
Write-Host "Parallel mode: $Parallel (Max concurrency: $MaxJobs)" -ForegroundColor Yellow
Write-Host ""

if ($Parallel) {
    # Parallel execution with MaxJobs throttling
    $pending = [System.Collections.Generic.Queue[object]]::new()
    foreach ($file in $taskFiles) { $pending.Enqueue($file) }
    $jobs = @()

    while ($pending.Count -gt 0 -or $jobs.Count -gt 0) {
        while ($pending.Count -gt 0 -and $jobs.Count -lt $MaxJobs) {
            $file = $pending.Dequeue()
            $agentId = $file.BaseName
            $job = Start-Job -Name $agentId -ScriptBlock {
                param($taskPath, $resultPath, $agentName)
                $task = Get-Content $taskPath -Raw
                $startTime = Get-Date
                
                try {
                    $result = claude -p $task 2>&1
                    $exitCode = $LASTEXITCODE
                    if ($exitCode -ne 0) {
                        throw "claude exited with code $exitCode"
                    }
                    $endTime = Get-Date
                    $duration = ($endTime - $startTime).TotalSeconds
                    
                    # Write result
                    @"
# Agent Execution Result

## Execution Info
- Agent: $agentName
- Status: ✅ Success
- Start: $startTime
- End: $endTime
- Duration: $duration seconds

## Output

$result
"@ | Out-File $resultPath -Encoding UTF8
                    
                    return @{
                        Agent = $agentName
                        Status = "Success"
                        Duration = $duration
                    }
                }
                catch {
                    $endTime = Get-Date
                    @"
# Agent Execution Result

## Execution Info
- Agent: $agentName
- Status: ❌ Failed
- Start: $startTime
- End: $endTime
- Error: $($_.Exception.Message)
"@ | Out-File $resultPath -Encoding UTF8
                    
                    return @{
                        Agent = $agentName
                        Status = "Failed"
                        Error = $_.Exception.Message
                    }
                }
            } -ArgumentList $file.FullName, "$ResultDir/$agentId-result.md", $agentId
            $jobs += $job
        }

        $finished = Wait-Job -Job $jobs -Any
        if ($null -ne $finished) {
            $result = Receive-Job $finished
            if ($result.Status -eq "Success") {
                Write-Host "✅ $($result.Agent): Success ($([math]::Round($result.Duration, 2))s)" -ForegroundColor Green
            } else {
                $anyFailed = $true
                Write-Host "❌ $($result.Agent): Failed - $($result.Error)" -ForegroundColor Red
            }
            Remove-Job $finished
            $jobs = @($jobs | Where-Object { $_.Id -ne $finished.Id })
        }
    }
    
    Write-Host ""
    Write-Host "═══════════════════════════════════════════════════" -ForegroundColor Green
    Write-Host "                   Execution Complete" -ForegroundColor Green
    Write-Host "═══════════════════════════════════════════════════" -ForegroundColor Green
}
else {
    # Serial execution
    foreach ($file in $taskFiles) {
        $agentId = $file.BaseName
        Write-Host "▶ Executing $agentId..." -ForegroundColor Cyan
        
        $task = Get-Content $file.FullName -Raw
        $startTime = Get-Date
        
        try {
            $result = claude -p $task 2>&1
            $exitCode = $LASTEXITCODE
            if ($exitCode -ne 0) {
                throw "claude exited with code $exitCode"
            }
            $endTime = Get-Date
            $duration = ($endTime - $startTime).TotalSeconds
            
            $result | Out-File "$ResultDir/$agentId-result.md" -Encoding UTF8
            Write-Host "  ✅ Completed ($([math]::Round($duration, 2))s)" -ForegroundColor Green
        }
        catch {
            $anyFailed = $true
            $endTime = Get-Date
            @"
# Agent Execution Result

## Execution Info
- Agent: $agentId
- Status: ❌ Failed
- Start: $startTime
- End: $endTime
- Error: $($_.Exception.Message)
"@ | Out-File "$ResultDir/$agentId-result.md" -Encoding UTF8
            Write-Host "  ❌ Failed: $($_.Exception.Message)" -ForegroundColor Red
        }
    }
}

Write-Host ""
Write-Host "Results saved to: $ResultDir" -ForegroundColor Yellow

if ($anyFailed) {
    Write-Error "One or more agents failed. Exiting with non-zero status."
    exit 1
}
```

### Bash Script (run-agents.sh)

```bash
#!/bin/bash

# Distributed Task Orchestration - Agent Launch Script
# Usage: ./run-agents.sh -t "YYMMDD-task-name" [-p] [-j 4]

PARALLEL=false
MAX_JOBS=4
TASK_ID=""
any_failed=false

while getopts "t:pj:" opt; do
    case $opt in
        t) TASK_ID=$OPTARG ;;
        p) PARALLEL=true ;;
        j) MAX_JOBS=$OPTARG ;;
    esac
done

if [ -z "$TASK_ID" ]; then
    echo "Error: Task ID is required. Use -t <task-id>"
    exit 1
fi

TASK_DIR=".agentdocs/runtime/$TASK_ID/agent_tasks"
RESULT_DIR=".agentdocs/runtime/$TASK_ID/results"

if $PARALLEL && ! command -v parallel >/dev/null 2>&1; then
    echo "Error: GNU parallel is required for -p mode"
    exit 1
fi

# Ensure result directory exists
mkdir -p "$RESULT_DIR"

shopt -s nullglob
task_files=("$TASK_DIR"/*.md)
shopt -u nullglob

if [ ${#task_files[@]} -eq 0 ]; then
    echo "No task files found in $TASK_DIR" >&2
    exit 1
fi

echo "═══════════════════════════════════════════════════"
echo "       🚀 Distributed Task Orchestration - Agent Executor"
echo "═══════════════════════════════════════════════════"
echo ""
echo "Task ID: $TASK_ID"

task_count=${#task_files[@]}
echo "Found $task_count tasks"
echo "Parallel mode: $PARALLEL (Max concurrency: $MAX_JOBS)"
echo ""

run_agent() {
    local task_file=$1
    local agent_id=$(basename "$task_file" .md)
    local result_file="$RESULT_DIR/${agent_id}-result.md"

    local start_time=$(date +%s)

    if claude -p "$(cat "$task_file")" > "$result_file" 2>&1; then
        local end_time=$(date +%s)
        local duration=$((end_time - start_time))
        echo "✅ $agent_id: Success (${duration}s)"
        return 0
    else
        echo "❌ $agent_id: Failed"
        return 1
    fi
}

export -f run_agent
export RESULT_DIR

if $PARALLEL; then
    if ! printf '%s\n' "${task_files[@]}" | parallel -j "$MAX_JOBS" run_agent {}; then
        any_failed=true
    fi
else
    for task_file in "${task_files[@]}"; do
        if ! run_agent "$task_file"; then
            any_failed=true
        fi
    done
fi

echo ""
echo "Results saved to: $RESULT_DIR"

if $any_failed; then
    echo "One or more agents failed. Exiting with non-zero status." >&2
    exit 1
fi
```

---

## 6. Quick Initialization Template

### Initialization Script (init-orchestrator.ps1)

```powershell
# Initialize distributed task orchestration directory structure
# Usage: .\init-orchestrator.ps1 -TaskId "YYMMDD-task-name"
param(
    [Parameter(Mandatory=$true)]
    [string]$TaskId
)

$baseDir = ".agentdocs"
$runtimeDir = "$baseDir/runtime/$TaskId"
$workflowPath = "$baseDir/workflow/$TaskId.md"

# Create directory structure
$dirs = @(
    $baseDir,
    "$baseDir/workflow",
    "$baseDir/workflow/done",
    $runtimeDir,
    "$runtimeDir/agent_tasks",
    "$runtimeDir/results"
)

foreach ($dir in $dirs) {
    if (-not (Test-Path $dir)) {
        New-Item -ItemType Directory -Path $dir -Force | Out-Null
        Write-Host "✅ Created directory: $dir" -ForegroundColor Green
    }
}

# Create workflow doc stub
$workflowDoc = @"
# .agentdocs/workflow/$TaskId.md

## Task Overview
[Fill in brief description]

## Current Analysis
[Fill in problem analysis, constraints, considerations]

## Solution Design
[Fill in approach and key decisions]

## Complexity Assessment
- Atomic steps: [count] → [score]
- Parallel streams: [yes/no] → [score]
- Modules/systems/services: [count] → [score]
- Long step (>5 min): [yes/no] → [score]
- Persisted review artifacts: [yes/no] → [score]
- OpenCode available: [yes/no] → [score]
- **Total score**: [number]
- **Chosen mode**: [Direct / Lightweight / Full orchestration]

## Implementation Plan

### Phase 1: [Phase Name]
- [ ] T-01: [Task description]

## Notes
[Important observations, blockers, decisions]
- Plan maintenance: [T-XX completed / added / abandoned / retried / failed / blocked] — [reason]
"@

$workflowDoc | Out-File $workflowPath -Encoding UTF8
Write-Host "✅ Created workflow doc: $workflowPath" -ForegroundColor Green

# Create master plan file
$masterPlan = @"
# 🎯 Distributed Task Plan: $TaskId

## Workflow Reference
\`.agentdocs/workflow/$TaskId.md\`

## Original Request
> [Fill in user request here]

## Goal Definition
**Primary Goal**: [Goal description]
**Success Criteria**: [Success criteria]

## Complexity Assessment

| Signal | Observation | Score |
|--------|-------------|-------|
| Atomic steps | [count / note] | [score] |
| Parallel streams | [yes/no + note] | [score] |
| Modules/systems/services | [count / note] | [score] |
| Long step (>5 min) | [yes/no] | [score] |
| Persisted review artifacts | [yes/no] | [score] |
| OpenCode available | [yes/no] | [score] |
| **Total** |  | **[score]** |

- **Chosen mode**: [Direct / Lightweight / Full orchestration]
- **Routing rationale**: [Why this mode fits]

If the mode is `Direct`, copy the same block into the response or task notes before execution because no workflow doc will be created.

---

## 📋 Task Decomposition

| Task ID | Task Name | Description | Dependencies | Priority |
|---------|-----------|-------------|--------------|----------|
| T-01 | | | None | P0 |

---

## 🤖 Agent Assignment

| Task ID | Agent | Status | Start Time | End Time |
|---------|-------|--------|------------|----------|
| T-01 | Agent-01 | 🟡 Pending | - | - |

---

## 📝 Execution Log

### [$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')] Initialization
- Task plan created
- Routing score recorded
"@

$masterPlan | Out-File "$runtimeDir/master_plan.md" -Encoding UTF8
Write-Host "✅ Created master plan: $runtimeDir/master_plan.md" -ForegroundColor Green

Write-Host ""
Write-Host "🎉 Initialization complete!" -ForegroundColor Cyan
Write-Host "Next step: Edit $workflowPath and $runtimeDir/master_plan.md to define tasks" -ForegroundColor Yellow
```

---

## 7. Durable Memory Templates

### `.agentdocs/index.md` Full Template

This is the canonical structure for the knowledge entry point. Create this file when setting up a new project:

```markdown
# .agentdocs/index.md

## Current Task Documents
<!-- Add active workflow docs here -->
<!-- Format: - [YYMMDD-task-name](workflow/YYMMDD-task-name.md) — brief description -->

## Completed Tasks
<!-- Optionally summarize completed tasks here -->
<!-- Format: - [YYMMDD-task-name] — outcome summary -->

## Architecture Decisions
- [YYYY-MM-DD] [Decision] — [Why] — [Scope/Impact]

## Coding Conventions
- [Rule] — [Example path]

## Known Pitfalls
- [YYYY-MM-DD] [Symptom] → [Root cause] → [Fix]

## Global Important Memory
- [Long-lived constraint or preference]
```

### Memory Entry Template (append/update)

Use the concise 1–2 line bullet format matching the four `index.md` sections:

```markdown
# Architecture Decisions
- [YYYY-MM-DD] [Decision] — [Why] — [Scope/Impact]

# Coding Conventions
- [Rule] — [Example path]

# Known Pitfalls
- [YYYY-MM-DD] [Symptom] → [Root cause] → [Fix]

# Global Important Memory
- [Long-lived constraint or preference]
```

> Do NOT use multi-line structured entries — keep each item to 1–2 lines max.

### Memory Extraction Checklist

- [ ] Is this reusable beyond the current task?
- [ ] Is this non-obvious and worth remembering?
- [ ] Is this already present in index.md (deduplicated)?
- [ ] Is the entry concise (1-2 lines, plus date if needed)?

### Memory Sync Prompt Template

```markdown
[CONTEXT]
You are syncing durable project memory for agentdocs-orchestrator.

[INPUT]
1) Current memory file: `.agentdocs/index.md`
2) Completed workflow doc: `.agentdocs/workflow/<task-id>.md` or `.agentdocs/workflow/done/<task-id>.md`
3) Execution result: `.agentdocs/runtime/<task-id>/final_output.md` (if exists)

[TASK]
Extract reusable knowledge and update only these sections in `index.md`:
- Architecture Decisions
- Coding Conventions
- Known Pitfalls
- Global Important Memory

[RULES]
- Keep each memory entry to 1-2 lines
- Do not include runtime logs or one-off details
- Deduplicate: update existing similar entries instead of adding duplicates
- Include date for Architecture Decisions and Known Pitfalls
- Respect `.agentdocs/local-rules.md` if present

[OUTPUT FORMAT]
Return three blocks:

## Update Existing Entries
- [Section] [Old entry] -> [New entry]

## Add New Entries
- [Section] [New memory entry]

## Skip (Not Durable)
- [Reason] [Candidate detail]
```
