---
description: "Clear workflow state (keeps files)"
---

# Vrau Reset

Read `.claude/vrau/state.local.json`.

## Validation

If file not found or state is `idle`:
- Tell user: "No active workflow to reset."
- Stop here.

## Confirmation

Display current state and ask:

```
Current workflow: <workflow>
State: <state>
Branch: <branch>

This will clear the workflow state but keep all files.

1. Confirm reset
2. Cancel
```

## Reset Action

If user confirms:

1. Update `state.local.json`:
   ```json
   {
     "state": "idle"
   }
   ```

2. Display:
   ```
   ✓ Workflow state reset to idle
   ✓ Files preserved in: .claude/vrau/workflows/<workflow>/

   Use /vrau:start to begin a new workflow.
   ```
