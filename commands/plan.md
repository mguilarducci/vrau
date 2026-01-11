---
description: "Continue or refine the current plan"
---

# Vrau Plan

Read `.claude/vrau/state.local.json`.

## Validation

If state is not `planning` or `plan_review`:
- Tell user: "Cannot plan in current state: <state>. Use `/vrau:status` to see details."
- Stop here.

## Actions

### If state is `planning`:

Continue with `superpowers:writing-plans` skill.

Reference the brainstorm document at:
- `.claude/vrau/workflows/<workflow>/brainstorm.md`

When plan is complete, save to workflow folder:
- `.claude/vrau/workflows/<workflow>/plan.md`

Update state to `plan_review`.

### If state is `plan_review`:

Ask user:

```
Plan is complete. What would you like to do?

1. Refine plan
   → Return to planning phase

2. View current plan
   → Display the plan.md contents

3. Proceed to review
   → Present review options
```

Based on choice:
- **Option 1**: Update state to `planning`, continue with `superpowers:writing-plans`
- **Option 2**: Read and display `.claude/vrau/workflows/<workflow>/plan.md`
- **Option 3**: Present review options (checklist / subagent / both / skip)
