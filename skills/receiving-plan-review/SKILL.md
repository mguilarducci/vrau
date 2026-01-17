---
name: receiving-plan-review
description: Use when receiving reviewer feedback on implementation plans. Requires technical verification of feasibility concerns, not blind agreement.
---

# Receiving Plan Review

## Overview

Handle reviewer feedback on implementation plans with technical rigor. Plans have different concerns than brainstorms: feasibility, ordering, dependencies, completeness.

**Plans are promises about what you will implement.** Review feedback questions whether those promises are keepable.

## When to Use

- After vrau-reviewer returns feedback on a plan
- When reviewer suggests REVISE or RETHINK
- When iterating on plan based on review
- Before making ANY changes to the plan based on reviewer feedback

## Core Principle

**Plans are promises. Review feedback questions if promises are keepable.**

## Research Before Responding

Before accepting or rejecting reviewer feedback on feasibility, verify claims:

1. **Check available MCP tools** for relevant documentation
2. **Use web search** to verify technical feasibility claims
3. **Fetch current docs** for libraries/APIs mentioned in feedback

Example: Reviewer says "This library doesn't support that feature" → Verify with current docs before accepting.

Unlike brainstorm reviews (which focus on design quality), plan reviews focus on:
- **Can we actually build this?** (feasibility)
- **In what order?** (dependencies)
- **Is anything missing?** (completeness)
- **Is scope realistic per task?** (sizing)
- **How do we verify it works?** (testing)

## Critical Process

For EACH reviewer point, follow this sequence:

1. **State the Concern**: Quote or paraphrase the reviewer's plan-specific concern
2. **Verify for THIS Plan**: Is this valid for OUR implementation? Check:
   - Does this dependency actually exist in our codebase?
   - Is this task genuinely too large or is it appropriately scoped?
   - Is this test gap real or is the reviewer over-testing?
   - Will this ordering actually cause problems?
3. **Decide with Reasoning**: Accept, partially accept, or disagree - WITH technical justification
4. **Act**: Update plan structure OR document why you're keeping it as-is

## Plan-Specific Concerns

Unlike brainstorm reviews, plan reviews focus on:

### 1. Task Ordering and Dependencies
- **Valid concern**: "Task 3 requires the database schema from Task 1, but Task 2 runs in parallel"
- **Invalid concern**: "These tasks should be in alphabetical order" (ordering preference, not dependency)

### 2. Missing Steps
- **Valid concern**: "You're deploying to production but there's no migration task"
- **Invalid concern**: "You should add a task to research best practices" (brainstorm-level, not implementation)

### 3. Unrealistic Scope Per Task
- **Valid concern**: "Task 2 says 'refactor entire auth system' - that's too large"
- **Invalid concern**: "Task 2 should be split into 5 sub-tasks" (micro-management without justification)

### 4. Test Coverage Gaps
- **Valid concern**: "You're changing the payment flow but have no test task"
- **Invalid concern**: "You need unit tests, integration tests, E2E tests for this trivial config change"

### 5. Integration Points
- **Valid concern**: "How do the frontend and backend tasks coordinate?"
- **Invalid concern**: "You should add a task to document the API" (if docs aren't required for implementation)

## Red Flags - You're Doing It Wrong

These patterns indicate you're NOT following the skill:

- **Accepting Scope Expansion**: Reviewer suggests "add monitoring setup" and you accept without questioning if monitoring is in scope
- **Adding Tasks "Just In Case"**: Adding defensive tasks without verifying necessity
- **Ignoring Dependency Concerns**: Reviewer points out ordering issues and you just agree without actually fixing the plan
- **Defensive About Task Ordering**: Rejecting dependency feedback without technical reasoning
- **Not Verifying Test Needs**: Accepting all test suggestions without checking if they're appropriate for complexity
- **Batch Processing**: "I've updated the plan to address all concerns" without explaining WHY each change improves feasibility
- **Confusing Design Review**: Accepting brainstorm-level feedback (architecture changes) when you're reviewing an implementation plan

## Scope Discipline

Plans are promises. Be disciplined about scope:

| Reviewer Suggestion | Question to Ask |
|---------------------|-----------------|
| "Add task X" | Is X required to complete the stated goal? |
| "Split task Y into 3" | Will splitting actually reduce risk or just add overhead? |
| "Add comprehensive tests" | Does the complexity level justify this testing investment? |
| "Add documentation task" | Is documentation a deliverable or nice-to-have? |

**Push back on scope creep.** "Adding monitoring setup would expand scope beyond the original goal. If monitoring is critical, it should be a separate workflow."

## Response Framework

Structure your response:

```markdown
## Plan Review Response

### Accepted Changes

#### Issue: [Reviewer's concern about feasibility/ordering/completeness]
**Assessment**: [Is this technically valid for OUR plan?]
**Reasoning**: [WHY this change improves the plan]
**Action**: [Specific plan update - added task, reordered, modified scope]

[Repeat for each accepted issue]

### Partial Acceptance

#### Issue: [Reviewer's concern]
**Assessment**: [Valid part vs. over-engineering part]
**Reasoning**: [WHY we're taking a middle path]
**Action**: [Compromise solution]

### Respectful Disagreement

#### Issue: [Reviewer's concern]
**Assessment**: [Why this doesn't apply to our context]
**Reasoning**: [Technical justification for keeping plan as-is]
**Action**: No change

## Updated Plan Summary

- Tasks added: N (list each with brief description)
- Tasks modified: N (list each with what changed)
- Tasks removed: N (list each with why removed)
- Task order changed: Y/N (if yes, explain dependency fix)
- Scope change: [Expanded/Reduced/Same] (justify if expanded)
```

## Common Reviewer Patterns to Verify

Watch for these patterns and verify each:

1. **Scope Creep**: "You should also add..." (is "also" expanding the goal?)
2. **Over-Testing**: "Add comprehensive test coverage" (is it appropriate for complexity?)
3. **Micro-Management**: "Split this into 10 tiny tasks" (does granularity help or hurt?)
4. **Premature Optimization**: "Add caching before implementing" (is this premature?)
5. **Missing Context**: "This task is too large" (compared to what? do they understand the actual work?)

## Valid Reasons to Accept

- Reviewer identified a real dependency you missed
- Task ordering will actually cause build failures
- Scope is genuinely unrealistic for a single task
- Missing critical integration or test step
- Suggestion improves plan feasibility without expanding scope

## Valid Reasons to Disagree

- Reviewer is expanding scope beyond stated goal
- Suggestion adds overhead without reducing risk
- Reviewer doesn't understand the codebase constraints
- Over-testing for task complexity
- Breaking tasks down too far (more overhead than benefit)

## Examples

### Good: Dependency Verification

```
#### Issue: Reviewer says "Task 3 (API integration) depends on Task 5 (auth setup)"
**Assessment**: Checked the codebase - API integration code DOES require auth middleware
from Task 5. This is a real dependency we missed.
**Reasoning**: Running Task 3 before Task 5 will fail because auth middleware won't exist.
**Action**: Reordered plan - Task 5 moves before Task 3. Updated Task 3 dependencies.
```

### Bad: Blind Acceptance

```
"Great catch! I've reordered the tasks as you suggested."
[No verification if the dependency actually exists in the codebase]
```

### Good: Scope Discipline

```
#### Issue: Reviewer suggests "Add task for setting up monitoring dashboard"
**Assessment**: The original goal is "Add user authentication". Monitoring is not mentioned
in the brainstorm or requirements.
**Reasoning**: While monitoring is valuable, adding it here expands scope beyond the stated goal.
This should be a separate workflow if needed.
**Action**: Respectfully disagree. Keeping plan focused on auth implementation only.
```

### Bad: Scope Creep

```
"Good point! I've added tasks for monitoring setup, alerting configuration, and dashboard creation."
[Accepted scope expansion without questioning if it's part of the goal]
```

### Good: Test Calibration

```
#### Issue: Reviewer wants "comprehensive test coverage including unit, integration, and E2E"
**Assessment**: The task is changing a simple config flag (complexity: trivial). Current plan
has one smoke test to verify the flag works.
**Reasoning**: For a trivial config change, one verification test is appropriate. Comprehensive
testing would be over-investment.
**Action**: Partial acceptance - keeping the smoke test, adding one integration test to verify
the flag affects the right systems. Not adding E2E tests for a config flag.
```

## Verification Checklist

Before responding to review, check:

- [ ] Have I verified EACH point is valid for OUR plan in OUR codebase?
- [ ] Have I checked for scope creep in suggested additions?
- [ ] Have I verified dependency concerns are real (not preferences)?
- [ ] Have I calibrated test suggestions to task complexity?
- [ ] Have I provided technical reasoning for ALL decisions?
- [ ] Am I making changes because they improve FEASIBILITY, not to appease?
- [ ] Have I respectfully pushed back on over-engineering?

## Output Requirements

Your response MUST include:

1. **Per-Issue Analysis**: For each reviewer point, explicit verification and decision
2. **Technical Reasoning**: WHY you're accepting, partially accepting, or disagreeing
3. **Specific Plan Changes**: Exact tasks added/modified/removed/reordered (not vague "updated")
4. **Scope Tracking**: Clear statement if scope expanded, reduced, or stayed the same
5. **Documented Disagreements**: Where you disagree, clear explanation with technical justification

Do NOT just say "I've updated the plan based on your feedback." Show your work and justify why each change improves the plan's feasibility.

## Key Difference from Brainstorm Review

| Aspect | Brainstorm Review | Plan Review |
|--------|-------------------|-------------|
| **Focus** | Is the DESIGN sound? | Can we IMPLEMENT it? |
| **Questions** | Does this solve the problem well? | Can we build this? In what order? |
| **Scope** | Design-level concerns | Implementation-level concerns |
| **Ordering** | Doesn't matter | Critical (dependencies) |
| **Valid Feedback** | Architecture, approach, edge cases | Task ordering, missing steps, scope sizing |
| **Invalid Feedback** | "Over-engineered for complexity" | "You should redesign the architecture" |

When receiving plan review, focus on **feasibility and execution**, not design quality (that was the brainstorm phase).
