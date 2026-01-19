---
description: "Verify GitHub issue exists or create one, then link to workflow"
model: haiku
---

# Start Job

Link a vrau workflow to a GitHub issue for tracking.

## Steps

1. **Check for issue number**
   - If issue number provided: verify exists with `gh issue view <number>`
   - If no issue number: create one with `gh issue create --title "<task title>" --body "<task description>"`

2. **Update workflow README.md**
   - Add/update the `Issue:` line with the issue number
   - Format: `**Issue:** #<number>`

3. **Update execution-log.md**
   - Set `Issue:` field to `#<number>`

4. **Return**
   - Issue number
   - Issue URL (from `gh issue view <number> --json url -q .url`)
