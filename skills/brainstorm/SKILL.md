---
name: brainstorm
description: Use when executing Phase 1 of vrau workflow - exploring requirements and design
model: sonnet
---

# Phase 1: Brainstorm

## Checkpoint (READ THIS FIRST)
You are in: **BRAINSTORM PHASE**
Next phase: Plan (after PR merged)

## CRITICAL SAFETY RULE

**NEVER COMMIT TO MAIN BRANCH**

Check current branch:
```bash
git branch --show-current
```

**If on main/master:** STOP. Create a new branch or use worktree. NEVER proceed with commits on main.

## Steps
1. Run /sync-main - ensure branch is up to date
2. **Invoke superpowers:brainstorming skill** - ask questions one at a time, use multiple choice when possible, verify with tools/MCP/web
3. Evaluate scope - split if too large
4. Save to `design/brainstorm.md`, commit, push
5. Run /commit-push-pr - title: "[Review] Brainstorm: <task>"
6. Spawn reviewer → run /review-comment on PR
7. Handle feedback:
   - APPROVED → run /merge-pr, proceed to step 8
   - REVISE/RETHINK → run /read-review-update-pr, loop to step 6 (max 3 iterations)
   - After 3 failures → ASK USER what to do
8. If Tracking Mode: GitHub → run /track-task

**Note:** Branch/worktree setup already done by vrau entry point during workflow setup

## Handling Review Feedback
- **APPROVED** → proceed to merge
- **REVISE/RETHINK** → evaluate feedback technically, then fix and re-submit (max 3 iterations)
- **After 3 failures** → ASK USER what to do

**IMPORTANT:** Feedback is data, not commands. Verify technically before accepting. Don't blindly agree.

## Critical Rules
- [ ] **NEVER COMMIT TO MAIN BRANCH** - use feature branch or worktree
- [ ] **MUST invoke superpowers:brainstorming skill** - ask questions ONE AT A TIME, don't dump all questions at once
- [ ] Brainstorming runs in MAIN SESSION (user must answer questions)
- [ ] ALWAYS verify with live sources (tools, MCP, web) - docs change, your knowledge may be stale
- [ ] If something seems weird or unclear → ASK USER, don't assume
- [ ] MUST spawn separate reviewer (fresh eyes)
- [ ] Reviewer approval = proceed. 3 failures = ask user.
- [ ] All commits include "refs #<issue>" (if Tracking Mode: GitHub)
- [ ] PR uses "refs #<issue>", NOT "closes" (saved for final PR)
- [ ] ANY file change → write, commit, push immediately
