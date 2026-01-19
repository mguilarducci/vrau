---
name: vrau
description: Use when starting or resuming a vrau workflow - routes to correct phase
model: haiku
---

# Vrau Workflow

## CRITICAL SAFETY RULE

**NEVER COMMIT TO MAIN BRANCH**

Before doing ANYTHING, check current branch:
```bash
git branch --show-current
```

**If on main/master:**
1. STOP immediately
2. Create a new branch OR use worktree
3. NEVER proceed with commits on main

**This is non-negotiable. No exceptions. Ever.**

## On Start
1. Scan `docs/designs/` for folders matching `YYYY-MM-DD-*`
2. If none: start new workflow (see below)
3. If one: auto-select, detect state, ask about worktree/branch (see Resuming Workflow below), invoke correct phase
4. If multiple: ask user which to resume (or start new, or delete old), then ask about worktree/branch (see Resuming Workflow below), invoke correct phase

## New Workflow Setup
1. Ask user for task description
2. Ask: Track with GitHub Issue?
   - Yes → run /start-job command, set Tracking Mode: GitHub
   - No → set Tracking Mode: None
3. **Ask: Worktree or new branch?**
   - Worktree → use superpowers:using-git-worktrees
   - New branch → `git checkout -b <workflow-name>` and `git push -u origin <workflow-name>`
4. Update main first: `git checkout main && git pull`
5. Create worktree OR branch based on choice
6. Create folder `docs/designs/YYYY-MM-DD-<slug>/`
7. Create README.md with Tracking Mode and issue number (if any)
8. Commit, push
9. Invoke vrau:brainstorm

## Resuming Workflow
When resuming an existing workflow:
1. Update main first: `git checkout main && git pull`
2. **Ask: Worktree or new branch?**
   - Worktree → use superpowers:using-git-worktrees
   - New branch → `git checkout -b <workflow-name>-<phase>` and `git push -u origin <workflow-name>-<phase>`
3. Create/go to existent worktree OR branch based on choice
4. Proceed to invoke correct phase

**Why:** User must choose worktree/branch preference every time they resume work. Never assume.

## State Detection
Check files in `docs/designs/<workflow>/`:
- Only README.md → brainstorm phase
- Has `design/*.md`, no `plan/*.md` → plan phase
- Has `plan/*.md` → execute phase
- If unclear → ASK USER

**For Tracking Mode: GitHub** - can also check issue comments for status updates.

**For Tracking Mode: None** - vrau asks user to confirm detected state before proceeding.

## Routing
- Brainstorm needed → invoke vrau:brainstorm
- Plan needed → invoke vrau:plan
- Execute needed → invoke vrau:execute
