# Notes: Distributed Task Orchestration System Design

## Core Concepts

### 0. Design Principles

**Minimal Documentation Principle**
- Keep documentation footprint minimal and focused
- Avoid creating unnecessary summary or explanation documents
- All task outputs should consolidate in designated locations
- Clean up temporary files after task completion

**Documentation Hierarchy**
1. **Workflow documents** (persistent) - Planning, analysis, decisions
2. **Runtime final_output.md** (temporary) - Execution results
3. **No additional files** - Avoid README, SUMMARY, or extra docs

**Anti-Patterns to Avoid**
- ❌ Creating summary documents after task completion
- ❌ Generating feature documentation unless explicitly requested
- ❌ Leaving temporary files in project root
- ❌ Creating redundant documentation that duplicates workflow docs

### 1. Two-Layer Architecture

**Workflow Layer (Persistent)**
- Task planning and thinking documentation
- Stored in `.agentdocs/workflow/`
- Captures analysis, design decisions, TODOs
- Moves to `done/` when completed

**Runtime Layer (Temporary, Task-Isolated)**
- Distributed execution coordination
- Stored in `.agentdocs/runtime/<task-id>/`
- Each task has its own isolated runtime directory
- Agent task files and results
- Cleaned up after specific task completion

### 2. Task Decomposition
- Break complex requests into independent atomic tasks
- Identify dependencies between tasks
- Define input/output for each task

### 3. Virtual Agents
- Agent-01, Agent-02, ... Agent-N
- Each agent is responsible for one or more atomic tasks
- Agents can be:
  - Sequential execution (in-context)
  - Independent processes launched via Claude CLI

### 4. Task State Management
```
Pending → Running → Completed
              ↓
           Failed
```

### 5. Claude CLI Integration Methods
```bash
# Basic call
claude -p "Task description" --output-format json

# Call with context
claude -p "$(cat context.md)\n\nTask description"

# Background execution
Start-Process claude -ArgumentList '-p "Task description"' -NoNewWindow
```

### 6. LSP Protocol Priority
- When environment supports LSP, prefer LSP operations
- Use LSP for: code navigation, symbol lookup, refactoring, diagnostics
- Fallback to text-based operations when LSP unavailable

## File Structure Design

```
distributed-task-orchestrator/
├── SKILL.md                 # Skill main entry
├── workflow.md              # Detailed workflow description
├── templates.md             # Task plan and status table templates
├── cli-integration.md       # Claude CLI integration guide
└── examples.md              # Usage examples
```

## Runtime Files (Generated in User's Project)

```
[user-project]/
├── .agentdocs/
│   ├── index.md                    # Index document (knowledge entry)
│   ├── workflow/                   # Task planning (persistent)
│   │   ├── YYMMDD-task-name.md    # Task analysis, design, TODOs
│   │   └── done/                   # Completed task archive
│   └── runtime/                    # Distributed execution (temporary)
│       ├── YYMMDD-task-name/       # Task-specific runtime (isolated)
│       │   ├── master_plan.md      # Orchestration plan for this task
│       │   ├── agent_tasks/
│       │   │   ├── agent-01.md
│       │   │   └── ...
│       │   ├── results/
│       │   │   ├── agent-01-result.md
│       │   │   └── ...
│       │   └── final_output.md
│       └── YYMMDD-another-task/    # Another concurrent task
│           └── ...
```

## Five-Phase Workflow

### Phase 1: Task Analysis and Planning
- Check existing context from `index.md`
- Parse user intent
- Identify dependencies (DAG graph)
- **Auto-create workflow document** if complex task lacks planning
- Break down into atomic tasks

### Phase 2: Agent Assignment and Status Marking
- Assign Agent IDs
- Create task-specific runtime directory: `runtime/<task-id>/`
- Create status table in `runtime/<task-id>/master_plan.md`
- Generate task file for each Agent

### Phase 3: Parallel Execution
- Option: Sequential execution (in-context)
- Option: Launch subprocesses via CLI
- Record execution logs

### Phase 4: Result Aggregation and Integration
- Collect all Agent results from `runtime/<task-id>/results/`
- Merge according to dependencies
- Generate final output

### Phase 5: Status Sync and Cleanup
- **Update workflow document TODOs** after each task completion
- Move completed workflow docs to `done/`
- Clean up specific task's runtime: `rm -rf runtime/<task-id>/`

## Key Integration Points

### Multi-Task Concurrency
- Each task has isolated runtime directory
- Multiple tasks can execute concurrently without interference
- Task ID format: `YYMMDD-task-name` (matches workflow document)

### Auto-Planning
When user submits complex task without planning:
1. Create workflow document: `.agentdocs/workflow/YYMMDD-task-name.md`
2. Register in `index.md` under "Current Task Documents"
3. Create runtime directory: `.agentdocs/runtime/YYMMDD-task-name/`
4. Proceed with distributed execution

### Status Synchronization
After each atomic task completes:
1. Update `runtime/<task-id>/master_plan.md` status table
2. Mark corresponding TODO in workflow document as done
3. Check if all TODOs complete → archive workflow doc

### Cleanup Strategy
After specific task completion:
1. Move workflow doc to `done/`
2. Remove from "Current Task Documents" in `index.md`
3. Delete only that task's runtime: `rm -rf runtime/<task-id>/`

### Language Adaptation
- Detect user's language from their request
- All documents use the same language as the user
- Applies to: workflow docs, index entries, runtime plans, final output

## Claude CLI Parameter Reference

CLI parameters to use:
- `-p, --prompt` - Pass prompt directly
- `--output-format` - Output format (text/json)
- `--continue`, `-c` - Continue previous conversation
- `--print` - Non-interactive print mode (no `--no-interactive` flag exists)

> **Note**: `--context-file` does not exist in `@anthropic-ai/claude-code`.
> Pass file contents inline: `claude -p "$(cat context.md) \n\n$task"`
