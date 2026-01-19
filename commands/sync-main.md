---
description: "Update current branch from main"
model: haiku
---

# Sync Main

Fetch latest main and merge into current branch.

## Steps

1. **Fetch main**
   ```bash
   git fetch origin main
   ```

2. **Merge main into current branch**
   ```bash
   git merge origin/main
   ```

3. **Handle result**
   - If clean merge: return success
   - If conflicts: report conflicting files and STOP
     - Do NOT attempt to resolve conflicts automatically
     - User must resolve conflicts manually

4. **Return**
   - Success status
   - Number of commits merged (if any)
   - List of conflicting files (if conflicts)

## Notes
- Run this at the start of each phase to ensure you're working with latest code
- Never force push or reset to resolve conflicts
