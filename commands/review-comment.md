---
description: "Post structured review feedback as a PR comment"
model: sonnet
---

# Review and Comment

Analyze a PR and post structured review feedback as a comment.

## Inputs
- `pr_number` (optional): PR number to review (defaults to current branch's PR)

## Steps

1. **Get PR number if not provided**
   ```bash
   gh pr view --json number -q .number
   ```

2. **Get PR details**
   ```bash
   gh pr view <number> --json title,body,files,headRefName
   ```

3. **Determine document type from files changed**
   - If `files` includes `design/brainstorm.md` → brainstorm review
   - If `files` includes files in `plan/` → plan review
   - If both → review both, note in summary
   - If neither → ask what to review

4. **Read the document being reviewed**
   - For brainstorm reviews: read `design/brainstorm.md`
   - For plan reviews: read `plan/*.md`

5. **Apply review analysis**
   - **For Brainstorms:** Clarity, completeness, feasibility, risks identified?
   - **For Plans:** Dependencies correct? Parallel groups make sense? Commit points specified? Tasks actionable?
   - Verify claims against codebase (read relevant files, check APIs exist)

6. **Format review**
   ```markdown
   ## Verdict: APPROVED | REVISE | RETHINK

   ## Summary
   [1-2 sentences]

   ## Critical Issues (blockers)
   - [Must fix before proceeding]

   ## Important Issues (should fix)
   - [Significant but not blocking]

   ## Suggestions (nice-to-have)
   - [Optional improvements]
   ```

7. **Post comment using heredoc for proper formatting**
   ```bash
   gh pr comment <number> --body "$(cat <<'EOF'
   <formatted review content>
   EOF
   )"
   ```

8. **Get comment URL**
   ```bash
   gh pr view <number> --json comments --jq '.comments[-1].url'
   ```

9. **Return**
   - Verdict (APPROVED | REVISE | RETHINK)
   - Comment URL

## Review Guidelines
- Be constructive, not pedantic
- REVISE = minor fixes needed
- RETHINK = fundamental problems
- APPROVED = good to proceed
- Match review depth to task complexity
