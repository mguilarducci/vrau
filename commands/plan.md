---
description: "Continue planning for an existing workflow"
---

# Continue Plan

Use this to continue or refine a plan for an existing workflow.

## Find Workflow

Scan `.claude/vrau/workflows/` for folders.

If no workflows exist:
```
No workflows found. Use /vrau:start to begin.
```

If multiple workflows exist, list them with their state and ask which to continue.

## Check State

Read the workflow folder contents:
- If no `brainstorm.md` → "Cannot plan without brainstorm. Run /vrau:brainstorm first."
- If `plan.md` exists → offer to refine or view
- If `brainstorm.md` exists but no `plan.md` → start fresh plan

## Actions

### If plan.md exists:

```
Plan exists for this workflow.

1. Refine plan
   -> Continue with superpowers:writing-plans
   -> Update plan.md

2. View current plan
   -> Display contents

3. Re-run review
   -> Invoke vrau:vrau-reviewer
```

### If no plan.md (but brainstorm exists):

Invoke `superpowers:writing-plans` skill.

Reference the brainstorm at `.claude/vrau/workflows/<workflow>/brainstorm.md`.

When complete:
1. Save to `.claude/vrau/workflows/<workflow>/plan.md`
2. Invoke `vrau:vrau-reviewer` for review
3. Follow review loop (max 3 iterations, then ask user)
4. When approved, proceed to execution
