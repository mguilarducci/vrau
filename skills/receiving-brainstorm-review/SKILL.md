---
name: receiving-brainstorm-review
description: Use when receiving reviewer feedback on brainstorm/design documents. Requires technical rigor and verification, not blind agreement or defensive rejection.
---

# Receiving Brainstorm Review

## Overview

Handle reviewer feedback on brainstorm documents with intellectual rigor. Neither blindly accept nor defensively reject - evaluate each point on technical merit.

## When to Use

- After vrau-reviewer returns feedback on a brainstorm
- When reviewer suggests REVISE or RETHINK
- When iterating on design based on review
- Before making ANY changes based on reviewer feedback

## Core Principle

**Reviewer feedback is data, not commands.**

Evaluate each point on technical merit. The reviewer may be:
- Correct (accept)
- Partially correct (accept the valid parts)
- Incorrect for this context (respectfully disagree)
- Over-engineering (calibrate to complexity)

## Critical Process

For EACH reviewer point, follow this sequence:

1. **State the Concern**: Quote or paraphrase the reviewer's point clearly
2. **Verify Technically**: Is this valid in THIS context? Check:
   - Does this problem actually exist here?
   - Is the suggested solution appropriate for this complexity level?
   - Are there trade-offs the reviewer missed?
3. **Decide with Reasoning**: Accept, partially accept, or disagree - WITH technical justification
4. **Act**: Make changes OR document why you're not making them

## Red Flags - You're Doing It Wrong

These patterns indicate you're NOT following the skill:

- **Reflexive Agreement**: Saying "great point" or "you're right" BEFORE verification
- **Blind Acceptance**: Making all suggested changes without questioning fit
- **Performative Compliance**: Changing design just to appease reviewer
- **Missing Verification**: Not checking if concern applies to this specific context
- **Ignoring Complexity**: Accepting over-engineering suggestions for simple tasks
- **Defensive Rejection**: Dismissing feedback without technical reasoning
- **Batch Processing**: "I've addressed all your concerns" without explaining WHY each change makes sense

## Complexity Calibration

Always consider task complexity when evaluating feedback:

| Task Complexity | Appropriate Review Depth | Red Flag |
|-----------------|-------------------------|----------|
| **trivial** | Does it work? | Reviewer demanding extensive error handling, monitoring, retry logic |
| **simple** | + obvious gaps | Reviewer suggesting enterprise patterns for a CLI tool |
| **moderate** | + architecture, testability | Reviewer not considering security implications |
| **complex** | + security, failure modes, observability | Reviewer saying "LGTM" without thorough review |

**Push back when reviewer is over-engineering.** "For a simple script that runs once, adding retry logic and circuit breakers would be over-engineering."

## Response Framework

Structure your response:

```markdown
## Review Response

### Issue: [Reviewer's concern]
**Assessment**: [Is this technically valid in our context?]
**Decision**: [Accept/Partial/Disagree]
**Reasoning**: [WHY - technical justification]
**Action**: [What we're doing about it]

[Repeat for each issue]

## Changes Made

[List of actual changes to the brainstorm, if any]

## Disagreements Documented

[Issues where we respectfully disagree, with reasoning]
```

## Common Reviewer Patterns to Verify

Watch for these reviewer patterns and verify each:

1. **Over-Engineering**: "You should add circuit breakers" (for a simple script?)
2. **Missing Context**: "This won't scale" (does it need to?)
3. **False Equivalence**: "Other projects do X" (are they the same complexity?)
4. **Premature Optimization**: "This could be slow" (have we profiled? is it hot path?)
5. **Cargo Culting**: "Best practice is Y" (is it best practice FOR THIS case?)

## Valid Reasons to Accept

- Reviewer identified a real technical gap
- Suggestion improves the design without adding unnecessary complexity
- You missed a security/correctness issue
- Suggestion aligns with task complexity level

## Valid Reasons to Disagree

- Reviewer is over-engineering for task complexity
- Suggestion adds complexity without proportional benefit
- Reviewer missed important context about constraints
- Alternative approach is technically superior for this use case

## Examples

### Good: Technical Verification

```
### Issue: Reviewer suggests adding retry logic for API calls
**Assessment**: The brainstorm describes a one-time setup script that calls a local API.
If the local API is down, retry won't help - the user needs to fix the service first.
**Decision**: Disagree
**Reasoning**: For a local setup script, immediate failure is better UX than hanging with
retries. The user can re-run the script after fixing the service.
**Action**: No change. Adding note in brainstorm about failure modes.
```

### Bad: Blind Agreement

```
"Great point about retry logic! I've added comprehensive retry with exponential backoff
and circuit breakers to the design."
[No verification if this is appropriate for the task complexity]
```

### Good: Partial Agreement

```
### Issue: Reviewer wants extensive error handling throughout
**Assessment**: Some error handling is needed (file I/O can fail), but comprehensive
error recovery would over-engineer a simple migration script.
**Decision**: Partial acceptance
**Reasoning**: We should handle file errors gracefully, but the reviewer's suggestion
of transaction rollback is overkill for a one-time migration.
**Action**: Adding basic error handling for file operations. Not adding full transaction system.
```

## Verification Checklist

Before responding to review, check:

- [ ] Have I verified EACH point technically before accepting?
- [ ] Have I considered task complexity for each suggestion?
- [ ] Have I provided technical reasoning for decisions?
- [ ] Am I making changes because they improve the design, not to appease?
- [ ] Have I respectfully disagreed where appropriate?
- [ ] Have I avoided reflexive "great point" responses?

## Output Requirements

Your response MUST include:

1. **Per-Issue Analysis**: For each reviewer point, explicit verification and decision
2. **Technical Reasoning**: WHY you're accepting, partially accepting, or disagreeing
3. **Actual Changes**: Specific changes made to brainstorm (not vague "addressed")
4. **Documented Disagreements**: Where you disagree, clear explanation why

Do NOT just say "I've updated the brainstorm to address your feedback." Show your work.
