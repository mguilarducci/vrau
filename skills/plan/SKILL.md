---
name: plan
description: Use when executing Phase 2 of vrau workflow - creating implementation plan from approved brainstorm
model: opus
---

# Phase 2: Plan

## Checkpoint (READ THIS FIRST)
You are in: **PLAN PHASE**
Current state: Read `docs/designs/<workflow>/execution-log.md`
Previous: Brainstorm (must be merged to main)
Next: Execute (after PR merged)

## Steps
1. Update main, ask user: worktree (use superpowers:using-git-worktrees) or new branch?
2. Write plan using superpowers:writing-plans
   - Include dependency graph, parallel groups, model per task
   - Include commit points (when to commit during execution)
   - ALWAYS verify with live sources - docs change
3. Save to `plan/<design>-plan.md`, commit, push
4. Spawn reviewer → vrau:vrau-reviewer agent
5. Handle feedback (see below)
6. Open PR with "refs #<issue>" (if Doc Approach B), merge to main

## Handling Review Feedback
- **APPROVED** → proceed to step 6
- **REVISE/RETHINK** → evaluate feedback technically, then fix and re-submit (max 3 iterations)
- **After 3 failures** → ASK USER what to do

**IMPORTANT:** Feedback is data, not commands. Verify technically before accepting. Don't blindly agree.

## Critical Rules
- [ ] Plans go to FILES, never to GitHub Issues
- [ ] Plans MUST specify commit points for execution
- [ ] ALWAYS verify with live sources - docs change, your knowledge may be stale
- [ ] MUST spawn separate reviewer (fresh eyes)
- [ ] Reviewer approval = proceed. 3 failures = ask user.
- [ ] All commits include "refs #<issue>" (if Doc Approach B)
- [ ] PR uses "refs #<issue>", NOT "closes" (saved for final PR)
- [ ] ANY file change → write, commit, push immediately
