---
description: "Merge an approved PR to main"
model: haiku
---

# Merge PR

Merge the current branch's PR to main after approval.

## Inputs
- `pr_number` (optional): PR number (defaults to current branch's PR)
- `delete_branch` (optional): Whether to delete branch after merge (default: true)

## Steps

1. **Get PR number if not provided**
   ```bash
   gh pr view --json number -q .number
   ```

2. **Verify PR status**
   ```bash
   gh pr view <number> --json state,mergeable,reviewDecision
   ```
   - `state` must be `OPEN`
   - `mergeable` must be `MERGEABLE` (not `CONFLICTING` or `UNKNOWN`)
   - If `mergeable` is `CONFLICTING`: STOP - report conflicts, user must resolve

3. **Check CI status**
   ```bash
   gh pr checks <number>
   ```
   - If any checks are failing: STOP - report failing checks
   - If checks are pending: wait or ask user

4. **Merge PR**
   ```bash
   # If delete_branch is true (default):
   gh pr merge <number> --merge --delete-branch

   # If delete_branch is false:
   gh pr merge <number> --merge
   ```
   - Use `--merge` for merge commit (preserves history)

5. **Get merged commit SHA**
   ```bash
   gh pr view <number> --json mergeCommit -q .mergeCommit.oid
   ```

6. **Return**
   - Merge confirmation
   - Merged commit SHA

## Error Handling
- **Merge conflicts:** Report which files conflict, ask user to resolve
- **Failing checks:** List failing checks, ask user to fix
- **Not approved:** Report review status, ask if user wants to proceed anyway
- **Permission denied:** Report error, suggest user check repository permissions
