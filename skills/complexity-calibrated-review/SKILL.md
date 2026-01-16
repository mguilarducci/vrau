---
name: complexity-calibrated-review
description: Use when reviewing documents (brainstorms, plans, designs, PRs) where review depth should match task complexity. Prevents over-engineering simple tasks while ensuring thorough review of complex ones.
---

# Complexity-Calibrated Review

## Overview

Adjust review depth based on task complexity. Over-engineering is a failure mode - demanding enterprise patterns for a simple script is as bad as missing security in a payment system.

## When to Use

- Reviewing brainstorm documents
- Reviewing implementation plans
- Reviewing designs or PRs
- Any review where appropriate depth matters

## Complexity Levels

Determine complexity BEFORE reviewing:

| Level | Examples | Signals |
|-------|----------|---------|
| trivial | poem script, single config | One file, no external deps |
| simple | CLI tool, small feature | Few files, clear scope |
| moderate | API endpoint, multi-file | Multiple components, some deps |
| complex | payment system, distributed | Security-critical, many integrations |

## Review Depth by Complexity

| Complexity | Lenses Applied | Bias |
|------------|----------------|------|
| **trivial** | Completeness only | APPROVE unless fundamentally broken |
| **simple** | + Obvious gaps | APPROVE with minor notes |
| **moderate** | + Security, Architecture, Testability, Maintainability | Balanced |
| **complex** | + Performance, Failure modes, Observability | Thorough |

## Output Format

```markdown
## Summary
[2-3 sentences: what this proposes, overall assessment]

## Critical Issues
[Blocking problems - for trivial tasks, almost always empty]

## Important Issues
[Should address - significant but not blocking]

## Suggestions
[Nice to have - proportional to complexity]

## Verdict
APPROVE | REVISE | RETHINK
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Demanding retry policies for a script | Check complexity level first |
| Rubber-stamping complex security code | Complex = thorough review |
| Same depth for all reviews | Always calibrate to complexity |
