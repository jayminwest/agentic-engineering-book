---
title: Context
description: Managing the information available to an agent at any given moment
created: 2025-12-08
last_updated: 2025-12-10
tags: [foundations, context, memory, retrieval]
part: 1
part_title: Foundations
chapter: 4
section: 0
order: 1.4.0
---

# Context

Context is everything the agent is aware of at any given time. More concretely, it's all the information currently in a model's context window, plus the space remaining.

Context management sits at the heart of agentic engineering. It determines what the agent knows, how capable it is, and whether it can complete its task. Poor context management leads to context rot, capability degradation, and failed tasks. Effective context management enables agents to work efficiently within their limits.

**Example contrast:**

```
# Poor context management (80% filled)
- 15k tokens of verbose logging output
- 10k tokens of redundant tool definitions
- 8k tokens of tangential file reads
[Request] Implement user authentication
→ Agent struggles, outputs low-quality code

# Effective context management (30% filled)
- 2k tokens of authentication best practices
- 5k tokens of relevant security code examples
- 3k tokens of project structure
[Request] Implement user authentication
→ Agent succeeds, produces production-ready code
```

The difference: deliberate context composition focusing on signal over volume.

---

## Chapter Overview

This chapter builds from fundamental principles through advanced multi-agent patterns:

### [1. Context Fundamentals](1-context-fundamentals.md)

Core principles of context as agent working memory. Covers the capability capacity model (context fill = capability drain), the difference between context and memory, and the "one agent, one task" principle. Establishes the mental model: context is ephemeral, finite, and the single biggest determinant of output quality.

**Key concepts:**
- Context vs. memory
- Capability capacity model
- One agent, one prompt, one task
- Quality over quantity in context

### [2. Context Management Strategies](2-context-strategies.md)

Practical techniques for handling context limits, compression, and the injection vs. retrieval balance. Introduces frequent intentional compaction—proactive compression at 40-60% capacity to maintain quality, not salvage it. Contrasts with emergency compaction at 95%.

**Key concepts:**
- Context window limits and capability degradation
- When to boot fresh vs. compact
- Injection for priming, retrieval for discovery
- Frequent intentional compaction pattern
- Structured context (markdown, JSON, XML)

### [3. Advanced Context Patterns](3-context-patterns.md)

Sophisticated patterns for complex scenarios. Progressive disclosure enables unlimited expertise within fixed context budgets. Context loading treats context as curated payloads, not accumulated logs. The ACE framework challenges assumptions—contexts should grow with learned knowledge in knowledge-intensive domains.

**Key concepts:**
- Progressive disclosure (metadata → content → resources)
- Context loading vs. accumulation
- Agentic Context Engineering (ACE)
- Growing playbooks vs. compressed prompts
- Generator/Reflector/Curator architecture

### [4. Multi-Agent Context](4-multi-agent-context.md)

How context management changes in multi-agent systems. Context isolation keeps orchestrator context clean by having subagents return synthesized summaries, not raw data. Persistent state vs. ephemeral context—the critical distinction for cross-session continuity.

**Key concepts:**
- Multi-agent context isolation
- Orchestrator context cleanliness
- Persistent state vs. ephemeral context
- State scoping (session, user, app, temp)
- Cross-session continuity

### [5. Context Management Architectures](5-context-management-architectures.md)

Architectural approaches to active context management. Surveys three points on the design spectrum: passive accumulation (the default), lossless preservation (LCM), and continuous curation (Sapling). Comparative analysis of trade-offs, with practitioner guidance on when to use each approach.

**Key concepts:**
- Passive accumulation and its failure modes
- Lossless Context Management (LCM) — immutable store, hierarchical DAG, operator-level recursion
- Continuous curation (Sapling) — five-stage inter-turn pipeline, operation model
- Design spectrum and composability

---

## When to Read What

**New to context management?** Start with [Context Fundamentals](1-context-fundamentals.md) to understand the core mental model.

**Hitting context limits?** See [Context Management Strategies](2-context-strategies.md) for practical techniques.

**Building sophisticated systems?** Explore [Advanced Context Patterns](3-context-patterns.md) for progressive disclosure, context loading, and ACE.

**Working with multiple agents?** Read [Multi-Agent Context](4-multi-agent-context.md) for isolation patterns and persistent state.

**Evaluating context management systems?** See [Context Management Architectures](5-context-management-architectures.md) for architectural options beyond the passive default.

---

## Connections

- **To [Prompt](../2-prompt/_index.md):** Context determines how agents respond to prompts. Prompts are context, but not all context is prompts.
- **To [Model](../3-model/_index.md):** Context limits vary by model. Small models work in orchestrator patterns via context loading.
- **To [Tool Use](../5-tool-use/_index.md):** Tools modify context—reading files, searching codebases. Progressive disclosure via tool metadata. See [MCP Tool Declarations](../5-tool-use/3-tool-restrictions.md#mcp-tool-declarations-in-frontmatter) and [Scaling Tool Use](../5-tool-use/4-scaling-tools.md).
- **To [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md):** Context isolation via sub-agents. Emergency Context Rewriting anti-pattern shows why reactive compaction fails.
- **To [Claude Code](../10-practitioner-toolkit/1-claude-code.md):** Skills implement progressive disclosure in production.
- **To [Google ADK](../10-practitioner-toolkit/2-google-adk.md):** State prefixes demonstrate persistent state scoping.
