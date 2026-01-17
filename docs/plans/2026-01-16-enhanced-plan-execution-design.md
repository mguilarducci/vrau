# Enhanced Plan Execution Design

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Enhance vrau workflow with dependency-aware parallel execution, per-task model recommendations, and research integration.

**Architecture:** Create vrau-specific wrappers for writing-plans and executing-plans that add dependency graphs, parallel groups, and model assignments. Add mandatory research steps to all brainstorm and review phases.

**Tech Stack:** Claude Code skills, MCP tools integration, Task tool with parallel dispatch

---

## Design Decisions

1. **Both** per-task dependencies AND summary graph (redundancy for clarity)
2. **Explicit parallel groups** with wave-style organization
3. **Hard model recommendation** per task in plan
4. **Mandatory research step** checking any available MCP/tools
5. **Override in vrau only** - no superpowers fork needed

---

## Enhanced Plan Format

### Per-Task Additions

```markdown
### Task 3: User Authentication API

**Depends on:** Task 1, Task 2
**Parallel group:** B
**Model:** sonnet

**Files:**
- Create: `src/auth/api.ts`
...
```

### Dependency Graph Section (Top of Plan)

```markdown
## Task Dependency Graph

```
Task 0 (setup) ─┬─► Task 1 ─┬─► Task 3 ─► Task 5
                │           │
                └─► Task 2 ─┘

Task 4 (independent) ─────────────► Task 6
```

## Parallel Execution Groups

| Group | Tasks | Can Start After |
|-------|-------|-----------------|
| A | 0 | (none) |
| B | 1, 2, 4 | Group A complete |
| C | 3 | Tasks 1, 2 complete |
| D | 5, 6 | Tasks 3, 4 complete |

## Model Assignments

| Task | Model | Rationale |
|------|-------|-----------|
| 0 | haiku | Simple setup |
| 1, 2 | sonnet | Standard implementation |
| 3 | opus | Complex auth logic |
| 4 | haiku | Config only |
| 5, 6 | sonnet | Integration work |
```

### Model Assignment Rules

Following vrau conventions:
- **haiku**: Setup, config, simple file operations
- **sonnet**: Standard implementation, tests, integration
- **opus**: Complex logic, security-sensitive, architectural decisions

---

## Parallel Execution Flow

```
Read plan → Extract dependency graph → Execute by groups

Group A: Task 0 (sequential - only one task)
    ↓ wait for completion
Group B: Tasks 1, 2, 4 (parallel dispatch)
    ↓ wait for ALL to complete
Group C: Task 3 (sequential - only one task)
    ↓ wait for completion
Group D: Tasks 5, 6 (parallel dispatch)
    ↓ wait for ALL to complete
Done
```

### Integration with dispatching-parallel-agents

The `execute-phase` skill will:

1. Parse the plan's dependency graph and parallel groups
2. For groups with 2+ tasks: invoke `superpowers:dispatching-parallel-agents`
3. For single-task groups: use normal sequential execution
4. Wait for group completion before starting next group

### Model Enforcement

Each dispatched agent receives the model specified in the plan:

```
Task tool:
  subagent_type: "general-purpose"
  model: "sonnet"  ← from plan's Model field
  prompt: "Implement Task 3..."
```

---

## Research Step

Added as **Step 0** before brainstorming/review begins:

```markdown
## Step 0: Research Available Tools (haiku)

Before starting, check what research tools are available:

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

**Output:** Brief summary of what was researched and key findings.
```

### Where This Step Appears

| Skill | Research Focus |
|-------|----------------|
| `brainstorm-phase` | Technology docs, best practices, prior art |
| `receiving-brainstorm-review` | Verify reviewer claims, check current docs |
| `plan-phase` | Implementation patterns, library APIs |
| `receiving-plan-review` | Verify feasibility concerns, check dependencies |

---

## Skills to Create/Modify

### New Skills

| Skill | Purpose |
|-------|---------|
| `vrau-writing-plans` | Wraps superpowers:writing-plans, adds dependency graph, parallel groups, model assignments |
| `vrau-executing-plans` | Wraps superpowers:executing-plans, adds parallel dispatch based on plan structure |

### Modified Skills

| Skill | Changes |
|-------|---------|
| `brainstorm-phase` | Add Step 0: Research Available Tools before brainstorming |
| `plan-phase` | Call `vrau-writing-plans` instead of `superpowers:writing-plans` |
| `execute-phase` | Call `vrau-executing-plans` instead of `superpowers:subagent-driven-development` |
| `receiving-brainstorm-review` | Add research step to verify reviewer claims |
| `receiving-plan-review` | Add research step to verify feasibility concerns |

### Skill Dependencies

```
brainstorm-phase
  └── (new) Step 0: Research
  └── superpowers:brainstorming (unchanged)

plan-phase
  └── (new) Step 0: Research
  └── vrau-writing-plans (new)
      └── superpowers:writing-plans (calls internally)

execute-phase
  └── vrau-executing-plans (new)
      └── superpowers:dispatching-parallel-agents (for parallel groups)
      └── superpowers:subagent-driven-development (for sequential tasks)
```

---

## Documentation Updates

### README.md Changes

1. Add new sections explaining enhanced plan format
2. Update phase diagrams to show research steps
3. Update model selection table for per-task models
4. Add parallel execution explanation
