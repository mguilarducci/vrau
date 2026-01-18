# Refactor Vrau Plugin for Token Efficiency

## Problem Statement

The vrau plugin has token efficiency and navigation issues:
- Claude gets lost early in workflows (not just context buildup)
- Phase transitions are problematic
- Skips steps within phases
- Confuses instructions across 15 different skills

## Solution: Radical Consolidation

Reduce from 15 skills (2,435 lines) to 5 skills (~700 lines).

### New Structure

| New Skill | Lines | Purpose |
|-----------|-------|---------|
| vrau/ | ~100 | Entry point + routing |
| brainstorm/ | ~150 | All of Phase 1 |
| plan/ | ~120 | All of Phase 2 |
| execute/ | ~150 | All of Phase 3 |
| vrau-reviewer/ | ~80 | Fresh-eyes review agent |

**Principle:** One skill per phase. Enter a phase skill, stay in it until phase ends.

---

## Skill Designs

### 1. vrau (Entry Point)

```markdown
---
name: vrau
description: Use when starting or resuming a vrau workflow - routes to correct phase
---

# Vrau Workflow

## On Start
1. Scan `docs/designs/` for folders matching `YYYY-MM-DD-*`
2. If none: start new workflow (see below)
3. If one: auto-select, detect state, invoke correct phase
4. If multiple: ask user which to resume (or start new, or delete old)

## New Workflow Setup
1. Ask user for task description
2. Ask: Start from GitHub issue?
   - Yes → get issue number, set Doc Approach B
   - No → set Doc Approach A
3. Create folder `docs/designs/YYYY-MM-DD-<slug>/`
4. Create README.md with Doc Approach and issue number (if any)
5. Create execution-log.md
6. Commit, push
7. Invoke vrau:brainstorm

## State Detection
Read `docs/designs/<workflow>/execution-log.md`:
- Phase: brainstorm | plan | execute
- Status of current phase

Fallback if no execution log - check files:
- Only README.md → brainstorm
- Has design/*.md, no plan/*.md → plan
- Has plan/*.md → execute

## Routing
- Brainstorm needed → invoke vrau:brainstorm
- Plan needed → invoke vrau:plan
- Execute needed → invoke vrau:execute
```

---

### 2. brainstorm (Phase 1)

```markdown
---
name: brainstorm
description: Use when executing Phase 1 of vrau workflow - exploring requirements and design
---

# Phase 1: Brainstorm

## Checkpoint (READ THIS FIRST)
Current state: Read `docs/designs/<workflow>/execution-log.md`
You are in: **BRAINSTORM PHASE**
Next phase: Plan (after PR merged)

## Steps
1. Update main, ask user: worktree (use superpowers:using-git-worktrees) or new branch?
2. Brainstorm with user (main session - use tools/MCP/web as needed)
3. Save to `design/brainstorm.md`, commit
4. Evaluate scope - split if too large
5. Self-review (optional)
6. Spawn reviewer → vrau:vrau-reviewer agent
7. Handle feedback:
   - APPROVED → proceed to step 8
   - REVISE/RETHINK → fix and re-submit (max 3 iterations)
   - After 3 failures → ASK USER what to do
8. Open PR with "refs #<issue>" (if Doc Approach B), merge to main

## Critical Rules
- [ ] Brainstorming runs in MAIN SESSION (user must answer questions)
- [ ] ALWAYS verify with live sources (tools, MCP, web) - docs change, your knowledge may be stale
- [ ] If something seems weird or unclear → ASK USER, don't assume
- [ ] MUST spawn separate reviewer (fresh eyes)
- [ ] Reviewer approval = proceed. 3 failures = ask user.
- [ ] All commits include "refs #<issue>" (if Doc Approach B)
- [ ] PR uses "refs #<issue>", NOT "closes" (saved for final PR)
- [ ] ANY file change (execution log, brainstorm.md) → write, commit, push immediately
```

---

### 3. plan (Phase 2)

```markdown
---
name: plan
description: Use when executing Phase 2 of vrau workflow - creating implementation plan from approved brainstorm
---

# Phase 2: Plan

## Checkpoint (READ THIS FIRST)
Current state: Read `docs/designs/<workflow>/execution-log.md`
You are in: **PLAN PHASE**
Previous: Brainstorm (must be merged to main)
Next: Execute (after PR merged)

## Steps
1. Update main, ask user: worktree (use superpowers:using-git-worktrees) or new branch?
2. Write plan using superpowers:writing-plans
   - Include dependency graph, parallel groups, model per task
   - Include commit points (when to commit during execution)
   - ALWAYS verify with live sources - docs change
3. Save to `plan/<design>-plan.md`, commit
4. Spawn reviewer → vrau:vrau-reviewer agent
5. Handle feedback:
   - APPROVED → proceed to step 6
   - REVISE/RETHINK → fix and re-submit (max 3 iterations)
   - After 3 failures → ASK USER what to do
6. Open PR with "refs #<issue>" (if Doc Approach B), merge to main

## Critical Rules
- [ ] Plans go to FILES, never to GitHub Issues
- [ ] Plans MUST specify commit points for execution
- [ ] ALWAYS verify with live sources - docs change, your knowledge may be stale
- [ ] MUST spawn separate reviewer (fresh eyes)
- [ ] Reviewer approval = proceed. 3 failures = ask user.
- [ ] All commits include "refs #<issue>" (if Doc Approach B)
- [ ] PR uses "refs #<issue>", NOT "closes" (saved for final PR)
- [ ] ANY file change (execution log, plan.md) → write, commit, push immediately
```

---

### 4. execute (Phase 3)

```markdown
---
name: execute
description: Use when executing Phase 3 of vrau workflow - implementing the approved plan
---

# Phase 3: Execute

## Checkpoint (READ THIS FIRST)
Current state: Read `docs/designs/<workflow>/execution-log.md`
You are in: **EXECUTE PHASE**
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
```

---

### 5. vrau-reviewer (Fresh Eyes Agent)

```markdown
---
name: vrau-reviewer
description: Use when spawned to review vrau brainstorm or plan documents with fresh eyes
---

# Vrau Reviewer

You are a fresh reviewer with no prior context. Your job is unbiased review.

## Input
You'll receive a document path to review (brainstorm.md or plan.md).

**If asked to review CODE instead:** Delegate to superpowers:requesting-code-review - that's not this skill's job.

## Review Process
1. Read the document thoroughly
2. ALWAYS verify claims with live sources (tools, MCP, web) - docs change
3. Evaluate based on document type:

**For Brainstorms:** Clarity, completeness, feasibility, risks identified?

**For Plans:** Dependencies correct? Parallel groups make sense? Commit points specified? Tasks actionable?

## Output Format
```
## Verdict: APPROVED | REVISE | RETHINK

## Summary
[1-2 sentences]

## Issues (if any)
- [Issue 1]
- [Issue 2]

## Suggestions (optional)
- [Nice-to-have improvements]
```

## Rules
- Be constructive, not pedantic
- REVISE = minor fixes needed
- RETHINK = fundamental problems
- APPROVED = good to proceed
```

---

## Migration Plan

### Delete (merge functionality into new skills)
- skills/brainstorm-phase/
- skills/brainstorm-step-research/
- skills/brainstorm-step-brainstorm/
- skills/receiving-brainstorm-review/
- skills/plan-phase/
- skills/plan-step-write/
- skills/vrau-writing-plans/
- skills/receiving-plan-review/
- skills/execute-phase/
- skills/vrau-executing-plans/
- skills/review-step-spawn-reviewer/
- skills/find-workflow/
- skills/complexity-calibrated-review/
- skills/strategic-decomposition/
- skills/execution-log-pattern.md

### Create
- skills/vrau/SKILL.md (refactored entry point)
- skills/brainstorm/SKILL.md
- skills/plan/SKILL.md
- skills/execute/SKILL.md
- skills/vrau-reviewer/SKILL.md

### Keep (unchanged)
- commands/start.md
- .claude/vrau/templates/README.template.md
- agents/vrau-reviewer.md → move to skills/vrau-reviewer/

---

## Key Design Decisions

1. **One skill per phase** - Reduces confusion about which skill is active
2. **Self-contained skills** - Each phase skill has everything needed, no jumping between skills
3. **Checkpoint at top** - Every skill starts with "READ THIS FIRST" showing current state
4. **Critical rules as checklist** - Non-negotiable items are explicit and scannable
5. **Minimal hand-holding** - Trust Claude's intelligence, only explain vrau-specific behavior
6. **Always verify with live sources** - Docs change, don't trust stale knowledge
7. **Commit discipline** - All file changes immediately persisted (write, commit, push)
8. **Issue references** - All commits/PRs use "refs #issue", only final PR uses "closes #issue"

---

## Success Criteria

1. **Never skips critical steps** - Reviews happen, PRs get created, issues close correctly
2. **Stays on track** - Clear phase boundaries prevent confusion
3. **Recovers gracefully** - Execution log enables resumption after interruption
4. **Token efficient** - ~700 lines total vs 2,435 lines (71% reduction)
