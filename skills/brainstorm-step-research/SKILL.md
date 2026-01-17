---
name: brainstorm-step-research
description: Use when executing research step in brainstorm phase - enforces haiku model usage
---

# Brainstorm Step: Research Available Tools

**This skill enforces haiku model usage. You cannot skip this check.**

## Model Enforcement Check

**FIRST: Check your current session model.**

Are you running on **haiku** model right now?

### If YES (you are haiku):

Execute the research step directly in this session. Skip to "Research Instructions" below.

### If NO (you are sonnet or opus):

**You MUST dispatch a Task tool with haiku model.** Do NOT execute in current session.

```
Task tool:
- subagent_type: "general-purpose"
- model: "haiku"
- description: "Research available tools"
- prompt: "You are in the brainstorm phase of a vrau workflow.

TASK: Research available tools before brainstorming.

INSTRUCTIONS:
1. List available MCP tools (check for documentation fetchers like context7, claude-docs, web search, API explorers)
2. Identify which tools are relevant for this task
3. Use relevant tools to fetch current documentation, search for patterns, gather context
4. Output: Brief summary of what was researched and key findings

Include this research context when the brainstorm step runs."
```

**STOP HERE.** Wait for the task to complete, then continue to next step in brainstorm phase.

---

## Research Instructions

**(Only execute if you are haiku model)**

### 1. List Available MCP Tools

Check for:
- Documentation fetchers (context7, claude-docs, etc.)
- Web search capabilities
- API explorers or code search tools

### 2. Identify Relevant Tools

Ask yourself:
- Does the task mention specific technologies? (React, Claude API, etc.)
- Are there docs that should be fetched?
- Would web search help find current best practices?

### 3. Use Relevant Tools

- Fetch current documentation for mentioned technologies
- Search for recent changes or deprecations
- Gather context before brainstorming begins

### 4. Output Summary

Provide a brief summary of:
- What was researched
- Key findings
- Any relevant documentation or patterns found

**This context will be passed to the brainstorming step.**
