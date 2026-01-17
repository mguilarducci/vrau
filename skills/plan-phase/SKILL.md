---
name: plan-phase
description: Use when executing Phase 2 of a vrau workflow - after brainstorm is complete, when plan/*.md doesn't exist yet
---

# Phase 2: Plan

**Important:** Plans always go to files, NOT issues. Do not update GitHub Issues with plan content.

### Model Selection for Phase 2

| Step | Model | Rationale |
|------|-------|-----------|
| Pre-plan setup | haiku | Simple file listing, git operations |
| Write plan | opus | Quality planning requires deep thinking |
| Review loop | opus | Via reviewer agent |

---

## Step 0: Research Available Tools (haiku)

**Model enforcement:** If current session is not haiku, dispatch Task tool:
```
Task(subagent_type="general-purpose", model="haiku", prompt="[Step 0 instructions]")
```
If current session is haiku, run in current session.

Before writing the plan, check what research tools are available:

1. **List available MCP tools** - documentation fetchers, web search, API explorers
2. **Identify relevant tools** - does the design mention technologies with MCP tools?
3. **Use relevant tools** - fetch current docs, search for implementation patterns

**Output:** Brief summary of research findings to inform the plan.

---

## Pre-Plan Setup (haiku)

**Model enforcement:** If current session is not haiku, dispatch Task tool:
```
Task(subagent_type="general-purpose", model="haiku", prompt="[Pre-Plan Setup instructions]")
```
If current session is haiku, run in current session.

**Recommend new session:** "Consider starting a fresh session for planning."

### Select Design to Plan

Ask which design to plan (file names only, don't read content):

```
Which design do you want to plan?

Files in docs/designs/<workflow>/design/:
1. 01-design-feature-x.md
2. 02-design-feature-y.md

Or issues in README.md:
3. #123 - Feature X

Select: [1-N]
```

### Branch Setup

**Follow the same process as Phase 1 Branch Setup:**

1. **Update default branch:**
   ```bash
   git fetch origin && git checkout main && git pull
   ```

2. **Ask user about branch strategy:**
   ```
   Branch setup options:

   A) Git worktree (isolated, recommended for complex work)
   B) New branch from main
   C) Continue in current branch

   Which option? [A/B/C]
   ```

3. **Execute based on choice:**

   **If A (worktree):**
   - Use `superpowers:using-git-worktrees` skill

   **If B (new branch):**
   ```bash
   git checkout -b vrau/<workflow>/plan
   git push -u origin vrau/<workflow>/plan
   ```

   **If C (current branch):**
   - Continue in current branch

## Write Plan

**Invoke wrapper skill to enforce opus model:**

```
Skill tool:
- skill: "vrau:plan-step-write"
```

The skill will:
- Check current session model
- If not opus, dispatch Task tool with opus model
- If opus, invoke vrau:vrau-writing-plans skill directly
- Create plan with dependency graph, parallel groups, model assignments
- Commit after each major section

**Output:** Plan document saved to `docs/designs/<workflow>/plan/<design-name>-plan.md`

## Review Loop

**FIRST: Recommend session compaction**

> "Before I request a plan review, consider compacting the session to reduce token usage."

**Invoke wrapper skill to enforce opus model:**

```
Skill tool:
- skill: "vrau:review-step-spawn-reviewer"
```

The skill will:
- Check current session model
- If not opus, dispatch Task tool with opus model
- If opus, execute review process directly:
  - Read plan document
  - Spawn vrau-reviewer agent
  - If REVISE/RETHINK: Use receiving-plan-review skill
  - If APPROVED: Signal completion

**When plan is approved:** Proceed to Phase 3.

## Phase 2 Complete

When plan is approved, report to user:

> "Phase 2 (Plan) is complete. Ready to proceed to Phase 3 (Execute)?"

Wait for user confirmation, then invoke:

```
Skill tool:
- skill: "vrau:execute-phase"
```
