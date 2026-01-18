# Vrau Plugin Token Efficiency Refactor - Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Consolidate 15 skills (2,517 lines) into 5 skills (~650 lines) to improve token efficiency and prevent Claude from getting lost during workflow execution.

**Architecture:** Replace fragmented skill structure with one-skill-per-phase design. Each phase skill is self-contained with checkpoint headers, numbered steps, and critical rules checklists. Model selection via YAML metadata instead of runtime enforcement.

**Tech Stack:** Claude Code skills (Markdown with YAML frontmatter)

---

## Dependency Graph

```
┌─────────────────────────────────────────────────────────────────┐
│                        GROUP 1 (Parallel)                        │
│  Create new skills structure (no dependencies between skills)   │
├─────────────────────────────────────────────────────────────────┤
│ Task 1: vrau/SKILL.md     Task 2: brainstorm/SKILL.md          │
│ Task 3: plan/SKILL.md     Task 4: execute/SKILL.md             │
│ Task 5: vrau-reviewer/SKILL.md                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        GROUP 2 (Parallel)                        │
│              Update references and templates                     │
├─────────────────────────────────────────────────────────────────┤
│ Task 6: commands/start.md     Task 7: README.template.md        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        GROUP 3 (Sequential)                      │
│                      Delete old skills                           │
├─────────────────────────────────────────────────────────────────┤
│ Task 8: Delete 15 old skill files/folders                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        GROUP 4 (Sequential)                      │
│                       Final verification                         │
├─────────────────────────────────────────────────────────────────┤
│ Task 9: Verify plugin structure and test                        │
└─────────────────────────────────────────────────────────────────┘
```

## Parallel Groups Summary

| Group | Tasks | Can Parallelize | Commit Point |
|-------|-------|-----------------|--------------|
| 1 | Tasks 1-5 | Yes (5 agents) | After all 5 complete |
| 2 | Tasks 6-7 | Yes (2 agents) | After both complete |
| 3 | Task 8 | No | After deletion |
| 4 | Task 9 | No | Final commit (cleanup) |

---

## Task 1: Create vrau/SKILL.md (Entry Point)

**Model:** haiku
**Files:**
- Create: `skills/vrau/SKILL.md` (overwrite existing)

**Step 1: Write the new vrau entry point skill**

```markdown
---
name: vrau
description: Use when starting or resuming a vrau workflow - routes to correct phase
model: haiku
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
5. Create execution-log.md (see format below)
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
- If unclear → ASK USER

## Execution Log Format
```
# Execution Log: <workflow>

## Workflow Context
- **Task:** <description>
- **Phase:** brainstorm | plan | execute
- **Branch:** <branch name>
- **Issue:** #<number> or (none)

## Status
- **Current Step:** <step number and name>
- **Last Updated:** <timestamp>
```

## Routing
- Brainstorm needed → invoke vrau:brainstorm
- Plan needed → invoke vrau:plan
- Execute needed → invoke vrau:execute
```

**Step 2: Verify file was created correctly**

Run: `head -20 skills/vrau/SKILL.md`
Expected: YAML frontmatter with name, description, model fields

---

## Task 2: Create brainstorm/SKILL.md (Phase 1)

**Model:** sonnet
**Files:**
- Create: `skills/brainstorm/SKILL.md`

**Step 1: Create the brainstorm skill directory**

Run: `mkdir -p skills/brainstorm`

**Step 2: Write the brainstorm phase skill**

```markdown
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
```

**Step 3: Verify file was created correctly**

Run: `head -10 skills/brainstorm/SKILL.md`
Expected: YAML frontmatter with model: sonnet

---

## Task 3: Create plan/SKILL.md (Phase 2)

**Model:** opus
**Files:**
- Create: `skills/plan/SKILL.md`

**Step 1: Create the plan skill directory**

Run: `mkdir -p skills/plan`

**Step 2: Write the plan phase skill**

```markdown
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
```

**Step 3: Verify file was created correctly**

Run: `head -10 skills/plan/SKILL.md`
Expected: YAML frontmatter with model: opus

---

## Task 4: Create execute/SKILL.md (Phase 3)

**Model:** sonnet
**Files:**
- Create: `skills/execute/SKILL.md`

**Step 1: Create the execute skill directory**

Run: `mkdir -p skills/execute`

**Step 2: Write the execute phase skill**

```markdown
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
```

**Step 3: Verify file was created correctly**

Run: `head -10 skills/execute/SKILL.md`
Expected: YAML frontmatter with model: sonnet

---

## Task 5: Create vrau-reviewer/SKILL.md (Fresh Eyes Agent)

**Model:** sonnet
**Files:**
- Create: `skills/vrau-reviewer/SKILL.md`

**Step 1: Create the vrau-reviewer skill directory**

Run: `mkdir -p skills/vrau-reviewer`

**Step 2: Write the vrau-reviewer skill**

```markdown
---
name: vrau-reviewer
description: Use when spawned to review vrau brainstorm or plan documents with fresh eyes
model: sonnet
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

## Complexity Calibration
Match review depth to task complexity:
- **Simple tasks:** Don't over-engineer. Brief review.
- **Complex tasks:** Thorough analysis. Check edge cases.

## Output Format
```
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

## Rules
- Be constructive, not pedantic
- REVISE = minor fixes needed
- RETHINK = fundamental problems
- APPROVED = good to proceed
- Don't nitpick simple tasks
```

**Step 3: Verify file was created correctly**

Run: `head -10 skills/vrau-reviewer/SKILL.md`
Expected: YAML frontmatter with model: sonnet

---

## COMMIT POINT: After Group 1

After Tasks 1-5 are complete, commit all new skills:

```bash
git add skills/vrau/SKILL.md skills/brainstorm/SKILL.md skills/plan/SKILL.md skills/execute/SKILL.md skills/vrau-reviewer/SKILL.md
git commit -m "feat(vrau): create consolidated 5-skill structure

- vrau: entry point with workflow detection and routing (haiku)
- brainstorm: Phase 1 with checkpoint header and critical rules (sonnet)
- plan: Phase 2 with superpowers:writing-plans integration (opus)
- execute: Phase 3 with parallel group code review (sonnet)
- vrau-reviewer: fresh eyes document reviewer (sonnet)

Token reduction: 2,517 → ~650 lines (74% reduction)"
git push
```

---

## Task 6: Update commands/start.md

**Model:** haiku
**Files:**
- Modify: `commands/start.md`

**Step 1: Simplify start.md to invoke new vrau skill**

Replace entire contents with:

```markdown
---
description: "Begin or resume a vrau workflow"
---

# Start Vrau Workflow

Invoke the vrau entry point skill:

```
Skill tool:
- skill: "vrau:vrau"
```

The vrau skill handles:
- Scanning for existing workflows
- Starting new workflows
- Detecting state and routing to correct phase
```

**Step 2: Verify the update**

Run: `cat commands/start.md`
Expected: Simplified content invoking vrau:vrau

---

## Task 7: Simplify README.template.md

**Model:** haiku
**Files:**
- Modify: `.claude/vrau/templates/README.template.md`

**Step 1: Remove per-step model configuration (now in skill metadata)**

Replace entire contents with:

```markdown
# {{TASK_TITLE}}

**Task:** {{TASK_DESCRIPTION}}

**Branch:** {{BRANCH_NAME}}
**Date:** {{DATE}}
**Doc Approach:** {{DOC_APPROACH}}
**Issue:** {{ISSUE_NUMBER}}

## Links

### Design Documents
{{DESIGN_LINKS}}

### Plan Documents
{{PLAN_LINKS}}
```

**Step 2: Verify the update**

Run: `cat .claude/vrau/templates/README.template.md`
Expected: Simplified template without Model Configuration section

---

## COMMIT POINT: After Group 2

After Tasks 6-7 are complete, commit the updates:

```bash
git add commands/start.md .claude/vrau/templates/README.template.md
git commit -m "refactor(vrau): simplify start command and README template

- start.md: now just invokes vrau:vrau (routing handled by skill)
- README.template.md: removed per-step model config (now in skill metadata)"
git push
```

---

## Task 8: Delete Old Skills

**Model:** haiku
**Files:**
- Delete: 15 skill files/folders

**Step 1: Delete old skill folders and files**

```bash
rm -rf skills/brainstorm-phase
rm -rf skills/brainstorm-step-research
rm -rf skills/brainstorm-step-brainstorm
rm -rf skills/receiving-brainstorm-review
rm -rf skills/plan-phase
rm -rf skills/plan-step-write
rm -rf skills/vrau-writing-plans
rm -rf skills/receiving-plan-review
rm -rf skills/execute-phase
rm -rf skills/vrau-executing-plans
rm -rf skills/review-step-spawn-reviewer
rm -rf skills/find-workflow
rm -rf skills/complexity-calibrated-review
rm -rf skills/strategic-decomposition
rm skills/execution-log-pattern.md
```

**Step 2: Verify deletion**

Run: `ls skills/`
Expected: Only vrau/, brainstorm/, plan/, execute/, vrau-reviewer/ directories remain

---

## COMMIT POINT: After Group 3

After Task 8 is complete, commit the deletion:

```bash
git add -A
git commit -m "refactor(vrau): delete 15 old skills (merged into 5 new skills)

Deleted:
- brainstorm-phase, brainstorm-step-research, brainstorm-step-brainstorm
- receiving-brainstorm-review, plan-phase, plan-step-write
- vrau-writing-plans, receiving-plan-review, execute-phase
- vrau-executing-plans, review-step-spawn-reviewer, find-workflow
- complexity-calibrated-review, strategic-decomposition
- execution-log-pattern.md

These are now consolidated into:
- vrau (entry point + routing)
- brainstorm (all Phase 1)
- plan (all Phase 2)
- execute (all Phase 3)
- vrau-reviewer (document review)"
git push
```

---

## Task 9: Final Verification

**Model:** sonnet
**Files:**
- Verify: All skill files in correct locations with correct content

**Step 1: Verify skill structure**

Run: `find skills -name "SKILL.md" | xargs wc -l`
Expected: 5 SKILL.md files, total ~650 lines

**Step 2: Verify YAML frontmatter in all skills**

Run: `for f in skills/*/SKILL.md; do echo "=== $f ==="; head -5 "$f"; done`
Expected: Each skill has name, description, and model fields

**Step 3: Verify model assignments are correct**

| Skill | Expected Model |
|-------|---------------|
| vrau | haiku |
| brainstorm | sonnet |
| plan | opus |
| execute | sonnet |
| vrau-reviewer | sonnet |

**Step 4: Verify commands/start.md invokes vrau:vrau**

Run: `grep "vrau:vrau" commands/start.md`
Expected: Match found

**Step 5: Count lines saved**

Run: `find skills -name "*.md" | xargs wc -l`
Expected: ~650 total lines (vs 2,517 before = 74% reduction)

---

## Summary

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Skills | 15 | 5 | -67% |
| Lines | 2,517 | ~650 | -74% |
| Phase confusion | High | Low | One skill per phase |
| Step skipping | Common | Rare | Numbered steps + checklists |
| Recovery | Difficult | Easy | Execution log + fallback detection |
