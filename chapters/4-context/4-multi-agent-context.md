---
title: Multi-Agent Context
description: Context isolation patterns and persistent state management in multi-agent systems
created: 2025-12-10
last_updated: 2026-01-11
tags: [foundations, context, multi-agent, state, isolation]
part: 1
part_title: Foundations
chapter: 4
section: 4
order: 1.4.4
---

# Multi-Agent Context

How context management changes in multi-agent systems: isolation patterns that prevent pollution, and the critical distinction between ephemeral context and persistent state.

---

## Multi-Agent Context Isolation

*[2025-12-09]*: In multi-agent systems, context isolation is a deliberate architectural choice that fundamentally differs from single-agent context management.

**The Pattern**: Each subagent maintains its own separate context window. Rather than sharing a massive context, subagents work in isolation and return only synthesized, relevant information to the orchestrator. The orchestrator's context stays clean because it receives summaries and conclusions, not raw data.

### Why It Works

- Prevents context pollution—one agent's research doesn't bloat another's working memory
- Enables true parallelism—agents can process information simultaneously without stepping on each other
- Natural compression—synthesis happens at the boundary, not as a cleanup step
- Predictable quality—each agent starts fresh with focused context

### The Trade-off

This approach uses significantly more tokens—research shows ~15× more than single-agent approaches for complex multi-tool tasks. However, token usage explains 80% of performance variance in multi-agent systems. The architectural value lies in deterministic quality, not speed (both single and multi-agent achieve similar ~40s latency for complex tasks).

*[2025-12-10]*: The 15× token multiplier and 80% variance explanation come from analysis of multi-agent orchestration for decision support tasks. See [Sources](#sources) for the multi-agent orchestration research paper.

### When to Use

Multi-agent context isolation excels in specific scenarios:
- **Complex tasks** benefiting from parallel analysis by specialized experts
- **High-reliability requirements** where near-zero quality variance matters
- **Research synthesis** where multiple sources need independent analysis
- **Decision support** requiring deterministic, reproducible outputs

For simple tasks, multi-agent context isolation introduces unnecessary overhead.

### Contrast with Single-Agent

Single-agent approaches accumulate everything in one context window. Context compounds over time, eventually degrading capability. Multi-agent systems isolate by design—each agent maintains focused context for its specialized role. The orchestrator aggregates final outputs, not intermediate states, keeping its own context clean.

### See Also

[Orchestrator Pattern: Context Isolation via Sub-Agents](../6-patterns/3-orchestrator-pattern.md#context-isolation-via-sub-agents) — Implementation details on how sub-agents act as disposable context buffers, returning synthesized summaries rather than raw data to keep orchestrator context clean.

### Sub-Agent Forking

*[2026-01-11]*: Claude Code 2.1.0 introduced explicit sub-agent forking via `context: fork` in agent, skill, or slash command frontmatter. This provides a declarative mechanism for context isolation without relying on the implicit isolation of Task tool spawning.

**Syntax:**

```yaml
---
name: isolated-researcher
description: Research task with fresh context
context: fork
tools: Read, Grep, Glob, WebFetch
---

[Agent instructions...]
```

**When `context: fork` activates:**
- The sub-agent starts with a fresh context window
- No conversation history from the parent carries over
- Results return to the parent as a synthesized summary
- Parent context remains unmodified by sub-agent operations

**Use Cases:**

| Scenario | Benefit of Forking |
|----------|-------------------|
| Research tasks | Fresh context prevents existing knowledge from biasing investigation |
| Skill testing | Test skill behavior without polluting main conversation |
| Parallel analysis | Multiple forks analyze independently, results aggregate cleanly |
| Sensitive operations | Isolate potentially context-polluting operations |

**Contrast with Implicit Isolation:**

The Task tool already provides context isolation—subagents maintain separate context windows. `context: fork` makes this explicit at the definition level:

| Approach | Isolation Timing | Declaration |
|----------|------------------|-------------|
| Task tool | At spawn time | Implicit in Task invocation |
| `context: fork` | At definition time | Explicit in frontmatter |

**When to Use:**

- **Prefer `context: fork`** when isolation is inherent to the agent's purpose (e.g., "always start fresh for this type of task")
- **Prefer Task tool defaults** when isolation depends on invocation context (e.g., "sometimes share context, sometimes isolate")

**Sources:** [Claude Code Changelog 2.1.0](https://code.claude.com/docs/en/changelog)

---

## Persistent State vs. Ephemeral Context

*[2025-12-09]*: Context is ephemeral—it lives in the active window and dies with the session. State is persistent—it survives across sessions, agents, and even application restarts. The distinction matters for production systems.

### The Confusion

Most discussions conflate context and state. "Give the agent more context" often means "persist information across sessions." But these are fundamentally different:

| Aspect | Ephemeral Context | Persistent State |
|--------|-------------------|------------------|
| **Lifetime** | Single session/conversation | Survives restarts |
| **Scope** | One agent's working memory | Shared across agents/sessions |
| **Cost** | Tokens in context window | Storage + retrieval |
| **Failure Mode** | Context rot, capacity limits | Stale data, sync issues |

### What Belongs Where

- **Context**: Current task details, recent tool outputs, working hypotheses, intermediate reasoning
- **State**: User preferences, conversation history, accumulated knowledge, learned corrections, cross-session continuity

Trying to reconstruct state from context each session is expensive and error-prone. Trying to persist everything as state creates sync nightmares.

### Google ADK's State Prefixes

ADK demonstrates elegant scoping with prefixes that determine persistence lifetime:

| Prefix | Lifetime | Use Case |
|--------|----------|----------|
| `session:` | Current conversation | Working memory, intermediate results |
| `user:` | Across sessions for this user | Preferences, accumulated knowledge |
| `app:` | Application-wide | Shared configuration, global state |
| `temp:` | Single turn | Scratch space, discardable |

The elegance: namespace by lifecycle, not by feature. No configuration overhead—the prefix declares intent.

### Example: State Prefix Usage

```python
# Google ADK state management example

# Session-scoped: Current task context
await state.set("session:current_file", "auth.py")
await state.set("session:last_edit", "line 42")

# User-scoped: Preferences across sessions
await state.set("user:preferred_style", "functional")
await state.set("user:editor_config", {"indent": 2})

# App-scoped: Shared configuration
await state.set("app:api_endpoint", "https://api.example.com")
await state.set("app:rate_limit", 100)

# Temp-scoped: Discardable scratch data
await state.set("temp:calculation_result", 3.14159)
await state.set("temp:search_buffer", ["result1", "result2"])
```

The prefix determines persistence behavior without additional configuration. When the session ends, `session:*` and `temp:*` keys disappear. `user:*` and `app:*` persist for future sessions.

### Practical Implications

1. **Multi-session workflows** need persistent state, not reconstructed context
2. **User preferences** belong in state, not re-injected each session
3. **Conversation continuity** requires state—context windows aren't infinite
4. **Knowledge accumulation** (learned corrections, discovered patterns) must persist

### The Missing Piece in Claude Code

Claude Code's context management is sophisticated for ephemeral context (progressive disclosure, multi-agent isolation) but lacks native persistent state beyond conversation history. For cross-session state, you need external storage—databases, files, or services like VertexAI's session management.

This is a real gap. When an agent learns something valuable mid-session, how does that learning persist? Currently: it doesn't, unless you build external persistence.

### See Also

[Google ADK: State and Memory Architecture](../9-practitioner-toolkit/2-google-adk.md#state-and-memory-architecture) — Concrete implementation of state scoping

---

## Connections

- **To [Context Fundamentals](1-context-fundamentals.md):** How multi-agent isolation changes the "One Agent, One Task" model
- **To [Advanced Context Patterns](3-context-patterns.md):** ACE playbooks are persistent state loaded into ephemeral context
- **To [Orchestrator Pattern](../6-patterns/3-orchestrator-pattern.md):** How orchestrators use context isolation to maintain clean working memory
- **To [Expert Swarm Pattern](../6-patterns/8-expert-swarm-pattern.md):** Path-passing protocol implements expertise sharing without context pollution. Workers receive `EXPERTISE_PATH: /path/to/expertise.yaml` and read relevant sections themselves. Prevents context pollution while maintaining consistency.
- **To [Google ADK](../9-practitioner-toolkit/2-google-adk.md):** Production implementation of persistent state

---

## Sources

- [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — Multi-agent context isolation in production
- [Multi-Agent LLM Orchestration Achieves Deterministic Decision Support](https://arxiv.org/html/2511.15755) — Research on token usage and performance variance
- [Simon Willison's Analysis](https://simonwillison.net/2025/Jun/14/multi-agent-research-system/) — Practitioner perspective on multi-agent patterns
