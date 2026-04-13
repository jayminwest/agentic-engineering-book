---
title: Tool Use
description: How agents use tools to affect the world
created: 2025-12-08
last_updated: 2025-12-10
tags: [foundations, tools, function-calling, actions, tool-use]
part: 1
part_title: Foundations
chapter: 5
section: 0
order: 1.5.0
---

# Tool Use

Tools are how agents affect the world. Agents without tools are chatbots. Agents with tools can affect the world.

---

## Chapter Overview

This chapter explores how agents interact with tools—from design principles to scaling patterns. Tool use is where the agent's reasoning meets the real world.

### What You'll Learn

1. **[Tool Design](1-tool-design.md)**: Principles for creating well-designed tools that agents can use effectively
2. **[Tool Selection and Routing](2-tool-selection.md)**: How agents decide which tool to use and what causes selection failures
3. **[Tool Restrictions and Security](3-tool-restrictions.md)**: Using tool access controls as security boundaries
4. **[Scaling Tool Use](4-scaling-tools.md)**: Patterns for managing large numbers of tools without overwhelming context
5. **[Skills and Meta-Tools](5-skills-and-meta-tools.md)**: How skills blur the boundary between tools and prompts

---

## Your Mental Model

**Tools extend the agent's capabilities beyond pure reasoning.** Think of them as:
- **Hands**: Writing/action tools that modify the world
- **Senses**: Reading/observation tools that gather information
- **Skills**: Temporary behavioral modifications that change how the agent reasons

The boundary between what the model does (reasoning) and what tools do (actions) defines the agent's architecture.

---

## Key Insights

### Communication and Coordination

*[2025-12-08]*: For multi-agent systems, providing CRUD operations on a shared communications database via tool calling enables effective collaboration. Agents can read what others have written, post updates, and query for relevant context. This is particularly useful for faster-moving, more flexible structures than what GitHub's issue/PR system provides—think of it as giving agents their own Slack or shared scratchpad.

---

## Connections

- **To [Prompt](../2-prompt/_index.md):** Tool descriptions are themselves prompts—see [Model-Invoked vs. User-Invoked Prompts](../2-prompt/_index.md#model-invoked-vs-user-invoked-prompts)
- **To [Model](../3-model/_index.md):** Which models are best at tool selection and use?
- **To [Context](../4-context/_index.md):** How do tool results become context for next steps? See [Progressive Disclosure](../4-context/3-context-patterns.md#progressive-disclosure-pattern)
- **To [Cost and Latency](../8-practices/3-cost-and-latency.md):** Token cost models differ by feature type—tools vs. Skills vs. subagents vs. MCP
- **To [Google ADK](../10-practitioner-toolkit/2-google-adk.md):** MCP deployment patterns and tool filtering at runtime

