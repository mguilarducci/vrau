# Vrau

Workflow orchestration plugin for Claude Code. Guides complex tasks through structured phases with review checkpoints.

## Requirements

- [Claude Code](https://github.com/anthropics/claude-code)
- [Superpowers](https://github.com/anthropics/superpowers) plugin

## Installation

```bash
claude plugins add mguilarducci/vrau
```

## Usage

```
/vrau:start
```

That's it. One command handles everything.

## How It Works

### Starting a New Workflow

1. Run `/vrau:start`
2. Describe what you're working on
3. Choose branch setup (worktree, new branch, or current)
4. Vrau creates a workflow folder and guides you through phases

### Resuming an Existing Workflow

1. Run `/vrau:start`
2. Vrau detects existing workflows and shows options:
   - Resume a workflow
   - Start new
   - Delete old workflow

### The Three Phases

```
┌─────────────┐     ┌──────────┐     ┌─────────────┐
│  BRAINSTORM │ ──▶ │   PLAN   │ ──▶ │   EXECUTE   │
│   + Review  │     │ + Review │     │   + PR      │
└─────────────┘     └──────────┘     └─────────────┘
```

---

### Phase 1: Brainstorm

```
┌──────────────────────────────────────────────────────────────────┐
│  STEP 0: RESEARCH AVAILABLE TOOLS (haiku)                        │
│  Check MCP tools, fetch relevant docs, web search                │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  PRE-CHECKS (haiku)                                              │
│  Run tests, start dev server, establish baseline                 │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  STEP 1: BRAINSTORMING (opus)                                    │
│  Invoke superpowers:brainstorming skill                          │
│  → Clarifying questions → Design exploration → Document          │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  STEP 2: SAVE OUTPUT (sonnet)                                    │
│  Save to design/brainstorm.md, commit immediately                │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  STEP 3: EVALUATE SCOPE FOR BREAKDOWN (sonnet) ◄── MANDATORY     │
│  Always check: can this be split into smaller workflows?         │
│  Smaller, focused workflows preferred over large ones            │
└──────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
    ┌─────────────────┐             ┌─────────────────┐
    │  SPLIT NEEDED   │             │  NO SPLIT       │
    │  Create sub-    │             │  Continue to    │
    │  workflows,     │             │  Step 4         │
    │  restart vrau   │             │                 │
    └─────────────────┘             └─────────────────┘
              │                               │
              ▼                               ▼
        [STOP - restart              ┌──────────────────────────────┐
         for each sub-workflow]      │  STEP 4: SELF-REVIEW (sonnet)│
                                     │  Optional quality check      │
                                     └──────────────────────────────┘
                                                      │
                                                      ▼
                                     ┌──────────────────────────────┐
                                     │  STEP 5: FORMAL REVIEW       │
                                     │  (opus via reviewer agent)   │
                                     └──────────────────────────────┘
                                                      │
                              ┌───────────────────────┴──────────┐
                              │                                  │
                              ▼                                  ▼
                    ┌─────────────────┐                ┌─────────────────┐
                    │  APPROVED       │                │  NEEDS REVISION │
                    │  Continue to    │                │  Update, re-    │
                    │  Step 6         │                │  submit review  │
                    └─────────────────┘                └─────────────────┘
                              │                                  │
                              │                    ┌─────────────┘
                              │                    │ (max 3 iterations)
                              │                    ▼
                              │         ┌─────────────────────────────┐
                              │         │  Loop back to FORMAL REVIEW │
                              │         └─────────────────────────────┘
                              ▼
                    ┌──────────────────────────────────────────────────┐
                    │  STEP 6: OPEN PR AND MERGE (haiku)               │
                    │  Create PR, merge to main                        │
                    └──────────────────────────────────────────────────┘
                              │
                              ▼
                        [PHASE 2: PLAN]
```

---

### Phase 2: Plan

```
┌──────────────────────────────────────────────────────────────────┐
│  STEP 0: RESEARCH AVAILABLE TOOLS (haiku)                        │
│  Check MCP tools, fetch implementation patterns                  │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  PRE-PLAN SETUP (haiku)                                          │
│  Select design to plan, branch setup, update from main           │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  WRITE PLAN (opus)                                               │
│  Invoke superpowers:writing-plans skill                          │
│  → Task breakdown → Dependencies → Implementation steps          │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  REVIEW LOOP (opus via reviewer agent)                           │
│  Spawn vrau-reviewer, process feedback                           │
└──────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
    ┌─────────────────┐             ┌─────────────────┐
    │  APPROVED       │             │  REVISE/RETHINK │
    │  Continue to    │             │  Use receiving- │
    │  Phase 3        │             │  plan-review    │
    └─────────────────┘             │  skill, update  │
              │                     └─────────────────┘
              │                               │
              │                    ┌──────────┘
              │                    │ (max 3 iterations)
              │                    ▼
              │         ┌─────────────────────────────┐
              │         │  Loop back to REVIEW        │
              │         └─────────────────────────────┘
              ▼
        [PHASE 3: EXECUTE]
```

---

### Phase 3: Execute

```
┌──────────────────────────────────────────────────────────────────┐
│  PRE-EXECUTION CHECKS (sonnet)                                   │
│  Verify plan dependencies, ask permission to proceed             │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  EXECUTION (model per task from plan)                            │
│  Invoke vrau-executing-plans                                     │
│  → Parse dependency graph → Dispatch parallel groups             │
│  → Model per task from plan                                      │
│                                                                  │
│  Parallel execution:                                             │
│  • Tasks in same group run concurrently                          │
│  • Wait for group completion before next group                   │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  POST-EXECUTION (haiku)                                          │
│  Update README with execution log                                │
│  Open PR with gh pr create                                       │
└──────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                        [WORKFLOW COMPLETE]
```

## Workflow Files

All workflow artifacts are stored in `docs/designs/`:

```
docs/designs/2026-01-16-feat-user-auth/
├── README.md           # Workflow metadata
├── design/
│   └── brainstorm.md   # Phase 1 output
└── plan/
    └── plan.md         # Phase 2 output
```

## Enhanced Plan Format

Vrau plans include dependency tracking for parallel execution:

### Task Dependencies

Each task specifies what it depends on:

```markdown
### Task 3: Feature Implementation

**Depends on:** Task 1, Task 2
**Parallel group:** B
**Model:** sonnet
```

### Parallel Execution Groups

Tasks in the same parallel group run concurrently:

| Group | Tasks | Runs After |
|-------|-------|------------|
| A | 0, 1 | (start) |
| B | 2, 3, 4 | Group A |
| C | 5 | Group B |

### Model Per Task

Each task specifies which model to use:
- **haiku**: Simple setup, config
- **sonnet**: Standard implementation
- **opus**: Complex logic (requires approval)

## State Detection

Vrau knows where you are by checking what files exist:

| Files Present | State | What Happens |
|---------------|-------|--------------|
| README.md only | Not started | Begin brainstorm |
| + design/*.md | Brainstorm done | Begin planning |
| + plan/*.md | Plan done | Begin execution |
| Execution markers | Done | Show summary |

## Philosophy

**Autonomous execution, user-driven decisions.**

Vrau automates the tedious parts while keeping you in control of all key decisions. Each phase has a review checkpoint where you can adjust course.

## License

MIT
