---
name: execute
description: Use when executing Phase 3 of vrau workflow - implementing the approved plan
model: sonnet
---

# Phase 3: Execute

## Checkpoint (READ THIS FIRST)
You are in: **EXECUTE PHASE**
Current state: Read `docs/designs/<workflow>/execution-log.md`
Previous: Plan (must be merged to main)
Next: Done (PR merged, issue closed if applicable)

## Steps
1. Update main, ask user: worktree (use superpowers:using-git-worktrees) or new branch?
2. Read plan, execute tasks by parallel groups
   - Dispatch parallel agents for independent tasks
   - ALWAYS verify with live sources - docs change
3. After EACH parallel group: use superpowers:requesting-code-review (fresh eyes)
   - APPROVED → continue to next group
   - Issues found → use superpowers:receiving-code-review, then request NEW fresh review
4. Run verification (superpowers:verification-before-completion)
5. Delete execution log file
6. Use superpowers:finishing-a-development-branch
   - If workflow started from GitHub issue: include "Closes #<issue>" in PR
   - Otherwise: standard PR

## Critical Rules
- [ ] ALWAYS verify with live sources - docs change, your knowledge may be stale
- [ ] Code review after EVERY parallel group (fresh eyes each time)
- [ ] MUST run verification before PR - never skip
- [ ] Delete execution log BEFORE creating PR (part of PR changes)
- [ ] All commits include "refs #<issue>" (if Doc Approach B)
- [ ] Only final PR uses "closes #<issue>" (if Doc Approach B)
- [ ] Execution log updates → write, commit, push immediately
- [ ] Code changes → commit as specified in the plan
