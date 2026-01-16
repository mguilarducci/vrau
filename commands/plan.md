---
description: "Continue planning for an existing workflow"
---

# Continue Plan

## Find Workflow

Use the `find-workflow` skill to discover and select the target workflow.

## Check State

- If no `brainstorm.md` → "Cannot plan without brainstorm. Run /vrau:brainstorm first."
- If `plan.md` exists → offer to refine or view
- If `brainstorm.md` exists but no `plan.md` → start fresh plan

## Actions

### If plan.md exists:

```
Plan exists for this workflow.

1. Refine plan
   → Continue with superpowers:writing-plans
   → Update plan.md

2. View current plan
   → Display contents

3. Re-run review
   → Invoke vrau:vrau-reviewer with complexity-calibrated-review
```

### If no plan.md (but brainstorm exists):

1. Invoke `superpowers:writing-plans` skill
2. Reference brainstorm at `.claude/vrau/workflows/<workflow>/brainstorm.md`
3. Save to `.claude/vrau/workflows/<workflow>/plan.md`
4. Run review (read [_phases/plan.md](_phases/plan.md) for review process)
5. When approved, proceed to execution
