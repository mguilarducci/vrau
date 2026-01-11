---
description: "Abandon workflow completely"
---

# Vrau Abort

Read `.claude/vrau/state.local.json`.

## Validation

If file not found or state is `idle`:
- Tell user: "No active workflow to abort."
- Stop here.

## Confirmation

Display current state and ask:

```
⚠ Abort Workflow

Current workflow: <workflow>
State: <state>
Branch: <branch>

What do you want to do with workflow files?

1. Keep files
   → Files stay in .claude/vrau/workflows/<workflow>/

2. Archive files
   → Move to .claude/vrau/archive/<workflow>/

3. Delete files
   → Remove workflow folder entirely

4. Cancel
   → Return without aborting
```

## Abort Action

Based on choice (1, 2, or 3):

1. Perform file action (keep/archive/delete)

2. Reset `state.local.json`:
   ```json
   {
     "state": "idle"
   }
   ```

3. Display:
   ```
   ✓ Workflow aborted
   ✓ Files: <kept|archived|deleted>
   ✓ State reset to idle

   Use /vrau:start to begin a new workflow.
   ```
