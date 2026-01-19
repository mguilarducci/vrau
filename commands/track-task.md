---
description: "Comment on linked GitHub issue with a single operation log"
model: haiku
---

# Track Task

Post a single operation log to the linked GitHub issue. Each call = one comment about what just happened.

## Inputs
- `operation`: What just happened (e.g., "Started brainstorm phase", "PR #12 created for review", "Reviewer approved brainstorm")
- `issue_number` (optional): Issue number (reads from README.md if not provided)

## Steps

1. **Get issue number if not provided**
   - Read from workflow `README.md` (look for `**Issue:** #<number>`)
   - If no issue linked: skip and return "No issue linked - Tracking Mode: None"

2. **Format single-line comment**
   ```markdown
   **[<timestamp>]** <operation>
   ```
   Example: `**[2026-01-19T10:30:00Z]** Started brainstorm phase`

3. **Post comment**
   ```bash
   gh issue comment <number> --body "**[<timestamp>]** <operation>"
   ```

4. **Return**
   - Comment URL

## When to Invoke (Tracking Mode: GitHub)

**Brainstorm phase:**
- Started brainstorm phase
- Brainstorm document saved to design/brainstorm.md
- PR #X created for brainstorm review
- Reviewer verdict: APPROVED/REVISE/RETHINK
- Brainstorm PR merged

**Plan phase:**
- Started plan phase
- Plan document saved to plan/X.md
- PR #X created for plan review
- Reviewer verdict: APPROVED/REVISE/RETHINK
- Plan PR merged

**Execute phase:**
- Started execute phase
- Completed task group A (tasks 1, 2)
- Code review: APPROVED/issues found
- Completed task group B (tasks 3, 4, 5)
- All tasks complete, running verification
- Final PR #X created (closes this issue)
