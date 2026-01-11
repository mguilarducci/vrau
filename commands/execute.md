---
description: "Continue execution of the current plan"
---

# Vrau Execute

Read `.claude/vrau/state.local.json`.

## Validation

If state is not `executing`:
- Tell user: "Cannot execute in current state: <state>. Use `/vrau:status` to see details."
- Stop here.

## Show Progress

Read `.claude/vrau/workflows/<workflow>/execution-log.local.md` if it exists.

Display current progress:

```
Execution Progress:
- Completed: X/Y tasks
- Current: <task name or "None">
- Errors: <count or "None">
```

## Continue Execution

Ask user:

```
How would you like to proceed?

1. Continue execution
   → Resume from current task

2. View execution log
   → Show detailed progress

3. View plan
   → Show the plan being executed

4. Abort execution
   → Stop and return to idle
```

Based on choice:
- **Option 1**: Continue with the execution method originally chosen (subagent or parallel)
- **Option 2**: Display `execution-log.local.md`
- **Option 3**: Display `plan.md`
- **Option 4**: Update state to `idle`, offer cleanup options

## Execution Completion

When all tasks complete:
1. Update state to `done`
2. Invoke `superpowers:finishing-a-development-branch`
3. Present cleanup options
