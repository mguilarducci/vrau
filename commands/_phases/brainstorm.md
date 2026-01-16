# Phase 1: Brainstorm

You're now in Phase 1 (Brainstorm) of the vrau workflow.

**Goal:** Produce a thorough, reviewed brainstorm document exploring requirements, design, and implementation approach.

---

## Before You Start: Pre-Brainstorm Checks

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

## Step 1: Invoke Brainstorming Skill

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

## Step 2: Save Brainstorm Output

Immediately after brainstorming completes, save the output:

```
.claude/vrau/workflows/<workflow>/brainstorm.md
```

**Commit immediately:**
```bash
git add .claude/vrau/workflows/<workflow>/brainstorm.md
git commit -m "vrau(<workflow>): complete initial brainstorm"
```

**Why commit now?** Claude Code is stateless. Each save/commit creates a recovery point. If the session ends, you can resume from this checkpoint.

---

## Step 3: Self-Review (Optional but Recommended)

Before requesting formal review, do a quick self-check:

1. **Completeness:** Does the brainstorm cover requirements, design, and implementation approach?
2. **Clarity:** Would another engineer understand the approach?
3. **Risks:** Are major risks and unknowns identified?

If something is missing, add it now. Save and commit:

```bash
git add .claude/vrau/workflows/<workflow>/brainstorm.md
git commit -m "vrau(<workflow>): refine brainstorm after self-review"
```

---

## Step 4: Request Formal Review

**IMPORTANT: Before requesting review, remind the user about session compaction:**

> "Before I request a formal review, this is a good time to compact the session to reduce token usage and cost. Would you like me to proceed with the review now, or would you prefer to compact first?"

Wait for user response. If user wants to compact, pause here and let them handle it.

**Once ready for review, use the receiving-brainstorm-review skill:**

```
Skill tool:
- skill: "superpowers:receiving-brainstorm-review"
- args: ".claude/vrau/workflows/<workflow>/brainstorm.md"
```

**Let the skill handle the review process:**
- It will fetch the brainstorm content
- Request review from a vrau reviewer agent
- Handle approval, rejection, or revision requests
- Guide you through any necessary revisions

**The skill will signal when review is complete:**
- "APPROVED" → proceed to Step 5
- "NEEDS_REVISION" → skill will guide revision loop
- "MAX_ITERATIONS" → skill will ask user how to proceed

---

## Step 5: Attempt Brainstorm Breakdown (Optional)

If the brainstorm is large or complex, consider breaking it down into smaller feature-sized pieces:

**Ask yourself:**
- Is this brainstorm larger than a single feature?
- Could it be split into independent sub-designs?
- Would multiple workflows be clearer than one large workflow?

**If YES to any of above:**

1. **Create breakdown document:**
   ```
   .claude/vrau/workflows/<workflow>/brainstorm-breakdown.md
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
   git add .claude/vrau/workflows/<workflow>/brainstorm-breakdown.md
   git commit -m "vrau(<workflow>): propose brainstorm breakdown"
   ```

4. **Ask user:**
   > "I've created a breakdown proposal in brainstorm-breakdown.md. Would you like to:
   > A) Proceed with separate workflows for each sub-design
   > B) Continue with the current workflow as-is"

**If user chooses A (separate workflows):**
- Use `/vrau:abort` to delete current workflow
- Use `/vrau:start` for each sub-design
- Each becomes its own vrau workflow with independent brainstorm → plan → execute phases

**If user chooses B (continue as-is):**
- Keep breakdown.md for reference
- Proceed to Phase 2 (Plan)

**If NO breakdown needed:**
- Skip this step entirely
- Proceed directly to Phase 2 (Plan)

---

## Step 6: Open PR and Merge

Once brainstorm is approved (and optionally broken down), open a PR for the brainstorm phase:

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
   See `.claude/vrau/workflows/<workflow>/brainstorm.md`

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

Wait for user confirmation, then:

```
Skill tool:
- skill: "vrau:plan"
```

This will load Phase 2 instructions and continue the workflow.
