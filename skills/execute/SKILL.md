---
name: execute
description: Use when executing Phase 3 of vrau workflow - implementing the approved plan
model: sonnet
---

# Phase 3: Execute

## Checkpoint (READ THIS FIRST)
You are in: **EXECUTE PHASE**
Previous: Plan (must be merged to main)
Next: Done (PR merged, issue closed if applicable)

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
3. If Tracking Mode: GitHub → run /track-task "Started execute phase"
4. Read plan, execute tasks by parallel groups
   - Dispatch parallel agents for independent tasks
   - ALWAYS verify with live sources - docs change
   - After each group: /track-task "Completed task group X (tasks N, M, ...)"
5. After EACH parallel group: use superpowers:requesting-code-review (fresh eyes)
   - If Tracking Mode: GitHub → run /track-task "Code review for group X: <VERDICT>"
   - APPROVED → continue to next group
   - Issues found → use superpowers:receiving-code-review, then request NEW fresh review
6. If Tracking Mode: GitHub → run /track-task "All tasks complete, running verification"
7. Run verification (superpowers:verification-before-completion)
8. Use superpowers:finishing-a-development-branch
   - If Tracking Mode: GitHub → include "Closes #<issue>" in final PR
   - If Tracking Mode: GitHub → run /track-task "Final PR #X created (closes #<issue>)"
   - Otherwise: standard PR

## Critical Rules
- [ ] **NEVER COMMIT TO MAIN BRANCH** - use feature branch or worktree
- [ ] ALWAYS verify with live sources - docs change, your knowledge may be stale
- [ ] Code review after EVERY parallel group (fresh eyes each time)
- [ ] MUST run verification before PR - never skip
- [ ] All commits include "refs #<issue>" (if Tracking Mode: GitHub)
- [ ] Only final PR uses "closes #<issue>" (if Tracking Mode: GitHub)
- [ ] Code changes → commit as specified in the plan
