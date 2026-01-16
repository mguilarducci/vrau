---
description: "Continue execution for an existing workflow"
---

# Continue Execution

## Find Workflow

Use the `find-workflow` skill to discover and select the target workflow.

## Check State

- If no `plan.md` → "Cannot execute without plan. Run /vrau:plan first."
- If `execution-log.md` exists → show progress and offer to continue
- If `plan.md` exists but no `execution-log.md` → start execution

## Actions

### If execution-log.md exists:

```
Execution Progress:
- Completed: X/Y tasks
- Last task: <task name>

1. Continue execution
   → Resume from where we left off

2. View execution log
   → Display full log

3. View plan
   → Display plan.md

4. Restart execution
   → Start from beginning
```

### If no execution-log.md (but plan exists):

Read [_phases/execute.md](_phases/execute.md) for execution options.

When complete:
1. Write log to `.claude/vrau/workflows/<workflow>/execution-log.md`
2. Invoke `superpowers:finishing-a-development-branch`
