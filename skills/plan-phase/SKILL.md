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

## Pre-Plan Setup (haiku)

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

Same as Phase 1 Step 0:
1. Update default branch: `git fetch origin && git checkout main && git pull`
2. Branch/worktree selection (use superpowers:using-git-worktrees if worktree)
3. Push new branch: `git push -u origin <branch>`

## Write Plan (opus)

1. Invoke `superpowers:writing-plans` skill with opus model
2. Provide context: reference the selected design document
3. Write plan to: `docs/designs/<workflow>/plan/<design-name>-plan.md`

**Commit discipline:** Commit after each major section:
```bash
git add docs/designs/<workflow>/plan/
git commit -m "plan: <description> [#issue-number]"
git push
```

**Plan requirements:**
- List all task dependencies explicitly
- Reference the design document
- Follow bite-sized task format from writing-plans skill

## Review Loop (opus)

**FIRST: Recommend session compaction**

### Review Process

1. Ensure plan is saved and committed
2. Spawn reviewer:
   ```
   subagent_type: "vrau:vrau-reviewer"
   prompt:
     1. Task: <one-line description>
     2. Complexity: <same as brainstorm>
     3. Content: <plan.md content>
     4. Request: "Review this plan for completeness and feasibility"
   ```

3. If reviewer returns REVISE or RETHINK:
   - **REQUIRED SKILL:** Use `vrau:receiving-plan-review`
   - Process each issue with technical rigor
   - Update plan
   - Save, commit, push
   - Re-invoke reviewer

4. Loop constraints: Max 3 iterations, then ask user

## Phase 2 Complete

When plan is approved, report to user:

> "Phase 2 (Plan) is complete. Ready to proceed to Phase 3 (Execute)?"

Wait for user confirmation, then invoke:

```
Skill tool:
- skill: "vrau:execute-phase"
```
