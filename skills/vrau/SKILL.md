---
name: vrau
description: Use when starting or resuming a vrau workflow - routes to correct phase
model: haiku
---

# Vrau Workflow

## On Start
1. Scan `docs/designs/` for folders matching `YYYY-MM-DD-*`
2. If none: start new workflow (see below)
3. If one: auto-select, detect state, invoke correct phase
4. If multiple: ask user which to resume (or start new, or delete old)

## New Workflow Setup
1. Ask user for task description
2. Ask: Start from GitHub issue?
   - Yes → get issue number, set Doc Approach B
   - No → set Doc Approach A
3. (Optional) Ask: Local-only mode? → set Doc Approach C, create .no-commit.local
4. Create folder `docs/designs/YYYY-MM-DD-<slug>/`
5. Create README.md with Doc Approach and issue number (if any)
6. Create execution-log.md (see format below)
7. Commit, push (skip if Doc Approach C)
8. Invoke vrau:brainstorm

## State Detection
Read `docs/designs/<workflow>/execution-log.md`:
- Phase: brainstorm | plan | execute
- Status of current phase

Fallback if no execution log - check files:
- Only README.md → brainstorm
- Has design/*.md, no plan/*.md → plan
- Has plan/*.md → execute
- If unclear → ASK USER

## Execution Log Format
```
# Execution Log: <workflow>

## Workflow Context
- **Task:** <description>
- **Phase:** brainstorm | plan | execute
- **Branch:** <branch name>
- **Issue:** #<number> or (none)

## Status
- **Current Step:** <step number and name>
- **Last Updated:** <timestamp>
```

## Routing
- Brainstorm needed → invoke vrau:brainstorm
- Plan needed → invoke vrau:plan
- Execute needed → invoke vrau:execute
