---
title: "Design as Bottleneck"
description: "Mental models for multi-agent systems at scale — when implementation is automated, design becomes the constraint"
created: 2026-02-11
last_updated: 2026-02-11
tags: [mental-models, multi-agent, scale, design, gastown, theory-of-constraints]
part: 3
part_title: Perspectives
chapter: 8
section: 6
order: 3.8.6
---

# Design as Bottleneck

When 20-30 agents churn through implementation in minutes, the constraint moves upstream. Design, architecture, and decomposition become the scarce resources. This section presents five mental models for thinking about agentic systems at scale—each grounded in production evidence from multi-agent systems like Gas Town.

---

## Core Questions

### Constraint Identification
- Where does the bottleneck actually sit when implementation is automated?
- What happens to traditional software roles when coding is no longer the slow part?
- How does Theory of Constraints apply to agentic workflows?

### Execution Models
- Are agents conversational partners or execution units?
- What execution contract eliminates idle-agent overhead?
- How does persistent identity coexist with ephemeral sessions?

### Scale and Accountability
- When does a "workshop" become a "factory floor"?
- What infrastructure appears at each scale transition?
- How does attribution change system behavior over time?

---

## Model 1: Design as Bottleneck

**When implementation is automated, design becomes the constraint.**

### The Core Idea

The Theory of Constraints (Goldratt, 1984) states that every system has exactly one bottleneck limiting throughput. In traditional software development, that bottleneck is often implementation—writing, testing, and debugging code. Developers spend weeks translating designs into working software.

Multi-agent systems invert this. A swarm of 20-30 agents can implement a well-decomposed feature set in minutes. The implementation phase, once the bottleneck, becomes nearly instantaneous relative to design. The constraint moves upstream.

```
Traditional Software Development:

Design          Implementation          Testing
  │                  │                    │
  ▼                  ▼                    ▼
┌──────┐     ┌──────────────┐      ┌──────────┐
│ Fast │────►│   BOTTLENECK │─────►│ Moderate │
│ 2 hrs│     │   2 weeks    │      │  3 days  │
└──────┘     └──────────────┘      └──────────┘


Multi-Agent Development:

Design              Implementation          Testing
  │                      │                    │
  ▼                      ▼                    ▼
┌──────────────┐   ┌──────────┐        ┌──────────┐
│  BOTTLENECK  │──►│   Fast   │───────►│ Moderate │
│   2-4 hours  │   │ 5 minutes│        │  30 min  │
└──────────────┘   └──────────┘        └──────────┘
```

The implementation rectangle shrank by orders of magnitude. But design didn't shrink at all—it may even have grown, because now the specification must be precise enough for agents to execute without ambiguity.

### What This Replaces

| Old Model | New Model |
|-----------|-----------|
| "The hard part is coding" | "The hard part is decomposition" |
| Invest in faster typing, better IDEs | Invest in specification quality, architectural thinking |
| Developer productivity = lines/hour | Developer productivity = specifications/hour |
| Junior devs bottleneck on syntax | Junior devs bottleneck on design clarity |
| "Ship faster" = more developers | "Ship faster" = better decomposition |

### Why This Happens

Three properties of multi-agent implementation create the shift:

1. **Parallelism eliminates serial implementation time.** Twenty agents working simultaneously reduce wall-clock time by 10-20x compared to a single developer, even accounting for coordination overhead.

2. **Agents don't need ramp-up time.** A human developer joining a feature spends hours understanding context. An agent receives its specification and begins immediately.

3. **Agent throughput scales with specification quality.** Vague specifications produce incorrect implementations that require rework. Precise specifications produce correct implementations on the first pass. The specification is the leverage point.

### Evidence

*[2026-02-11]*: Gas Town (multi-agent orchestration platform) users report that issue decomposition quality directly determines swarm output quality. The system churns through implementation so quickly that design and planning become the dominant time cost. Teams that invest in decomposition skills consistently outperform those optimizing implementation speed.

### Implications for Practitioners

- **Invest in decomposition skills.** The ability to break complex problems into well-scoped, independent units is the highest-leverage skill in agentic systems.
- **Treat specifications as the primary artifact.** See [Specs as Source Code](3-specs-as-source-code.md)—specifications are not throwaway scaffolding; they are the program.
- **Measure design throughput, not implementation throughput.** Track how quickly well-formed specifications emerge, not how quickly agents write code.
- **Front-load architectural decisions.** Ambiguity in architecture creates cascading failures across parallel agents. Resolve architectural questions before spawning the swarm.

---

## Model 2: Agents as Pistons

**Agents are not conversational partners. They are pistons in an engine.**

### The Core Idea

Most mental models for AI agents draw from human interaction: agents as assistants, collaborators, or team members. These metaphors import conversational expectations—agents should acknowledge tasks, negotiate scope, report readiness, and confirm understanding.

The piston model discards all of this. A piston fires when compressed gas expands. It doesn't confirm, negotiate, or report status. The mechanical contract is the assignment: compression happens, the piston fires.

```
Conversational Model:              Piston Model:

Orchestrator: "Can you do X?"      ┌─────────────┐
Agent: "Let me check..."           │   Hook fires │
Agent: "Yes, I can do X"           │      │       │
Orchestrator: "Please proceed"     │      ▼       │
Agent: "Starting X now..."         │   Agent runs │
Agent: "X is 50% complete..."      │      │       │
Agent: "X is done"                 │      ▼       │
                                   │   Work done  │
  7 messages                       └─────────────┘
  ~3,500 tokens overhead
                                     0 messages
                                     0 tokens overhead
```

### The Execution Contract

The piston model replaces conversation with a contract: **if there is work on your hook, you must run it.** No waiting for confirmation. No polling for instructions. No announcing readiness and idling.

This contract has three properties:

1. **The hook IS the assignment.** Placing work on an agent's hook is the complete instruction. The agent does not need to be "told" to start—the presence of work is the trigger.

2. **Execution is immediate.** There is no negotiation phase. The agent begins work the moment it detects its hook has fired.

3. **Completion is the only signal.** The agent communicates exactly once: when the work is done. No progress updates, no status checks, no mid-task conversations.

### What This Replaces

| Conversational Model | Piston Model |
|---------------------|--------------|
| Agents announce readiness | Agents are always ready |
| Orchestrator assigns tasks via messages | Hook fires, agent runs |
| Progress updates flow continuously | Completion is the only signal |
| Agents negotiate scope | Scope is defined by the hook payload |
| Idle agents wait for instructions | No idle state exists |
| Multi-turn assignment protocol | Zero-turn assignment |

### The Idle Agent Anti-Pattern

The conversational model creates a characteristic failure: **agents that announce readiness and wait.** An agent that says "I'm ready for my next task" is consuming resources while producing nothing. Worse, it creates a coordination dependency—something must respond to the readiness announcement.

```
Idle Agent Anti-Pattern:

Agent A: "Ready for work"         ─── idle ───
                                              │
Orchestrator: "Here's task 3"     ◄───────────┘
                                              │
Agent A: "Starting task 3"        ─── idle ───
                                              │
Agent A: [actual work begins]     ◄───────────┘

Time wasted: orchestrator latency + 2 message round trips
```

In the piston model, this scenario cannot occur. There is no "ready" state. There is only "working" or "not instantiated."

### When This Model Applies

**Good fit:**
- High-throughput systems with many agents
- Tasks with clear, complete specifications
- Workflows where coordination overhead dominates
- Systems where agent count exceeds 5-10

**Poor fit:**
- Exploratory tasks requiring clarification
- Creative work needing iterative refinement
- Tasks where the specification is genuinely incomplete
- Single-agent interactions where conversation is natural

### Evidence

*[2026-02-11]*: Gas Town's architecture embodies the piston model through its "propulsion principle"—GUPP (Gas Town Universal Propulsion Principle): if there is work on your hook, you must run it. This principle drives the entire system architecture, eliminating idle-agent overhead and enabling predictable throughput scaling. The absence of negotiation reduces per-task token overhead to near zero.

---

## Model 3: Persistent Identity, Ephemeral Execution

**Agents are like employees—permanent identity, but each workday is fresh.**

### The Core Idea

Two failing models dominate thinking about agent state:

1. **Persistent sessions** that accumulate context until they bloat and degrade. The agent "remembers everything" but drowns in irrelevant history.
2. **Fully stateless agents** that start fresh every time. The agent "forgets everything" and repeats mistakes, rediscovers solutions, and cannot learn.

The middle path separates **who** from **how**:

```
┌─────────────────────────────────────────────────────┐
│                PERSISTENT IDENTITY                   │
│                                                      │
│  Name: Agent-7                                       │
│  Role: Security Reviewer                             │
│  Skills: [auth, crypto, input-validation]            │
│  Track Record: 47 reviews, 3 critical finds          │
│  Known Patterns: prefers OWASP checklist approach    │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │Session 1 │  │Session 2 │  │Session 3 │  ...      │
│  │          │  │          │  │          │           │
│  │ Fresh    │  │ Fresh    │  │ Fresh    │           │
│  │ context  │  │ context  │  │ context  │           │
│  │ window   │  │ window   │  │ window   │           │
│  │          │  │          │  │          │           │
│  │ Dispose  │  │ Dispose  │  │ Dispose  │           │
│  │ after    │  │ after    │  │ after    │           │
│  └──────────┘  └──────────┘  └──────────┘           │
│                                                      │
│             EPHEMERAL EXECUTION                      │
└─────────────────────────────────────────────────────┘
```

The persistent layer holds identity, skills, track record, and accumulated expertise. The ephemeral layer holds the current session's context window—tools, files, and working memory for the immediate task. Sessions are disposable. Identity accumulates.

### The Three Layers

| Layer | Persists Across Sessions | Contains | Size |
|-------|------------------------|----------|------|
| **Identity** | Always | Name, role, skills, configuration | Small (~500 tokens) |
| **History** | Always (append-only) | Past session summaries, decisions, outcomes | Growing (compressed) |
| **Session** | Never | Current context, working files, tool state | Large (fills context window) |

### Seancing: Querying the Past

When an agent needs context from a prior session, it performs what Gas Town calls **seancing**—querying past session records for decisions and rationale. This is analogous to institutional memory in organizations: no employee remembers every meeting, but meeting notes make past decisions retrievable.

```
Current Session:

Agent: "What authentication approach did we choose for the API?"

  │
  ▼  [searches session history]

Session 12 Summary:
  "Chose JWT with RS256 over session cookies.
   Rationale: stateless scaling, mobile client support.
   Decision maker: Architecture-Agent-3."

  │
  ▼

Agent: [proceeds with JWT context loaded]
```

Seancing differs from loading full session history. It is targeted retrieval—searching for specific decisions, patterns, or outcomes rather than replaying everything.

### What This Replaces

| Failing Model | Problem | Middle Path |
|--------------|---------|-------------|
| **Persistent sessions** | Context bloat, degraded performance after 10+ interactions | Sessions are ephemeral; dispose after task |
| **Stateless agents** | Cannot learn, repeats mistakes, no institutional memory | Identity and history persist; sessions query them |
| **Full history replay** | Prohibitive token cost, irrelevant context dilution | Seancing retrieves targeted decisions only |

### Identity Accumulation in Practice

The persistent identity layer grows through structured updates:

```yaml
# agent-7-cv.yaml (persists across all sessions)
name: Agent-7
role: security-reviewer
sessions_completed: 47
specializations:
  - authentication-flows
  - input-validation
  - cryptographic-implementations
notable_outcomes:
  - session_12: "Identified JWT key rotation gap"
  - session_31: "Caught SQL injection in parameterized query"
  - session_45: "Flagged timing attack in comparison logic"
patterns_learned:
  - "Check revocation lists when JWT validation present"
  - "Verify constant-time comparison for all secret comparisons"
```

Each session adds to the CV. New sessions restore context by reading the CV, not by replaying old sessions. The identity grows richer while each session starts clean.

### Evidence

*[2026-02-11]*: Gas Town implements this model through "polecats"—agents with persistent CVs that accumulate track records across sessions. New sessions restore context via hook-based initialization that loads identity and queries relevant history. The CV structure enables capability-based routing: the orchestrator assigns security reviews to agents with demonstrated security expertise, not to the next available agent.

### Implications for Practitioners

- **Design identity schemas early.** What persists across sessions determines what agents can learn. Invest in the structure of the persistent layer.
- **Make sessions disposable by default.** The temptation to preserve session state "in case it's useful" leads to bloat. Default to disposal; query when needed.
- **Build seancing into workflows.** Agents should actively query past decisions when entering domains they've worked in before. This is not automatic—it requires explicit tooling.
- **Use CVs for routing.** Agent identity enables intelligent task assignment. An orchestrator that knows Agent-7 found three critical security issues routes security-sensitive work to Agent-7.

---

## Model 4: Work as Ledger

**Every agent action is a timestamped entry in a permanent record.**

### The Core Idea

Most agent systems treat execution as ephemeral—work happens, results appear, the process vanishes. The ledger model treats execution as accounting: every action, decision, and outcome is a timestamped, attributed entry in a permanent record.

```
Fire-and-Forget Model:            Ledger Model:

Task → Agent → Result             Task → Agent → Result
                                        │
                                        ▼
                                   ┌──────────────────────┐
                                   │ LEDGER               │
                                   │                      │
                                   │ 14:23 Agent-7 READ   │
                                   │   auth.py (247 lines)│
                                   │                      │
                                   │ 14:24 Agent-7 FOUND  │
                                   │   JWT validation gap │
                                   │   severity: critical │
                                   │                      │
                                   │ 14:25 Agent-7 WROTE  │
                                   │   fix.patch (12 lines)│
                                   │   model: opus-4      │
                                   │   tokens: 3,847      │
                                   │                      │
                                   │ 14:26 Agent-7 DONE   │
                                   │   result: success    │
                                   └──────────────────────┘
```

### What the Ledger Enables

Attribution is not a compliance afterthought—it is system intelligence. The ledger enables capabilities that fire-and-forget systems cannot support:

| Capability | How the Ledger Enables It |
|-----------|--------------------------|
| **Debugging** | Trace failures to specific agent actions and decisions |
| **Capability routing** | Route tasks to agents with proven track records |
| **Performance management** | Identify agents that consistently produce quality vs. rework |
| **Model comparison** | Compare output quality across models (opus vs. sonnet) for the same task type |
| **Cost attribution** | Know which agents and tasks consume the most tokens |
| **Regression detection** | Detect when agent performance degrades over time |

### The Batch-Closure Heresy

One anti-pattern threatens ledger integrity: **closing entries retroactively.** When an agent marks work as complete after the fact—or worse, when a supervisor closes entries on behalf of agents—the ledger loses its causal ordering.

```
Correct Ledger:                    Corrupted Ledger:

14:23 START task-42                14:23 START task-42
14:25 READ auth.py                 14:50 [BATCH CLOSE]
14:27 FOUND vulnerability            - READ auth.py     ← no timestamp
14:28 WROTE fix.patch                - FOUND vuln       ← retroactive
14:29 COMPLETE task-42                - WROTE fix        ← rewritten
                                      - COMPLETE task-42 ← bundled

Causal order preserved.            Causal order destroyed.
Debugging: straightforward.        Debugging: impossible.
```

Batch closure is tempting because it reduces logging overhead. But it destroys the property that makes ledgers valuable: the ability to reconstruct exactly what happened, in what order, and why.

### Ledger vs. Logging

The ledger model is not simply "add logging." Logs are diagnostic artifacts—searched when something breaks. The ledger is an operational artifact—queried continuously for routing, attribution, and improvement.

| Property | Logging | Ledger |
|----------|---------|--------|
| **Purpose** | Diagnosis after failure | Continuous system intelligence |
| **Queried** | When something breaks | Every task assignment, every review |
| **Retention** | Rotated, compressed, archived | Permanent, append-only |
| **Structure** | Free-form text | Structured entries with agent, action, outcome |
| **Attribution** | Optional | Required for every entry |
| **Consumers** | Humans debugging | Orchestrators, agents, and humans |

### When This Model Applies

**Good fit:**
- Multi-agent systems where multiple agents contribute to outcomes
- Systems requiring auditability or compliance
- Long-running projects where agent performance must improve over time
- Environments comparing multiple models or agent configurations

**Poor fit:**
- Single-agent, single-session interactions
- Prototyping where overhead must be minimal
- Tasks where the process is less important than the result
- Ephemeral scripts and one-off automation

### Evidence

*[2026-02-11]*: Gas Town implements the ledger model through its "Bead" system—a permanent work record where every agent action is a timestamped, attributed entry. The Bead system enables capability routing (assigning tasks based on past performance), regression detection (flagging agents whose quality has declined), and cost attribution (tracking token spend per agent and task type). The "Batch-Closure Heresy"—closing entries retroactively—is explicitly forbidden because it corrupts the causal ordering that makes the ledger useful.

### Implications for Practitioners

- **Design attribution into the system from day one.** Retrofitting attribution onto an existing system is orders of magnitude harder than building it in.
- **Make the ledger queryable, not just writable.** A ledger that can only be read by humans is a log. A ledger that orchestrators query for routing decisions is system intelligence.
- **Enforce causal ordering.** Every entry must have a timestamp and an agent attribution. Batch closures destroy the causal structure.
- **Use the ledger for routing.** Agent assignment should consider past performance on similar tasks, not just availability.

---

## Model 5: Factory Floor vs. Workshop

**The transition from 3 agents to 30 is not gradual—it is a phase change.**

### The Core Idea

Two distinct models exist for organizing agentic work:

```
WORKSHOP (1-3 agents):            FACTORY FLOOR (10-30 agents):

┌─────────────────────┐           ┌──────────────────────────────────┐
│                     │           │  ┌──────────┐   ┌──────────┐    │
│  Human ◄──► Agent   │           │  │Supervisor│   │  Health   │    │
│         ◄──► Agent  │           │  │  Agent   │   │ Monitor  │    │
│                     │           │  └─────┬────┘   └──────────┘    │
│  Direct management  │           │        │                         │
│  Conversational     │           │  ┌─────┼──────────────────┐     │
│  Each agent visible │           │  │     │   Merge Queue    │     │
│                     │           │  │  ┌──┴──┐ ┌─────┐ ┌───┐│     │
└─────────────────────┘           │  │  │A│B│C│ │D│E│F│ │...││     │
                                  │  │  └─────┘ └─────┘ └───┘│     │
                                  │  └────────────────────────┘     │
                                  │                                  │
                                  │  Infrastructure required:        │
                                  │  - Supervisors                   │
                                  │  - Merge queues                  │
                                  │  - Health monitoring              │
                                  │  - Conflict resolution           │
                                  │  - Work attribution              │
                                  └──────────────────────────────────┘
```

The workshop is intimate. A human directly manages each agent, sees all output, and handles coordination through conversation. This works well for 1-3 agents—the cognitive load is manageable, and the overhead of infrastructure is not justified.

The factory floor is industrial. No human can directly manage 20+ agents. Infrastructure replaces direct management: supervisors route work, merge queues prevent conflicts, health monitors detect failures, and attribution tracks accountability.

### The Phase Change

The transition between these models is not gradual. It is a phase change—a discontinuity where the old model stops working and new infrastructure becomes necessary.

```
Agent Count:    1    3    5    8    10   15   20   30
                │    │    │    │    │    │    │    │
Workshop:       ████████████████░░░░░░░░░░░░░░░░░░
                              ▲
                              │ Phase transition
                              │ (chaos zone)
                              ▼
Factory:        ░░░░░░░░░░░░░░████████████████████

Infrastructure needed at each level:

1-3 agents:   Nothing. Direct management works.
4-7 agents:   Merge strategy. File conflicts appear.
8-12 agents:  Supervisors. Humans cannot track all agents.
13-20 agents: Health monitoring. Silent failures occur.
20+ agents:   Full infrastructure. Attribution, queues,
              conflict resolution, automated routing.
```

### Scale Infrastructure Table

| Agent Count | Infrastructure Required | Why |
|-------------|------------------------|-----|
| 1-3 | None | Human directly manages each agent |
| 4-7 | **Merge strategy** | Multiple agents editing overlapping files |
| 8-12 | **Supervisors** | Human cannot track status of all agents simultaneously |
| 13-20 | **Health monitoring** | Silent agent failures go undetected without automated checks |
| 20-30 | **Full orchestration** | Attribution, queuing, conflict resolution, automated routing |
| 30+ | **Hierarchical management** | Single orchestrator cannot manage all agents; sub-orchestrators needed |

### Two Failure Modes

**Premature Factory Infrastructure:**
Building supervisor agents, merge queues, and health monitors for a 2-agent system. The overhead exceeds the value. Simple direct management works better. The infrastructure becomes complexity without benefit—bureaucracy for a team of two.

**Delayed Factory Infrastructure:**
Running 15 agents with workshop-style direct management. Merge conflicts multiply, silent failures go undetected, and the human becomes the bottleneck—unable to review all output, missing critical issues, and drowning in coordination overhead. The system appears to work but produces lower quality than a well-managed 5-agent workshop.

### Steve Yegge's Eight Levels

Steve Yegge's framework for AI-assisted development describes a spectrum from Level 1 (AI as autocomplete) to Level 8 (AI-first development). Most practitioners operate at Levels 1-4 (workshop). Factory-floor systems like Gas Town target Levels 7-8, where the development model fundamentally changes.

| Level Range | Mode | Agent Relationship |
|-------------|------|-------------------|
| 1-2 | Autocomplete, chat | AI assists human coding |
| 3-4 | Pair programming, delegation | AI handles defined subtasks |
| 5-6 | Supervised autonomy | AI works independently with checkpoints |
| 7-8 | Factory floor | Human designs, AI swarm implements |

The levels are not a maturity model where everyone should reach Level 8. They describe different modes suitable for different contexts. Most tasks are well-served at Levels 3-4. Factory-floor orchestration is necessary only when the task scope genuinely demands 10+ concurrent agents.

### What This Replaces

| Workshop Assumption | Factory Floor Reality |
|--------------------|----------------------|
| "Add more agents to go faster" | More agents without infrastructure causes chaos |
| "I can manage 10 agents like I manage 2" | Cognitive load grows superlinearly with agent count |
| "Infrastructure is premature optimization" | Infrastructure becomes necessary at ~8 agents |
| "Scaling is linear" | Scaling has phase transitions requiring new architecture |

### Evidence

*[2026-02-11]*: Gas Town's entire architecture is factory-floor infrastructure—supervisors, health monitoring, merge queues, work attribution, and automated routing. The system targets Yegge's Level 7-8 operations, where 20-30 agents execute in parallel. Teams that attempt Gas Town's scale without its infrastructure (or equivalent) consistently encounter the chaos zone: merge conflicts, silent failures, and human bottlenecks. The architecture itself is evidence that the phase transition is real and requires deliberate infrastructure investment.

### When to Transition

The decision to move from workshop to factory is not about ambition—it is about observable symptoms:

```
Transition Signals (Workshop → Factory):

  Merge conflicts appearing?
  ├── No → Stay in workshop
  └── Yes
       └── Agent failures going undetected?
            ├── No → Add merge strategy only
            └── Yes
                 └── Human overwhelmed by review volume?
                      ├── No → Add health monitoring
                      └── Yes → Build factory infrastructure
```

Each symptom maps to specific infrastructure. Build only what the symptoms demand—premature infrastructure is its own failure mode.

---

## The Models in Combination

These five models form an interlocking system. Each addresses a different aspect of multi-agent work at scale:

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│  DESIGN AS BOTTLENECK                                  │
│  (what to invest in)                                   │
│         │                                              │
│         ▼                                              │
│  ┌──────────────┐     ┌────────────────────────┐       │
│  │ AGENTS AS    │     │ PERSISTENT IDENTITY,   │       │
│  │ PISTONS      │     │ EPHEMERAL EXECUTION    │       │
│  │              │     │                        │       │
│  │ (how agents  │     │ (how agents persist    │       │
│  │  execute)    │     │  across sessions)      │       │
│  └──────┬───────┘     └───────────┬────────────┘       │
│         │                         │                    │
│         └──────────┬──────────────┘                    │
│                    │                                   │
│                    ▼                                   │
│         ┌──────────────────┐                           │
│         │ WORK AS LEDGER   │                           │
│         │                  │                           │
│         │ (how work is     │                           │
│         │  tracked)        │                           │
│         └────────┬─────────┘                           │
│                  │                                     │
│                  ▼                                     │
│         ┌──────────────────┐                           │
│         │ FACTORY FLOOR    │                           │
│         │ vs. WORKSHOP     │                           │
│         │                  │                           │
│         │ (what scale      │                           │
│         │  demands)        │                           │
│         └──────────────────┘                           │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Design as Bottleneck** identifies where to invest: upstream, in decomposition and specification.

**Agents as Pistons** defines the execution contract: fire-and-forget, no conversation overhead.

**Persistent Identity, Ephemeral Execution** solves the state management problem: agents learn without bloating.

**Work as Ledger** provides accountability: every action is recorded, attributed, and queryable.

**Factory Floor vs. Workshop** determines what infrastructure these models require at each scale level.

Each model is useful independently. Together, they describe a coherent philosophy for building multi-agent systems that scale.

---

## Anti-Patterns

### Conversational Orchestration at Scale

**The mistake:** Using multi-turn message exchanges to assign work to 15+ agents. "Agent-3, are you available? Great, here's task 7. Agent-3, how's it going? Agent-3, are you done yet?"

**Why it fails:** Orchestration overhead grows quadratically with agent count. At 20 agents, the orchestrator spends more tokens coordinating than agents spend working.

**The fix:** Piston model. Hook fires, agent runs, completion signal returns. Zero conversational overhead.

### Stateless Swarms

**The mistake:** Spawning fresh, identity-free agents for every task. No history, no expertise accumulation, no capability routing.

**Why it fails:** The system cannot improve. Agent-7's excellent security review in Session 12 is invisible to Session 13's task assignment. Every task is assigned to a random agent regardless of fit.

**The fix:** Persistent identity with ephemeral execution. CVs accumulate track records. Orchestrators query CVs for routing.

### Fire-and-Forget Without Attribution

**The mistake:** Agents produce outputs but the system does not record who did what, when, or how. Results appear but the process is opaque.

**Why it fails:** Debugging requires replaying entire sessions. Performance improvement is impossible because there is no data. Cost optimization is guesswork because token spend is not attributed.

**The fix:** Ledger model. Every action is a timestamped, attributed entry. The ledger is queryable by orchestrators, not just searchable by humans.

### Workshop Mentality at Factory Scale

**The mistake:** A human attempting to directly manage 20 agents the same way they manage 2. Reading every output, making every routing decision, manually resolving every conflict.

**Why it fails:** The human becomes the bottleneck. Agent throughput is limited by human review speed. The entire point of scaling—parallel throughput—is negated by serial human review.

**The fix:** Factory floor infrastructure. Supervisors, health monitoring, merge queues, automated routing. The human designs and reviews; infrastructure manages execution.

---

## Open Questions

- **Where is the next bottleneck after design?** If design automation improves (agents generating specifications from high-level intent), what becomes the new constraint? Requirements? Strategy? Taste?

- **Can the piston model support creative work?** Creative tasks may require the conversational back-and-forth that the piston model eliminates. Is there a hybrid model that preserves execution efficiency for implementation while allowing conversation for creative exploration?

- **What is the optimal CV structure?** Too detailed and the CV consumes excessive context. Too sparse and routing decisions lack signal. What information density maximizes routing quality per token?

- **How large can ledgers grow before they become unwieldy?** Append-only ledgers grow without bound. What compression, summarization, or archival strategies maintain queryability without unbounded growth?

- **Is the phase transition predictable?** The workshop-to-factory transition appears to occur around 8-12 agents, but this number likely varies by domain and task complexity. Can the transition point be predicted from task characteristics?

- **What happens above 100 agents?** Gas Town operates at 20-30 agents. Model-native swarm systems (e.g., Kimi K2.5) demonstrate 100+ concurrent subagents. Does factory-floor infrastructure scale to this level, or does a third organizational model emerge?

- **How does the ledger interact with privacy constraints?** In regulated environments, permanent attribution records may conflict with data retention policies. How does the ledger model adapt to privacy requirements?

---

## Connections

- **[Execution Topologies](5-execution-topologies.md):** The five topologies describe *shapes* of agent work; these models describe *principles* for operating agent systems at scale. Factory floor infrastructure enables wider and deeper topologies. The piston model reduces friction in the measurement framework.

- **[Specs as Source Code](3-specs-as-source-code.md):** Design as Bottleneck is the logical consequence of treating specs as source code. If specifications are the primary programming surface, and implementation is automated, then specification quality is the constraint.

- **[Pit of Success](1-pit-of-success.md):** The piston model is a pit-of-success design for agent execution. The easiest path (hook fires, agent runs) is also the correct path. No negotiation means no negotiation failures.

- **[Self-Improving Experts](../6-patterns/2-self-improving-experts.md):** Persistent Identity, Ephemeral Execution is the mental model behind self-improving experts. The pattern (expertise files, session learning) implements the model (persistent identity, ephemeral sessions).

- **[Orchestrator Pattern](../6-patterns/3-orchestrator-pattern.md):** Factory floor infrastructure is what orchestrator patterns become at scale. The orchestrator pattern describes coordination; Factory Floor vs. Workshop describes when coordination requires dedicated infrastructure.

- **[Expert Swarm Pattern](../6-patterns/8-expert-swarm-pattern.md):** Expert swarms operate at factory-floor scale. The swarm pattern implements the piston model (agents as execution units) and the ledger model (work attribution across swarm members).

- **[Workflow Coordination](../7-practices/5-workflow-coordination.md):** Operational implementation of the infrastructure that factory-floor scale demands—merge strategies, health monitoring, and conflict resolution.

- **[Context Fundamentals](../4-context/1-context-fundamentals.md):** Ephemeral execution is a context management strategy. Disposing sessions and loading identity via CV keeps context windows fresh and relevant.

- **[Operating Agent Swarms](../7-practices/7-operating-agent-swarms.md)**: Operational practices for when the factory floor is running—cost management, incident response, and scale transitions that emerge once design bottlenecks are addressed.

---

## Sources

- Eliyahu M. Goldratt, *The Goal* (1984) — Theory of Constraints: every system has exactly one bottleneck limiting throughput
- Gas Town multi-agent orchestration platform — production evidence for piston model, persistent identity, ledger system, and factory-floor infrastructure
- Steve Yegge, "Eight Levels of AI-Assisted Development" — framework for classifying human-AI development modes
- Rico Mariani, .NET Framework design — original "Pit of Success" concept, applied here to agent execution contracts
