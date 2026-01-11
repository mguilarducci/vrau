---
description: "Show current vrau workflow status"
---

# Vrau Status

Read `.claude/vrau/state.local.json`. If not found, display:

```
No active vrau workflow.
Use /vrau:start to begin.
```

Otherwise, display status:

```
┌─────────────────────────────────────────────┐
│ VRAU Status                                 │
├─────────────────────────────────────────────┤
│ State:    <state>                           │
│ Branch:   <branch>                          │
│ Workflow: <workflow>                        │
│ Started:  <startedAt>                       │
│ Workspace: <workspaceType>                  │
│                                             │
│ Files:                                      │
│   <✓|○> brainstorm.md                       │
│   <✓|○> plan.md                             │
│   <✓|○> execution-log.local.md              │
└─────────────────────────────────────────────┘
```

Check for file existence:
- `✓` = file exists
- `○` = file does not exist

If state is `executing`, also show progress from `execution-log.local.md`:
- Count completed tasks
- Show current task if in progress
