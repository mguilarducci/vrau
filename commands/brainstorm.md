---
description: "Continue brainstorming for an existing workflow"
---

# Continue Brainstorm

Use this to continue or refine a brainstorm for an existing workflow.

## Find Workflow

Scan `.claude/vrau/workflows/` for folders.

If no workflows exist:
```
No workflows found. Use /vrau:start to begin.
```

If multiple workflows exist, list them and ask which to continue:
```
Found workflows:
1. 2026-01-10-add-auth (has: brainstorm.md, plan.md)
2. 2026-01-12-fix-api (has: brainstorm.md)
3. 2026-01-13-refactor-db (empty)

Which workflow?
```

## Check State

Read the workflow folder contents:
- If `brainstorm.md` exists → offer to refine or view
- If no `brainstorm.md` → start fresh brainstorm

## Actions

### If brainstorm.md exists:

```
Brainstorm exists for this workflow.

1. Refine brainstorm
   -> Continue with superpowers:brainstorming
   -> Update brainstorm.md

2. View current brainstorm
   -> Display contents

3. Re-run review
   -> Invoke vrau:vrau-reviewer
```

### If no brainstorm.md:

Invoke `superpowers:brainstorming` skill.

When complete:
1. Save to `.claude/vrau/workflows/<workflow>/brainstorm.md`
2. Invoke `vrau:vrau-reviewer` for review
3. Follow review loop (max 3 iterations, then ask user)
4. When approved, proceed to planning (invoke `superpowers:writing-plans`)
