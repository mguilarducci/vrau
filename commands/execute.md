---
description: "Continue execution for an existing workflow"
---

# Continue Execution

Use this to continue or start execution for an existing workflow.

## Find Workflow

Scan `.claude/vrau/workflows/` for folders.

If no workflows exist:
```
No workflows found. Use /vrau:start to begin.
```

If multiple workflows exist, list them with their state and ask which to continue.

## Check State

Read the workflow folder contents:
- If no `plan.md` → "Cannot execute without plan. Run /vrau:plan first."
- If `execution-log.md` exists → show progress and offer to continue
- If `plan.md` exists but no `execution-log.md` → start execution

## Actions

### If execution-log.md exists:

Read the log and show progress:
```
Execution Progress:
- Completed: X/Y tasks
- Last task: <task name>

1. Continue execution
   -> Resume from where we left off

2. View execution log
   -> Display full log

3. View plan
   -> Display plan.md

4. Restart execution
   -> Start from beginning
```

### If no execution-log.md (but plan exists):

```
Plan approved! Ready to execute.

1. Subagent-driven (this session)
   -> Uses superpowers:subagent-driven-development
   -> Fresh subagent per task

2. Parallel session (separate terminal)
   -> Uses superpowers:executing-plans
   -> Open new terminal
```

No auto-review for execution.

When execution completes:
1. Write execution log to `.claude/vrau/workflows/<workflow>/execution-log.md`
2. Invoke `superpowers:finishing-a-development-branch`
