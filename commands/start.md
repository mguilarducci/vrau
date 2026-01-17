---
description: "Begin or resume a vrau workflow"
---

# Vrau Workflow

## Step 0: Check for Existing Workflows

Use the `find-workflow` skill to scan `docs/designs/` for existing workflows.

**If workflows exist:**

```
Found existing workflow(s):

1. 2026-01-15-feat-user-auth
   State: brainstorm_complete

2. 2026-01-10-fix-api-errors
   State: plan_complete

Options:
A. Resume workflow [1-N]
B. Start new workflow
C. Delete workflow [1-N]
```

- **Resume**: Detect phase from files, load appropriate `_phases/*.md`
- **New**: Continue to Step 1
- **Delete**: `rm -rf docs/designs/<workflow>`, then re-check

**If no workflows exist:** Continue to Step 1.

---

## Step 1: Setup New Workflow

### 1.1 Get Task Description

```
What are you working on?
> [Wait for user to describe task]
```

### 1.2 Branch Setup

```
How do you want to work?

1. Create worktree (isolated workspace)
   → Uses superpowers:using-git-worktrees

2. Create new branch
   → git checkout -b <branch-name>

3. Use current branch
   → Stay where you are
```

Based on choice:
- **Option 1**: Invoke `superpowers:using-git-worktrees`
- **Option 2**: `git checkout -b <branch-name> && git push -u origin <branch-name>`
- **Option 3**: No git operations

### 1.3 Create Workflow Folder

```bash
WORKFLOW=$(date +%Y-%m-%d)-<branch-name-with-dashes>
mkdir -p docs/designs/$WORKFLOW/design
mkdir -p docs/designs/$WORKFLOW/plan
```

### 1.4 Create README from Template

Copy `.claude/vrau/templates/README.template.md` to `docs/designs/$WORKFLOW/README.md`

Fill template variables and commit:

```bash
git add docs/designs/$WORKFLOW/README.md
git commit -m "vrau: initialize workflow for <task>"
git push
```

---

## Phase 1: Brainstorm

Invoke the brainstorm-phase skill:
```
Skill tool:
- skill: "vrau:brainstorm-phase"
```

---

## Phase 2: Plan

Invoke the plan-phase skill:
```
Skill tool:
- skill: "vrau:plan-phase"
```

---

## Phase 3: Execute

Invoke the execute-phase skill:
```
Skill tool:
- skill: "vrau:execute-phase"
```

---

## State Detection

| Folder Contents | State | Action |
|-----------------|-------|--------|
| README.md only | not_started | Start brainstorm |
| + design/*.md | brainstorm_complete | Start plan |
| + plan/*.md | plan_complete | Start execute |
| Execution markers in README | done | Show summary |
