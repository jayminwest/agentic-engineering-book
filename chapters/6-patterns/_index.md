---
title: Patterns
description: Recurring architectures and approaches for building agentic systems
created: 2025-12-08
last_updated: 2026-01-21
tags: [patterns, architecture]
part: 2
part_title: Craft
chapter: 6
section: 0
order: 2.6.0
---

# Patterns

Patterns are solutions that keep showing up. They're not prescriptions—they're options with tradeoffs.

---

## Pattern Catalog

| Pattern | When to Use | Key Tradeoff |
|---------|-------------|--------------|
| [Plan-Build-Review](1-plan-build-review.md) | Complex domains with accumulating expertise | Learning vs. simplicity |
| [Self-Improving Experts](2-self-improving-experts.md) | Commands that learn from production experience | Maintenance overhead vs. continuous improvement |
| [Orchestrator](3-orchestrator-pattern.md) | Multi-agent coordination with parallel experts | Power vs. complexity overhead |
| [Autonomous Loops](4-autonomous-loops.md) | Mechanical tasks with clear completion criteria | Iteration resilience vs. token cost |
| [Progressive Disclosure](../4-context/_index.md#progressive-disclosure-pattern) | Large knowledge bases with tight context budgets | Slight latency vs. dramatic capacity gains |
| ReAct Loop | General-purpose reasoning + action | Flexibility vs. efficiency |
| Human-in-the-Loop | High-stakes, uncertain, or preference-sensitive | Safety vs. autonomy |

*Add more patterns as you identify them*

---

## Questions to Answer Here

- What patterns have you seen that work consistently across different domains?
- What patterns looked promising but turned out to be over-engineered for most use cases?
- How do you decide which pattern to start with for a new agent?
- What's the simplest pattern that solves most problems?
- When do patterns compose well, and when do they conflict?

---

## Pattern Selection

*A decision tree or heuristic for choosing patterns:*



---

## Anti-Patterns

*Document what doesn't work and why:*

### Emergency Context Rewriting

**What it is:** Full context rewrites when hitting token limits, using the LLM to "summarize" or "compress" existing context to free up space.

**Why it fails:**

1. **Brevity bias:** LLMs progressively collapse toward short, generic prompts that discard domain-specific information
2. **Context collapse:** Monolithic rewrites compress contexts into uninformative summaries
3. **Lossy compression:** Destroys hard-won knowledge accumulated through experience—tool-use guidelines, failure modes, domain heuristics all get deleted
4. **Kills specificity:** The precise details that make context useful are exactly what gets removed

**Evidence:** The ACE paper demonstrated that Dynamic Cheatsheet (which uses LLM rewrites) suffered progressive degradation over time. Each rewrite made the context slightly less useful, compounding over iterations.

**Better Alternatives:**

1. **Incremental deltas:** ACE's add/update/remove pattern for specific entries rather than full rewrites
2. **Frequent intentional compaction:** Proactive cleanup at 40-60% capacity, not reactive emergency fixes at 95%
3. **Fresh agent boots:** The "one agent, one task" approach—start fresh rather than trying to compress unbounded context
4. **Progressive disclosure:** Load context on-demand rather than trying to fit everything upfront

See also: [Context Management](../4-context/_index.md) for proactive strategies.

---

## Connections

- **To [Prompts/Structuring](../2-prompt/2-structuring.md)**: Patterns rely on prompt design to separate concerns. Mutable expertise sections enable pattern evolution.
- **To [Context Management](../4-context/_index.md)**: Context strategies support pattern implementation. Progressive disclosure and context isolation are foundational to orchestration.
- **To [Tool Use](../5-tool-use/_index.md)**: Patterns coordinate tool selection and restrictions. Tool constraints force delegation patterns.
- **To [Evaluation](../7-practices/2-evaluation.md)**: Patterns require evaluation to verify effectiveness. Self-improving experts need measurement to validate that improvement is happening.


