# RULES.md

Status: Active
Version: 1.2.0
Last Updated: 2026-03-20
Owner: Project Maintainer

## 1) Purpose

This file defines the **default** implementation guardrails for projects using the agentdocs-orchestrator skill.
When process rules conflict across documents, this file takes precedence **unless the target project defines overrides**
in `.agentdocs/local-rules.md`, which takes highest precedence for that project.

## 2) Scope

These rules apply to all implementation tasks, bug fixes, and refactors within the target project.
They do not block read-only analysis tasks.

**Project-level customization**: Create `.agentdocs/local-rules.md` in your project to extend,
restrict, or override any rule here. Local rules take precedence over this file.

## 3) Hard Gates Before Coding

All items below must be satisfied before writing implementation code:

- [ ] Plan is written and includes goal, impacted files, risk analysis, and verification strategy
- [ ] If the agentdocs skill is active for the task, a written Complexity Assessment and chosen mode are recorded before decomposition or direct execution
- [ ] Explicit user approval is received. Approval signals include:
  - English: "approved", "go ahead", "start", "ok", "yes", "proceed", "sounds good", "let's do it"
  - Chinese: "好的", "开始", "开始吧", "可以", "没问题", "同意", "确认", "去做"
- [ ] Verification baseline is defined

If any item is missing: stop at planning and do not implement.

## 4) Segmented Change Rule (>3 Files)

If expected changes touch more than 3 files:

1. Split work into stages
2. Each stage must touch at most 3 files
3. Verify each stage before starting the next stage

Do not run one-pass broad edits across many files.

## 5) TDD Rule for Bug Fixes

Bug-fix tasks must follow Red-Green-Refactor:

1. Red: create a failing repro script/test first
2. Green: implement the minimal fix
3. Refactor: improve code while keeping tests green

No repro evidence means no fix implementation.

## 6) Defensive Programming Deliverables

After implementation, always attach both outputs:

1. Potential Bug Checklist
   - Regression risks
   - Edge cases
   - Error-path gaps
   - Performance or memory side effects

2. Test Case Checklist
   - Happy path
   - Edge case
   - Failure path

For bug fixes, also include:

- Red evidence (before fix)
- Green evidence (after fix)

## 7) Completion Definition

A task is not complete until all items are true:

- [ ] Approval gate passed
- [ ] For bug fixes, Red/Green evidence recorded
- [ ] If >3 files, segmented stages followed
- [ ] Potential Bug Checklist attached
- [ ] Test Case Checklist attached
- [ ] Verification commands and outcomes recorded
- [ ] Completed tasks are checked off in `.agentdocs/workflow/<task-id>.md`
- [ ] Full orchestration only: matching `.agentdocs/runtime/<task-id>/master_plan.md` statuses are synced to final task state
- [ ] Implementation plan reflects actual work, including failed, retried, abandoned, newly discovered, obsolete, blocked, or unplanned tasks
- [ ] No finished task remains marked pending in any plan artifact

## 8) Self-Evolution Protocol

When the user corrects process behavior:

1. Update `.agentdocs/local-rules.md` in the **current project** with the new rule
   (do NOT modify the skill's own RULES.md — that file lives in the skill directory and is not project-editable)
2. Add a new entry in Rule Changelog below
3. Apply the new rule immediately to all subsequent tasks in this project

## 9) Violation Handling

If a mandatory gate is violated:

1. Stop further implementation
2. Report the violation clearly
3. Return to planning and request approval

Do not continue with partial compliance.

## 10) Rule Changelog

| Date | Version | Change | Trigger | Effective |
|------|---------|--------|---------|-----------|
| 2026-02-06 | 1.0.0 | Initial rules baseline | User-defined process constraints | Immediate |
| 2026-02-26 | 1.1.0 | Scope clarification; self-evolution targets local-rules.md; multilingual approval signals | Skill review | Immediate |
| 2026-03-20 | 1.2.0 | Require written complexity scoring before routing; require workflow/master-plan marker sync for completion | User-reported routing + status-sync gaps | Immediate |
| 2026-07-04 | 1.3.0 | Require implementation-plan maintenance as a completion gate for completed, failed, retried, abandoned, added, obsolete, blocked, and unplanned work | User-reported AI completion drift | Immediate |
