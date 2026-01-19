---
description: "Commit changes, push, and create or update PR atomically"
model: haiku
---

# Commit, Push, PR

Stage all changes, commit, push, and create/update a PR in one atomic operation.

## Inputs
- `message`: Commit message
- `pr_title`: PR title
- `pr_body`: PR body (markdown)
- `issue_number` (optional): GitHub issue to reference

## Steps

1. **Stage all changes**
   ```bash
   git add -A
   ```

2. **Commit**
   - If `issue_number` provided: append `refs #<issue_number>` to message
   ```bash
   git commit -m "<message>"
   ```

3. **Push**
   ```bash
   git push -u origin $(git branch --show-current)
   ```

4. **Check for existing PR**
   ```bash
   gh pr view --json url 2>/dev/null
   ```

5. **Create or report PR**
   - If no PR exists:
     ```bash
     gh pr create --title "<pr_title>" --body "<pr_body>"
     ```
   - If PR exists: report existing PR URL

6. **Return**
   - PR URL
   - Commit SHA (from `git rev-parse HEAD`)
