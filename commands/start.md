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

Read [_phases/brainstorm.md](_phases/brainstorm.md) for the complete brainstorm process.

## Phase 2: Plan

Read [_phases/plan.md](_phases/plan.md) for the complete planning process.

## Phase 3: Execute

Read [_phases/execute.md](_phases/execute.md) for execution options.

---

## State Recovery

| Files in workflow folder | State | Resume with |
|-------------------------|-------|-------------|
| (empty) | Not started | `/vrau:start` |
| brainstorm.md | Brainstorm done | `/vrau:plan` |
| brainstorm.md + plan.md | Plan done | `/vrau:execute` |
| + execution-log.md | Done | `/vrau:status` |
