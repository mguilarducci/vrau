---
description: "Begin a new vrau workflow"
---

# Start Vrau Workflow

First, check if `.claude/vrau/state.local.json` exists and has a state other than `idle`. If so:
- Tell user: "A workflow is already in progress. Use `/vrau:status` to see details or `/vrau:abort` to abandon it."
- Stop here.

## Step 1: Task Description

Ask the user:

```
What are you working on?
> [Wait for user to describe task in one sentence]
```

## Step 2: Branch Name

Generate a suggested branch name from the task description (kebab-case, prefix with fix/, feat/, refactor/, etc.).

Present:

```
Suggested branch: <generated-name>

1. Confirm
2. Edit name
```

Branch names must match: `^[a-zA-Z0-9/_-]+$`

## Step 3: Workspace Setup

Present options:

```
How do you want to work?

1. Create worktree (isolated workspace)
   → Uses superpowers:using-git-worktrees
   → New directory with separate branch
   → Best for: larger features, experiments, parallel work

2. Work in current branch
   → Stay in current directory, no branch change
   → Best for: small fixes, quick changes

3. Create new branch (no worktree)
   → New branch in current directory (auto checkout)
   → Best for: medium tasks, standard git flow
```

Based on choice:
- **Option 1**: Invoke `superpowers:using-git-worktrees` with branch name
- **Option 2**: No git operations
- **Option 3**: Run `git checkout -b <branch-name>`

## Step 4: Initialize Workflow

1. Create directories:
   ```bash
   mkdir -p .claude/vrau/workflows/$(date +%Y-%m-%d)-<branch-name>
   ```

2. Create `state.local.json`:
   ```json
   {
     "branch": "<branch-name>",
     "workflow": "YYYY-MM-DD-<branch-name>",
     "state": "brainstorming",
     "workspaceType": "<worktree|current_branch|new_branch>",
     "startedAt": "<ISO-timestamp>"
   }
   ```

3. Ensure `.claude/vrau/**/*.local.md` and `.claude/vrau/**/*.local.json` are in `.gitignore`

## Step 5: Confirmation and Start

Display:

```
✓ Branch created: <branch-name>
✓ Workflow initialized
✓ State: brainstorming

Starting brainstorm phase...
```

Then invoke `superpowers:brainstorming` skill with the task description.
