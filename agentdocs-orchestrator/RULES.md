# RULES.md

Status: Active  
Version: 1.0.0  
Last Updated: 2026-02-06  
Owner: Project Maintainer

## 1) Purpose

This file is the single source of truth for implementation behavior in this repository.
When process rules conflict across documents, this file takes precedence.

## 2) Scope

These rules apply to all implementation tasks, bug fixes, and refactors.
They do not block read-only analysis tasks.

## 3) Hard Gates Before Coding

All items below must be satisfied before writing implementation code:

- [ ] Plan is written and includes goal, impacted files, risk analysis, and verification strategy
- [ ] Explicit user approval is received (for example: "approved", "go ahead", "start")
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

## 8) Self-Evolution Protocol

When the user corrects process behavior:

1. Update this RULES.md
2. Add a new entry in Rule Changelog
3. Apply the new rule immediately to all subsequent tasks

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
