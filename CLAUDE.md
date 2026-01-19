# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vrau is a Claude Code plugin that orchestrates complex development workflows through three phases: Brainstorm → Plan → Execute. Each phase has PR-based review checkpoints.

**Requires:** Claude Code + Superpowers plugin

## Architecture

```
vrau/
├── skills/           # Phase-specific workflow logic (SKILL.md files with YAML frontmatter)
│   ├── vrau/         # Entry point - routes to correct phase
│   ├── brainstorm/   # Phase 1: Design exploration
│   ├── plan/         # Phase 2: Task breakdown
│   ├── execute/      # Phase 3: Implementation
│   └── vrau-reviewer/ # Fresh-eyes review agent
├── commands/         # Atomic git/GitHub operations (markdown with YAML frontmatter)
├── agents/           # Subagent configurations
├── .claude-plugin/   # Plugin metadata (plugin.json, marketplace.json)
└── docs/designs/     # Workflow artifacts (created at runtime)
```

### Skills vs Commands

- **Skills** (`skills/<name>/SKILL.md`): Multi-step workflows with decision logic. Frontmatter: `name`, `description`, `model`
- **Commands** (`commands/<name>.md`): Single atomic operations. Frontmatter: `description`, `model`

### Tracking Modes

Two tracking modes for workflow state:
- **GitHub Issue**: Status tracked via issue comments, PRs for reviews
- **None**: File-based state detection only

### State Detection

Determined by files in `docs/designs/<workflow>/`:
- `README.md` only → brainstorm phase
- `design/*.md` present → plan phase
- `plan/*.md` present → execute phase

## Critical Rules

**NEVER COMMIT TO MAIN BRANCH** - All phases require feature branches or worktrees. Check branch before any commit:
```bash
git branch --show-current
```

## Version Bumping

Update version in `.claude-plugin/plugin.json` when making releases.
