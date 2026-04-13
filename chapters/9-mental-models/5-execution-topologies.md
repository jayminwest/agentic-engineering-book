---
title: Execution Topologies
description: A measurement framework for understanding and improving agent execution structures
created: 2026-01-15
last_updated: 2026-02-11
tags: [mental-models, measurement, execution, topologies, improvement, swarm]
part: 3
part_title: Perspectives
chapter: 9
section: 5
order: 3.9.5
---

# Execution Topologies

**Execution topologies** describe the shape of agent work—how tasks flow, branch, and converge. Understanding topologies enables a critical capability: measuring whether agentic systems actually improve over time. Without a classification framework, improvement becomes subjective ("feels better") rather than observable ("produces more value with less friction").

This framework, inspired by thread-based engineering work from indydevdan, provides vocabulary for describing execution structures and metrics for tracking their evolution.

---

## The Core Idea

### Boundaries of Agent Work

Engineers appear at two points in any agent workflow: the beginning and the end. At the beginning, engineers provide prompts, plans, or specifications. At the end, they review outputs, validate results, or approve artifacts. Everything between these two boundaries constitutes the **agent autonomy space**.

```
Engineer                                                Engineer
   │                                                       │
   │  ┌──────────────────────────────────────────────┐    │
   │  │                                              │    │
   ▼  │           AGENT AUTONOMY SPACE               │    ▼
Prompt/│                                              │ Review/
 Plan  │    Topology determines what happens here    │ Validate
   │  │                                              │    │
   │  └──────────────────────────────────────────────┘    │
```

The execution topology describes what happens in this autonomous space. Different topologies offer different trade-offs:

- **Parallel topologies** maximize throughput for independent work
- **Sequential topologies** ensure dependencies flow correctly
- **Synthesis topologies** combine multiple perspectives
- **Nested topologies** handle complex decomposition
- **Persistent topologies** accumulate expertise over time

### Why Topology Matters

Topology selection affects every system quality:

| Quality | Topology Impact |
|---------|-----------------|
| Latency | Sequential adds latency; parallel reduces wall-clock time |
| Cost | Parallel multiplies token spend; synthesis has highest cost |
| Quality | Synthesis often produces highest quality; sequential ensures consistency |
| Debuggability | Sequential is easiest to trace; nested creates complex call graphs |

Understanding the topology in use enables targeted optimization. Attempting to reduce latency in a necessarily sequential workflow wastes effort. Recognizing that a workflow is implicitly parallel reveals optimization opportunities.

---

## The Five Topologies

Each topology follows a consistent structure: definition, visual representation, mapping to existing book patterns, and measurement indicators.

### Parallel Topology

Multiple independent operations execute concurrently. Results aggregate after all branches complete.

```
                    ┌──────────┐
                    │  Branch  │
               ┌───►│    A     │───┐
               │    └──────────┘   │
┌─────────┐    │    ┌──────────┐   │    ┌───────────┐
│  Task   │────┼───►│  Branch  │───┼───►│ Aggregate │
└─────────┘    │    │    B     │   │    └───────────┘
               │    └──────────┘   │
               │    ┌──────────┐   │
               └───►│  Branch  │───┘
                    │    C     │
                    └──────────┘
```

**Book Mapping:** [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md) documents single-message parallelism where coordinators spawn multiple sub-agents simultaneously.

**When to Use:**
- Tasks decompose into independent subtasks
- Subtasks share no data dependencies
- Wall-clock time matters more than total token spend
- Results can merge without ordering constraints

**Measurement Indicators:**
- Branch count per task
- Success rate per branch
- Speedup factor vs. sequential baseline
- Aggregation overhead (time/tokens spent merging)

*[2026-01-30]*: Model-native swarm orchestration (e.g., Kimi K2.5) enables branch counts exceeding SDK orchestration limits. Kimi K2.5 demonstrates up to 100 concurrent subagents—well beyond typical SDK parallelism (5-10 branches). This shifts "wider" measurement ceiling: SDK orchestration saturates at ~10 branches; model-native swarm scales to 100+. See [Multi-Model Architectures: Model-Native Swarm](../../3-model/4-multi-model-architectures.md#model-native-swarm-orchestration) for architectural details and performance characteristics.

### Sequential Topology

Operations execute in strict order. Each phase produces artifacts consumed by the next.

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Phase 1 │───►│ Phase 2 │───►│ Phase 3 │───►│ Phase 4 │
│  Plan   │    │  Build  │    │ Review  │    │ Refine  │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
     │              │              │              │
     ▼              ▼              ▼              ▼
  Spec.md       Code/Files    Feedback.md    Final.md
```

**Book Mapping:** [Plan-Build-Review](../7-patterns/1-plan-build-review.md) documents the canonical sequential pattern with phase gates separating planning, execution, and validation.

**When to Use:**
- Strong dependencies exist between phases
- Later phases require artifacts from earlier phases
- Quality gates must prevent error propagation
- Audit trail matters (who decided what, when)

**Measurement Indicators:**
- Phase count per workflow
- Phase success rate (passes without rework)
- Artifact quality scores per phase
- Rework rate (phases requiring re-execution)

### Synthesis Topology

Multiple perspectives analyze the same problem. Results merge through comparison, voting, or reasoned integration.

```
                    ┌────────────┐
               ┌───►│  Expert A  │───┐
               │    │ (Security) │   │
               │    └────────────┘   │
┌──────────┐   │    ┌────────────┐   │    ┌───────────┐
│  Problem │───┼───►│  Expert B  │───┼───►│ Synthesize│
└──────────┘   │    │   (Perf)   │   │    └───────────┘
               │    └────────────┘   │
               │    ┌────────────┐   │
               └───►│  Expert C  │───┘
                    │   (UX)     │
                    └────────────┘
```

**Book Mapping:** [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md) covers expert synthesis and best-of-N approaches where multiple agents provide independent analyses that a coordinator integrates.

**When to Use:**
- Problem benefits from multiple perspectives
- No single expert covers all dimensions
- Quality requires cross-domain consideration
- Consensus or majority voting increases confidence

**Measurement Indicators:**
- Expert count per problem
- Unique insight rate (perspectives not redundant)
- Cross-cutting detection (issues spanning experts)
- Synthesis quality (integration coherence)

### Nested Topology

Parent operations spawn child operations. Results bubble up through the hierarchy.

```
┌─────────────────────────────────────────┐
│                Coordinator              │
│  ┌─────────────────────────────────┐   │
│  │           Sub-Agent A           │   │
│  │  ┌─────────┐    ┌─────────┐    │   │
│  │  │ Worker  │    │ Worker  │    │   │
│  │  │   A.1   │    │   A.2   │    │   │
│  │  └─────────┘    └─────────┘    │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │           Sub-Agent B           │   │
│  │  ┌─────────┐                    │   │
│  │  │ Worker  │                    │   │
│  │  │   B.1   │                    │   │
│  │  └─────────┘                    │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**Book Mapping:** [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md) and [Multi-Agent Context](../4-context/4-multi-agent-context.md) cover sub-agent spawning and coordinator patterns where context flows down and summaries flow up.

**When to Use:**
- Problems decompose hierarchically
- Subtasks require isolated context
- Coordination overhead is justified by complexity reduction
- Results require aggregation across levels

**Measurement Indicators:**
- Nesting depth (levels in hierarchy)
- Context isolation effectiveness (cross-contamination rate)
- Summary quality (information preserved through bubbling)
- Coordination overhead (tokens/time in coordination vs. work)

### Persistent Topology

Operations span multiple sessions. State persists in artifacts—files, databases, or memory systems.

```
Session 1                Session 2                Session 3
┌─────────┐             ┌─────────┐             ┌─────────┐
│  Work   │             │  Work   │             │  Work   │
│ + Learn │             │ + Learn │             │ + Learn │
└────┬────┘             └────┬────┘             └────┬────┘
     │                       │                       │
     ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────┐
│                   Persistent State                      │
│  expertise.yaml  │  patterns.md  │  learnings.md       │
└─────────────────────────────────────────────────────────┘
```

**Book Mapping:** [Self-Improving Experts](../7-patterns/2-self-improving-experts.md) documents the pattern of agents that accumulate domain expertise across sessions, storing learnings in structured files.

**When to Use:**
- Domain expertise develops over time
- Session work should inform future sessions
- Pattern recognition spans multiple interactions
- Learning investment compounds across uses

**Measurement Indicators:**
- Session count contributing to expertise
- Expertise growth rate (learnings per session)
- Pattern reuse rate (existing patterns applied)
- Expertise utilization (stored knowledge actually used)

---

## The Measurement Framework

### The Core Question

**"How do agents get better over time?"**

Without measurement, improvement claims remain aspirational. Execution topologies provide a framework for making improvement observable through four vectors.

### Four Improvement Vectors

```
              WIDER (more parallel)
                    ↑
                    │
    ┌───────────────┼───────────────┐
    │               │               │
THICKER ←───────────┼───────────────→ DEEPER
(more tools)        │          (longer chains)
    │               │               │
    └───────────────┼───────────────┘
                    │
                    ↓
           LESS FRICTION
         (fewer checkpoints)
```

| Vector | Description | Measurement | Improvement Looks Like |
|--------|-------------|-------------|----------------------|
| **Wider** | More parallel branches | Branch count per task | 1 branch → 5 branches |
| **Deeper** | Longer autonomous chains | Steps before human intervention | 3 steps → 12 steps |
| **Thicker** | More tool invocations | Tool calls per session | 5 calls → 50 calls |
| **Less Friction** | Fewer human checkpoints | Approval gates triggered | 5 approvals → 1 approval |

### Tool Calls as Impact Proxy

Tool invocations serve as a proxy for agent impact. An agent that reads 50 files, writes 10, and runs 5 tests accomplishes more than one that reads 2 files and writes nothing—assuming equivalent quality.

This connects to cost management: [Cost and Latency](../8-practices/3-cost-and-latency.md) documents token cost as a proxy for value delivered. Tool calls refine this—they represent actions taken, not just tokens consumed.

**Calibration:** Raw tool call count misleads if quality degrades. Pair tool call metrics with quality signals:
- Success rate of tool operations
- Downstream usage of tool outputs
- Human revision rate of agent work

### Tracking Improvement Over Time

A baseline measurement enables trend analysis:

```
Week 1: Parallel branches = 2, Depth = 4 steps, Tools = 12/session
Week 4: Parallel branches = 4, Depth = 8 steps, Tools = 35/session
Week 8: Parallel branches = 5, Depth = 15 steps, Tools = 80/session

Trend: All vectors improving → system learning effectively
```

When vectors plateau or regress, investigate:
- Wider plateau → Task structure doesn't support more parallelism
- Deeper regression → Error accumulation forcing earlier human intervention
- Thicker plateau → Tool coverage gaps limiting automation
- Friction increase → Trust erosion requiring more checkpoints

---

## Decision Heuristics

### Topology Selection Table

| Task Characteristic | Recommended Topology | Rationale |
|--------------------|---------------------|-----------|
| Independent subtasks | Parallel | No dependencies → maximize concurrency |
| Strict phase dependencies | Sequential | Later phases need earlier artifacts |
| Multi-perspective analysis | Synthesis | Problem requires diverse expertise |
| Hierarchical decomposition | Nested | Complexity requires divide-and-conquer |
| Recurring domain work | Persistent | Learning should compound across sessions |
| Unknown structure | Sequential → Parallel | Start simple, add complexity when justified |

### Trust Gradient

Trust level determines appropriate topology dimensions:

| Trust Level | Width | Depth | Thickness | Friction | Example Context |
|-------------|-------|-------|-----------|----------|-----------------|
| **Low** | Narrow (1-2) | Shallow (2-3) | Thin (5-10) | High (every step) | New domain, unproven agent |
| **Medium** | Moderate (3-4) | Medium (5-8) | Medium (20-40) | Medium (phase gates) | Established patterns, familiar domain |
| **High** | Wide (5+) | Deep (10+) | Thick (50+) | Low (final review) | Mature system, high-value automation |

Trust increases through demonstrated reliability:
- Low friction runs accumulate without failure
- Quality metrics remain stable as autonomy increases
- Edge cases handled gracefully without escalation

### Topology Combinations

Real systems often combine topologies:

```
Sequential Phases with Parallel Sub-Tasks:

┌─────────────────────────────────────────────────────────┐
│ Phase 1: Plan                                           │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 2: Build (Parallel)                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ Module A │  │ Module B │  │ Module C │              │
│  └──────────┘  └──────────┘  └──────────┘              │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 3: Review (Synthesis)                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                 │
│  │Security │  │  Perf   │  │  Style  │                 │
│  └─────────┘  └─────────┘  └─────────┘                 │
└─────────────────────────────────────────────────────────┘
```

This combination leverages:
- Sequential flow for phase dependencies
- Parallel execution within the build phase
- Synthesis for multi-perspective review

---

## Anti-Patterns by Topology

Each topology has characteristic failure modes:

| Topology | Anti-Pattern | Why It Fails | Detection Signal |
|----------|--------------|--------------|------------------|
| **Parallel** | Dependent branches | Branches serialize anyway; overhead without benefit | Branch wait times spike |
| **Sequential** | Unnecessary phases | Latency without value; busywork gates | Phases produce no unique artifacts |
| **Synthesis** | Redundant experts | Token cost without new perspectives | Expert outputs highly correlated |
| **Nested** | Shallow nesting | Coordination overhead exceeds work | Depth = 1 everywhere |
| **Persistent** | Static expertise | Learning without application; stale knowledge | Expertise files unchanged for weeks |

### Recognizing Anti-Pattern Emergence

Anti-patterns often emerge gradually:

1. **Scope creep** adds phases to sequential workflows
2. **Risk aversion** adds experts to synthesis without clear value
3. **Premature optimization** parallelizes tasks with hidden dependencies
4. **Over-architecture** nests when flat structures suffice

Regular topology audits catch drift before it calcifies.

---

## The Sixth Topology: Zero-Touch

Beyond the five core topologies lies the aspirational endpoint: **zero-touch execution**.

```
┌─────────────┐                              ┌─────────────┐
│   Trigger   │ ────────────────────────────►│   Result    │
│  (Event)    │         No human in loop     │ (Verified)  │
└─────────────┘                              └─────────────┘
```

Zero-touch represents maximum trust: agents detect events, execute workflows, and deliver verified results without human intervention. The topology itself becomes irrelevant—what matters is the system handles end-to-end.

**Prerequisites for zero-touch:**
- Demonstrated reliability across all edge cases
- Robust error handling and self-correction
- Clear boundaries (what triggers escalation)
- Audit trails for post-hoc review

Zero-touch is a horizon, not a destination. Each reduction in friction—each removed checkpoint—moves toward it while acknowledging that some domains may never reach full autonomy.

---

## Autonomy as a Topology Dimension

The five topologies describe how agents connect and what flows between them. **Autonomy** describes how much latitude each node has within that topology. A hub-spoke topology can operate with high autonomy (workers execute without approval) or low autonomy (every action requires orchestrator confirmation). Topology shape and autonomy level are independent design choices—orthogonal dimensions of the same system.

### Five-Level Autonomy Spectrum

*[2026-02-11]*: Multi-agent systems in early 2026 cluster across a five-level spectrum:

| Level | Description | Human Role | Example |
|-------|-------------|------------|---------|
| 1. Augmentation | Agents enhance human work | Hands-on collaborator | Single-agent chat assistants |
| 2. Supervised automation | Agents execute with checkpoints | Active reviewer | OpenClaw with configurable thinking depth |
| 3. Bounded autonomy | Agents operate within defined limits | Supervisor at decision points | AutoForge (spec-driven, test-gated execution) |
| 4. High autonomy | Agents execute relentlessly, humans review results | Factory floor operator | Gas Town (agents "must execute" per GUPP principle) |
| 5. Full autonomy | Minimal human oversight | Absent or strategic only | Not achieved by any current production system |

### Bounded Autonomy Dominates Production

The consensus across practitioners, frameworks, and industry analysis places Level 3 (bounded autonomy) as the current production sweet spot. Full automation is not the winning strategy—intentional human checkpoints at workflow decision points produce better outcomes than either constant oversight or unchecked agent execution.

### Autonomy Interacts with the Trust Gradient

Higher autonomy requires proportionally stronger safeguards. Gas Town's Level 4 autonomy (the GUPP principle: "If there is work on your hook, you MUST run it") combined with weak safeguards produced documented incidents: agents merging PRs with failing tests, deleting code unpredictably, and requiring forced repository resets. The lesson generalizes—autonomy level must be matched to safeguard maturity, not aspirational capability.

### Cost Scales with Autonomy

*[2026-02-11]*: Multi-agent systems at higher autonomy levels consume approximately 15x more tokens than standard single-agent interactions. Each autonomy level increase means more agent-initiated actions, more tool calls, more context consumption. The economic viability of higher autonomy depends on task value exceeding the proportionally higher token cost.

**Cross-reference:** See [The Multi-Agent Landscape: Autonomy](../7-patterns/10-multi-agent-landscape.md#autonomy--the-bounded-middle) for the full ecosystem analysis of autonomy trends.

---

## Connections

- **[Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md)**: Implements parallel, synthesis, and nested topologies through coordinator agents
- **[Plan-Build-Review](../7-patterns/1-plan-build-review.md)**: The canonical sequential topology with phase gates
- **[Self-Improving Experts](../7-patterns/2-self-improving-experts.md)**: Persistent topology realized through expertise accumulation
- **[Multi-Agent Context](../4-context/4-multi-agent-context.md)**: Context management for nested and parallel topologies
- **[Cost and Latency](../8-practices/3-cost-and-latency.md)**: Tool calls as impact proxy connects to cost management
- **[Workflow Coordination](../8-practices/5-workflow-coordination.md)**: Operational considerations for topology implementation
- **[Human-in-the-Loop](../7-patterns/6-human-in-the-loop.md)**: Checkpoint patterns that implement different autonomy levels within topologies
