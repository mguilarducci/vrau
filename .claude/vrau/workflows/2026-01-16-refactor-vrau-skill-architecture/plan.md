# Vrau Skill Architecture Refactor - Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Refactor vrau plugin to follow Claude Code best practices - extract shared patterns into reusable skills, improve discoverability, optimize context with progressive disclosure.

**Architecture:** Create two new skills (`find-workflow`, `complexity-calibrated-review`), enhance one existing skill (`vrau-workflow`), and restructure `start.md` by extracting phases into separate files. Commands will reference skills instead of inline duplication.

**Tech Stack:** Markdown skills, Claude Code plugin architecture

**Reviewer Clarifications Addressed:**
- `status.md` will also use `find-workflow` skill (5 commands total, not 4)
- Phase files are loaded on-demand via markdown links (progressive disclosure)
- `complexity-calibrated-review` skill teaches the PATTERN; `vrau-reviewer` agent IMPLEMENTS it

---

## Task 1: Create `find-workflow` Skill

**Files:**
- Create: `skills/find-workflow/SKILL.md`

**Step 1: Create skill directory**

```bash
mkdir -p skills/find-workflow
```

**Step 2: Write the skill file**

Create `skills/find-workflow/SKILL.md`:

```markdown
---
name: find-workflow
description: Use when needing to discover, list, or select an existing vrau workflow from .claude/vrau/workflows/. Use before any command that operates on an existing workflow.
---

# Find Workflow

## Overview

Discovers vrau workflows and handles user selection. Returns workflow path, name, state, and files.

## When to Use

- Before operating on an existing workflow
- When user runs /vrau:brainstorm, /vrau:plan, /vrau:execute, /vrau:abort, /vrau:status
- When resuming work after session ends

## Process

1. Scan `.claude/vrau/workflows/` for folders
2. If none exist → display "No workflows found. Use /vrau:start to begin."
3. If one exists → auto-select it
4. If multiple exist → present numbered list with state, ask user to choose

## State Derivation

Derive state from files present in workflow folder:

| Files Present | State |
|---------------|-------|
| (empty folder) | not_started |
| brainstorm.md only | brainstorm_complete |
| brainstorm.md + plan.md | plan_complete |
| + execution-log.md | executing |

## Output Format

After selection, provide:
- **Path:** `.claude/vrau/workflows/<workflow-name>/`
- **Name:** `<workflow-name>`
- **State:** One of the states above
- **Files:** List of files present

## Display Format (for multiple workflows)

```
Found workflows:

1. 2026-01-10-feat-add-auth
   State: plan_complete
   Files: brainstorm.md, plan.md

2. 2026-01-12-fix-api-error
   State: brainstorm_complete
   Files: brainstorm.md

Which workflow? [1-2]
```
```

**Step 3: Verify file created**

```bash
cat skills/find-workflow/SKILL.md | head -20
```

**Step 4: Commit**

```bash
git add skills/find-workflow/SKILL.md
git commit -m "feat: add find-workflow skill for workflow discovery"
```

---

## Task 2: Create `complexity-calibrated-review` Skill

**Files:**
- Create: `skills/complexity-calibrated-review/SKILL.md`

**Step 1: Create skill directory**

```bash
mkdir -p skills/complexity-calibrated-review
```

**Step 2: Write the skill file**

Create `skills/complexity-calibrated-review/SKILL.md`:

```markdown
---
name: complexity-calibrated-review
description: Use when reviewing documents (brainstorms, plans, designs, PRs) where review depth should match task complexity. Prevents over-engineering simple tasks while ensuring thorough review of complex ones.
---

# Complexity-Calibrated Review

## Overview

Adjust review depth based on task complexity. Over-engineering is a failure mode - demanding enterprise patterns for a simple script is as bad as missing security in a payment system.

## When to Use

- Reviewing brainstorm documents
- Reviewing implementation plans
- Reviewing designs or PRs
- Any review where appropriate depth matters

## Complexity Levels

Determine complexity BEFORE reviewing:

| Level | Examples | Signals |
|-------|----------|---------|
| trivial | poem script, single config | One file, no external deps |
| simple | CLI tool, small feature | Few files, clear scope |
| moderate | API endpoint, multi-file | Multiple components, some deps |
| complex | payment system, distributed | Security-critical, many integrations |

## Review Depth by Complexity

| Complexity | Lenses Applied | Bias |
|------------|----------------|------|
| **trivial** | Completeness only | APPROVE unless fundamentally broken |
| **simple** | + Obvious gaps | APPROVE with minor notes |
| **moderate** | + Security, Architecture, Testability, Maintainability | Balanced |
| **complex** | + Performance, Failure modes, Observability | Thorough |

## Output Format

```markdown
## Summary
[2-3 sentences: what this proposes, overall assessment]

## Critical Issues
[Blocking problems - for trivial tasks, almost always empty]

## Important Issues
[Should address - significant but not blocking]

## Suggestions
[Nice to have - proportional to complexity]

## Verdict
APPROVE | REVISE | RETHINK
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Demanding retry policies for a script | Check complexity level first |
| Rubber-stamping complex security code | Complex = thorough review |
| Same depth for all reviews | Always calibrate to complexity |
```

**Step 3: Verify file created**

```bash
cat skills/complexity-calibrated-review/SKILL.md | head -20
```

**Step 4: Commit**

```bash
git add skills/complexity-calibrated-review/SKILL.md
git commit -m "feat: add complexity-calibrated-review skill for YAGNI-aware reviews"
```

---

## Task 3: Enhance `vrau-workflow` Skill

**Files:**
- Modify: `skills/vrau/SKILL.md`

**Step 1: Read current file**

```bash
cat skills/vrau/SKILL.md
```

**Step 2: Replace with enhanced version**

Replace `skills/vrau/SKILL.md` with:

```markdown
---
name: vrau-workflow
description: |
  Use when detecting complex multi-step tasks that would benefit from
  structured brainstorm → plan → execute workflow. Triggers: new features,
  refactors, multi-file changes, bug investigations, architectural changes.
---

# Vrau Workflow Suggestion

Suggest vrau for complex tasks, but never force it.

## Complexity Estimation

Before suggesting vrau, estimate task complexity:

| Signal | Points |
|--------|--------|
| Multiple files mentioned | +1 |
| Multiple components/systems | +1 |
| Requires investigation first | +1 |
| User says "feature", "refactor", "redesign", "architecture" | +1 |
| User says "quick", "small", "just", "simple" | -2 |
| Single file explicitly mentioned | -1 |

**Threshold:** Suggest vrau when score >= 2

## When to Suggest

Score >= 2 AND task involves:
- New features with multiple components
- Refactoring spanning multiple files
- Bug fixes requiring investigation
- Tasks with multiple requirements
- Architectural changes or design decisions

## When NOT to Suggest

Score < 2 OR task is:
- Simple single-file changes
- Quick fixes or typo corrections
- Questions or research tasks
- Tasks user explicitly wants done immediately

## How to Suggest

Present naturally when score >= 2:

```
This looks like a multi-step task that could benefit from structured workflow.

Would you like to use vrau?
→ Brainstorm requirements (with auto-review)
→ Create detailed plan (with auto-review)
→ Execute with tracking

1. Yes, start vrau workflow
2. No, just help me directly
```

If user chooses "Yes", invoke `/vrau:start`.

## Key Principle

**Never force vrau.** Always offer "help directly" as an alternative.
```

**Step 3: Verify changes**

```bash
cat skills/vrau/SKILL.md
```

**Step 4: Commit**

```bash
git add skills/vrau/SKILL.md
git commit -m "feat: enhance vrau-workflow with complexity estimation"
```

---

## Task 4: Create Phase Files Directory

**Files:**
- Create: `commands/_phases/` directory

**Step 1: Create directory**

```bash
mkdir -p commands/_phases
```

**Step 2: Commit**

```bash
git add commands/_phases/.gitkeep 2>/dev/null || touch commands/_phases/.gitkeep && git add commands/_phases/.gitkeep
git commit -m "chore: add _phases directory for start.md progressive disclosure"
```

---

## Task 5: Create Brainstorm Phase File

**Files:**
- Create: `commands/_phases/brainstorm.md`

**Step 1: Write the phase file**

Create `commands/_phases/brainstorm.md`:

```markdown
# Phase 1: Brainstorm

## Invoke Brainstorming

Use `superpowers:brainstorming` skill with the task description.

Let the skill drive the conversation - it asks clarifying questions until producing a final brainstorm document.

**Completion signal:** Summary or "brainstorm complete" message.

## Save Output

Write brainstorm to:
```
.claude/vrau/workflows/<workflow>/brainstorm.md
```

## Review

Determine task complexity (trivial/simple/moderate/complex), then spawn reviewer:

```
Task tool:
- subagent_type: "vrau:vrau-reviewer"
- prompt:
  1. Task description (one line)
  2. Complexity: <level>
  3. brainstorm.md content
  4. Request: "Review this brainstorm"
```

**REQUIRED:** Reviewer uses complexity-calibrated-review pattern.

## Review Loop

1. Reviewer APPROVEs → proceed to Phase 2 (Plan)
2. Reviewer has issues → refine brainstorm, re-invoke reviewer
3. Max 3 iterations
4. After 3 without approval → ask user how to proceed
```

**Step 2: Commit**

```bash
git add commands/_phases/brainstorm.md
git commit -m "feat: extract brainstorm phase from start.md"
```

---

## Task 6: Create Plan Phase File

**Files:**
- Create: `commands/_phases/plan.md`

**Step 1: Write the phase file**

Create `commands/_phases/plan.md`:

```markdown
# Phase 2: Plan

## Invoke Planning

Use `superpowers:writing-plans` skill.

Provide context:
- Reference brainstorm at `.claude/vrau/workflows/<workflow>/brainstorm.md`
- Task complexity (same as brainstorm phase)

Let the skill drive until it produces a plan.

**Completion signal:** Plan document ready.

## Save Output

Write plan to:
```
.claude/vrau/workflows/<workflow>/plan.md
```

## Review

Spawn reviewer with same complexity level:

```
Task tool:
- subagent_type: "vrau:vrau-reviewer"
- prompt:
  1. Task description (one line)
  2. Complexity: <level>
  3. plan.md content
  4. Request: "Review this plan for completeness and feasibility"
```

**REQUIRED:** Reviewer uses complexity-calibrated-review pattern.

## Review Loop

Same as brainstorm: max 3 iterations, then ask user.

When approved → proceed to Phase 3 (Execute)
```

**Step 2: Commit**

```bash
git add commands/_phases/plan.md
git commit -m "feat: extract plan phase from start.md"
```

---

## Task 7: Create Execute Phase File

**Files:**
- Create: `commands/_phases/execute.md`

**Step 1: Write the phase file**

Create `commands/_phases/execute.md`:

```markdown
# Phase 3: Execute

## Execution Options

Present choices:

```
Plan approved! Ready to execute.

1. Subagent-driven (this session)
   → Uses superpowers:subagent-driven-development
   → Fresh subagent per task

2. Manual execution
   → Execute plan step by step
   → More control, same session
```

**No auto-review for execution.**

## When Complete

1. Write summary to `.claude/vrau/workflows/<workflow>/execution-log.md`
2. Invoke `superpowers:finishing-a-development-branch`
```

**Step 2: Commit**

```bash
git add commands/_phases/execute.md
git commit -m "feat: extract execute phase from start.md"
```

---

## Task 8: Refactor `commands/start.md`

**Files:**
- Modify: `commands/start.md`

**Step 1: Read current file**

```bash
cat commands/start.md
```

**Step 2: Replace with slimmed orchestrator**

Replace `commands/start.md` with:

```markdown
---
description: "Begin a new vrau workflow"
---

# Start Vrau Workflow

## Step 1: Task Description

Ask the user:

```
What are you working on?
> [Wait for user to describe task]
```

## Step 2: Branch Name

Generate a suggested branch name from the task description:
- Use kebab-case
- Prefix with fix/, feat/, refactor/, etc.
- Replace slashes with dashes for folder naming

```
Suggested branch: <generated-name>

1. Confirm
2. Edit name
```

Branch names must match: `^[a-zA-Z0-9/_-]+$`

## Step 3: Workspace Setup

```
How do you want to work?

1. Create worktree (isolated workspace)
   → Uses superpowers:using-git-worktrees

2. Work in current branch
   → Stay in current directory

3. Create new branch (no worktree)
   → git checkout -b <branch-name>
```

## Step 4: Create Workflow Folder

```bash
WORKFLOW=$(date +%Y-%m-%d)-<branch-name-with-dashes>
mkdir -p .claude/vrau/workflows/$WORKFLOW
```

Ensure `.gitignore` has: `.claude/vrau/**/*.local.*`

---

## Phase 1: Brainstorm

See [_phases/brainstorm.md](_phases/brainstorm.md)

## Phase 2: Plan

See [_phases/plan.md](_phases/plan.md)

## Phase 3: Execute

See [_phases/execute.md](_phases/execute.md)

---

## State Recovery

| Files in workflow folder | State | Resume with |
|-------------------------|-------|-------------|
| (empty) | Not started | `/vrau:start` |
| brainstorm.md | Brainstorm done | `/vrau:plan` |
| brainstorm.md + plan.md | Plan done | `/vrau:execute` |
| + execution-log.md | Done | `/vrau:status` |
```

**Step 3: Verify changes**

```bash
wc -l commands/start.md
# Should be ~80 lines (was 187)
```

**Step 4: Commit**

```bash
git add commands/start.md
git commit -m "refactor: slim start.md with progressive disclosure to phase files"
```

---

## Task 9: Refactor `commands/brainstorm.md`

**Files:**
- Modify: `commands/brainstorm.md`

**Step 1: Read current file**

```bash
cat commands/brainstorm.md
```

**Step 2: Replace with refactored version**

Replace `commands/brainstorm.md` with:

```markdown
---
description: "Continue brainstorming for an existing workflow"
---

# Continue Brainstorm

## Find Workflow

Use the `find-workflow` skill to discover and select the target workflow.

## Check State

- If `brainstorm.md` exists → offer to refine or view
- If no `brainstorm.md` → start fresh brainstorm

## Actions

### If brainstorm.md exists:

```
Brainstorm exists for this workflow.

1. Refine brainstorm
   → Continue with superpowers:brainstorming
   → Update brainstorm.md

2. View current brainstorm
   → Display contents

3. Re-run review
   → Invoke vrau:vrau-reviewer with complexity-calibrated-review
```

### If no brainstorm.md:

1. Invoke `superpowers:brainstorming` skill
2. Save to `.claude/vrau/workflows/<workflow>/brainstorm.md`
3. Run review (see [_phases/brainstorm.md](_phases/brainstorm.md) for review process)
4. When approved, proceed to planning
```

**Step 3: Commit**

```bash
git add commands/brainstorm.md
git commit -m "refactor: brainstorm.md uses find-workflow skill"
```

---

## Task 10: Refactor `commands/plan.md`

**Files:**
- Modify: `commands/plan.md`

**Step 1: Read current file**

```bash
cat commands/plan.md
```

**Step 2: Replace with refactored version**

Replace `commands/plan.md` with:

```markdown
---
description: "Continue planning for an existing workflow"
---

# Continue Plan

## Find Workflow

Use the `find-workflow` skill to discover and select the target workflow.

## Check State

- If no `brainstorm.md` → "Cannot plan without brainstorm. Run /vrau:brainstorm first."
- If `plan.md` exists → offer to refine or view
- If `brainstorm.md` exists but no `plan.md` → start fresh plan

## Actions

### If plan.md exists:

```
Plan exists for this workflow.

1. Refine plan
   → Continue with superpowers:writing-plans
   → Update plan.md

2. View current plan
   → Display contents

3. Re-run review
   → Invoke vrau:vrau-reviewer with complexity-calibrated-review
```

### If no plan.md (but brainstorm exists):

1. Invoke `superpowers:writing-plans` skill
2. Reference brainstorm at `.claude/vrau/workflows/<workflow>/brainstorm.md`
3. Save to `.claude/vrau/workflows/<workflow>/plan.md`
4. Run review (see [_phases/plan.md](_phases/plan.md) for review process)
5. When approved, proceed to execution
```

**Step 3: Commit**

```bash
git add commands/plan.md
git commit -m "refactor: plan.md uses find-workflow skill"
```

---

## Task 11: Refactor `commands/execute.md`

**Files:**
- Modify: `commands/execute.md`

**Step 1: Read current file**

```bash
cat commands/execute.md
```

**Step 2: Replace with refactored version**

Replace `commands/execute.md` with:

```markdown
---
description: "Continue execution for an existing workflow"
---

# Continue Execution

## Find Workflow

Use the `find-workflow` skill to discover and select the target workflow.

## Check State

- If no `plan.md` → "Cannot execute without plan. Run /vrau:plan first."
- If `execution-log.md` exists → show progress and offer to continue
- If `plan.md` exists but no `execution-log.md` → start execution

## Actions

### If execution-log.md exists:

```
Execution Progress:
- Completed: X/Y tasks
- Last task: <task name>

1. Continue execution
   → Resume from where we left off

2. View execution log
   → Display full log

3. View plan
   → Display plan.md

4. Restart execution
   → Start from beginning
```

### If no execution-log.md (but plan exists):

See [_phases/execute.md](_phases/execute.md) for execution options.

When complete:
1. Write log to `.claude/vrau/workflows/<workflow>/execution-log.md`
2. Invoke `superpowers:finishing-a-development-branch`
```

**Step 3: Commit**

```bash
git add commands/execute.md
git commit -m "refactor: execute.md uses find-workflow skill"
```

---

## Task 12: Refactor `commands/abort.md`

**Files:**
- Modify: `commands/abort.md`

**Step 1: Read current file**

```bash
cat commands/abort.md
```

**Step 2: Replace with refactored version**

Replace `commands/abort.md` with:

```markdown
---
description: "Delete a vrau workflow"
---

# Abort Workflow

## Find Workflow

Use the `find-workflow` skill to discover and select the workflow to delete.

If no workflows exist, display: "No workflows to abort."

## Confirmation

Show workflow contents and confirm:

```
Delete workflow: <workflow-name>

Contents:
  - <list files>

1. Delete
   → Remove folder and all files

2. Archive
   → Move to .claude/vrau/archive/

3. Cancel
```

## Execute

**Delete:**
```bash
rm -rf .claude/vrau/workflows/<workflow>
```

**Archive:**
```bash
mkdir -p .claude/vrau/archive
mv .claude/vrau/workflows/<workflow> .claude/vrau/archive/
```

Display: "Workflow deleted/archived."
```

**Step 3: Commit**

```bash
git add commands/abort.md
git commit -m "refactor: abort.md uses find-workflow skill"
```

---

## Task 13: Refactor `commands/status.md`

**Files:**
- Modify: `commands/status.md`

**Step 1: Read current file**

```bash
cat commands/status.md
```

**Step 2: Replace with refactored version**

Replace `commands/status.md` with:

```markdown
---
description: "Show vrau workflow status"
---

# Vrau Status

## Find Workflows

Scan `.claude/vrau/workflows/` for folders.

If none exist:
```
No vrau workflows found.
Use /vrau:start to begin.
```

## List Workflows

Use the `find-workflow` skill's state derivation for each workflow:

| Files Present | State |
|--------------|-------|
| (empty folder) | Not started |
| brainstorm.md only | Brainstorm complete |
| brainstorm.md + plan.md | Plan complete |
| brainstorm.md + plan.md + execution-log.md | Execution started |

Display:

```
Vrau Workflows:

  <workflow-name>
    State: <state>
    Files: <file list>

Commands:
  /vrau:start      - Begin new workflow
  /vrau:brainstorm - Continue brainstorming
  /vrau:plan       - Continue planning
  /vrau:execute    - Continue execution
  /vrau:abort      - Delete workflow
```

## Single Workflow Detail

If user asks about specific workflow:

```
Workflow: <name>

State: <state>
Created: <date from folder name>

Files:
  <file> (<size>)

Next step: <recommended command>
```
```

**Step 3: Commit**

```bash
git add commands/status.md
git commit -m "refactor: status.md references find-workflow skill"
```

---

## Task 14: Final Verification

**Step 1: Check all files exist**

```bash
ls -la skills/find-workflow/
ls -la skills/complexity-calibrated-review/
ls -la commands/_phases/
```

**Step 2: Count lines in refactored files**

```bash
echo "=== Line counts ==="
wc -l skills/find-workflow/SKILL.md
wc -l skills/complexity-calibrated-review/SKILL.md
wc -l skills/vrau/SKILL.md
wc -l commands/start.md
wc -l commands/_phases/*.md
wc -l commands/brainstorm.md commands/plan.md commands/execute.md commands/abort.md commands/status.md
```

**Step 3: Verify git status**

```bash
git log --oneline -10
git status
```

**Step 4: Final commit if needed**

```bash
git add -A
git status
# Only commit if there are uncommitted changes
```

---

## Summary

| Task | Files | Action |
|------|-------|--------|
| 1 | skills/find-workflow/SKILL.md | CREATE |
| 2 | skills/complexity-calibrated-review/SKILL.md | CREATE |
| 3 | skills/vrau/SKILL.md | MODIFY |
| 4 | commands/_phases/ | CREATE DIR |
| 5 | commands/_phases/brainstorm.md | CREATE |
| 6 | commands/_phases/plan.md | CREATE |
| 7 | commands/_phases/execute.md | CREATE |
| 8 | commands/start.md | MODIFY |
| 9 | commands/brainstorm.md | MODIFY |
| 10 | commands/plan.md | MODIFY |
| 11 | commands/execute.md | MODIFY |
| 12 | commands/abort.md | MODIFY |
| 13 | commands/status.md | MODIFY |
| 14 | - | VERIFY |

**Expected outcome:**
- 2 new skills created
- 1 skill enhanced
- 3 phase files extracted
- 6 commands refactored
- ~88 lines of duplication eliminated
- Progressive disclosure implemented
