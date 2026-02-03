---
title: Patterns
description: Recurring architectures and approaches for building agentic systems
created: 2025-12-08
last_updated: 2026-01-30
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
| [Progressive Disclosure](7-progressive-disclosure.md) | Large knowledge bases with tight context budgets | Slight latency vs. dramatic capacity gains |
| [ReAct Loop](5-react-pattern.md) | General-purpose reasoning + action | Flexibility vs. efficiency |
| [Human-in-the-Loop](6-human-in-the-loop.md) | High-stakes, uncertain, or preference-sensitive | Safety vs. autonomy |
| [Expert Swarm](8-expert-swarm-pattern.md) | Multiple independent tasks within single domain requiring consistency | Expertise governance vs. generic orchestration |

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

Choosing the right pattern depends on three factors: task type, execution requirements, and learning needs.

### Start Here: The Default

**If you're unsure, start with Plan-Build-Review.** It's the safest default because:
- Separates thinking from doing (catches errors before implementation)
- Scales down easily (skip Review phase for simple tasks)
- Scales up naturally (add Research phase for complex domains)

The other patterns address specific limitations of Plan-Build-Review:
- **Orchestrator**: When you need parallel experts or context isolation
- **Autonomous Loops**: When iteration beats planning for mechanical tasks
- **Self-Improving Experts**: When you want knowledge to accumulate across sessions
- **Human-in-the-Loop**: When certain decisions require human judgment
- **ReAct**: When real-time reasoning-action cycles beat upfront planning
- **Progressive Disclosure**: When context must be loaded incrementally

---

### Decision Tree

```
Start: What kind of task?
│
├─► Architectural/Creative Decision
│   └─► Does it need multiple expert perspectives?
│       ├─► Yes → Orchestrator (parallel domain experts)
│       └─► No → Plan-Build-Review (structured phases)
│
├─► Mechanical/Repetitive Task (migrations, refactoring)
│   └─► Are completion criteria machine-verifiable?
│       ├─► Yes → Autonomous Loops (iteration beats planning)
│       └─► No → Plan-Build-Review (human judges completion)
│
├─► Interactive/Exploratory Task
│   └─► Does each action inform the next?
│       ├─► Yes → ReAct (tight reasoning-action loop)
│       └─► No → Plan-Build-Review (plan upfront)
│
├─► High-Stakes/Uncertain Decision
│   └─► Must a human approve certain steps?
│       ├─► Yes → Human-in-the-Loop (explicit approval gates)
│       └─► No → Plan-Build-Review with review phase
│
└─► Large Knowledge Base Task
    └─► Can all context fit in one prompt?
        ├─► Yes → Direct approach (any pattern)
        └─► No → Progressive Disclosure (load on demand)
```

---

### Pattern Selection Matrix

| Task Characteristic | Primary Pattern | Secondary Pattern |
|---------------------|-----------------|-------------------|
| Needs multiple expert domains | Orchestrator | Plan-Build-Review |
| Needs domain expertise + parallelism | Expert Swarm | Orchestrator |
| Mechanical, 10+ iterations likely | Autonomous Loops | Plan-Build-Review |
| Tight action-feedback cycles | ReAct | Orchestrator |
| High-stakes decisions | Human-in-the-Loop | Plan-Build-Review |
| Context exceeds window | Progressive Disclosure | Orchestrator |
| Knowledge should accumulate | Self-Improving Experts | Plan-Build-Review |
| Quick one-off task | Skip patterns | Direct execution |
| Complex multi-phase workflow | Plan-Build-Review | Orchestrator |

---

### When Each Pattern Shines

| Pattern | Best For | Worst For |
|---------|----------|-----------|
| **Plan-Build-Review** | Complex domains, structured workflows, accumulating expertise | Simple one-off tasks, mechanical iteration |
| **Self-Improving Experts** | Recurring decisions, project-specific patterns | One-time projects, stable domains |
| **Orchestrator** | Multi-expert analysis, parallel execution, context isolation | Simple tasks, tight coupling required |
| **Autonomous Loops** | Migrations, coverage expansion, mechanical refactoring | Architectural decisions, subjective criteria |
| **ReAct** | Tool-heavy exploration, debugging, dynamic decisions | Well-defined specs, batch processing |
| **Human-in-the-Loop** | Security-sensitive, compliance, ambiguous requirements | Fully automated pipelines, trusted domains |
| **Progressive Disclosure** | Large codebases, documentation, knowledge-heavy tasks | Small context, everything fits in window |

---

### Multi-Pattern Composition

Patterns often combine. Common compositions:

**Orchestrator + Plan-Build-Review**
Each domain expert uses Plan-Build-Review internally while the orchestrator coordinates parallel execution.
```
Orchestrator
├─► Security Expert (Plan → Build → Review)
├─► Performance Expert (Plan → Build → Review)
└─► Architecture Expert (Plan → Build → Review)
```

**Autonomous Loops + Self-Improving Experts**
Run iteration loop, then extract learnings to improve future prompts.
```
Loop: Execute PROMPT.md → Commit → Check completion
After session: Run improve-agent → Update expertise.yaml
Next session: Benefits from accumulated knowledge
```

**Human-in-the-Loop + Orchestrator**
Approval gates between orchestrator phases.
```
Scout (auto) → Plan (auto) → [Human Review] → Build (auto) → [Human Review] → Deploy
```

**Progressive Disclosure + Any Pattern**
Load context incrementally before any other pattern's execution.
```
Load minimal context → Determine what's needed → Load specifics → Execute pattern
```

---

### Pattern Conflicts

Some patterns work against each other:

| Combination | Conflict | Resolution |
|-------------|----------|------------|
| Autonomous Loops + Orchestrator | Fresh context vs. coordinated state | Use orchestration *within* each iteration, fresh between |
| ReAct + Plan-Build-Review | Real-time cycles vs. upfront planning | ReAct for exploration, PBR for implementation |
| Self-Improving + Autonomous Loops | Accumulating knowledge vs. fresh starts | Extract learnings after loop sessions |

---

### Scaling Considerations

**Small tasks (< 1 hour)**
- Skip patterns entirely or use minimal Plan-Build
- Patterns add overhead that exceeds benefit

**Medium tasks (1-8 hours)**
- Plan-Build-Review is usually right
- Consider Orchestrator if multiple concerns

**Large tasks (multi-day)**
- Full pattern selection matters
- Often requires pattern composition
- Consider Autonomous Loops for mechanical portions

**Team/Production systems**
- Self-Improving Experts become essential
- Human-in-the-Loop for compliance
- Progressive Disclosure for large codebases



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


