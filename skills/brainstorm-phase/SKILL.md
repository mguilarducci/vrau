---
name: brainstorm-phase
description: Use when executing Phase 1 of a vrau workflow - after workflow setup, when design/brainstorm.md doesn't exist yet
---

# Phase 1: Brainstorm

You're now in Phase 1 (Brainstorm) of the vrau workflow.

**Goal:** Produce a thorough, reviewed brainstorm document exploring requirements, design, and implementation approach.

---

## Stateless Execution Pattern

**Each step is stateless.** Subagents have no memory of previous steps. Use the execution log to persist context.

**Execution log location:** `docs/designs/<workflow>/execution-log.md`

**Every step MUST:**
1. READ execution log at start (pass content to subagent)
2. DO the step's work
3. WRITE updated execution log with current status and context for next step
4. COMMIT AND PUSH immediately

```bash
git add docs/designs/<workflow>/execution-log.md
git commit -m "vrau(<workflow>): update execution log - <step name>"
git push
```

**When dispatching subagents:** Include execution log content in the prompt so the subagent has full context.

### Model Selection for Phase 1

| Step | Model | Rationale |
|------|-------|-----------|
| Pre-checks | haiku | Simple verification tasks |
| Brainstorming | opus | Creative, quality thinking |
| Save | sonnet | Standard operations |
| Breakdown check | sonnet | Always evaluate scope before review |
| Self-review | sonnet | Optional quality check |
| Formal review | opus | Via reviewer agent |
| PR/Merge | haiku | Mechanical git operations |

---

## Pre-Brainstorm Setup: Select Issue or File

**Ask the user what to brainstorm:**

```
What would you like to brainstorm?

Options:
1. GitHub issue (provide issue number)
2. Specific file or feature (provide description)
3. General task (provide description)

Please specify:
```

Wait for user response. Use their input as the task description for the rest of the phase.

### Initialize Execution Log

After getting user input, create the execution log:

```markdown
# Execution Log: <workflow>

## Workflow Context
- **Task:** <user's task description>
- **Phase:** brainstorm
- **Branch:** (pending)
- **Started:** <timestamp>

## Brainstorm
- **Status:** pending
- **File:** docs/designs/<workflow>/design/brainstorm.md

## Last Updated
<timestamp> - initialized
```

```bash
git add docs/designs/<workflow>/execution-log.md
git commit -m "vrau(<workflow>): initialize execution log"
git push
```

---

## Branch Setup (haiku)

**Model enforcement:** Always dispatch Task tool with haiku model:
```
Task(subagent_type="general-purpose", model="haiku", prompt="[Branch Setup instructions]")
```

### Update Default Branch

1. **Fetch and update main branch:**
   ```bash
   git fetch origin && git checkout main && git pull
   ```

### Create Feature Branch

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
   - Creates isolated workspace

   **If B (new branch):**
   ```bash
   git checkout -b vrau/<workflow>/brainstorm
   git push -u origin vrau/<workflow>/brainstorm
   ```

   **If C (current branch):**
   - Verify current branch is not main
   - If on main, force option A or B

**After branch setup, update execution log:**
- Set Branch to the created branch name
- Update Last Updated
- Commit and push

---

## Step 0: Research Available Tools

**Invoke wrapper skill to enforce haiku model:**

```
Skill tool:
- skill: "vrau:brainstorm-step-research"
```

The skill will:
- Always dispatch Task tool with haiku model
- Research available MCP tools and gather context

**Output:** Research summary to pass to brainstorming step.

**After research, update execution log:**
- Add `## Research Summary` section with findings
- Update Last Updated
- Commit and push

---

## Pre-Brainstorm Checks (haiku)

**Model enforcement:** Always dispatch Task tool with haiku model:
```
Task(subagent_type="general-purpose", model="haiku", prompt="[Pre-checks instructions]")
```

Run these checks ONCE at the start of the brainstorm phase:

1. **Run the test suite** (if one exists):
   ```bash
   npm test  # or pytest, cargo test, etc.
   ```
   - If tests fail, note failures in brainstorm context
   - Don't block on failures - proceed with brainstorming

2. **Start the dev server** (if applicable):
   ```bash
   npm run dev  # or similar
   ```
   - Verify it starts without errors
   - Note any startup issues in brainstorm context

These checks establish baseline health and inform the brainstorm.

**After pre-checks, update execution log:**
- Note any test failures or dev server issues in Research Summary
- Update Last Updated
- Commit and push

---

## Step 1: Invoke Brainstorming Skill

**Invoke wrapper skill to enforce opus model:**

```
Skill tool:
- skill: "vrau:brainstorm-step-brainstorm"
```

The skill will:
- Always dispatch Task tool with opus model
- Invoke superpowers:brainstorming skill
- Drive the brainstorming conversation

**Completion signal:** The skill produces a brainstorm document ready to save.

**After brainstorming, update execution log:**
- Set Brainstorm Status to `in_progress`
- Add one-line Summary of brainstorm
- Update Last Updated
- Commit and push

---

## Step 2: Save Brainstorm Output (sonnet)

**Model enforcement:** Always dispatch Task tool with sonnet model:
```
Task(subagent_type="general-purpose", model="sonnet", prompt="[Step 2 instructions]")
```

Immediately after brainstorming completes, save the output:

```
docs/designs/<workflow>/design/brainstorm.md
```

**Commit immediately:**
```bash
git add docs/designs/<workflow>/design/brainstorm.md
git commit -m "vrau(<workflow>): complete initial brainstorm"
```

**Why commit now?** Claude Code is stateless. Each save/commit creates a recovery point. If the session ends, you can resume from this checkpoint.

**After save, update execution log:**
- Set Brainstorm Status to `complete`
- Update Last Updated
- Commit and push

---

## Step 3: Evaluate Scope for Breakdown (sonnet)

**Model enforcement:** Always dispatch Task tool with sonnet model:
```
Task(subagent_type="general-purpose", model="sonnet", prompt="[Step 3 instructions]")
```

**Always check if the brainstorm should be split.** Smaller, focused workflows are preferred over large ones.

**Evaluate:**
- Is this brainstorm larger than a single feature?
- Could it be split into independent sub-designs?
- Would multiple workflows be clearer than one large workflow?
- Are there distinct components that could be delivered separately?

**If breakdown makes sense:**

1. **Create breakdown document:**
   ```
   docs/designs/<workflow>/design/brainstorm-breakdown.md
   ```

2. **Content format:**
   ```markdown
   # Brainstorm Breakdown: <workflow>

   ## Original Scope
   <one-line summary of original brainstorm>

   ## Proposed Sub-Designs
   1. **<sub-design-1>**: <one-line description>
   2. **<sub-design-2>**: <one-line description>
   3. **<sub-design-3>**: <one-line description>

   ## Rationale
   <why splitting makes sense>

   ## Dependencies
   <any ordering constraints between sub-designs>
   ```

3. **Commit breakdown:**
   ```bash
   git add docs/designs/<workflow>/design/brainstorm-breakdown.md
   git commit -m "vrau(<workflow>): propose brainstorm breakdown"
   ```

4. **Ask user:**
   > "I've analyzed the scope and recommend splitting this into smaller workflows:
   > [list sub-designs]
   >
   > Options:
   > A) Proceed with separate workflows for each sub-design (recommended)
   > B) Continue with the current workflow as-is"

**If user chooses A (separate workflows):**
- Delete current workflow: `rm -rf docs/designs/<workflow>`
- Use `/vrau:start` for each sub-design
- Each becomes its own vrau workflow with independent brainstorm -> plan -> execute phases
- **STOP HERE** - phase 1 restarts for each sub-workflow

**If user chooses B (continue as-is):**
- Keep breakdown.md for reference
- Continue to Step 4

**If breakdown doesn't make sense:**
- Document why in a brief note (scope is already focused, components are tightly coupled, etc.)
- Continue to Step 4

**After breakdown check, update execution log:**
- Note breakdown decision (split vs continue)
- Update Last Updated
- Commit and push

---

## Step 4: Self-Review (sonnet) - Optional

**Model enforcement:** Always dispatch Task tool with sonnet model:
```
Task(subagent_type="general-purpose", model="sonnet", prompt="[Step 4 instructions]")
```

Before requesting formal review, do a quick self-check:

1. **Completeness:** Does the brainstorm cover requirements, design, and implementation approach?
2. **Clarity:** Would another engineer understand the approach?
3. **Risks:** Are major risks and unknowns identified?

If something is missing, add it now. Save and commit:

```bash
git add docs/designs/<workflow>/design/brainstorm.md
git commit -m "vrau(<workflow>): refine brainstorm after self-review"
```

**After self-review, update execution log:**
- Note any refinements made
- Update Last Updated
- Commit and push

---

## Step 5: Request Formal Review

**IMPORTANT: Before requesting review, remind the user about session compaction:**

> "Before I request a formal review, this is a good time to compact the session to reduce token usage and cost. Would you like me to proceed with the review now, or would you prefer to compact first?"

Wait for user response. If user wants to compact, pause here and let them handle it.

**Invoke wrapper skill to enforce opus model:**

```
Skill tool:
- skill: "vrau:review-step-spawn-reviewer"
```

The skill will:
- Always dispatch Task tool with opus model
- Read brainstorm document
- **MANDATORY: Spawn a SEPARATE vrau-reviewer agent for unbiased fresh-eyes review**
- The orchestrator must NOT review the document itself
- If REVISE/RETHINK: Use receiving-brainstorm-review skill
- If APPROVED: Signal completion

**When review is approved:** Proceed to Step 6.

**After review approved, update execution log:**
- Set Brainstorm Status to `approved`
- Update Last Updated
- Commit and push

---

## Step 6: Open PR and Merge (haiku)

**Model enforcement:** Always dispatch Task tool with haiku model:
```
Task(subagent_type="general-purpose", model="haiku", prompt="[Step 6 instructions]")
```

Once brainstorm is approved, open a PR for the brainstorm phase:

1. **Ensure all changes are committed:**
   ```bash
   git status  # should be clean
   ```

2. **Push branch:**
   ```bash
   git push -u origin vrau/<workflow>/brainstorm
   ```

3. **Create PR:**
   ```bash
   gh pr create --title "vrau(<workflow>): Phase 1 - Brainstorm" --body "$(cat <<'EOF'
   ## Summary
   - Completed brainstorm phase for <workflow>
   - Addressed review feedback
   - Ready to proceed to planning phase

   ## Brainstorm Document
   See `docs/designs/<workflow>/design/brainstorm.md`

   ## Next Steps
   - Merge this PR
   - Proceed to Phase 2 (Plan)
   EOF
   )"
   ```

4. **Merge PR:**
   ```bash
   gh pr merge --squash --delete-branch
   ```

5. **Update local main:**
   ```bash
   git checkout main
   git pull
   ```

---

## Phase 1 Complete

**Update execution log for next phase:**
- Set Phase to `plan`
- Add `## Plan` section with Status: `pending`
- Update Last Updated
- Commit and push

Brainstorm phase is now complete. Report to user:

> "Phase 1 (Brainstorm) is complete and merged to main. Ready to proceed to Phase 2 (Plan)?"

Wait for user confirmation, then invoke:

```
Skill tool:
- skill: "vrau:plan-phase"
```

This will load Phase 2 instructions and continue the workflow.
