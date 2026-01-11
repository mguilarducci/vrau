---
description: "Continue vrau workflow - auto-detects current state and takes appropriate action"
---

# Vrau Workflow

Read the current state from `.claude/vrau/state.local.json`.

## State Routing

Based on the `state` field:

| State | Action |
|-------|--------|
| File not found OR `idle` | Tell user: "No active workflow. Use `/vrau:start` to begin a new workflow." |
| `brainstorming` | Continue with `superpowers:brainstorming` skill. When complete, transition to `brainstorm_review`. |
| `brainstorm_review` | Present review options (see Review Options below) |
| `planning` | Continue with `superpowers:writing-plans` skill. When complete, transition to `plan_review`. |
| `plan_review` | Present review options (see Review Options below) |
| `executing` | Show execution progress from `execution-log.local.md`. Ask if user wants to continue execution. |
| `done` | Present cleanup options (Keep / Archive / Delete) |

## Review Options

Present these options:

```
How would you like to review?

1. Guided checklist
   → Quick self-review with key questions
   → You check items, then approve/revise

2. Subagent review
   → Fresh-eyes analysis by reviewer agent
   → Returns Critical/Important/Minor feedback

3. Both in parallel
   → Checklist shown now
   → Subagent runs in background
   → Combined feedback for decision

4. Skip review
   → Proceed directly to next phase
```

After review choice is made:
- If `brainstorm_review`: Update state to `planning`, invoke `superpowers:writing-plans`
- If `plan_review`: Update state to `executing`, present execution options

## Execution Options

After plan is approved:

```
Plan approved! Ready to execute.

How do you want to run the plan?

1. Subagent-driven (this session)
   → Uses superpowers:subagent-driven-development
   → Fresh subagent per task
   → Autonomous execution

2. Parallel session (separate terminal)
   → Uses superpowers:executing-plans
   → Open new terminal in worktree/branch
   → Return here when done
```

## State File Format

```json
{
  "branch": "fix/payment-api-timeout",
  "workflow": "2026-01-11-fix-payment-api-timeout",
  "state": "executing",
  "workspaceType": "new_branch",
  "startedAt": "2026-01-11T10:30:00Z"
}
```
