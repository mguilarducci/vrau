---
description: "Continue or refine the current brainstorm"
---

# Vrau Brainstorm

Read `.claude/vrau/state.local.json`.

## Validation

If state is not `brainstorming` or `brainstorm_review`:
- Tell user: "Cannot brainstorm in current state: <state>. Use `/vrau:status` to see details."
- Stop here.

## Actions

### If state is `brainstorming`:

Continue with `superpowers:brainstorming` skill.

If the brainstorm is complete, save to workflow folder:
- `.claude/vrau/workflows/<workflow>/brainstorm.md`

Update state to `brainstorm_review`.

### If state is `brainstorm_review`:

Ask user:

```
Brainstorm is complete. What would you like to do?

1. Refine brainstorm
   → Return to brainstorming phase

2. View current brainstorm
   → Display the brainstorm.md contents

3. Proceed to review
   → Present review options
```

Based on choice:
- **Option 1**: Update state to `brainstorming`, continue with `superpowers:brainstorming`
- **Option 2**: Read and display `.claude/vrau/workflows/<workflow>/brainstorm.md`
- **Option 3**: Present review options (checklist / subagent / both / skip)
