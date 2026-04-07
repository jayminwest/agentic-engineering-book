---
title: "The Multi-Agent Landscape"
description: "Communication protocols, topology models, safeguards, memory architectures, and autonomy levels across the multi-agent ecosystem"
created: 2026-02-11
last_updated: 2026-02-11
tags: [patterns, multi-agent, protocols, communication, topology, safeguards, memory, autonomy]
part: 2
part_title: Craft
chapter: 6
section: 10
order: 2.6.10
---

# The Multi-Agent Landscape

Multi-agent systems in early 2026 converge on shared architectural patterns despite divergent implementations. Across 15+ active frameworks, five dimensions define the design space: how agents communicate, how they organize, what constrains them, how they remember, and how much latitude they receive.

This chapter maps those dimensions. Projects appear as evidence for conceptual claims, not as the focus. The goal: identify the structural forces shaping multi-agent design so practitioners can make informed architectural decisions regardless of framework choice.

---

## Communication — The Emerging Protocol Stack

*[2026-02-11]*: The multi-agent communication landscape has stratified into distinct layers, each addressing a different integration problem. No single protocol covers the full stack.

### Protocol Stratification

| Layer | Protocol | Purpose | Transport | Adoption Signal |
|-------|----------|---------|-----------|-----------------|
| **Vertical** (agent-to-tool) | MCP (Anthropic) | Standardized tool integration | JSON-RPC 2.0 | 97M monthly SDK downloads |
| **Horizontal** (agent-to-agent) | A2A (Google / Linux Foundation) | Cross-vendor agent interop | REST / JSON-RPC | 50+ launch partners |
| **Lightweight** (legacy / REST) | ACP (IBM BeeAI) | Low-barrier agent coordination | REST | Open-source community |
| **Decentralized** (identity) | ANP (W3C) | Peer-to-peer agent discovery | DID-based | Early specification stage |

The stratification reflects a fundamental insight: agent-to-tool communication (vertical) and agent-to-agent communication (horizontal) solve different problems. MCP standardizes how agents discover and invoke tools. A2A standardizes how agents discover and invoke each other. Attempting to solve both with one protocol produces an unwieldy specification.

Organizations adopting standardized protocols report 60-70% reduction in integration time compared to custom agent messaging implementations.

### Three Communication Paradigms

Beyond protocols, three paradigms govern how agents actually exchange information during execution:

**Handoff** — Agent transfers control mid-conversation with state attached. Clean and deterministic, but inherently sequential. One agent acts at a time.
- *Observed in:* OpenAI Agents SDK, Pydantic AI
- *Trade-off:* Simplicity vs. parallelism

**Shared State** — Agents read and write to a common store (SQLite, graph state, session objects). Enables fast reuse of intermediate results, but risks overwrite conflicts and stale reads.
- *Observed in:* AutoForge (SQLite features DB), LangGraph (graph state), Google ADK (`output_key`)
- *Trade-off:* Speed of reuse vs. consistency guarantees

**Async Mail / Hooks** — Agents work independently, checking persistent message queues for coordination signals. Maximally decoupled, but coordination overhead increases with team size.
- *Observed in:* Gas Town (hooks with GUPP principle), OpenClaw (Beads messaging)
- *Trade-off:* Decoupling vs. coordination latency

### The Structured Communication Trend

*[2026-02-11]*: A clear trend across frameworks: structured protocols over natural language for inter-agent communication. JSON-RPC 2.0 and REST dominate transport. The reasoning is practical — structured messages are parseable, validatable, and debuggable. Natural language between agents introduces the same ambiguity problems that prompt engineering solves for human-to-agent communication.

**Opt-in communication as safety pattern:** Some frameworks default agent-to-agent communication to OFF, requiring explicit allowlisting per agent pair. This treats cross-talk as a capability to be granted, not a default — a meaningful architectural choice that limits blast radius when individual agents misbehave.

### Protocol Selection Guidance

| Integration Need | Recommended Protocol | Rationale |
|-----------------|---------------------|-----------|
| Tool discovery and invocation | MCP | Mature ecosystem, wide adoption, standardized tool descriptions |
| Cross-vendor agent interop | A2A | Industry backing, enterprise focus, task lifecycle management |
| Legacy system integration | ACP | Low barrier, REST-native, minimal infrastructure |
| Decentralized agent networks | ANP | DID-based identity, no central authority required |
| Internal team coordination | Framework-native | Lower overhead than protocol adoption for single-vendor deployments |

The decision is not "which protocol?" but "which protocols?" — most production systems need at least two (vertical + horizontal) and often three (adding framework-native for internal coordination).

---

## Topology — Dynamic Over Fixed

The question is not "which topology is best?" but "which topology fits this task's coordination requirements?"

### Seven Topologies in Active Use

| Topology | Structure | Best For | Example Implementations |
|----------|-----------|----------|------------------------|
| **Hub-spoke / Orchestrator-worker** | Central coordinator delegates to specialists | Production deployments — clear control and debugging | Anthropic Claude Code, Google ADK Coordinator, Gas Town Mayor |
| **Sequential pipeline** | Agent output feeds next agent's input | Linear workflows with clear stage boundaries | Google ADK SequentialAgent, AutoForge |
| **Parallel fan-out/gather** | Multiple agents execute simultaneously, results aggregated | Independent subtasks with synthesis | Google ADK ParallelAgent, MassGen |
| **Hierarchical tree** | Multi-level delegation (coordinator → sub-coordinator → workers) | Complex decomposition with scope isolation | Gas Town (Town → Rig → Crew), smolagents |
| **Flat / peer-to-peer** | Agents communicate directly without central coordinator | Routing-based specialization | OpenClaw, A2A-enabled systems |
| **Dynamic / swarm** | Team composition changes based on task requirements | Elastic workloads, evolving requirements | CrewAI crews, Swarms Enterprise |
| **Generator-critic** | One agent produces, another evaluates, iterating until quality threshold | Iterative refinement with explicit quality gates | Google ADK pattern, Constitutional AI variants |

### Dynamic Hierarchy as the Trend

*[2026-02-11]*: The trend is not away from hierarchy — it is toward *dynamic* hierarchy. Orchestrators that spawn, reassign, and dissolve teams as task requirements shift outperform static configurations. Fixed topologies force tasks into predetermined coordination shapes; dynamic topologies adapt coordination to the task.

**LangChain's production guidance** codifies the scaling path: "Start with a single agent. Add tools before adding agents. Adopt multi-agent only when facing context management constraints." This sequential escalation prevents premature complexity — most tasks that seem to need multi-agent coordination actually need better tool design.

### Swarm vs. Supervisor Trade-offs

LangChain benchmarks show swarm architecture slightly outperforms supervisor across the board, but supervisor systems are easier to debug and reason about. The performance gap is narrow; the observability gap is wide.

| Dimension | Swarm | Supervisor |
|-----------|-------|------------|
| **Raw performance** | Slightly higher | Slightly lower |
| **Debugging** | Distributed traces, harder to follow | Central log, linear trace |
| **Scalability** | Horizontal (add more agents) | Vertical (smarter coordinator) |
| **Failure isolation** | Natural (agent-level) | Requires explicit handling |
| **Coordination cost** | Implicit (emergence) | Explicit (orchestration logic) |

### Orchestration vs. Choreography

Code-based orchestration (deterministic routing) and LLM-based routing (flexible delegation) represent opposite ends of a control spectrum. Most production systems use hybrid approaches — deterministic routing for well-understood paths, LLM-based routing for novel situations. Pure orchestration is brittle; pure choreography is unpredictable.

| Approach | Control Model | Strengths | Weaknesses |
|----------|--------------|-----------|------------|
| **Code-based orchestration** | Deterministic routing via explicit rules | Predictable, debuggable, auditable | Brittle under novel conditions |
| **LLM-based routing** | Model decides delegation dynamically | Flexible, adapts to unforeseen tasks | Unpredictable, harder to audit |
| **Hybrid** | Deterministic default, LLM for novel paths | Balance of control and flexibility | Complexity of two routing systems |

### Topology Selection Heuristic

```
Is the workflow linear with clear stages?
    Yes → Sequential Pipeline
    No  → Continue

Do subtasks share state during execution?
    Yes → Hub-spoke with shared state
    No  → Continue

Are subtasks independent and parallelizable?
    Yes → Parallel fan-out/gather
    No  → Continue

Does the task require iterative refinement?
    Yes → Generator-critic
    No  → Continue

Must team composition change during execution?
    Yes → Dynamic swarm
    No  → Hub-spoke (safest default)
```

---

## Safeguards — Bounded Autonomy as Standard

Defense-in-depth is the baseline pattern, not a nice-to-have. Every production multi-agent system studied implements multiple independent safety layers.

### The Defense-in-Depth Stack

```
┌─────────────────────────────────────────────┐
│  Manual Review (human approval gates)        │
├─────────────────────────────────────────────┤
│  Agent Config Protection                     │
│  (block writes to CLAUDE.md, .cursorrules,   │
│   MCP configs)                               │
├─────────────────────────────────────────────┤
│  Command Allowlists                          │
│  (only permitted commands execute)           │
├─────────────────────────────────────────────┤
│  Filesystem Restrictions                     │
│  (tiered: none → read-only → read-write)    │
├─────────────────────────────────────────────┤
│  Credential Injection                        │
│  (minimal at start, task-scoped, short-lived)│
├─────────────────────────────────────────────┤
│  OS-Level Sandbox                            │
│  (gVisor, Kata containers, macOS Seatbelt)   │
└─────────────────────────────────────────────┘
```

Each layer catches failures that slip through layers above it. Application-level controls alone are insufficient — attackers use indirection, calling restricted tools through approved ones. OS-level enforcement catches what application guards miss.

### The Four Pillars of Platform Control

*[2026-02-11]*: CNCF's 2026 framework for agent governance identifies four complementary control mechanisms:

1. **Golden paths** — Paved roads that make the secure approach the easy approach
2. **Guardrails** — Automated checks that prevent known-bad configurations
3. **Safety nets** — Catch-all systems that detect anomalies the guardrails miss
4. **Manual review** — Human checkpoints at high-consequence decision points

These map cleanly to multi-agent systems: golden paths are well-tested agent templates, guardrails are filesystem and command restrictions, safety nets are watchdog processes, and manual review is human-in-the-loop approval.

### Prompt Injection Remains Unsolved

Every framework studied acknowledges prompt injection as an open risk. No framework claims prevention — only mitigation through defense-in-depth.

**Concrete mitigations observed across frameworks:**
- **Locked DMs** — Agents cannot message arbitrary endpoints; communication restricted to declared peers
- **Mention gating** — Agents only respond when explicitly invoked, preventing injection through ambient context
- **Instruction-hardened models** — Training-time resistance to injection attempts (model-level, not application-level)
- **Input sanitization** — Stripping or escaping control characters from external data before injecting into agent context
- **Output validation** — Verifying agent actions against declared intent before execution

None of these is sufficient alone. The defense-in-depth principle applies: assume each layer will fail, and design the stack so that no single failure compromises the system.

### Trust Creep as Primary Risk

*[2026-02-11]*: The most dangerous failure mode is not a dramatic exploit but gradual erosion. Trust creep — gradually relaxing approval gates because "agents usually get it right" — produces systemic quality degradation. The fix is structural: tie approval requirements to task classification, not to recent success rates.

Only 21% of organizations report mature agent governance frameworks despite 73% citing security as their top AI risk. The gap between perceived risk and implemented controls defines the current landscape.

---

## Memory — The Urgent Frontier

*[2026-02-11]*: Memory is the most urgent unsolved frontier in agent development. ICLR 2026 dedicated a workshop to "MemAgents: Memory for LLM-Based Agentic Systems," reflecting the gap between memory's importance and its current maturity.

### Three Memory Models in Production

| Model | Mechanism | Strengths | Risks |
|-------|-----------|-----------|-------|
| **Shared pool** | All agents access common store (vector DB, document store) | Fast reuse, no duplication | Overwrite conflicts, stale reads, contamination |
| **Local with sync** | Each agent owns private memory, shares via periodic synchronization | Isolation by default, fewer contention issues | Sync latency, divergence between syncs |
| **Event bus** | Agents as independent actors with private state, communicating asynchronously | Maximal decoupling, clean interfaces | Requires event schema discipline, eventual consistency |

### The Two-Tier Pattern

A recurring architecture across projects: ephemeral working memory paired with persistent knowledge storage. The specific implementations differ, but the structural pattern is consistent.

**Ephemeral layer** (session-scoped, disposable):
- Daily logs, work queues, conversation context
- Cheap to create, acceptable to lose

**Persistent layer** (cross-session, curated):
- Curated knowledge bases, semantic search indexes, structured databases
- Expensive to build, critical to preserve

| Project | Ephemeral Layer | Persistent Layer | Retrieval |
|---------|----------------|------------------|-----------|
| OpenClaw | Daily logs (`memory/YYYY-MM-DD.md`) | Curated `MEMORY.md` | Hybrid vector/BM25 search |
| Gas Town | Hooks (work queues) | Beads (git-backed JSONL knowledge graph with semantic decay) | Graph traversal + semantic search |
| AutoForge | Fresh context per session (deliberate amnesia) | SQLite features DB | Structured queries |

### Virtual Memory for Cognition

*[2026-02-11]*: OpenClaw's metaphor reframes context management: the LLM context window functions as cache, persistent storage as source of truth. The model only "remembers" what gets written to disk. This shifts the mental model from "fitting everything in the window" to "managing a memory hierarchy" — analogous to how operating systems manage RAM and disk.

The implications are practical:
- **Write-back policy matters.** Deciding what persists and what evaporates requires explicit design, not implicit hope.
- **Cache invalidation applies.** Stale information in the context window is worse than missing information — it produces confident wrong answers.
- **Prefetch strategies help.** Loading likely-needed context before the agent requests it reduces latency (progressive disclosure applied to memory).

### Memory Contamination

Incorrect information spreading across agents via shared memory is a named risk in every shared-pool implementation. The trend is toward fine-grained access control — restricting which agents can write to which memory segments — rather than all-or-nothing sharing. Read-many, write-few architectures reduce contamination surface.

**Contamination mitigation strategies observed:**

| Strategy | Mechanism | Trade-off |
|----------|-----------|-----------|
| **Write gating** | Only designated agents can write to shared memory | Reduces contamination but bottlenecks information flow |
| **Versioned entries** | Every write creates a new version; reads get latest or pinned | Enables rollback but increases storage cost |
| **Semantic decay** | Entries lose relevance weight over time unless refreshed | Self-cleaning but risks losing valid long-term knowledge |
| **Provenance tracking** | Every memory entry tagged with source agent and confidence | Enables blame tracing but adds metadata overhead |

### Seancing: Cross-Session Context Recovery

*[2026-02-11]*: Gas Town's "seancing" pattern addresses context loss at session boundaries. New agent sessions query predecessors about unfinished work, reconstructing intent from artifacts rather than relying on memory transfer. The pattern treats session boundaries as inevitable rather than preventable — building recovery into the architecture instead of trying to maintain continuity.

### Fresh Context as Deliberate Strategy

AutoForge takes the opposite approach: starting each session with an empty context window prevents context pollution while SQLite provides continuity for structured state. This trades recall for cleanliness — the agent cannot reference previous conversations but also cannot be poisoned by stale context from them.

The choice between seancing (recover prior context) and fresh starts (discard prior context) depends on whether the task benefits more from continuity or from clean reasoning.

---

## Autonomy — The Bounded Middle

The dominant strategy in 2026 is not maximum automation but intentional constraint. Bounded autonomy — clear operational limits combined with escalation paths and comprehensive audit trails — outperforms both excessive restriction and excessive freedom.

### Five-Level Autonomy Spectrum

| Level | Description | Agent Behavior | Human Role | Current Examples |
|-------|-------------|---------------|------------|------------------|
| **Augmentation** | Agents enhance human work | Suggest, draft, complete | Active creator | Most chat assistants, Copilot |
| **Supervised automation** | Agents execute with checkpoints | Act within approved scope | Reviewer at gates | OpenClaw (configurable thinking depth) |
| **Bounded autonomy** | Agents operate within defined limits | Full execution within boundaries | Supervisor, exception handler | AutoForge (spec + test gates), Google ADK |
| **High autonomy** | Agents execute relentlessly, humans review results | Execute until done or stuck | Post-hoc reviewer | Gas Town (GUPP: "must execute") |
| **Full autonomy** | Minimal human oversight | Theoretically unbounded | Absent or passive | Not achieved by any current system |

### Bounded Autonomy as Consensus

*[2026-02-11]*: Bounded autonomy is the 2026 consensus position. Full automation is not the dominant strategy — practitioners report better outcomes with intentional human checkpoints positioned at high-consequence decision points. The human role shifts from reviewing all outputs to functioning as an "agent supervisor" positioned at workflow decision points. The job changes from proofreader to factory floor manager.

### The Economics of Multi-Agent Systems

Multi-agent systems consume approximately 15x more tokens than standard single-agent chat (Anthropic production data). This cost multiplier means multi-agent architectures are economically justified only for tasks where the value of parallelism, specialization, or consistency exceeds the token premium.

| Architecture | Relative Token Cost | Best Justified When |
|-------------|-------------------|---------------------|
| Single agent | 1x (baseline) | Most tasks — default choice |
| Two-agent (generator-critic) | 2-3x | Quality-critical outputs needing verification |
| Orchestrator + 3-5 specialists | 5-8x | Multi-domain analysis requiring parallel expertise |
| Full swarm (10+ agents) | 10-15x | Large-scale parallel execution with tight deadlines |

**Role-based design reduces failure by 35%.** Separating Planner, Executor, Verifier, and Optimizer roles mirrors human team structure and produces measurable quality improvement over monolithic agent architectures. The specialization benefit compounds: planners develop better plans when they never execute, and executors produce better output when they follow explicit plans rather than self-planning.

### The 60% Failure Rate

*[2026-02-11]*: Multi-agent deployments fail in the majority of enterprises — not due to agent capability limitations but due to architectural misunderstanding of legacy systems and undocumented dependencies. The systems agents interact with were not designed for automated consumption. Undocumented APIs, implicit workflows, and tribal knowledge create failure surfaces that no amount of agent sophistication can navigate without explicit mapping.

### The GUPP Lesson: High Autonomy Without Proportional Safeguards

Gas Town's GUPP principle ("If there is work on your hook, you MUST run it") represents the high-autonomy extreme. The documented consequences: agents merged PRs with failing tests, deleted code unpredictably, and required forced repository resets. High autonomy without proportional safeguards produces chaos — the safeguards must scale with the latitude.

This is not an argument against high autonomy. It is evidence that autonomy level and safeguard depth must be calibrated together. The failure was not too much autonomy per se, but a mismatch between autonomy granted and constraints enforced.

### Autonomy Calibration Framework

Match autonomy level to task risk and reversibility:

```
Is the action easily reversible?
    Yes → Higher autonomy acceptable (bounded autonomy or above)
    No  → Continue

Is the action visible to external parties?
    Yes → Supervised automation (human gate before external actions)
    No  → Continue

Does the action modify shared state?
    Yes → Bounded autonomy with explicit checkpoints
    No  → Higher autonomy acceptable for isolated execution
```

**The calibration principle:** Autonomy level × safeguard depth should remain roughly constant. Increasing autonomy without proportionally increasing safeguards creates the mismatch that produced the GUPP failures. Decreasing safeguards without reducing autonomy produces the same mismatch from the other direction.

---

## Connections

- **To [Expert Swarm Pattern](8-expert-swarm-pattern.md):** File ownership and flat coordination represent a specific instantiation of the topologies and communication paradigms described here. Expert Swarm's expertise inheritance protocol is one implementation of the shared-state communication paradigm.

- **To [Production Multi-Agent Systems](11-production-multi-agent-systems.md):** Operational patterns for the topologies and safeguards discussed here. Production systems must implement the defense-in-depth stack and memory architectures at scale, with autonomous recovery for the failure modes this chapter identifies.

- **To [Multi-Agent Context](../4-context/4-multi-agent-context.md):** Memory architectures in this chapter extend the context isolation patterns documented there. The two-tier memory pattern (ephemeral + persistent) maps directly to context management strategies for multi-agent systems.

- **To [Operating Agent Swarms](../7-practices/7-operating-agent-swarms.md):** Operational practices for the safeguards and autonomy levels described here. The bounded autonomy consensus translates into specific operational procedures for monitoring, escalation, and recovery.

- **To [Execution Topologies](../8-mental-models/5-execution-topologies.md):** Autonomy as a measurement dimension for topology selection. The seven topologies documented here map to the execution topology mental models, with autonomy level as an additional selection criterion.

---

## Open Questions

- How do the communication protocols (MCP, A2A, ACP, ANP) compose when an agent needs both vertical and horizontal integration simultaneously?
- What governance structures prevent trust creep in organizations with hundreds of deployed agents?
- As memory architectures mature, does the two-tier pattern (ephemeral + persistent) stabilize or fragment into more specialized tiers?
- What observability standards emerge for tracing decisions across multi-agent topologies? Current debugging remains ad hoc.
- How does the bounded autonomy consensus shift as model capabilities improve — does the "bounded middle" move toward higher autonomy, or do safeguard requirements scale proportionally?
- What architectural patterns address the 60% enterprise failure rate — is the solution better agent design or better legacy system instrumentation? This question is addressed in [Context at Codebase Scale](../4-context/6-context-at-codebase-scale.md).
- How do memory contamination defenses compose with the performance benefits of shared memory pools?
- When (if ever) does full autonomy become viable, and what safeguard architecture makes it safe?
