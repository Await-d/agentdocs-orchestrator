# Examples: Distributed Task Orchestration Practical Examples

## Example 1: Codebase Analysis Task

### User Request
> "Analyze my TypeScript project, check code quality, security vulnerabilities, and performance issues, and generate a complete report"

---

### Phase 1️⃣ Task Analysis and Decomposition

```markdown
# .agentdocs/runtime/260112-code-analysis/master_plan.md

## Original Request
> Analyze TypeScript project for code quality, security vulnerabilities, and performance issues

## Goal Definition
**Primary Goal**: Generate a comprehensive code analysis report
**Success Criteria**: Cover code quality, security, and performance dimensions

## Task Decomposition

### Dependency Graph
```
[T-01: Code Scan] ──┬──→ [T-02: Quality Analysis]
                    ├──→ [T-03: Security Scan]
                    └──→ [T-04: Performance Analysis]
                               ↓
                    [T-05: Generate Report] ←─────┘
```

### Task List
| Task ID | Task Name | Description | Dependencies | Priority |
|---------|-----------|-------------|--------------|----------|
| T-01 | Code Scan | Read all .ts/.tsx files | None | P0 |
| T-02 | Quality Analysis | Check type usage, code style | T-01 | P1 |
| T-03 | Security Scan | Check vulnerabilities and sensitive info | T-01 | P1 |
| T-04 | Performance Analysis | Analyze performance issue patterns | T-01 | P1 |
| T-05 | Generate Report | Integrate all analysis results | T-02,T-03,T-04 | P2 |
```

---

### Phase 2️⃣ Agent Assignment

```markdown
## Task Status Table
| Task ID | Agent | Status | Start Time | End Time |
|---------|-------|--------|------------|----------|
| T-01 | Agent-01 | 🟡 Pending | - | - |
| T-02 | Agent-02 | ⏸️ Waiting | - | - |
| T-03 | Agent-03 | ⏸️ Waiting | - | - |
| T-04 | Agent-04 | ⏸️ Waiting | - | - |
| T-05 | Agent-05 | ⏸️ Waiting | - | - |
```

**Agent-01 Task File** (`.agentdocs/runtime/260112-code-analysis/agent_tasks/agent-01.md`):
```markdown
# Agent-01 Task: Code Scan

## Task Description
Scan the project's src/ directory, read all TypeScript files (.ts, .tsx)

## Expected Output
1. File list (path, line count, size)
2. Project statistics (total files, total lines)
3. Main entry file identification

## Output Format
```json
{
  "files": [{"path": "...", "lines": 100, "size": 2048}],
  "stats": {"totalFiles": 15, "totalLines": 2847}
}
```
```

---

### Phase 3️⃣ Sequential Execution (in-context)

```
══════════════════════════════════════════════════════════════════
                    🚀 Execution Batch #1
══════════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────────────
│ 🤖 Agent-01 [T-01: Code Scan]
├──────────────────────────────────────────────────────────────────
│ 📥 Instruction: Scan all TypeScript files in src/ directory
│ ⚙️ Execution:
│    → Traverse src/ directory structure
│    → Identify .ts and .tsx files
│    → Read file contents and calculate statistics
│ 📤 Output: 
│    {
│      "files": [
│        {"path": "src/App.tsx", "lines": 45},
│        {"path": "src/components/Button.tsx", "lines": 32},
│        ... (15 files total)
│      ],
│      "stats": {"totalFiles": 15, "totalLines": 2847}
│    }
│ ⏱️ Duration: 1.8s
│ ✅ Status: Completed
└──────────────────────────────────────────────────────────────────

══════════════════════════════════════════════════════════════════
                    🚀 Execution Batch #2 (Parallel)
══════════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────────────
│ 🤖 Agent-02 [T-02: Quality Analysis]     🔄 Parallel Execution
├──────────────────────────────────────────────────────────────────
│ 📥 Instruction: Analyze code quality and type usage
│ ⚙️ Execution:
│    → Check any type usage
│    → Check unused variables
│    → Check code complexity
│ 📤 Output:
│    - any type usage: 8 occurrences
│    - Unused variables: 3 occurrences
│    - High complexity functions: 2
│ ✅ Completed (3.2s)
├──────────────────────────────────────────────────────────────────
│ 🤖 Agent-03 [T-03: Security Scan]     🔄 Parallel Execution
├──────────────────────────────────────────────────────────────────
│ 📥 Instruction: Scan for security vulnerabilities
│ ⚙️ Execution:
│    → Check hardcoded keys
│    → Check SQL injection risks
│    → Check XSS risks
│ 📤 Output:
│    - Warning: Found 1 potential API key
│    - Suggestion: 2 places need input validation
│ ✅ Completed (2.8s)
├──────────────────────────────────────────────────────────────────
│ 🤖 Agent-04 [T-04: Performance Analysis]     🔄 Parallel Execution
├──────────────────────────────────────────────────────────────────
│ 📥 Instruction: Analyze performance issues
│ ⚙️ Execution:
│    → Check unnecessary re-renders
│    → Check large dependencies
│    → Check memory leak patterns
│ 📤 Output:
│    - Missing useMemo: 5 places
│    - Missing useCallback: 3 places
│    - Suggest code splitting: 1 place
│ ✅ Completed (2.5s)
└──────────────────────────────────────────────────────────────────

══════════════════════════════════════════════════════════════════
                    🚀 Execution Batch #3
══════════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────────────
│ 🤖 Agent-05 [T-05: Generate Report]
├──────────────────────────────────────────────────────────────────
│ 📥 Instruction: Integrate all analysis results to generate report
│ ⚙️ Execution:
│    → Read Agent-02,03,04 results
│    → Sort issues by priority
│    → Generate Markdown report
│ 📤 Output: .agentdocs/runtime/260112-code-analysis/final_output.md
│ ✅ Completed (1.5s)
└──────────────────────────────────────────────────────────────────
```

---

### Phase 4️⃣ Result Aggregation

```markdown
# .agentdocs/runtime/260112-code-analysis/final_output.md

## 📊 Code Analysis Report

### Execution Summary
| Metric | Value |
|--------|-------|
| Files analyzed | 15 |
| Total lines of code | 2,847 |
| Issues found | 23 |
| Execution time | 11.8 seconds |

### 🔴 Critical Issues (Immediate Action Required)

#### 1. Security Issues
- **API Key Exposure** (src/api/config.ts:12)
  ```typescript
  const API_KEY = "sk-abc123...";  // ❌ Hardcoded
  ```
  **Recommendation**: Use environment variable `process.env.API_KEY`

### 🟡 Warnings (Recommended to Address)

#### 2. Code Quality
- 8 occurrences of `any` type
- 3 unused variables
- 2 high complexity functions (cyclomatic complexity > 10)

#### 3. Performance Optimization
- 5 places missing `useMemo`
- 3 places missing `useCallback`
- Recommend code splitting for `HeavyComponent`

### 📋 Detailed Findings List

[Complete list of issues...]
```

---

## Example 2: Multi-Document Translation Task

### User Request
> "Translate the 5 English documents in docs/ directory to Chinese"

---

### Task Decomposition

```markdown
## Task Decomposition

### Dependencies
No dependencies, all translation tasks can be fully parallelized

### Task List
| Task ID | File | Agent | Status |
|---------|------|-------|--------|
| T-01 | docs/intro.md | Agent-01 | 🟡 |
| T-02 | docs/getting-started.md | Agent-02 | 🟡 |
| T-03 | docs/api-reference.md | Agent-03 | 🟡 |
| T-04 | docs/tutorials.md | Agent-04 | 🟡 |
| T-05 | docs/faq.md | Agent-05 | 🟡 |
```

### Parallel Execution

```powershell
# All Agents launch simultaneously
$taskId = (Get-Date -Format 'yyMMdd') + "-translate"
$resultDir = ".agentdocs/runtime/$taskId/results"
New-Item -ItemType Directory -Path $resultDir -Force | Out-Null

$docs = Get-ChildItem "docs/*.md"
$jobs = foreach ($doc in $docs) {
    $index = [array]::IndexOf($docs, $doc) + 1
    $agentId = "agent-{0:D2}" -f $index
    Start-Job -Name $agentId -ScriptBlock {
        param($file, $resultPath)
        $content = Get-Content $file -Raw
        $result = claude -p "Translate the following English content to Chinese, maintain Markdown format: $content"
        $result | Out-File $resultPath -Encoding UTF8
    } -ArgumentList $doc.FullName, "$resultDir/$agentId-result.md"
}

# Wait for all to complete in parallel
$jobs | Wait-Job | Out-Null
$jobs | Remove-Job
Write-Host "Results saved to $resultDir"
```

### Execution Result

```
══════════════════════════════════════════════════════════════════
              🚀 Parallel Execution: 5 translation tasks running simultaneously
══════════════════════════════════════════════════════════════════

Agent-01 [intro.md]         ████████████████████░░░░ 80%
Agent-02 [getting-started]  ██████████████████████░░ 90%
Agent-03 [api-reference]    ████████████░░░░░░░░░░░░ 50%
Agent-04 [tutorials.md]     ██████████████████░░░░░░ 75%
Agent-05 [faq.md]           ████████████████████████ 100% ✅

[Waiting...]

Agent-02 completed ✅
Agent-01 completed ✅
Agent-04 completed ✅
Agent-03 completed ✅

══════════════════════════════════════════════════════════════════
                    ✅ All translations complete
══════════════════════════════════════════════════════════════════
Total duration: 45 seconds (Serial estimate: 180 seconds)
Parallel efficiency: 4x speedup
```

---

## Example 3: API Endpoint Testing Task

### User Request
> "Test all API endpoints for response time and correctness"

---

### Task Decomposition

```markdown
## Endpoint List
| Endpoint | Method | Agent |
|----------|--------|-------|
| /api/users | GET | Agent-01 |
| /api/users/:id | GET | Agent-02 |
| /api/users | POST | Agent-03 |
| /api/products | GET | Agent-04 |
| /api/orders | GET | Agent-05 |
```

### Agent Task Template

```markdown
# Agent-01 Task: Test GET /api/users

## Test Cases
1. Normal request - Expect 200 OK
2. Invalid parameters - Expect 400 Bad Request
3. Unauthorized - Expect 401 Unauthorized

## Validation Points
- Response time < 500ms
- Returns JSON format
- Contains pagination info

## Output Format
```json
{
  "endpoint": "/api/users",
  "tests": [
    {"case": "Normal request", "status": "pass", "responseTime": 123}
  ],
  "summary": {"total": 3, "pass": 3, "fail": 0}
}
```
```

### Final Report

```markdown
# API Test Report

## Overview
| Metric | Value |
|--------|-------|
| Endpoints tested | 5 |
| Test cases | 15 |
| Passed | 14 |
| Failed | 1 |
| Average response time | 156ms |

## Failed Cases

### ❌ POST /api/users - Large data volume test
- Status: Timeout
- Response time: 5023ms (Threshold: 1000ms)
- Recommendation: Optimize database write performance

## Endpoint Performance Ranking

| Rank | Endpoint | Avg Response Time |
|------|----------|-------------------|
| 1 | GET /api/products | 89ms |
| 2 | GET /api/users | 123ms |
| 3 | GET /api/users/:id | 145ms |
| 4 | GET /api/orders | 198ms |
| 5 | POST /api/users | 856ms |
```

---

## Example 4: Real Execution Using Claude CLI (Full Orchestration)

> **Note:** This example uses full orchestration mode (runtime dir + agent files). It decomposes the request into multiple tasks via claude, then runs them in parallel. Use this for tasks with 5+ steps or multiple independent streams.

### Complete PowerShell Script

```powershell
# orchestrate.ps1 - Distributed Task Orchestration Example (Full Orchestration)

param(
    [string]$Request = "Analyze current directory code structure"
)

Write-Host "═══════════════════════════════════════════════════" -ForegroundColor Cyan
Write-Host "       🤖 Distributed Task Orchestration System" -ForegroundColor Cyan
Write-Host "═══════════════════════════════════════════════════" -ForegroundColor Cyan
Write-Host ""

# Phase 1: Task Decomposition
Write-Host "📋 Phase 1: Analyzing request and decomposing tasks..." -ForegroundColor Yellow

$decomposePrompt = @"
You are a task decomposition expert. Please decompose the following request into 3-5 independent atomic tasks.

Request: $Request

Output format (JSON):
{
  "tasks": [
    {"id": "T-01", "name": "Task name", "description": "Detailed description", "deps": []}
  ]
}
"@

$taskJson = claude --output-format json -p $decomposePrompt 2>$null
try { $tasks = $taskJson | ConvertFrom-Json } catch { Write-Error "Failed to parse JSON from claude output: $_"; exit 1 }

Write-Host "  ✅ Decomposition complete: $($tasks.tasks.Count) tasks" -ForegroundColor Green

# Phase 2: Create Agent task files
Write-Host ""
Write-Host "🤖 Phase 2: Assigning agents..." -ForegroundColor Yellow

$taskId = (Get-Date -Format 'yyMMdd') + "-orchestrate"
$orchestratorDir = ".agentdocs/runtime/$taskId"
$workflowPath = ".agentdocs/workflow/$taskId.md"
New-Item -ItemType Directory -Path ".agentdocs/workflow" -Force | Out-Null
@"
# .agentdocs/workflow/$taskId.md

## Task Overview
$Request

## Implementation Plan
- [ ] Run distributed orchestration via orchestrate.ps1
"@ | Out-File $workflowPath -Encoding UTF8
New-Item -ItemType Directory -Path "$orchestratorDir/agent_tasks" -Force | Out-Null
New-Item -ItemType Directory -Path "$orchestratorDir/results" -Force | Out-Null

$agentIndex = 1
foreach ($task in $tasks.tasks) {
    $agentId = "agent-{0:D2}" -f $agentIndex
    
    $taskContent = @"
# $agentId Task

## Task ID: $($task.id)
## Task Name: $($task.name)

$($task.description)

Please complete this task and output the result.
"@
    
    $taskContent | Out-File "$orchestratorDir/agent_tasks/$agentId.md" -Encoding UTF8
    Write-Host "  📝 Created $agentId -> $($task.name)" -ForegroundColor Gray
    $agentIndex++
}

# Phase 3: Parallel Execution
Write-Host ""
Write-Host "🚀 Phase 3: Parallel execution..." -ForegroundColor Yellow

$taskFiles = Get-ChildItem "$orchestratorDir/agent_tasks/*.md"
$startTime = Get-Date

$jobs = foreach ($file in $taskFiles) {
    $agentId = $file.BaseName
    Write-Host "  ▶ Launching $agentId" -ForegroundColor Cyan
    
    Start-Job -Name $agentId -ScriptBlock {
        param($taskPath, $resultPath)
        $task = Get-Content $taskPath -Raw
        $result = claude -p $task 2>&1
        $result | Out-File $resultPath -Encoding UTF8
        return $result
    } -ArgumentList $file.FullName, "$orchestratorDir/results/$agentId-result.md"
}

# Wait for completion
$jobs | Wait-Job | Out-Null
$endTime = Get-Date
$duration = ($endTime - $startTime).TotalSeconds

Write-Host ""
foreach ($job in $jobs) {
    $status = if ($job.State -eq 'Completed') { "✅" } else { "❌" }
    Write-Host "  $status $($job.Name) completed" -ForegroundColor Green
}

# Phase 4: Result Aggregation
Write-Host ""
Write-Host "📊 Phase 4: Aggregating results..." -ForegroundColor Yellow

$allResults = Get-ChildItem "$orchestratorDir/results/*.md" | ForEach-Object {
    "## $($_.BaseName)`n`n$(Get-Content $_ -Raw)"
} | Out-String

$mergePrompt = @"
Please integrate the following multiple subtask execution results into a concise report:

$allResults

Requirements: Generate executive summary and key findings
"@

$finalReport = claude -p $mergePrompt 2>$null
$finalReport | Out-File ".agentdocs/runtime/$taskId/final_output.md" -Encoding UTF8

Write-Host "  ✅ Report generation complete" -ForegroundColor Green

# Cleanup jobs
$jobs | Remove-Job

# Phase 5: Status sync, memory sync, and cleanup
Write-Host ""
Write-Host "🧠 Phase 5: Syncing status and cleaning up..." -ForegroundColor Yellow

# Update workflow TODO (manual step — mark items done in .agentdocs/workflow/$taskId.md)
Write-Host "  ✏️  Update .agentdocs/workflow/$taskId.md — mark all TODOs as done" -ForegroundColor Gray

# Memory sync reminder
Write-Host "  💾 Extract durable memory into .agentdocs/index.md before archiving" -ForegroundColor Gray

# Archive workflow doc
# mv ".agentdocs/workflow/$taskId.md" ".agentdocs/workflow/done/$taskId.md"

# Review final report before cleanup
Write-Host "  👀 Review the final report before removing runtime artifacts" -ForegroundColor Gray

# Optional cleanup after report has been reviewed or copied elsewhere
# Remove-Item -Recurse -Force $orchestratorDir
# Write-Host "  🗑️  Runtime dir cleaned: $orchestratorDir" -ForegroundColor Gray

# Output summary
Write-Host ""
Write-Host "═══════════════════════════════════════════════════" -ForegroundColor Green
Write-Host "                   ✅ Execution Complete" -ForegroundColor Green
Write-Host "═══════════════════════════════════════════════════" -ForegroundColor Green
Write-Host ""
Write-Host "📄 Final report: .agentdocs/runtime/$taskId/final_output.md" -ForegroundColor Yellow
Write-Host "⏱️ Total duration: $([math]::Round($duration, 2)) seconds" -ForegroundColor Yellow
```

---

## Example 5: Error Recovery Pattern

### Scenario: Agent-03 Execution Failed

```
══════════════════════════════════════════════════════════════════
                    🚀 Execution Batch #2
══════════════════════════════════════════════════════════════════

Agent-01 ✅ Completed (2.1s)
Agent-02 ✅ Completed (1.8s)
Agent-03 ❌ Failed - Timeout
Agent-04 ✅ Completed (2.3s)

══════════════════════════════════════════════════════════════════
                    🔄 Error Recovery
══════════════════════════════════════════════════════════════════

Agent-03 failure detected
→ Update master_plan.md error log
→ Retry #1/3...
→ Agent-03 re-executing
→ ✅ Retry successful (3.5s)

══════════════════════════════════════════════════════════════════
                    Continuing Execution
══════════════════════════════════════════════════════════════════
```

### Error Log Update

```markdown
## ⚠️ Error Log

| Time | Agent | Error | Retries | Result |
|------|-------|-------|---------|--------|
| 14:30:22 | Agent-03 | Execution timeout (>60s) | 1 | ✅ Success |
```

---

## Example 6: Bug Fix with Approval Gate + TDD + Segmented Changes

### User Request
> "Fix intermittent 500 error in order API."

### Phase 0️⃣ Mandatory Gates

```markdown
## Approval Gate
- Plan:
  - Goal: Fix order API intermittent 500 under concurrent requests
  - Impacted files (estimated):
    1. src/api/orders/controller.ts
    2. src/api/orders/service.ts
    3. src/api/orders/repository.ts
    4. tests/orders/concurrency.test.ts
  - Risk: regression on normal order creation flow
  - Verification: run targeted order API tests + concurrency repro script

- User Approval: ✅ Explicitly approved before coding
```

### TDD Red Step (Reproduce Before Fix)

```bash
# Failing reproduction script
node scripts/repro-order-500.js

# Expected failing signal
# Error: 500 Internal Server Error (observed in >= 3/50 concurrent requests)
```

### Segmented Change Plan (>3 files)

```markdown
| Stage | Files (<=3) | Goal | Verification |
|-------|-------------|------|--------------|
| S1 | controller.ts, service.ts | Add request-level guard and idempotency path | Existing order API tests pass |
| S2 | repository.ts, concurrency.test.ts | Make write path atomic + add regression test | Repro script turns green |
```

### Green Step (Minimal Fix)

```
Stage S1 ✅ Completed
- No 500 on standard request path

Stage S2 ✅ Completed
- Concurrency test added and passing
- Repro script: 0/50 failures
```

### Defensive Programming Output (Required)

```markdown
## Potential Bug Checklist
- [x] Checked regression risk on normal order create flow
- [x] Reviewed edge case: duplicate request retries
- [x] Reviewed failure path: DB timeout fallback
- [x] Reviewed side effects: lock contention/performance

## Test Case Checklist
| Case Type | Scenario | Result |
|-----------|----------|--------|
| Happy path | create order with valid payload | ✅ pass |
| Edge case | duplicate request id | ✅ pass |
| Failure path | DB timeout injection | ✅ pass |

## TDD Evidence
- Red: `scripts/repro-order-500.js` failed before fix
- Green: same script passed after fix (0/50 failures)
```

### Self-Evolution Update Example

When the user adds a new process rule during review:

```markdown
User correction: "Always include rollback notes for production-impacting changes."

Action:
1. Add this rule to the **target project's** `.agentdocs/local-rules.md` (do NOT edit the skill's own RULES.md/SKILL.md/workflow.md)
2. Apply it as default for all subsequent tasks in this project
3. Record update in current execution log
```

---

## Tips Summary

### 1. Task Granularity Control
- ✅ Good: "Analyze code quality in src/components directory"
- ❌ Bad: "Analyze entire codebase" (too large)
- ❌ Bad: "Check one variable" (too small)

### 2. Minimize Dependencies
- Design independent tasks whenever possible
- Use files to pass intermediate results
- Avoid circular dependencies

### 3. Parallel Efficiency
- More independent tasks = greater parallel benefits
- Control concurrency (recommend 4-8)
- Monitor resource usage

### 4. Error Handling
- Log all failures
- Implement automatic retry
- Preserve partial results
