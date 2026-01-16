---
description: "Continue brainstorming for an existing workflow"
---

# Continue Brainstorm

## Find Workflow

Use the `find-workflow` skill to discover and select the target workflow.

## Check State

- If `brainstorm.md` exists → offer to refine or view
- If no `brainstorm.md` → start fresh brainstorm

## Actions

### If brainstorm.md exists:

```
Brainstorm exists for this workflow.

1. Refine brainstorm
   → Continue with superpowers:brainstorming
   → Update brainstorm.md

2. View current brainstorm
   → Display contents

3. Re-run review
   → Invoke vrau:vrau-reviewer with complexity-calibrated-review
```

### If no brainstorm.md:

1. Invoke `superpowers:brainstorming` skill
2. Save to `.claude/vrau/workflows/<workflow>/brainstorm.md`
3. Run review (read [_phases/brainstorm.md](_phases/brainstorm.md) for review process)
4. When approved, proceed to planning
