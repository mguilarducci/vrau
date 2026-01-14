---
name: vrau-workflow
description: |
  Suggests vrau workflow for complex multi-step tasks.
  Use when detecting features, refactors, or tasks that
  would benefit from brainstorming → planning → execution.
---

# Vrau Workflow Suggestion

When you detect a complex multi-step task, suggest using vrau.

## When to Suggest

Suggest vrau for:
- New features with multiple components
- Refactoring efforts spanning multiple files
- Bug fixes requiring investigation and multi-step resolution
- Tasks the user describes with multiple requirements

Do NOT suggest vrau for:
- Simple single-file changes
- Quick fixes or typo corrections
- Questions or research tasks
- Tasks the user explicitly wants done immediately

## How to Suggest

Present the choice naturally:

```
This looks like a multi-step task that could benefit from structured workflow.

Would you like to use vrau?
-> Brainstorm requirements (with auto-review)
-> Create a detailed plan (with auto-review)
-> Execute with tracking

1. Yes, start vrau workflow
2. No, just help me directly
```

If user chooses "Yes", invoke `/vrau:start` with the task description.

## Vrau Flow

```
/vrau:start
  -> Ask task, branch, workspace setup
  -> Brainstorm (asks questions, then writes brainstorm.md)
  -> Auto-review (max 3 iterations)
  -> Plan (asks questions, then writes plan.md)
  -> Auto-review (max 3 iterations)
  -> Execute
```

State is derived from workflow files - no state file needed.
Each workflow lives in `.claude/vrau/workflows/<date>-<branch>/`.

## Key Principle

**Never force vrau.** Always let the user decide.
