# Phase 1: Brainstorm

## Invoke Brainstorming

Use `superpowers:brainstorming` skill with the task description.

Let the skill drive the conversation - it asks clarifying questions until producing a final brainstorm document.

**Completion signal:** Summary or "brainstorm complete" message.

## Save Output

Write brainstorm to:
```
.claude/vrau/workflows/<workflow>/brainstorm.md
```

## Review

Determine task complexity (trivial/simple/moderate/complex), then spawn reviewer:

```
Task tool:
- subagent_type: "vrau:vrau-reviewer"
- prompt:
  1. Task description (one line)
  2. Complexity: <level>
  3. brainstorm.md content
  4. Request: "Review this brainstorm"
```

**REQUIRED:** Reviewer uses complexity-calibrated-review pattern.

## Review Loop

1. Reviewer APPROVEs → proceed to Phase 2 (Plan)
2. Reviewer has issues → refine brainstorm, re-invoke reviewer
3. Max 3 iterations
4. After 3 without approval → ask user how to proceed
