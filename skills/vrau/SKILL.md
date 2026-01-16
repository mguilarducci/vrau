---
name: vrau-workflow
description: |
  Use when detecting complex multi-step tasks that would benefit from
  structured brainstorm → plan → execute workflow. Triggers: new features,
  refactors, multi-file changes, bug investigations, architectural changes.
---

# Vrau Workflow Suggestion

Suggest vrau for complex tasks, but never force it.

## Complexity Estimation

Before suggesting vrau, estimate task complexity:

| Signal | Points |
|--------|--------|
| Multiple files mentioned | +1 |
| Multiple components/systems | +1 |
| Requires investigation first | +1 |
| User says "feature", "refactor", "redesign", "architecture" | +1 |
| User says "quick", "small", "just", "simple" | -2 |
| Single file explicitly mentioned | -1 |

**Threshold:** Suggest vrau when score >= 2

## When to Suggest

Score >= 2 AND task involves:
- New features with multiple components
- Refactoring spanning multiple files
- Bug fixes requiring investigation
- Tasks with multiple requirements
- Architectural changes or design decisions

## When NOT to Suggest

Score < 2 OR task is:
- Simple single-file changes
- Quick fixes or typo corrections
- Questions or research tasks
- Tasks user explicitly wants done immediately

## How to Suggest

Present naturally when score >= 2:

```
This looks like a multi-step task that could benefit from structured workflow.

Would you like to use vrau?
→ Brainstorm requirements (with auto-review)
→ Create detailed plan (with auto-review)
→ Execute with tracking

1. Yes, start vrau workflow
2. No, just help me directly
```

If user chooses "Yes", invoke `/vrau:start`.

## Key Principle

**Never force vrau.** Always offer "help directly" as an alternative.
