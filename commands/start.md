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
   -> Uses superpowers:using-git-worktrees
   -> Best for: larger features, parallel work

2. Work in current branch
   -> Stay in current directory
   -> Best for: small fixes

3. Create new branch (no worktree)
   -> git checkout -b <branch-name>
   -> Best for: standard git flow
```

Based on choice:
- **Option 1**: Invoke `superpowers:using-git-worktrees` with branch name. If it fails (dirty working directory, branch exists), ask user how to proceed.
- **Option 2**: No git operations
- **Option 3**: Run `git checkout -b <branch-name>`. If it fails, ask user.

## Step 4: Create Workflow Folder

Convert branch name to folder-safe format (replace `/` with `-`):
```bash
WORKFLOW=$(date +%Y-%m-%d)-<branch-name-with-dashes>
mkdir -p .claude/vrau/workflows/$WORKFLOW
```

**Note:** `$WORKFLOW` is used throughout this document to refer to the workflow folder name.

Check `.gitignore` for vrau patterns. If missing, add:
```
.claude/vrau/**/*.local.*
```

---

## Phase 1: Brainstorm

Invoke `superpowers:brainstorming` skill with the task description.

The brainstorming skill asks clarifying questions in the first turns. Let it drive the conversation until it produces a final brainstorm document.

**Completion signal:** The brainstorming skill will indicate when complete (usually with a summary or "brainstorm complete" message).

**Save output:** Write the brainstorm summary/spec to:
```
.claude/vrau/workflows/<workflow>/brainstorm.md
```

### Brainstorm Review

Use the Task tool to spawn `vrau:vrau-reviewer` subagent:

```
Task tool parameters:
- subagent_type: "vrau:vrau-reviewer"
- prompt: Include:
  1. Task description (one line)
  2. Task complexity: "trivial" | "simple" | "moderate" | "complex"
  3. The brainstorm.md content
  4. Request: "Review this brainstorm for completeness"
```

**Complexity calibration:**
| Complexity | Examples | Review depth |
|------------|----------|--------------|
| trivial | poem script, single file | Approve unless fundamentally broken |
| simple | CLI tool, small feature | Check for obvious gaps |
| moderate | API endpoint, multi-file | Check architecture, error handling |
| complex | payment system, distributed | Demand security, retry, testing |

**Review loop:**
1. Reviewer approves → proceed to Phase 2
2. Reviewer has issues → refine brainstorm based on feedback, re-invoke reviewer
3. Max 3 iterations total
4. After 3 without approval → ask user: "Reviewer still has concerns: [list]. How to proceed?"

---

## Phase 2: Plan

Invoke `superpowers:writing-plans` skill.

Provide context:
- Reference brainstorm at `.claude/vrau/workflows/<workflow>/brainstorm.md`
- Task complexity (same as brainstorm phase)

The planning skill asks clarifying questions in the first turns. Let it drive until it produces a plan.

**Completion signal:** The planning skill indicates when complete.

**Save output:** Write the plan to:
```
.claude/vrau/workflows/<workflow>/plan.md
```

### Plan Review

Use the Task tool to spawn `vrau:vrau-reviewer` subagent:

```
Task tool parameters:
- subagent_type: "vrau:vrau-reviewer"
- prompt: Include:
  1. Task description (one line)
  2. Task complexity: "trivial" | "simple" | "moderate" | "complex"
  3. The plan.md content
  4. Request: "Review this plan for completeness and feasibility"
```

Same complexity calibration and review loop as brainstorm (max 3 iterations, then ask user).

---

## Phase 3: Execute

Present execution options:

```
Plan approved! Ready to execute.

1. Subagent-driven (this session)
   -> Uses superpowers:subagent-driven-development
   -> Fresh subagent per task
   -> Progress tracked here

2. Manual execution
   -> You execute the plan step by step
   -> More control, same session
```

**No auto-review for execution.**

When execution completes:
1. Write summary to `.claude/vrau/workflows/<workflow>/execution-log.md`
2. Invoke `superpowers:finishing-a-development-branch`

---

## State Recovery

If session ends mid-workflow, state is derived from files:

| Files in workflow folder | State | Resume with |
|-------------------------|-------|-------------|
| (empty) | Not started | `/vrau:start` |
| brainstorm.md | Brainstorm done | `/vrau:plan` |
| brainstorm.md + plan.md | Plan done | `/vrau:execute` |
| + execution-log.md | Done | `/vrau:status` |

Use `/vrau:status` to see all workflows and their state.
