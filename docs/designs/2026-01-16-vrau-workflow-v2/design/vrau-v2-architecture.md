# Vrau Workflow v2 Architecture

## Executive Summary

Redesign vrau to support:
1. **Two-phase workflow**: Phase 1 (Design/Brainstorm) and Phase 2 (Plan/Execute)
2. **Issue tracking integration**: GitHub Issues via `gh` CLI
3. **Flexible document storage**: Files, issues, or local-only (no commit)
4. **Model selection per step**: User configures which model for each step
5. **Aggressive commit/push discipline**: Stateless brainstorming with frequent saves
6. **Brainstorm breakdown**: Large designs split into feature-sized pieces
7. **Session compaction**: Explicit "compact session" reminders at phase transitions

---

## Phase 1: Brainstorm/Design

### Step 0: Branch Setup (haiku)

```
1. Update default branch: git fetch origin && git checkout main && git pull
2. Branch/worktree selection:
   - Option 1: Create worktree (use superpowers:using-git-worktrees)
   - Option 2: Work in current branch
   - Option 3: Create new branch (git checkout -b <name>)
3. Push new branch: git push -u origin <branch>
```

**Issue tracking:** If using issues, include issue number in all commits: `git commit -m "feat: description (#123)"`

### Step 1: Step-by-Step Instructions (haiku)

**1.1 Document Approach Selection**

Ask user which approach for writing documents:

```
How should I store design documents?

A. File-based (default)
   → docs/designs/{date}-{branch-name}/design/{file}.md
   → Committed and pushed

B. GitHub Issues
   → Create issues for design documentation
   → Save reference links to docs/designs/{date}-{branch-name}/README.md

C. Local-only (NO COMMITS)
   → Same as A, but NEVER commit these files
   → Creates .local marker file
```

**Folder naming:** Replace `/` with `-` in branch name
- Branch: `feat/add-auth` → Folder: `feat-add-auth`

**After selection:**
- Record choice in README.md
- If option C: Create `docs/designs/{date}-{branch-name}/.no-commit.local` (gitignored)
- Commit and push README.md (unless option C)

**1.2 Model Selection**

Show default models for each step:

```
Model configuration (defaults shown):

Phase 1 - Brainstorm:
  - Branch setup: haiku
  - Instructions: haiku
  - Verifications: haiku
  - Brainstorm: opus
  - Review: opus
  - Breakdown: sonnet

Phase 2 - Plan:
  - Pre-plan: haiku
  - Write plan: opus
  - Plan review: opus

Phase 3 - Execute:
  - Simple code: haiku
  - Complex code: sonnet
  - Very complex: opus (will ask)
  - Review/quality: sonnet (minimum)

Accept defaults? [Y/n] or specify changes
```

**After confirmation:**
- Save model choices to README.md
- Commit and push (unless no-commit mode)

### Step 2: Brainstorm (opus)

**Process:**
1. Run verifications with haiku (tests, dev server, etc.)
2. Invoke `superpowers:brainstorming` with opus
3. After each major section of Q&A:
   - Save progress according to doc approach (file/issue)
   - Commit and push (unless no-commit mode)
4. Continue until design is ready
5. Do NOT open PR yet

**Commit discipline:**
- Small, frequent commits with descriptive messages
- Include issue number if applicable: `design: add auth flow (#123)`
- Goal: stateless brainstorming - can resume from any point

### Step 3: Brainstorm Review Loop (opus)

**FIRST: COMPACT THE SESSION**

Remind user: "Consider compacting the session before review to free up context."

**Process:**

1. Update files/issues with current state, commit
2. Spawn fresh reviewer subagent (opus) using `vrau:vrau-reviewer`
   - Input: task description, complexity, brainstorm content
   - Request: "Review this brainstorm"
   - Reviewer considers task complexity (don't over-review simple scripts)
3. If issues found:
   - Analyze feedback with opus
   - Make changes that make sense
   - Save, commit, push
4. Repeat until:
   - Reviewer approves, OR
   - 3 iterations reached → ask user if should continue

**Review skills needed:**
- Use existing `vrau:vrau-reviewer` for review
- Create new `vrau:receiving-brainstorm-review` for handling feedback (similar to superpowers:receiving-code-review)

### Step 4: Brainstorm Breakdown (sonnet)

**FIRST: COMPACT THE SESSION**

**Process:**

1. Analyze if design can be broken into smaller independent designs
   - Each piece should be feature/value-sized
   - If already small enough, skip breakdown

2. If breaking down:

**For file-based (option A/C):**
```
docs/designs/{date}-{branch}/design/
  00-executive-summary.md    # High-level overview only
  01-design-feature-x.md     # First sub-design
  02-design-feature-y.md     # Second sub-design
  ...
```
- Main file becomes executive summary
- Each sub-design references main file
- Update README.md with file list
- Commit and push (unless no-commit)

**For issue-based (option B):**
- Main issue becomes executive summary
- Create sub-issues with sequential prefix names
- Set main issue as parent of each sub-issue (GitHub feature)
- Update README.md with issue links
- Commit and push

3. **Ask user:** "Should I review each sub-design individually? [Y/n]"
   - If yes: run review loop on each sub-design
   - If no: proceed to PR

### Step 5: Open PR (sonnet)

1. Invoke `superpowers:finishing-a-development-branch`
2. Open PR (don't wait for user choice)
3. Merge PR

**PR description:**
- Link to design docs or issues
- Summary of what was designed
- List of sub-designs if broken down

---

## Phase 2: Plan/Execute

**Recommend:** Start with a new session (haiku)

Make it simple - provide copy-paste command or clear next steps.

**Important:** Plans always go to files, NOT issues. Do not update issues with plan content.

### Step 5: Pre-Write Plan (haiku)

1. Ask which design to plan:
   ```
   Which design do you want to plan?

   Files in docs/designs/{date}-{branch}/design/:
   1. 01-design-feature-x.md
   2. 02-design-feature-y.md

   Or issues listed in README.md:
   3. #123 - Feature X
   4. #124 - Feature Y

   Select: [1-4]
   ```
   - Only show file/issue names, don't read content yet

2. Run branch setup (same as Phase 1 Step 0)

### Step 6: Write Plan (opus)

1. Invoke `superpowers:writing-plans`
2. Write plan to: `docs/designs/{date}-{branch}/plan/{design-name}-plan.md`
3. Commit and push frequently (small commits)

**Plan requirements:**
- List all task dependencies explicitly
- Reference the design document

### Step 7: Plan Review Loop (opus)

**FIRST: COMPACT THE SESSION**

Same as brainstorm review loop:
1. Spawn fresh reviewer (opus)
2. Max 3 iterations
3. Ask user after 3

**Skills needed:**
- Use `vrau:vrau-reviewer` for review
- Create `vrau:receiving-plan-review` for handling feedback

### Step 8: Execution

**FIRST: COMPACT THE SESSION**

**Process:**

0. Verify dependencies (sonnet):
   - Check if all dependencies in plan file are implemented
   - If not, notify user before proceeding

1. Ask permission to proceed

2. Start with `superpowers:subagent-driven-development`:
   - Simple tasks: haiku
   - Complex tasks: sonnet
   - Very complex tasks: **ask** before using opus
   - Quality/review: sonnet minimum (never haiku)

3. After permission granted:
   - Execute without asking for each step
   - Ask only if doubts or decisions needed

4. Follow superpower skill until complete

5. Open PR and add comment: `@claude, review`

---

## File Structure Changes

### Current (to be replaced)
```
.claude/vrau/workflows/{date}-{branch}/
  brainstorm.md
  plan.md
  execution-log.md
```

### New
```
docs/designs/{date}-{branch-dashes}/
  README.md                    # Full context, choices, links
  .no-commit.local             # Present if option C (gitignored)
  design/
    00-executive-summary.md    # Or single design.md
    01-design-xxx.md
    02-design-yyy.md
  plan/
    {design-name}-plan.md

.claude/vrau/
  config.json                  # Team defaults
  config.local.json            # Personal overrides (gitignored)
```

### Gitignore additions
```
docs/designs/**/.no-commit.local
docs/designs/**/*.local.md
.claude/vrau/**/*.local.*
```

---

## Skills to Create/Modify

### New Skills

1. **`vrau:receiving-brainstorm-review`**
   - Similar to `superpowers:receiving-code-review`
   - Handles reviewer feedback on brainstorms
   - Requires technical rigor, not blind agreement

2. **`vrau:receiving-plan-review`**
   - Similar to above but for plans
   - Handles reviewer feedback on implementation plans

### Modified Skills/Commands

1. **`commands/start.md`** - Complete rewrite for new flow
2. **`commands/_phases/brainstorm.md`** - Add commit discipline, breakdown
3. **`commands/_phases/plan.md`** - Add dependency check, commit discipline
4. **`commands/_phases/execute.md`** - Add model selection, @claude review
5. **`skills/find-workflow/SKILL.md`** - Update paths from `.claude/vrau/workflows/` to `docs/designs/`

---

## State Machine

```
START
  ↓
[Branch Setup]
  ↓
[Doc Approach Selection] → A: Files / B: Issues / C: Local-only
  ↓
[Model Selection]
  ↓
[Brainstorm] ←→ [Save/Commit frequently]
  ↓
[Brainstorm Review] ←→ [Revise] (max 3)
  ↓
[Breakdown?] → Yes → [Break into sub-designs] → [Review each?]
  ↓                                                    ↓
  No ←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←
  ↓
[Open PR & Merge] → END PHASE 1
  ↓
[New Session Recommended]
  ↓
[Select Design to Plan]
  ↓
[Branch Setup]
  ↓
[Write Plan] ←→ [Save/Commit frequently]
  ↓
[Plan Review] ←→ [Revise] (max 3)
  ↓
[Check Dependencies]
  ↓
[Execute with subagents]
  ↓
[Open PR + @claude review] → END PHASE 2
```

---

## Resolved Questions

1. **Storage location:** Fully migrate to `docs/designs/` - remove `.claude/vrau/workflows/`
2. **README template:** Use a template file at `.claude/vrau/templates/README.template.md`
3. **Issue tracking:** GitHub Issues only, no project board
