# Phase 2: Plan

## Invoke Planning

Use `superpowers:writing-plans` skill.

Provide context:
- Reference brainstorm at `.claude/vrau/workflows/<workflow>/brainstorm.md`
- Task complexity (same as brainstorm phase)

Let the skill drive until it produces a plan.

**Completion signal:** Plan document ready.

## Save Output

Write plan to:
```
.claude/vrau/workflows/<workflow>/plan.md
```

## Review

Spawn reviewer with same complexity level:

```
Task tool:
- subagent_type: "vrau:vrau-reviewer"
- prompt:
  1. Task description (one line)
  2. Complexity: <level>
  3. plan.md content
  4. Request: "Review this plan for completeness and feasibility"
```

**REQUIRED:** Reviewer uses complexity-calibrated-review pattern.

## Review Loop

Same as brainstorm: max 3 iterations, then ask user.

When approved → proceed to Phase 3 (Execute)
