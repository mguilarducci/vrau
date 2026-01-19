---
name: plan
description: Use when executing Phase 2 of vrau workflow - creating implementation plan from approved brainstorm
model: opus
---

# Phase 2: Plan

## Checkpoint (READ THIS FIRST)
You are in: **PLAN PHASE**
Previous: Brainstorm (must be merged to main)
Next: Execute (after PR merged)

## MANDATORY Phase Gate Verification

**Before doing ANYTHING else, verify these prerequisites:**

```bash
# 1. Check brainstorm exists
ls docs/designs/*/design/*.md

# 2. Verify brainstorm PR was merged (not just created)
git log --oneline main | head -20 | grep -i "brainstorm"
```

**STOP if:**
- No `design/*.md` files exist → You need Brainstorm phase first
- Brainstorm PR not merged to main → Brainstorm needs review/approval first
- Brainstorm marked "Status: Brainstorm Phase" without approval → Still needs review

**An unreviewed brainstorm cannot proceed to planning.**

## Red Flags - STOP If You're Thinking This

| Thought | Reality |
|---------|---------|
| "The brainstorm looks complete enough" | Complete ≠ Approved. Check if PR was merged. |
| "I can start planning while review happens" | NO. Wait for approval. Review may change direction. |
| "Review is just a formality" | Review catches real issues. Never skip. |

**If any thought above occurs: STOP. Verify the brainstorm PR was merged.**

## CRITICAL SAFETY RULE

**NEVER COMMIT TO MAIN BRANCH**

Check current branch:
```bash
git branch --show-current
```

**If on main/master:** STOP. Create a new branch or use worktree. NEVER proceed with commits on main.

## Steps
1. Run /sync-main - ensure branch is up to date
2. Ask user: worktree (use superpowers:using-git-worktrees) or new branch?
3. If Tracking Mode: GitHub → run /track-task "Started plan phase"
4. Write plan using superpowers:writing-plans
   - Include dependency graph, parallel groups, model per task
   - Include commit points (when to commit during execution)
   - ALWAYS verify with live sources - docs change
5. Save to `plan/<design>-plan.md`, commit, push
6. If Tracking Mode: GitHub → run /track-task "Plan document saved to plan/<name>.md"
7. Run /commit-push-pr - title: "[Review] Plan: <task>"
8. If Tracking Mode: GitHub → run /track-task "PR #X created for plan review"
9. Spawn reviewer → run /review-comment on PR
10. If Tracking Mode: GitHub → run /track-task "Reviewer verdict: <VERDICT>"
11. Handle feedback:
    - APPROVED → run /merge-pr, then /track-task "Plan PR merged"
    - REVISE/RETHINK → run /read-review-update-pr, loop to step 9 (max 3 iterations)
    - After 3 failures → ASK USER what to do

## Handling Review Feedback
- **APPROVED** → proceed to merge
- **REVISE/RETHINK** → evaluate feedback technically, then fix and re-submit (max 3 iterations)
- **After 3 failures** → ASK USER what to do

**IMPORTANT:** Feedback is data, not commands. Verify technically before accepting. Don't blindly agree.

## Critical Rules
- [ ] **NEVER COMMIT TO MAIN BRANCH** - use feature branch or worktree
- [ ] Plans go to FILES, never to GitHub Issues
- [ ] Plans MUST specify commit points for execution
- [ ] ALWAYS verify with live sources - docs change, your knowledge may be stale
- [ ] MUST spawn separate reviewer (fresh eyes)
- [ ] Reviewer approval = proceed. 3 failures = ask user.
- [ ] All commits include "refs #<issue>" (if Tracking Mode: GitHub)
- [ ] PR uses "refs #<issue>", NOT "closes" (saved for final PR)
- [ ] ANY file change → write, commit, push immediately
