---
title: "Gas Town: Multi-Agent Workspace Manager"
description: "Case study of Steve Yegge's Gas Town system for orchestrating 20-30 parallel coding agents"
created: 2026-02-11
last_updated: 2026-04-11
tags: [case-study, multi-agent, orchestration, gastown, production]
part: 4
part_title: Appendices
chapter: 10
section: 3
order: 4.10.3
---

# Gas Town: Multi-Agent Workspace Manager

Gas Town is a multi-agent orchestration system created by Steve Yegge for coordinating 20-30 parallel AI coding agents on real software projects. Built in Go (~189K lines of code), MIT licensed, with 9,000+ GitHub stars, Gas Town represents one of the most ambitious publicly documented attempts at production-scale agent coordination. Two core binaries power the system: `gt` (workspace manager) and `bd` (git-backed issue tracking).

The system matters because it confronts problems that most agent frameworks defer: persistent identity across sessions, structured work decomposition at scale, quality verification without human bottlenecks, and cost management when running dozens of concurrent agents at ~$100/hour in Claude API tokens.

---

## Core Questions

### Architecture and Design
- How does Gas Town maintain coordination across 20-30 simultaneous agents?
- What persistence mechanisms survive context exhaustion and session restarts?
- How does the role hierarchy distribute responsibilities without creating bottlenecks?

### Novel Mechanisms
- What does "persistent identity, ephemeral sessions" mean in practice?
- How do agent CVs enable capability-based routing?
- What role does "seancing" play in cross-session knowledge transfer?

### Production Realities
- What does $100/hour in token costs actually buy?
- How does the system handle the design bottleneck when agents write faster than humans can plan?
- What quality gates prevent low-quality code from reaching main branches?

### Broader Implications
- Which Gas Town patterns transfer to smaller-scale agent systems?
- Where does the zoomorphic naming convention help versus hinder adoption?
- What does Gas Town reveal about the future of human-agent collaboration?

---

## Your Mental Model

**Gas Town treats agents as a workforce, not a tool.** Where most agent frameworks model AI as a sophisticated function call, Gas Town models agents as employees in a factory: persistent identities with work histories, performance reviews, capability profiles, and structured career paths. The metaphor runs deep enough to include hiring (spawning), firing (termination for poor performance), and rehabilitation (re-assigning after failures).

**The steam engine metaphor is the core design principle.** Yegge describes the system through the Gas Town Universal Propulsion Principle (GUPP): "If there is work on your Hook, YOU MUST RUN IT." Agents are pistons in a steam engine. An idle piston is a system failure. This relentless forward motion, combined with git-backed persistence, means the system grinds forward even when individual agents fail, stall, or produce garbage.

---

## Philosophy

Gas Town rests on three named philosophical principles and five design axioms that distinguish it from conventional multi-agent frameworks.

### GUPP: Gas Town Universal Propulsion Principle

> "If there is work on your Hook, YOU MUST RUN IT."

GUPP eliminates the concept of an idle agent. Every agent (called a **Polecat** in Gas Town terminology) has a **Hook**---a queue of pending work items. When the hook contains work, the agent must execute. No negotiation, no prioritization delay, no waiting for permission. The analogy is to pistons in a steam engine: each piston fires when pressure arrives.

This creates a fundamentally different system dynamic from orchestrator-based approaches. In hub-and-spoke orchestration, the coordinator decides when agents work. In Gas Town, agents self-activate based on hook state. The coordinator (Mayor) manages *what work enters hooks*, not *when agents run*.

**Implications:**
- System throughput scales linearly with agent count (no coordinator bottleneck)
- Individual agent failures affect only their assigned work, not system-wide scheduling
- Monitoring reduces to hook inspection: empty hooks mean the system is idle or finished

### NDI: Nondeterministic Idempotence

Gas Town embraces the reality that AI agents are unreliable individual performers. **Nondeterministic Idempotence (NDI)** formalizes the principle that useful outcomes emerge from orchestration of unreliable processes, not from any single reliable execution.

Individual operations may fail, hallucinate, or produce incorrect code. NDI ensures eventual completion through:

1. **Persistent tracking** --- every work item survives individual session failures
2. **Oversight agents** --- supervisory roles (Witnesses, Deacons) detect and remediate stuck work
3. **Retry mechanisms** --- failed work returns to hooks for re-execution
4. **Quality gates** --- output validation catches errors before they propagate

The relationship to idempotence: re-running a failed operation should produce a useful outcome (even if not identical to the first attempt). The "nondeterministic" qualifier acknowledges that each execution may take a different path to completion.

### MEOW: Molecular Expression of Work

Gas Town decomposes work through a chemistry-inspired hierarchy called **MEOW (Molecular Expression of Work)**:

```
Formulas (TOML templates)
    └── Define reusable work patterns
         └── Protomolecules (frozen templates)
              └── Snapshots with specific parameters
                   └── Molecules (active instances)
                        └── Currently executing work units
                             └── Wisps (ephemeral patrol molecules)
                                  └── Lightweight monitoring/maintenance units
```

| Level | Purpose | Lifecycle | Example |
|-------|---------|-----------|---------|
| Formula | Reusable work template | Permanent | "Fix bug from issue" |
| Protomolecule | Parameterized snapshot | Until instantiated | "Fix bug #1234 in auth module" |
| Molecule | Active work unit | Until completion | Assigned to Polecat, executing |
| Wisp | Ephemeral maintenance | Minutes | Check if merge queue is stuck |

This decomposition enables work tracking at multiple granularities. A human designs at the Formula level. The system instantiates through Protomolecules. Agents execute at the Molecule level. Background maintenance runs as Wisps.

### Five Design Axioms

Gas Town codifies five non-negotiable design principles:

| Axiom | Statement | Implication |
|-------|-----------|-------------|
| 1. Attribution | Every action has an actor | Full audit trail; no anonymous operations |
| 2. Work as Data | Work is structured and queryable | Not just tickets; machine-parseable state |
| 3. History Matters | Track records determine trust | Agent CVs enable capability routing |
| 4. Scale Assumed | Multi-repo, multi-agent, multi-org from day one | No single-agent simplifications in core |
| 5. Verification over Trust | Quality gates are first-class citizens | Every output validated before acceptance |

These axioms interact. Attribution (1) feeds History (3), which enables Verification (5). Work as Data (2) makes Scale (4) tractable. Together they create a system where trust is earned incrementally through verifiable performance, not assumed through configuration.

---

## Architecture

### Role Taxonomy

Gas Town organizes agents into a hierarchical role taxonomy spanning two levels: **Town-level** (global coordination) and **Rig-level** (per-workspace execution).

```
TOWN LEVEL (Global)
├── Mayor ─────────── Global coordinator, work assignment
├── Deacon ────────── Background supervisor daemon (continuous patrol)
└── Dogs ──────────── Maintenance helpers
    └── Boot ──────── Health triage (ephemeral)

RIG LEVEL (Per-Workspace)
├── Polecat ───────── Worker agent (persistent identity, ephemeral sessions)
├── Crew ──────────── Human workspace
├── Witness ───────── Per-rig supervisor (quality monitoring)
└── Refinery ──────── Merge queue processor
```

**Town-Level Roles:**

| Role | Type | Lifecycle | Responsibility |
|------|------|-----------|----------------|
| Mayor | AI coordinator | Persistent | Global work assignment, formula instantiation, priority management |
| Deacon | AI supervisor | Continuous daemon | Background patrol, health monitoring, stuck-work detection |
| Boot | AI triage | Ephemeral | Health triage when Deacon detects anomalies |
| Dogs | AI helpers | Ephemeral | Maintenance tasks (cleanup, bookkeeping) |

**Rig-Level Roles:**

| Role | Type | Lifecycle | Responsibility |
|------|------|-----------|----------------|
| Polecat | AI worker | Persistent identity, ephemeral sessions | Execute molecules, produce code, submit to merge queue |
| Crew | Human | Persistent | Human workspace; direct coding, review, oversight |
| Witness | AI supervisor | Per-rig persistent | Monitor Polecat quality, flag issues, escalate to Deacon |
| Refinery | AI processor | Persistent | Merge queue management, conflict resolution, validation |

### Workspace Structure

Each Polecat operates in an isolated workspace (**rig**) with:
- Dedicated git worktree (independent branch)
- Isolated filesystem (no cross-agent interference)
- Private hook queue (work items assigned to this agent)
- Session context (loaded fresh each spawn)

The isolation guarantees that agent failures are contained. A Polecat that corrupts its workspace affects only its own rig. The Refinery handles merge conflicts when integrating work from multiple rigs into the main branch.

### Three-Tier Watchdog Chain

Supervision in Gas Town operates through a three-tier escalation:

```
Tier 1: Daemon (Go process)
│  Mechanical health checks
│  Process monitoring
│  Resource limits
│
▼
Tier 2: Boot → Deacon (AI agents)
│  Ephemeral triage (Boot) → Continuous patrol (Deacon)
│  Semantic analysis of stuck work
│  Cross-agent pattern detection
│
▼
Tier 3: Witnesses / Refineries (AI agents)
   Per-rig quality monitoring
   Merge queue validation
   Code review before main
```

**Tier 1 (Daemon)** is a Go process performing mechanical checks: Is the process alive? Has it exceeded resource limits? Is the session responsive? No AI reasoning involved---pure system monitoring.

**Tier 2 (Boot + Deacon)** introduces AI reasoning for triage. Boot is ephemeral, spawned when the Daemon detects an anomaly. Deacon is persistent, continuously patrolling for stuck work, stalled molecules, and cross-agent conflicts. Deacon can reassign work, restart agents, or escalate to humans.

**Tier 3 (Witnesses + Refineries)** provides per-rig quality supervision. Witnesses monitor individual Polecat output quality. Refineries validate merged code before it reaches main. These agents can request rework, reject submissions, or flag concerns.

### Mail Protocol

Inter-agent communication uses a typed message protocol with defined routes:

| Message Type | Sender | Receiver | Purpose |
|--------------|--------|----------|---------|
| `POLECAT_DONE` | Polecat | Refinery | Work complete, ready for merge |
| `MERGE_READY` | Refinery | Mayor | Merge successful, update tracking |
| `REWORK_REQUEST` | Witness/Refinery | Polecat | Quality insufficient, redo required |
| `HELP` | Any agent | Deacon | Agent stuck, needs intervention |
| `HANDOFF` | Polecat | Polecat | Transfer work between agents |

The typed message approach contrasts with unstructured natural language communication between agents. Each message type has a defined schema, enabling automated routing and monitoring. The system can track message flow rates to detect bottlenecks (too many `REWORK_REQUEST` messages from a specific Witness might indicate miscalibrated quality thresholds).

### Convoy-Based Work Tracking

Gas Town introduces **Convoys** as persistent tracking units that group related issues across repositories:

```
Convoy: "Authentication Overhaul"
├── Issue #1234 (repo: auth-service)
├── Issue #1235 (repo: user-api)
├── Issue #1236 (repo: frontend)
└── Issue #1237 (repo: shared-types)
    │
    ├── Swarm: [Polecat-A, Polecat-B, Polecat-C]  ← Ephemeral
    └── Convoy tracking ──────────────────────────── ← Persistent
```

The critical distinction: the **Swarm** (set of worker agents) is ephemeral. Agents come and go as sessions start and end. The **Convoy** persists, tracking overall progress regardless of which specific agents have contributed. This separation means work tracking survives agent turnover, session restarts, and even complete system reboots.

---

## Novel Contributions

Gas Town introduces several mechanisms not commonly found in existing agent frameworks.

### Persistent Identity, Ephemeral Sessions

Most agent frameworks treat each session as independent. Gas Town separates **identity** from **session**:

| Aspect | Persistent (Identity) | Ephemeral (Session) |
|--------|-----------------------|---------------------|
| What survives | Name, CV, work history, capability profile | Nothing---context window discarded |
| Storage | Git-backed Beads (JSONL + SQLite) | In-memory only |
| Purpose | Routing, trust, performance tracking | Actual task execution |
| Analogy | Employee record | Shift at work |

A Polecat named "Polecat-7" maintains its identity across hundreds of sessions. Each session is disposable---when context fills or the task completes, the session terminates and a fresh one spawns. The identity persists in git-backed storage, accumulating:

- **Work history**: What tasks were completed, success/failure rates
- **Capability profile**: Derived from performance (strong at Go, weak at CSS)
- **Trust level**: Accumulated through verified successful completions

### Agent CVs and Capability Routing

Every Polecat accumulates a **CV (Curriculum Vitae)** documenting work history. The system uses CVs for:

1. **Capability matching**: Route Go work to agents with proven Go track records
2. **Performance management**: Identify consistently failing agents for reassignment or termination
3. **Model A/B testing**: Compare performance across different underlying models
4. **Skill derivation**: Infer capabilities from evidence rather than static configuration

```
Polecat CV Structure:
├── Identity
│   ├── Name: Polecat-7
│   ├── Model: claude-opus-4-6
│   └── Created: 2026-01-15
├── Work History
│   ├── Tasks completed: 347
│   ├── Success rate: 89%
│   ├── Avg completion time: 12 min
│   └── Rework rate: 8%
├── Capability Profile (derived)
│   ├── Go: Strong (142 tasks, 94% success)
│   ├── TypeScript: Moderate (89 tasks, 85% success)
│   ├── CSS: Weak (23 tasks, 61% success)
│   └── Testing: Strong (93 tasks, 91% success)
└── Recent Performance
    ├── Last 7 days: 92% success
    ├── Trend: Improving
    └── Flags: None
```

This evidence-based routing contrasts with static capability declarations. Rather than configuring "Agent X handles Go," the system observes "Agent X succeeds at Go 94% of the time" and routes accordingly.

### Seancing: Cross-Session Knowledge Transfer

When a new session spawns for a Polecat, it must recover context from prior sessions. Gas Town calls this process **seancing**---querying predecessors' records to reconstruct relevant knowledge.

The seancing process:

1. New session loads Polecat identity from persistent storage
2. Queries work history for current molecule/convoy context
3. Retrieves relevant prior session summaries
4. Loads current molecule state (where in the workflow)
5. Resumes execution with reconstructed context

This solves a fundamental problem in long-running agent work: context window exhaustion. Rather than attempting to preserve a single massive context, Gas Town treats context as renewable. Each session gets a fresh window, populated through targeted retrieval from persistent state.

**Trade-offs:**

| Benefit | Cost |
|---------|------|
| Fresh context prevents cascading errors | Reconstruction loses nuance from prior sessions |
| Survives crashes and restarts | Latency cost for context reconstruction |
| Enables indefinite work duration | Depends on quality of persistent summaries |
| Scales to arbitrarily complex projects | Information compression introduces lossy abstraction |

### Self-Cleaning Workers

Polecats exist in exactly three states. No idle state exists.

```
┌─────────┐     gt hook      ┌──────────┐
│ Working  │ ◄──────────────  │ Spawned  │
└────┬─────┘                  └──────────┘
     │
     │ gt done
     ▼
┌──────────┐   push branch    ┌──────────┐   nuke sandbox
│ Stalled  │ ──────────────►  │ Submitting│ ──────────────► Session terminated
└──────────┘                  └──────────┘

     │ (timeout)
     ▼
┌──────────┐
│  Zombie  │ ──────────────► Deacon intervenes
└──────────┘
```

| State | Meaning | Duration | Exit Condition |
|-------|---------|----------|----------------|
| Working | Actively executing a molecule step | Minutes to hours | Step completion or stall |
| Stalled | No progress detected | Minutes | Deacon intervention or timeout |
| Zombie | Unresponsive, presumed dead | N/A | Forced termination and restart |

The `gt done` command triggers a complete cleanup sequence:
1. Push branch to remote
2. Submit work to merge queue (Refinery)
3. Destroy sandbox (nuke local workspace)
4. Terminate session

No cleanup tasks remain. No state lingers. The next session starts completely fresh. This eliminates an entire category of bugs related to stale state, partial cleanup, and resource leaks.

### Refinery Merge Queue

Workers never push directly to main. Every code change flows through the **Refinery**, a dedicated agent that manages the merge queue:

```
Polecat-A ──► branch-a ──┐
Polecat-B ──► branch-b ──┤
Polecat-C ──► branch-c ──┼──► Refinery ──► main
Polecat-D ──► branch-d ──┤
Polecat-E ──► branch-e ──┘
```

The Refinery performs:
1. **Validation**: Tests pass, linting clean, no regressions
2. **Conflict resolution**: Merge conflicts between concurrent branches
3. **Re-imagination**: When conflicts are complex, the Refinery can re-implement a change against the updated main rather than attempting mechanical merge
4. **Ordering**: Sequence merges to minimize conflict cascades

The "re-imagination" capability is particularly notable. When two Polecats modify overlapping code, a simple three-way merge often fails. The Refinery can read both changes, understand the intent, and produce a fresh implementation that incorporates both goals. This is fundamentally different from traditional merge resolution.

---

## The Propulsion Loop

Gas Town's execution model reduces to a tight loop that drives all agent work:

```
┌─────────────────────────────────────────────┐
│                                             │
│  1. gt hook        → What's hooked?         │
│  2. bd mol current → Where in molecule?     │
│  3. Execute step   → Do the work            │
│  4. bd close <step> --continue              │
│     └── Close step, auto-advance to next    │
│  5. GOTO 2                                  │
│                                             │
└─────────────────────────────────────────────┘
```

Step 4 is critical: `bd close <step> --continue` atomically closes the current step and advances the molecule to the next step. This prevents the **Batch-Closure Heresy**---closing steps retroactively, which corrupts the timeline and makes audit trails unreliable.

The loop continues until:
- All molecule steps complete (success)
- A step fails and cannot be retried (escalation)
- The session hits context limits (seancing into a new session)
- The Deacon intervenes (external termination)

---

## Production Realities

### Cost and Economics

Gas Town operates at significant token cost:

| Metric | Value | Context |
|--------|-------|---------|
| Hourly cost | ~$100 | 20-30 concurrent agents on Claude API |
| Monthly estimate | $1,000-$3,000 | Sustainable development pace |
| Target developer level | "Level 7-8" | Senior engineers managing agent swarms |
| Agent concurrency | 10-30 | Typical working set |

The economics work when agent output replaces or augments expensive senior engineering time. At $100/hour for 20-30 agents, the cost per agent-hour is $3-5---far below human developer rates. The question is whether agent output quality justifies the cost, which the CV system attempts to measure empirically.

### Quality Trade-offs

Gas Town makes explicit quality trade-offs:

**Accepted:**
- Individual agent outputs may be mediocre (NDI compensates through persistence and retry)
- Some branches will be rejected by the Refinery (expected, not a failure)
- Human review remains necessary for design decisions

**Mitigated:**
- Witness agents provide continuous quality monitoring
- Refinery validates all code before main
- CV system routes work to capable agents
- Rework requests catch quality issues early

**Unresolved:**
- PRs sometimes require human cleanup before production readiness
- Complex architectural decisions exceed current agent capabilities
- Agent coordination overhead increases superlinearly with agent count

### Named Anti-Patterns

Gas Town explicitly names and prohibits two anti-patterns:

**The Idle Polecat Heresy**

An idle worker is a system failure, not a waiting state. If a Polecat has no work on its hook, the system has failed to distribute work properly. The Mayor should always have more work queued than agents available. Slack in the system represents wasted capacity.

This contrasts with frameworks that model agent idle time as normal. In Gas Town, idle detection triggers investigation: Why is this agent idle? Is work distribution stuck? Has the Mayor failed to decompose remaining work?

**The Batch-Closure Heresy**

Closing molecule steps retroactively (rather than sequentially as completed) corrupts the timeline. Every step must close in execution order with accurate timestamps. Batch-closing steps after the fact (e.g., "close steps 1-5 because they're all done") produces an audit trail that doesn't reflect actual execution sequence.

This matters because Gas Town derives trust from verifiable history. If timestamps are inaccurate, performance metrics become unreliable, capability routing degrades, and the entire evidence-based trust system erodes.

### The Design Bottleneck

Gas Town's most provocative insight concerns the shifting bottleneck in software development:

```
Traditional Software Development:
    Design ──► Code ──► Test ──► Deploy
    (Fast)     (SLOW)   (Slow)   (Medium)
                 ▲
                 └── Bottleneck: Writing code

Gas Town at Scale:
    Design ──► Code ──► Test ──► Deploy
    (SLOW)     (Fast)   (Fast)   (Medium)
      ▲
      └── Bottleneck: Human design decisions
```

When 20-30 agents can write code faster than humans can review it, **design becomes the bottleneck**. The scarce resource shifts from coding capacity to human judgment about *what* to build and *how* to architect it.

This has implications beyond Gas Town:
- Planning and specification become the highest-leverage activities
- Human time should concentrate on design, architecture, and review
- Agent management becomes a distinct skill ("Level 7-8 developer" managing agents)
- The value of clear specifications increases when agents consume them directly

---

## Scale Considerations

### Git-Backed Persistence

All Gas Town state resides in git-backed storage called **Beads**:

| Component | Format | Purpose |
|-----------|--------|---------|
| Beads (current) | JSONL + SQLite | Event log + queryable state |
| Beads (planned migration) | Dolt | Git-for-data with SQL queries |

Git-backed persistence provides:
- **Crash recovery**: State survives any failure (process, session, machine)
- **Audit trail**: Complete history of every state change
- **Branching**: Experimental states without affecting production tracking
- **Distribution**: State replicates through standard git mechanisms

The planned migration to Dolt (a git-compatible database) suggests that JSONL + SQLite reaches limits at Gas Town's scale. Dolt would provide SQL query capabilities over versioned data---useful for the CV system's performance analytics.

### Scaling Boundaries

| Dimension | Current Limit | Bottleneck |
|-----------|--------------|------------|
| Agent count | 20-30 | Human design bandwidth |
| Repo count | Multi-repo | Convoy tracking complexity |
| Session duration | Context-limited | Seancing reconstruction quality |
| Merge throughput | Refinery capacity | Sequential merge queue |

The merge queue represents a potential bottleneck at scale. With 30 agents producing concurrent changes, the Refinery must process merges sequentially to maintain consistency. Multiple Refineries could parallelize this, but conflict resolution becomes more complex.

---

## Criticisms and Limitations

### Naming Complexity

Gas Town's zoomorphic naming convention (Polecats, Wisps, Deacons, Dogs, Refineries, Convoys) creates a steep learning curve. Each concept maps to a functional role, but the metaphorical distance between the name and the function requires memorization:

| Gas Town Term | Functional Equivalent |
|---------------|----------------------|
| Polecat | Worker agent |
| Wisp | Ephemeral monitoring task |
| Deacon | Supervisor daemon |
| Boot | Triage agent |
| Rig | Isolated workspace |
| Molecule | Active work unit |
| Convoy | Cross-repo work group |
| Refinery | Merge queue processor |

The naming provides internal consistency and a vivid mental model (the "steam-powered town" metaphor holds together), but it adds cognitive overhead for newcomers. Practitioners must learn both the metaphor and the underlying concepts.

### Early Maturity

As of early 2026, Gas Town is production-capable but not production-proven at scale across diverse organizations. Key maturity indicators:

- **Single-team validation**: Primarily tested by Yegge's own team
- **Go-centric**: Built in Go, tested primarily on Go projects
- **Cost accessibility**: $100/hour limits adoption to well-funded teams
- **Review overhead**: PRs sometimes need human cleanup, offsetting some productivity gains

### Cognitive Load

Managing 20-30 concurrent agents creates what observers describe as "palpable stress." The human operator must:
- Monitor agent progress across multiple rigs
- Review merge queue submissions
- Make design decisions for blocked agents
- Investigate Deacon escalations
- Maintain formula quality for new work types

This cognitive load represents a genuine constraint on scaling. Even if the system supports 50 agents, human attention may not.

---

## Lessons for Practitioners

Gas Town's architecture encodes several principles applicable to multi-agent systems at any scale.

### 1. Separate Identity from Session

Even at smaller scales (3-5 agents), maintaining persistent agent identity enables:
- Performance tracking across sessions
- Capability-based work routing
- Debugging through work history analysis
- Progressive trust building

Implementation need not be as elaborate as Gas Town's CV system. A simple JSON log per agent identity captures the core benefit.

### 2. Design for Agent Failure

NDI formalizes what practitioners discover empirically: agents fail unpredictably. Systems that assume agent reliability build fragile architectures. Systems that assume agent *unreliability* and design recovery mechanisms build resilient ones.

Practical applications:
- Track work items independently from agents executing them
- Implement automatic retry for failed tasks
- Use quality gates before accepting agent output
- Monitor for stuck/zombie agents and intervene automatically

### 3. Invest in Merge Infrastructure

As agent count grows, merge infrastructure becomes critical. Gas Town's Refinery pattern---never letting agents push directly to main---prevents the "merge chaos" that occurs when multiple agents modify overlapping code simultaneously.

Even with 2-3 agents, a merge queue (even a simple one) prevents:
- Conflicting changes reaching main simultaneously
- Test failures from incompatible concurrent modifications
- Loss of work when manual conflict resolution goes wrong

### 4. Make Work Decomposition Explicit

Gas Town's Formula/Protomolecule/Molecule hierarchy may be over-engineered for smaller systems, but the underlying principle transfers: **work decomposition should be structured data, not implicit understanding.**

When work items are structured and queryable:
- Progress tracking becomes automatic
- Bottleneck detection becomes observable
- Agent routing becomes data-driven
- Recovery from failures becomes systematic

### 5. Accept the Design Bottleneck

Gas Town demonstrates that scaling agent execution capacity shifts the bottleneck to human design capacity. Practitioners benefit from:
- Investing disproportionately in specification quality
- Treating agent management as a dedicated skill
- Designing work decomposition templates (Formulas) for repeated patterns
- Accepting that human attention---not compute---limits throughput

### 6. Build Observable Systems

Gas Town's attribution axiom ("every action has an actor") enables comprehensive observability. The principle: if an agent did it, the system knows which agent, when, and in what context. This traceability is essential for:
- Debugging production failures
- Identifying consistently problematic agents
- Measuring actual productivity (not just activity)
- Building trust in agent outputs through verifiable history

---

## Comparison with Other Approaches

| Dimension | Gas Town | Claude Code Agent Teams | Traditional Orchestrator |
|-----------|----------|------------------------|------------------------|
| Agent identity | Persistent (CVs, histories) | Ephemeral (per-session) | Ephemeral (per-task) |
| Coordination | Hook-based self-activation | Peer-to-peer messaging | Hub-and-spoke |
| Work tracking | Convoy + Molecule hierarchy | Shared task list | Orchestrator state |
| Quality gates | Witness + Refinery chain | Human review | Orchestrator validation |
| Scale target | 20-30 agents | 2-10 agents | 2-10 subagents |
| Persistence | Git-backed Beads | None (session-scoped) | None (session-scoped) |
| Cost model | ~$100/hr (20-30 agents) | Variable (per-session) | Variable (per-task) |
| Design philosophy | Workforce management | Team collaboration | Function composition |
| Failure handling | NDI (eventual completion) | Manual retry | Orchestrator retry |
| Merge strategy | Refinery (dedicated agent) | Manual or CI/CD | Manual or CI/CD |

---

## Technical Implementation Details

### The `gt` Binary (Workspace Manager)

Core commands:

| Command | Purpose |
|---------|---------|
| `gt hook` | Check what work is pending on this agent's hook |
| `gt done` | Complete current work, push branch, nuke sandbox, terminate |
| `gt spawn` | Create a new rig (workspace) for an agent |
| `gt status` | View system-wide agent and work status |

### The `bd` Binary (Issue Tracking)

Git-backed issue tracking commands:

| Command | Purpose |
|---------|---------|
| `bd mol current` | Show current molecule and step |
| `bd close <step> --continue` | Atomically close step and advance |
| `bd create` | Create a new issue/work item |
| `bd query` | Search work items with structured filters |

### System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Gas Town System                       │
│                                                         │
│  ┌─────────┐   Formulas    ┌──────────────────────┐    │
│  │  Mayor   │ ────────────► │  Work Distribution   │    │
│  └─────────┘               │  (Hook Assignment)    │    │
│       │                    └──────────┬───────────┘    │
│       │                               │                 │
│       │          ┌────────────────────┼──────────┐     │
│       │          │                    │          │     │
│       │     ┌────▼────┐   ┌────▼────┐  ┌────▼────┐  │
│       │     │ Rig-1   │   │ Rig-2   │  │ Rig-N   │  │
│       │     │Polecat-A│   │Polecat-B│  │Polecat-N│  │
│       │     │Witness-1│   │Witness-2│  │Witness-N│  │
│       │     └────┬────┘   └────┬────┘  └────┬────┘  │
│       │          │             │             │        │
│       │          └──────┬──────┘─────────────┘        │
│       │                 │                              │
│       │          ┌──────▼──────┐                       │
│       │          │  Refinery   │                       │
│       │          │ (Merge Queue)│                       │
│       │          └──────┬──────┘                       │
│       │                 │                              │
│  ┌────▼────┐     ┌──────▼──────┐                       │
│  │ Deacon  │     │    main     │                       │
│  │(Patrol) │     │  (branch)   │                       │
│  └─────────┘     └─────────────┘                       │
│                                                         │
│  ┌─────────────────────────────────────┐               │
│  │         Git-Backed Beads            │               │
│  │  (JSONL + SQLite → Dolt planned)    │               │
│  └─────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────┘
```

---

## Open Questions

- **Cross-organization scaling**: How does Gas Town's trust model extend when multiple organizations contribute agents? Does the CV system need federation?
- **Model diversity**: What happens when different Polecats run different underlying models? The CV system tracks performance, but model-specific capabilities may confound routing.
- **Formula evolution**: How should Formulas evolve as agent capabilities improve? Static templates may encode outdated assumptions about agent limitations.
- **Merge queue throughput**: At 50+ agents, does the single-Refinery model become a bottleneck? Can multiple Refineries coordinate without introducing merge conflicts?
- **Human attention scaling**: The design bottleneck suggests a ceiling on agent count tied to human design bandwidth. What tools or processes could raise this ceiling?
- **Quality convergence**: Does the CV system converge to stable capability assessments, or do model updates and task variation prevent convergence?
- **Cost optimization**: Can Gas Town's economics improve through model cascading (using cheaper models for simpler molecules)?
- **Seancing fidelity**: How much information is lost during seancing? Can retrieval-augmented approaches improve cross-session knowledge transfer?

---

## Sources

### Primary Sources
- **Steve Yegge** --- Gas Town creator, blog posts on Medium documenting design philosophy and architecture
- **DoltHub Blog** --- "A Day in Gas Town" (detailed operational walkthrough)
- **GitHub** --- github.com/steveyegge/gastown (MIT licensed, ~189K LOC Go)

### Analysis and Commentary
- **Maggie Appleton**, "Gas Town's Agent Patterns, Design Bottlenecks, and Vibecoding at Scale" (January 2026) --- Designer-perspective analysis framing Gas Town as speculative design fiction rather than a production blueprint; introduces six-factor code-proximity framework; argues Gas Town itself is "poorly designed" because Yegge did not deliberate which metaphors and primitives to use — a productive counterpoint to Yegge's own case-study framing. https://maggieappleton.com/gastown/
- **TWiT Network** --- Coverage of Gas Town and multi-agent development trends
- **Pragmatic Engineer Newsletter** --- Industry context for AI-assisted development at scale

---

## Connections

- **To [Orchestrator Pattern](../../../chapters/6-patterns/3-orchestrator-pattern.md):** Gas Town's Mayor operates as an orchestrator but delegates scheduling to hook-based self-activation, contrasting with hub-and-spoke orchestration where the coordinator controls execution timing. The "push work to hooks" model eliminates the coordinator bottleneck at scale.

- **To [Expert Swarm Pattern](../../../chapters/6-patterns/8-expert-swarm-pattern.md):** Gas Town's Polecat swarm shares the parallel-workers-with-shared-context model, but adds persistent identity, CV-based routing, and structured work decomposition (MEOW) that the Expert Swarm pattern leaves to the orchestrator.

- **To [Autonomous Loops](../../../chapters/6-patterns/4-autonomous-loops.md):** The Propulsion Loop resembles the Ralph Wiggum pattern (while loop with fresh context), but Gas Town adds structured step tracking (molecules), supervision (Witnesses/Deacon), and quality gates (Refinery) around the core loop.

- **To [Workflow Coordination](../../../chapters/7-practices/5-workflow-coordination.md):** Gas Town implements workflow coordination through git-backed Beads and typed mail protocols, demonstrating a production-scale implementation of "structured metadata as coordination layer." Convoys extend the pattern to cross-repository work tracking.

- **To [Production Concerns](../../../chapters/7-practices/4-production-concerns.md):** The three-tier watchdog chain (Daemon/Deacon/Witness) represents a comprehensive production supervision architecture. The named anti-patterns (Idle Polecat Heresy, Batch-Closure Heresy) codify operational discipline rarely documented in agent frameworks.

- **To [Execution Topologies](../../../chapters/8-mental-models/5-execution-topologies.md):** Gas Town primarily employs a persistent parallel topology with nested sequential execution within molecules. The Convoy structure maps to the nested topology pattern, while the Refinery represents a synthesis node.

- **To [Multi-Model Architectures](../../../chapters/3-model/4-multi-model-architectures.md):** Gas Town's CV system enables empirical model routing---tracking which underlying model performs best for which task types---rather than relying on static model assignment. This evidence-based approach complements the cost-optimization strategies documented in multi-model architecture design.

- **To [Context Strategies](../../../chapters/4-context/2-context-strategies.md):** Seancing represents a novel context strategy---targeted retrieval from persistent per-agent state rather than from a shared knowledge base. Combined with the ephemeral session model, it demonstrates an alternative to the "preserve context at all costs" approach.
