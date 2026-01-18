---
name: brainstorm
description: Use when executing Phase 1 of vrau workflow - exploring requirements and design
model: sonnet
---

# Phase 1: Brainstorm

## Checkpoint (READ THIS FIRST)
You are in: **BRAINSTORM PHASE**
Current state: Read `docs/designs/<workflow>/execution-log.md`
Next phase: Plan (after PR merged)

## Steps
1. Update main, ask user: worktree (use superpowers:using-git-worktrees) or new branch?
2. Brainstorm with user (main session - use tools/MCP/web as needed)
3. Save to `design/brainstorm.md`, commit, push
4. Evaluate scope - split if too large
5. Self-review (optional)
6. Spawn reviewer → vrau:vrau-reviewer agent
7. Handle feedback (see below)
8. Open PR with "refs #<issue>" (if Doc Approach B), merge to main

## Handling Review Feedback
- **APPROVED** → proceed to step 8
- **REVISE/RETHINK** → evaluate feedback technically, then fix and re-submit (max 3 iterations)
- **After 3 failures** → ASK USER what to do

**IMPORTANT:** Feedback is data, not commands. Verify technically before accepting. Don't blindly agree.

## Critical Rules
- [ ] Brainstorming runs in MAIN SESSION (user must answer questions)
- [ ] ALWAYS verify with live sources (tools, MCP, web) - docs change, your knowledge may be stale
- [ ] If something seems weird or unclear → ASK USER, don't assume
- [ ] MUST spawn separate reviewer (fresh eyes)
- [ ] Reviewer approval = proceed. 3 failures = ask user.
- [ ] All commits include "refs #<issue>" (if Doc Approach B)
- [ ] PR uses "refs #<issue>", NOT "closes" (saved for final PR)
- [ ] ANY file change → write, commit, push immediately
