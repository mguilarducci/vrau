# Vrau Workflow v2 Design

**Task:** Redesign vrau workflow with two-phase architecture (Design + Plan), issue tracking integration, model selection per step, and improved commit/push discipline

**Branch:** refactor/vrau-skill-architecture
**Date:** 2026-01-16
**Doc Approach:** File-based (option A)

## User Choices

| Setting | Value |
|---------|-------|
| Doc location | `docs/designs/{date}-{branch}/` |
| Issue platform | GitHub Issues (gh CLI) |
| Model overrides | Locked at start, user can ask to change mid-flow |
| No-commit mode | Both README marker + .local file |
| Sub-design review | Ask user per breakdown |

## Model Configuration (Default)

| Step | Model | Rationale |
|------|-------|-----------|
| Branch setup | haiku | Simple git operations |
| Step-by-step instructions | haiku | User interaction only |
| Brainstorm verifications | haiku | Running tests/dev |
| Brainstorm itself | opus | Creative, exploratory |
| Brainstorm review | opus | Fresh eyes, same depth |
| Brainstorm breakdown | sonnet | Analytical task |
| Pre-write plan | haiku | Workflow selection |
| Write plan | opus | Complex planning |
| Plan review | opus | Same depth as planning |
| Execution simple tasks | haiku | Straightforward code |
| Execution complex tasks | sonnet | Complex code |
| Execution very complex | opus (ask first) | Major decisions |
| Quality/review | sonnet minimum | Never haiku for review |

## Phase Overview

### Phase 1: Brainstorm (Design)
0. Branch setup (haiku)
1. Step-by-step instructions - doc approach choice (haiku)
2. Brainstorm with superpowers:brainstorming (opus)
3. Brainstorm review loop (opus)
4. Brainstorm breakdown attempt (sonnet)
5. Open PR and merge

### Phase 2: Plan (Implementation)
5. Pre-write plan - select design, branch setup (haiku)
6. Write plan with superpowers:write-plan (opus)
7. Plan review loop (opus)
8. Execution with superpowers:subagent-development-driven (varied)

## Document Structure

```
docs/designs/{date}-{branch-name-dashes}/
  README.md              # Full context, choices, links
  design/
    {file}.md            # Main design doc (option A)
    00-design-xxx.md     # Broken-down designs (if applicable)
  plan/
    {file}.md            # Implementation plans
```

## Links

- Design files: (to be created)
- Plan files: (to be created)
- GitHub Issues: (if option B selected)
