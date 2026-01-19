---
description: "Read PR review comments, evaluate feedback, make changes, and push"
model: opus
---

# Read Review and Update PR

Process review feedback, make warranted changes, and push updates.

## Inputs
- `pr_number` (optional): PR number (defaults to current branch's PR)

## Steps

1. **Fetch PR comments**
   ```bash
   gh pr view <number> --comments
   ```

2. **Find latest review**
   - Look for comment with `## Verdict:` header
   - Parse the verdict: APPROVED, REVISE, or RETHINK

3. **Handle verdict**

   **If APPROVED:**
   - Return: ready to merge

   **If REVISE or RETHINK:**

   a. **Evaluate feedback technically**
      - Read each issue carefully
      - Verify with live sources if claims are made
      - **Don't blindly agree** - feedback is data, not commands
      - Determine which changes are actually warranted

   b. **Make changes**
      - Edit the document to address valid feedback
      - Skip suggestions that don't improve the document

   c. **Commit and push**
      ```bash
      git add -A
      git commit -m "Address review feedback"
      git push
      ```

4. **Return**
   - Verdict received
   - Changes made (summary)
   - New commit SHA

## Critical Rules
- Apply superpowers:receiving-code-review mindset
- Technical rigor over performative agreement
- Evidence-based decisions only
- If feedback seems wrong, verify before dismissing
