---
title: Multi-Agent Context
description: Context isolation patterns and persistent state management in multi-agent systems
created: 2025-12-10
last_updated: 2026-02-11
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

### Agent Teams vs Subagents

*[2026-02-05]*: Context isolation differs significantly between subagents and agent teams (TeammateTool), reflecting different coordination models.

**Subagents (Traditional Pattern):**
- Isolated context windows (no parent conversation history)
- Return summaries to orchestrator
- No inter-subagent communication (coordination via orchestrator)
- Coordination state flows through spec files or orchestrator context
- Pattern: Hub-and-spoke architecture

**Agent Teams (Experimental):**
- Isolated context windows (independent sessions for each teammate)
- Peer-to-peer messaging without orchestrator mediation
- Shared task list and message queue (observable at `~/.claude/teams/<team_id>/`)
- Automatic message delivery when teammates check for updates
- Pattern: Peer coordination with optional orchestrator oversight

**Key Distinction:** Subagents isolate context to prevent pollution and enable parallelism. Agent teams isolate context while adding direct communication channels—teammates coordinate through messages, not through shared context.

**When Context Isolation Matters:**
- **Subagents**: Prevents orchestrator context pollution from worker search/analysis operations
- **Agent teams**: Prevents peer context pollution while enabling coordination messages

Both patterns achieve context hygiene, but agent teams add collaboration primitives that subagents lack.

**See:** [Agent Teams documentation](../../10-practitioner-toolkit/1-claude-code.md#agent-teams-native-multi-agent-coordination-experimental) for coordination patterns, message primitives, and decision frameworks. [Official Claude Code docs](https://docs.claude.com/en/docs/claude-code/agent-teams) for setup and usage.

### Contrast with Single-Agent

Single-agent approaches accumulate everything in one context window. Context compounds over time, eventually degrading capability. Multi-agent systems isolate by design—each agent maintains focused context for its specialized role. The orchestrator aggregates final outputs, not intermediate states, keeping its own context clean.

### See Also

[Orchestrator Pattern: Context Isolation via Sub-Agents](../7-patterns/3-orchestrator-pattern.md#context-isolation-via-sub-agents) — Implementation details on how sub-agents act as disposable context buffers, returning synthesized summaries rather than raw data to keep orchestrator context clean.

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

[Google ADK: State and Memory Architecture](../10-practitioner-toolkit/2-google-adk.md#state-and-memory-architecture) — Concrete implementation of state scoping

---

## Memory Architectures for Multi-Agent Systems

*[2026-02-11]*: As multi-agent systems mature beyond single-session interactions, memory architecture becomes a defining design choice. Three models appear in production systems, each with distinct trade-off profiles.

### Three Production Models

**Shared Pool** — All agents access a common memory store (vector database, document store, SQLite). Fast knowledge reuse across agents. Risk: overwrite conflicts, stale reads, and **memory contamination** — where incorrect information written by one agent propagates across the system as "ground truth" for every subsequent reader.

**Local with Synchronization** — Each agent owns private memory, sharing selected information via periodic sync. Isolation by default means fewer contention issues and natural containment of errors. Scales better than shared pools but requires explicit synchronization protocol design.

**Event Bus** — Agents maintain private state and communicate asynchronously via structured events. Maximum decoupling, but requires disciplined event schema governance. Best suited for systems where agents operate on fundamentally different timescales or data domains.

| Model | Isolation | Reuse Speed | Contamination Risk | Scaling |
|-------|-----------|-------------|-------------------|---------|
| Shared Pool | None | Immediate | High | Limited by contention |
| Local + Sync | Default | Sync interval | Low (contained) | Horizontal |
| Event Bus | Full | Event latency | Minimal | Independent |

### The Two-Tier Pattern

Multiple production systems converge on a two-tier approach separating ephemeral from durable memory:

- **OpenClaw** uses daily log files (`memory/YYYY-MM-DD.md`) for running context alongside a curated `MEMORY.md` for durable facts and decisions, with hybrid vector + BM25 search across both tiers
- **Gas Town** uses git-backed JSONL (Beads) with hierarchical task organization and semantic "memory decay" that summarizes old closed tasks to save context window tokens
- **AutoForge** deliberately starts each session with a fresh context window while maintaining state in SQLite — trading recall for context cleanliness

The convergence is notable: all three separate short-lived operational context from long-lived knowledge, despite using entirely different storage backends.

### Virtual Memory for Cognition

*[2026-02-11]*: OpenClaw's architecture treats the LLM context window as cache and disk/database as source of truth — a "virtual memory for cognition" model. The implication: context management shifts from "fitting everything in the window" to "managing a memory hierarchy" with different tiers serving different access patterns and durability guarantees.

This reframes context strategy. Rather than optimizing a single flat window, effective multi-agent systems design memory hierarchies:

| Tier | Analog | Access Pattern | Durability |
|------|--------|---------------|------------|
| Context window | CPU cache | Immediate, capacity-limited | Session-scoped |
| Session logs | RAM | Fast retrieval, structured | Session or daily |
| Curated memory | Disk | Selective loading | Persistent |
| External database | Cold storage | Query-based | Permanent |

### Memory Contamination

When agents share memory, incorrect information spreads across the system. A single agent's hallucination written to shared state becomes "ground truth" for every subsequent agent that reads it. The trend in production systems is toward fine-grained access control — private by default with selective, explicit sharing — rather than all-or-nothing memory pools.

Mitigation strategies observed in production:
- **Write validation gates** — verify outputs before committing to shared state
- **Source attribution** — tag memory entries with authoring agent and confidence
- **Read isolation** — agents read snapshots rather than live state
- **Periodic audits** — scheduled verification of shared memory against ground truth

### Fresh Context as Strategy

*[2026-02-11]*: AutoForge's approach of starting every session with an empty context window represents a deliberate design choice, not a limitation. Fresh context prevents the accumulation of stale assumptions, outdated instructions, and hallucinated context that degrades long-running sessions. When paired with reliable external state (SQLite, git), each session operates on clean data rather than inherited noise.

This aligns with the token economics of multi-agent systems: spending tokens on fresh context loading is cheaper than debugging degraded reasoning from context contamination. The pattern works best when external state is well-structured and queryable — unstructured state dumps negate the benefits of starting fresh.

---

## Connections

- **To [Context Fundamentals](1-context-fundamentals.md):** How multi-agent isolation changes the "One Agent, One Task" model
- **To [Advanced Context Patterns](3-context-patterns.md):** ACE playbooks are persistent state loaded into ephemeral context
- **To [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md):** How orchestrators use context isolation to maintain clean working memory
- **To [Expert Swarm Pattern](../7-patterns/8-expert-swarm-pattern.md):** Path-passing protocol implements expertise sharing without context pollution. Workers receive `EXPERTISE_PATH: /path/to/expertise.yaml` and read relevant sections themselves. Prevents context pollution while maintaining consistency.
- **To [Google ADK](../10-practitioner-toolkit/2-google-adk.md):** Production implementation of persistent state

---

## Sources

- [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — Multi-agent context isolation in production
- [Multi-Agent LLM Orchestration Achieves Deterministic Decision Support](https://arxiv.org/html/2511.15755) — Research on token usage and performance variance
- [Simon Willison's Analysis](https://simonwillison.net/2025/Jun/14/multi-agent-research-system/) — Practitioner perspective on multi-agent patterns
