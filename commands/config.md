---
description: "View or edit vrau configuration"
---

# Vrau Config

## Load Configuration

1. Read `.claude/vrau/config.json` (team defaults, committed)
2. Read `.claude/vrau/config.local.json` (personal overrides, gitignored)
3. Merge: local overrides team defaults

## Display Current Config

```
Vrau Configuration:

  Reviewer lenses:
    - security
    - architecture
    - testability
    - maintainability

  Paths:
    workflows: .claude/vrau/workflows

Options:
1. Edit reviewer lenses
2. Reset to defaults
3. Done
```

## Edit Reviewer Lenses

```
Current lenses: security, architecture, testability, maintainability

Available lenses:
- security (vulnerabilities, auth, data exposure)
- architecture (patterns, coupling, scalability)
- testability (test coverage, mocking, isolation)
- maintainability (readability, complexity, documentation)
- performance (efficiency, caching, optimization)
- accessibility (a11y compliance, usability)

Enter lenses (comma-separated) or 'reset' for defaults:
```

Save changes to `.claude/vrau/config.local.json`.

## Default Configuration

```json
{
  "paths": {
    "workflows": ".claude/vrau/workflows"
  },
  "reviewer": {
    "lenses": ["security", "architecture", "testability", "maintainability"]
  }
}
```
