---
description: "Comment on linked GitHub issue with execution status"
model: haiku
---

# Track Task

Post a status update comment to the linked GitHub issue.

## Inputs
- `phase`: Current phase (brainstorm | plan | execute)
- `status`: Brief status message
- `issue_number` (optional): Issue number (reads from README.md if not provided)

## Steps

1. **Get issue number if not provided**
   - Read from workflow `README.md` (look for `**Issue:** #<number>`)
   - If no issue linked: skip and return "No issue linked - Tracking Mode: None"

2. **Format status comment**
   ```markdown
   ## Status Update

   **Phase:** <phase>
   **Status:** <status>
   **Timestamp:** <ISO timestamp>
   **Branch:** <current branch name>
   ```

3. **Post comment**
   ```bash
   gh issue comment <number> --body "$(cat <<'EOF'
   <formatted comment>
   EOF
   )"
   ```

4. **Return**
   - Comment URL (from `gh issue view <number> --json comments --jq '.comments[-1].url'`)

## When to Use
- After completing brainstorm phase (before plan)
- After completing plan phase (before execute)
- During execution at major checkpoints
- After final PR merged (workflow complete)
