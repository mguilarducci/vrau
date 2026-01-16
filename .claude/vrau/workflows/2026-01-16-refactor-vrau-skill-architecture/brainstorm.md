# Vrau Skill Architecture Refactoring - Brainstorm

## Goal

Refactor the vrau plugin to follow Claude Code skills/commands best practices:
- Extract shared patterns into reusable skills
- Improve discoverability with better trigger detection
- Optimize context usage with progressive disclosure

## Current State

- 2 skills (vrau-workflow: 63 lines, strategic-decomposition: 156 lines)
- 7 commands (start.md largest at 187 lines)
- 1 agent (vrau-reviewer: 103 lines)
- File-based state derivation

### Issues Identified

1. `commands/start.md` contains setup + all 3 phases inline (187 lines)
2. "Find Workflow" logic duplicated in 4 commands (~60 lines total)
3. Review loop pattern duplicated in multiple places
4. Basic trigger detection in vrau-workflow skill

## Design Decisions

### 1. New Skills

| Skill | Purpose |
|-------|---------|
| `find-workflow` | Discovers vrau workflows, handles selection UI, returns chosen workflow |
| `complexity-calibrated-review` | General-purpose YAGNI-aware review - adjusts depth based on task complexity |

### 2. Enhanced Existing Skill

`vrau-workflow` will be enhanced with:
- Better description keywords (CSO-optimized)
- Complexity estimation before suggesting vrau (score-based threshold)

### 3. Command Restructuring

Split `commands/start.md` into:
```
commands/
  start.md                    # Slimmed orchestrator (~80 lines)
  _phases/
    brainstorm.md             # Phase 1 details (~50 lines)
    plan.md                   # Phase 2 details (~50 lines)
    execute.md                # Phase 3 details (~40 lines)
```

### 4. Refactored Commands

Commands will reference skills instead of inline logic:
- `brainstorm.md`, `plan.md`, `execute.md`, `abort.md` → use `find-workflow` skill
- Review sections → reference `complexity-calibrated-review` skill

## Skill Specifications

### find-workflow

**Triggers:** When needing to find, list, or select an existing vrau workflow.

**Behavior:**
1. Scan `.claude/vrau/workflows/` for folders
2. If none → return "no workflows" message
3. If one → auto-select
4. If multiple → present list with state, ask user to choose
5. Return: { path, name, state, files[] }

**State derivation:**
| Files Present | State |
|---------------|-------|
| (empty) | not_started |
| brainstorm.md | brainstorm_complete |
| brainstorm.md + plan.md | plan_complete |
| + execution-log.md | executing/done |

### complexity-calibrated-review

**Triggers:** When reviewing any document where appropriate depth matters.

**Input:** Document + complexity level (trivial/simple/moderate/complex)

**Depth by complexity:**
| Complexity | Lenses | Bias |
|------------|--------|------|
| trivial | Completeness only | Approve unless broken |
| simple | + Obvious gaps | Approve with minor notes |
| moderate | + Security, Architecture, Testability | Balanced |
| complex | + Performance, Failure modes, Observability | Thorough |

**Output:** Summary, Critical/Important/Suggestions, Verdict (APPROVE/REVISE/RETHINK)

### vrau-workflow (enhanced)

**New description keywords:**
- Triggers: features, refactors, multi-file changes, bug investigations, architectural changes
- Anti-triggers: single-file edits, typo fixes, questions, quick changes

**Complexity estimation:**
| Signal | Points |
|--------|--------|
| Multiple files mentioned | +1 |
| Multiple components/systems | +1 |
| Requires investigation | +1 |
| "feature", "refactor", "redesign" | +1 |
| "quick", "small", "just" | -2 |
| Single file explicit | -1 |

Suggest vrau when score >= 2.

## File Changes Summary

### New Files
- `skills/find-workflow/SKILL.md` (~70 lines)
- `skills/complexity-calibrated-review/SKILL.md` (~110 lines)
- `commands/_phases/brainstorm.md` (~50 lines)
- `commands/_phases/plan.md` (~50 lines)
- `commands/_phases/execute.md` (~40 lines)

### Modified Files
- `skills/vrau/SKILL.md` - Enhanced triggers (~80 lines, was 63)
- `commands/start.md` - Slimmed orchestrator (~80 lines, was 187)
- `commands/brainstorm.md` - Use find-workflow (~35 lines, was 61)
- `commands/plan.md` - Use find-workflow (~35 lines, was 56)
- `commands/execute.md` - Use find-workflow (~45 lines, was 69)
- `commands/abort.md` - Use find-workflow (~50 lines, was 67)

### Unchanged Files
- `skills/strategic-decomposition/SKILL.md`
- `agents/vrau-reviewer.md`
- `commands/status.md`
- `commands/config.md`

## Out of Scope

- Changes to the vrau-reviewer agent
- Changes to strategic-decomposition skill
- New commands
- Plugin configuration changes
