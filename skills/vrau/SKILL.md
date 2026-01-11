---
name: vrau-workflow
description: |
  Suggests vrau workflow for complex multi-step tasks.
  Use when detecting features, refactors, or tasks that
  would benefit from brainstorming → planning → execution.
---

# Vrau Workflow Suggestion

When you detect a complex multi-step task that would benefit from structured
brainstorming, planning, and execution, suggest using vrau.

## When to Suggest

Suggest vrau for:
- New features with multiple components
- Refactoring efforts spanning multiple files
- Bug fixes requiring investigation and multi-step resolution
- Tasks the user describes with multiple requirements
- Any task that would benefit from brainstorm → plan → execute flow

Do NOT suggest vrau for:
- Simple single-file changes
- Quick fixes or typo corrections
- Questions or research tasks
- Tasks the user explicitly wants done immediately

## How to Suggest

Present the choice naturally:

```
This looks like a multi-step task that could benefit from structured workflow.

Would you like to use vrau to orchestrate this?
→ Brainstorm requirements first
→ Create a detailed plan
→ Execute with tracking and verification

1. Yes, start vrau workflow
2. No, just help me directly
```

If user chooses "Yes", invoke `/vrau:start` with the task description.

If user chooses "No", proceed normally without vrau.

## Key Principle

**Never force vrau.** Always let the user decide. Vrau is for when structure helps,
not for imposing process on simple tasks.
