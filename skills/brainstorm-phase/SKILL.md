---
name: brainstorm-phase
description: Use when executing Phase 1 of a vrau workflow - after workflow setup, when design/brainstorm.md doesn't exist yet
---

# Phase 1: Brainstorm

You're now in Phase 1 (Brainstorm) of the vrau workflow.

**Goal:** Produce a thorough, reviewed brainstorm document exploring requirements, design, and implementation approach.

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

## Step 0: Research Available Tools (haiku)

Before brainstorming, check what research tools are available:

1. **List available MCP tools:**
   - Check for documentation fetchers (context7, claude-docs, etc.)
   - Check for web search capabilities
   - Check for API explorers or code search tools

2. **Identify relevant tools for this task:**
   - Does the task mention specific technologies? (React, Claude API, etc.)
   - Are there docs that should be fetched?
   - Would web search help find current best practices?

3. **Use relevant tools:**
   - Fetch current documentation for mentioned technologies
   - Search for recent changes or deprecations
   - Gather context before brainstorming begins

**Output:** Brief summary of what was researched and key findings. Include this context when invoking the brainstorming skill.

---

## Pre-Brainstorm Checks (haiku)

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

---

## Step 1: Invoke Brainstorming Skill (opus)

Use the `superpowers:brainstorming` skill with the task description:

```
Skill tool:
- skill: "superpowers:brainstorming"
- args: <full task description from workflow>
```

**Let the skill drive the conversation:**
- It will ask clarifying questions
- Answer thoroughly and thoughtfully
- The skill will produce a final brainstorm document when complete

**Completion signal:** The skill produces a summary or "brainstorm complete" message.

---

## Step 2: Save Brainstorm Output (sonnet)

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

---

## Step 3: Evaluate Scope for Breakdown (sonnet)

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

---

## Step 4: Self-Review (sonnet) - Optional

Before requesting formal review, do a quick self-check:

1. **Completeness:** Does the brainstorm cover requirements, design, and implementation approach?
2. **Clarity:** Would another engineer understand the approach?
3. **Risks:** Are major risks and unknowns identified?

If something is missing, add it now. Save and commit:

```bash
git add docs/designs/<workflow>/design/brainstorm.md
git commit -m "vrau(<workflow>): refine brainstorm after self-review"
```

---

## Step 5: Request Formal Review (opus via reviewer agent)

**IMPORTANT: Before requesting review, remind the user about session compaction:**

> "Before I request a formal review, this is a good time to compact the session to reduce token usage and cost. Would you like me to proceed with the review now, or would you prefer to compact first?"

Wait for user response. If user wants to compact, pause here and let them handle it.

**Once ready for review, use the receiving-brainstorm-review skill:**

```
Skill tool:
- skill: "vrau:receiving-brainstorm-review"
- args: "docs/designs/<workflow>/design/brainstorm.md"
```

**Let the skill handle the review process:**
- It will fetch the brainstorm content
- Request review from a vrau reviewer agent
- Handle approval, rejection, or revision requests
- Guide you through any necessary revisions

**The skill will signal when review is complete:**
- "APPROVED" -> proceed to Step 6
- "NEEDS_REVISION" -> skill will guide revision loop
- "MAX_ITERATIONS" -> skill will ask user how to proceed

---

## Step 6: Open PR and Merge (haiku)

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

Brainstorm phase is now complete. Report to user:

> "Phase 1 (Brainstorm) is complete and merged to main. Ready to proceed to Phase 2 (Plan)?"

Wait for user confirmation, then invoke:

```
Skill tool:
- skill: "vrau:plan-phase"
```

This will load Phase 2 instructions and continue the workflow.
