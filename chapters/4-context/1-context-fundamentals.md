---
title: Context Fundamentals
description: Core principles of context as agent working memory and capability capacity
created: 2025-12-10
last_updated: 2026-02-05
tags: [foundations, context, memory, capability-capacity]
part: 1
part_title: Foundations
chapter: 4
section: 1
order: 1.4.1
---

# Context Fundamentals

Context is everything the agent is aware of at any given time. More concretely, it's all the information currently in a model's context window, plus the space remaining.

---

## Context vs Memory

Memory in current LLMs doesn't exist in any meaningful way by default. Each fresh agent instance starts from a blank slate with no awareness of previous agents' work. Throughout an agent's session, its context window fills with accumulated information: user messages, tool calls and responses, read files, and other encountered data.

The distinction matters for architecture. Context is ephemeral working memory that dies with the session. Memory would be persistent knowledge that survives restarts—but without external storage mechanisms, agents don't have this by default. See [Multi-Agent Context: Persistent State vs. Ephemeral Context](4-multi-agent-context.md#persistent-state-vs-ephemeral-context) for the full distinction.

*[2026-02-05]*: Claude Code's session memory feature (v2.1.32+) provides automatic persistent memory across sessions through background summarization. This external storage mechanism enables agents to recall context from previous sessions without manual persistence. See [Claude Code: Memory Management](../10-practitioner-toolkit/1-claude-code.md#memory-management) for implementation details.

---

## Context and Prompts

Context determines how an agent responds to prompts. The prompt triggers agent action, but the full context window shapes the response. All prompts become part of context, but not all context originates from prompts—tool outputs, read files, and intermediate reasoning also occupy the context window.

This relationship creates a foundational principle: prompts are inputs, context is state. Poor context management degrades prompt effectiveness regardless of prompt quality. For prompting techniques that account for context dynamics, see [Prompt](../2-prompt/_index.md).

---

## The Pit of Success Principle

Determining what information an agent needs remains situational and difficult to formalize. However, the "Pit of Success" principle provides a decision framework: Does the agent's current context increase the likelihood that the next generated token will be correct and aligned with the task?

This frames context management as capability engineering. High-signal context (relevant code, clear specifications, domain knowledge) shifts probability distributions toward correct outputs. Low-signal context (tangential files, logging output, redundant tool definitions) adds noise without improving token prediction quality.

---

## The Capability Capacity Model

### Context Fill Correlates with Capability Drain

*[2025-12-10]*: Observable patterns suggest context utilization above 40% correlates with early signs of capability degradation, though the precise relationship between context usage and capability remains uncharacterized.

The relationship may not be linear—there could be capability cliffs at certain thresholds. Without rigorous measurement, conservative practice treats 40% utilization as a caution point rather than a hard limit. Production deployments observing quality degradation typically find agents operating above 60% capacity.

### Context Type Matters More Than Context Volume

Agent context essentially fine-tunes the model dynamically. Filling 30% of context with logging output or verbose tool definitions produces different capability than filling 30% with well-structured, high-quality code examples.

**Example contrast:**

```
# Poor context composition (30% filled)
[15,000 tokens of JSON logging output]
[5,000 tokens of redundant tool definitions]
[Request] Refactor the authentication module

→ Model has seen mostly noise, predictions weighted toward log patterns
```

```
# High-quality context composition (30% filled)
[15,000 tokens of exemplary authentication code]
[5,000 tokens of security best practices]
[Request] Refactor the authentication module

→ Model has seen relevant patterns, predictions weighted toward quality code
```

The capability difference stems from what fills the context, not just how much. Context composition shapes the probability distribution for next-token prediction.

### Measuring Capability Degradation

Detecting "lackluster results" requires active monitoring, which doesn't scale well. Practical approaches include:

- **Spot-check alignment**: Compare intended outcome vs. actual output
- **Conversational validation**: Short verification exchanges catch drift early
- **Output quality metrics**: Track code complexity, test coverage, error rates
- **Context utilization tracking**: Log when agents cross 40%, 60%, 80% thresholds

This typically works better in conversational workflows than one-shot tasks. Large batch operations lack the feedback loops needed for early detection.

---

## One Agent, One Task

### Task Boundary Definition

The upper bound for "one task" remains poorly defined. Current practice suggests prompts of 1,000-2,000 lines work reliably, with potential extension to 5,000+ lines depending on task structure and context availability.

Task boundaries depend more on coherence than size. A well-scoped refactoring across 10 files may succeed where a vague "improve the codebase" request fails, regardless of token count.

### Handling Mid-Execution Scope Growth

When a task proves larger than initially scoped, recommendations include:

1. **Kill and rescope** — Stop the current agent, clear its work, break the original task into smaller focused parts
2. **Evaluate salvage cost** — Restarting is almost always cheaper than using additional agents to rescue degraded work
3. **Learn from the failure** — Adjust scoping heuristics for future similar tasks

The bias toward restarting reflects context economics: fresh agents start with clean context budgets, while salvage operations compound existing context bloat.

### Task Type Variations

This principle varies by task type. The focus here remains coding-specific workflows. Other domains (creative writing, research synthesis, conversational support) may exhibit different task boundary patterns.

---

## Your Mental Model

Context is the agent's working memory—ephemeral, finite, and the single biggest determinant of output quality.

**Core beliefs:**

1. **Context is capability.** An agent with 20% context used has more room to think than one at 80%. Treat context like a budget, not a dumping ground.

2. **Quality over quantity.** What's in the context matters more than how much is in it. High-signal context (relevant code, clear specs) outperforms high-volume context (logs, tangential files) even at the same fill level.

3. **Fresh starts are cheap.** When in doubt, boot a new agent. Trying to salvage a degraded context is almost never worth it. The cost of restarting is lower than the cost of fighting context rot.

4. **Injection for priming, retrieval for discovery.** Inject what the agent *must* know upfront. Let it retrieve what it *might* need. The balance shifts based on codebase size and task specificity.

5. **One agent, one task.** Scope tasks to fit comfortably within context limits. If a task balloons, kill and rescope rather than push through.

**Open questions:**

- Where exactly is the capability cliff? Is 80% the danger zone, or does it vary by model/task?
- What does a minimal viable "handoff payload" look like for agent continuity?
- Can context rot be detected programmatically, or is it always a judgment call?

---

## Connections

- **To [Prompt](../2-prompt/_index.md):** How do you teach the agent (via prompt) how to use its context?
- **To [Model](../3-model/_index.md):** How do context limits shape which models you can use?
- **To [Tool Use](../5-tool-use/_index.md):** How do tools interact with context? (reading files, searching, etc.)
- **To [Context Management Strategies](2-context-strategies.md):** Practical techniques for managing context windows
- **To [Advanced Context Patterns](3-context-patterns.md):** Progressive disclosure and other sophisticated patterns
