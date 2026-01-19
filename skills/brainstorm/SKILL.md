---
name: brainstorm
description: Use when executing Phase 1 of vrau workflow - exploring requirements and design
model: sonnet
---

# Phase 1: Brainstorm

## Checkpoint (READ THIS FIRST)
You are in: **BRAINSTORM PHASE**
Next phase: **Plan** (after PR merged) → NOT Execute

**After brainstorm approval, invoke vrau:plan. NEVER skip to vrau:execute.**

## Red Flags - STOP If You're Thinking This

| Thought | Reality |
|---------|---------|
| "This brainstorm is detailed enough to implement directly" | Brainstorm ≠ Plan. Plan phase creates actionable tasks. |
| "I can skip planning since I know what to do" | Plans get reviewed separately. Different concerns. |
| "After approval I can start coding" | After approval → Plan phase. Not coding. |
| "The brainstorm has specific code changes" | Still needs Plan phase to organize execution order. |
| "I can review it myself" | NO. You wrote it, you CANNOT review it. Spawn a separate agent. |
| "I'll just check it quickly before spawning reviewer" | That's self-review. Spawn reviewer INSTEAD of reviewing yourself. |
| "I understand it best since I wrote it" | That's WHY you can't review it. Fresh eyes catch what you missed. |

**If any thought above occurs: STOP. The workflow is Brainstorm → Plan → Execute. No shortcuts.**

## CRITICAL SAFETY RULE

**NEVER COMMIT TO MAIN BRANCH**

Check current branch:
```bash
git branch --show-current
```

**If on main/master:** STOP. Create a new branch or use worktree. NEVER proceed with commits on main.

## Steps
1. Run /sync-main - ensure branch is up to date
2. If Tracking Mode: GitHub → run /track-task "Started brainstorm phase"
3. **Invoke superpowers:brainstorming skill** - ask questions one at a time, use multiple choice when possible, verify with tools/MCP/web
4. Evaluate scope - split if too large
5. Save to `design/brainstorm.md`, commit, push
6. If Tracking Mode: GitHub → run /track-task "Brainstorm document saved to design/brainstorm.md"
7. Run /commit-push-pr - title: "[Review] Brainstorm: <task>"
8. If Tracking Mode: GitHub → run /track-task "PR #X created for brainstorm review"
9. **Spawn SEPARATE reviewer** (you CANNOT review your own work):
   - Use Task tool with `vrau:vrau-reviewer` subagent
   - Reviewer runs /review-comment on PR
   - You wrote it = you cannot review it. No exceptions.
10. If Tracking Mode: GitHub → run /track-task "Reviewer verdict: <VERDICT>"
11. Handle feedback:
    - APPROVED → run /merge-pr, then /track-task "Brainstorm PR merged"
    - REVISE/RETHINK → run /read-review-update-pr, loop to step 9 (max 3 iterations)
    - After 3 failures → ASK USER what to do

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
