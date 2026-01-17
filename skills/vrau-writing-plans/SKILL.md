---
name: vrau-writing-plans
description: Use when writing implementation plans in vrau workflow - adds dependency graphs, parallel execution groups, and per-task model assignments
---

# Vrau Writing Plans

Enhanced plan writing with dependency tracking, parallel groups, and model assignments.

## Overview

Wraps `superpowers:writing-plans` with vrau-specific enhancements for parallel execution.

**Announce:** "I'm using vrau-writing-plans to create the plan with dependency tracking."

## Step 0: Research Available Tools (haiku)

Before writing, check for useful MCP tools:

1. **List available MCP tools** - doc fetchers, web search, API explorers
2. **Identify relevant tools** - technologies mentioned in the design
3. **Use them** - fetch current docs, search for patterns

**Output:** Brief research summary to inform the plan.

## Required Plan Sections

After the standard header, every plan MUST include:

### 1. Task Dependency Graph

```
Task 0 ──┬──► Task 2 ──► Task 4
         │
Task 1 ──┘
```

### 2. Parallel Execution Groups

| Group | Tasks | Can Start After |
|-------|-------|-----------------|
| A | 0, 1 | (none) |
| B | 2 | Group A complete |

### 3. Model Assignments

| Task | Model | Rationale |
|------|-------|-----------|
| 0 | haiku | Simple setup |
| 1 | sonnet | Complex logic |

### 4. Per-Task Fields

```markdown
### Task N: [Name]

**Depends on:** Task X, Task Y (or "none")
**Parallel group:** A
**Model:** haiku/sonnet/opus

**Files:** ...
**Steps:** ...
```

## Model Selection Rules

| Complexity | Model | Examples |
|------------|-------|----------|
| trivial/simple | haiku | Setup, config, single file |
| moderate | sonnet | Multi-file, integration |
| complex | sonnet | Architecture, security |
| very complex | opus | Requires user approval |

**Quality/review:** sonnet minimum

## Dependency Analysis

1. **True dependencies** - Task B needs Task A's output
2. **No false dependencies** - ordering preference ≠ dependency
3. **Maximize parallelism** - independent tasks = same group
4. **Prevent conflicts** - parallel tasks must not edit same files

## Process

1. Invoke `superpowers:writing-plans` for standard task format
2. Add dependency graph section
3. Add parallel groups table
4. Add model assignments table
5. Add per-task fields (depends-on, group, model)

## Handoff

> "Plan complete with N tasks in M parallel groups. Ready for execution?"

Use `vrau:execute-phase` for parallel-aware execution.
