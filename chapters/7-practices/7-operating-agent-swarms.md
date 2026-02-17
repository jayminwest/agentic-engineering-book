---
title: "Operating Agent Swarms"
description: "Practices for running multi-agent systems at production scale, from cost management to incident response"
created: 2026-02-11
last_updated: 2026-02-13
tags: [practices, multi-agent, production, operations, cost, scale, gastown, overstory]
part: 2
part_title: Craft
chapter: 7
section: 7
order: 2.7.7
---

# Operating Agent Swarms

Running one agent is engineering. Running thirty is operations. The shift from single-agent development to multi-agent production introduces failure modes, cost dynamics, and coordination problems that don't exist at smaller scales. These practices emerge from production deployments running 20-30 concurrent agents on real codebases.

---

## Core Questions

### Economics
- What does it actually cost to run a multi-agent swarm in production?
- How does cost scale with agent count, and where are the optimization levers?
- At what point does the investment pay for itself?

### Operations
- What breaks when scaling from 1 agent to 30?
- How does incident response differ for agent failures vs. traditional software failures?
- What infrastructure emerges at each scale level?

### Human Factors
- What becomes the bottleneck when agents can implement faster than humans can design?
- How does a practitioner's role change when operating swarms?
- What cognitive load limits exist, and how are they managed?

---

## Your Mental Model

**Operating an agent swarm resembles running a factory floor, not writing software.** The daily experience of managing 20-30 concurrent agents has more in common with manufacturing operations than traditional engineering. Tasks flow through pipelines. Quality gates catch defects. Watchdogs monitor machine health. The practitioner's role shifts from writing code to designing work, routing tasks, and monitoring throughput.

**Agent economics follow linear scaling with diminishing human leverage.** Each additional agent consumes tokens independently—doubling agents roughly doubles cost. But human oversight doesn't scale linearly. One person can meaningfully monitor 3-5 agents. Beyond that, infrastructure must replace human attention: automated quality gates, structured attribution, escalation chains. The investment in infrastructure determines the ceiling on useful parallelism.

---

## Cost Realities of Multi-Agent Operations

### What Swarms Actually Cost

*[2026-02-11]*: Production evidence from Gas Town, a multi-agent development system running 20-30 concurrent agents on real codebases:

| Metric | Value | Context |
|--------|-------|---------|
| Hourly token cost | ~$100/hour | 20-30 agents running Claude models |
| Monthly sustainable budget | $1,000-$3,000 | Typical practitioner spend |
| Cost scaling | Linear with agent count | Each agent consumes tokens independently |
| Cost per agent-hour | ~$3-5 | Varies by model and task complexity |

**Linear scaling is the key constraint.** Unlike traditional compute where parallelism introduces sublinear cost growth through shared resources, agent swarms scale linearly. Agent 30 costs as much as agent 1. There are no economies of scale at the token level.

*[2026-02-13]*: Overstory's subscription-based cost model (Claude Code Pro subscription, fixed monthly cost) contrasts with Gas Town's API token model (~$100/hr for 20-30 agents). The fixed-cost approach changes economic incentives: practitioners optimize for throughput (maximize agents within subscription limits) rather than per-token efficiency. This shifts the bottleneck from token budget to subscription tier limits and human design capacity. The divergence suggests cost model choice (subscription vs pay-per-use) fundamentally shapes operational strategy.

### Cost Optimization Levers

Not all agents need the same model. Production swarms use **model selection per worker** to optimize cost without sacrificing quality where it matters.

| Task Type | Model Tier | Rationale |
|-----------|-----------|-----------|
| Orchestration, synthesis | Frontier (Opus-class) | Requires planning, judgment, coordination |
| Code implementation | Mid-tier (Sonnet-class) | Follows specs, doesn't need deep reasoning |
| Linting, formatting, migration | Fast/cheap (Haiku-class) | Mechanical transforms, pattern matching |
| Code review, security audit | Frontier | Requires nuanced analysis |

**Agent CVs enable model A/B testing.** When agents maintain work histories (see [Attribution and Accountability](#attribution-and-accountability)), deploying different models through different workers produces objective comparison data. Run half the implementation agents on Sonnet and half on a newer model, then compare completion rates, rework frequency, and quality gate pass rates.

### The Investment Reframe

At $100/hour, a swarm of 25 agents costs roughly $800 for a full workday. Compare this against the alternative:

| Approach | Daily Cost | Daily Output |
|----------|-----------|-------------|
| 25-agent swarm | ~$800 | 25 parallel implementation streams |
| 1 senior engineer | ~$600-800 (loaded) | 1 sequential implementation stream |
| 5 engineers | ~$3,000-4,000 (loaded) | 5 parallel streams + coordination overhead |

The swarm matches a senior engineer's daily cost while delivering 25× the parallel throughput—if the work is decomposable. The "if" matters. Poorly decomposed work produces 25 agents stepping on each other, not 25× throughput.

**See Also**: [Cost and Latency: Real-World ROI](3-cost-and-latency.md#real-world-roi-what-12kmonth-buys) — Economic analysis of single-agent ROI

---

## Design as the New Bottleneck

### The Constraint Shifts Upstream

When 20-30 agents can implement simultaneously, the limiting factor is no longer implementation capacity. It's the human's ability to design, decompose, and specify work at a rate that keeps the swarm fed.

```
Traditional Bottleneck:
  Design (fast) → Implement (slow) → Review (fast)
                   ^^^^^^^^^^^^^^^^
                   Human writes code

Swarm Bottleneck:
  Design (slow) → Implement (fast) → Review (moderate)
  ^^^^^^^^^^^^^
  Human designs and decomposes work
```

*[2026-02-11]*: Gas Town production experience confirms this pattern: "The system churns through implementation so quickly that design and planning become the bottleneck." A swarm of 25 agents can consume well-specified issues faster than a single practitioner can create them.

### Issue Decomposition Quality Drives Throughput

The quality of issue decomposition directly determines swarm throughput. Poorly specified issues cause agents to:
- Block on ambiguous requirements
- Produce work that conflicts with other agents
- Require rework that consumes more tokens than the original implementation

**Decomposition quality checklist:**

- [ ] Each issue is independently implementable (no hidden dependencies)
- [ ] Acceptance criteria are machine-verifiable (tests, linting, type checks)
- [ ] File scope is explicit (which files each agent owns)
- [ ] Interface contracts are defined (how components interact)
- [ ] Expected output format is specified (PR structure, commit convention)

### Practitioner Implications

The shift changes what skills matter:

| Traditional Engineering | Swarm Operations |
|------------------------|------------------|
| Writing code | Decomposing work into parallel streams |
| Debugging implementations | Debugging specifications |
| Code review | Research and plan review |
| Individual productivity | System throughput optimization |
| Technical depth | Architectural breadth |

**Invest in decomposition skills, not just prompt engineering.** The highest-leverage skill for swarm operators is breaking complex work into well-specified, independent units. A practitioner who can decompose a feature into 15 non-overlapping issues keeps 15 agents productive. A practitioner who writes brilliant prompts but produces tangled specifications keeps 15 agents blocked.

**See Also**: [High-Leverage Review](_index.md#high-leverage-review) — Why reviewing plans matters more than reviewing code

---

## Attribution and Accountability

### Every Action Traces to a Specific Agent

**Attribution is not optional.** At 20-30 concurrent agents, "something broke" is useless without "agent-17 broke it at 14:32 while modifying auth.ts." Attribution enables three capabilities that are impossible without it:

1. **Debugging** — Which agent introduced the regression? Check its work history.
2. **Capability routing** — Which agents handle TypeScript well? Check their success rates.
3. **Compliance** — Who changed the security policy? Check the audit trail.

### Implementation: Agent Identity and Work Records

Every agent action includes structured identity metadata:

```json
{
  "agent_id": "builder-17",
  "task_id": "ISSUE-342",
  "action": "edit",
  "file": "src/auth/login.ts",
  "timestamp": "2026-02-11T14:32:00Z",
  "model": "claude-sonnet-4-5-20250929",
  "session_id": "sess_abc123"
}
```

**Agent CVs** accumulate work history over time:

| Metric | Purpose |
|--------|---------|
| Tasks completed | Volume indicator |
| Quality gate pass rate | First-pass quality |
| Rework frequency | How often output needs correction |
| Average tokens per task | Efficiency indicator |
| Model version | Track behavior changes across model updates |
| Domain distribution | What types of work this agent handles |

These records enable objective performance evaluation. Instead of guessing which model or configuration works best, compare actual production data across agents.

*[2026-02-13]*: Overstory implements attribution through SQLite metrics store (`.overstory/metrics.db`) with per-agent session tracking. Agent CVs accumulate in `.overstory/agents/{name}/identity.yaml` with structured work history. The TypeScript implementation demonstrates that attribution infrastructure is language-agnostic—both Go (Gas Town) and TypeScript (Overstory) converge on persistent identity + structured logging as foundation for capability routing and performance analysis.

### Attribution Enables Capability Routing

With sufficient work history, the system can route tasks to agents with proven track records:

```
New task: "Refactor authentication module"

Agent capability lookup:
  builder-03: 12 auth tasks, 92% pass rate → Best candidate
  builder-11: 3 auth tasks, 67% pass rate  → Developing
  builder-17: 0 auth tasks                  → No data

Route to: builder-03
```

This moves from random assignment to evidence-based routing. The quality improvement compounds—agents that succeed at certain tasks get more of those tasks, building deeper context.

### When to Adopt Attribution

**Day one.** Retrofitting attribution onto an existing swarm is expensive and lossy. Start with structured logging from the first multi-agent deployment:

- Session start: log agent identity, model, task assignment
- Every tool use: log action, target, parameters
- Session end: log outcomes, quality gate results, token usage

The logging overhead is negligible relative to the token cost of agent operations. The debugging value is enormous.

---

## Quality Gates at Scale

### Verification Over Trust

**Quality gates are first-class primitives, not optional enhancements.** The anti-pattern of trusting agent output without verification scales dangerously. A single agent producing subtly wrong code is one bug. Thirty agents producing subtly wrong code is a systemic failure.

### The Merge Queue Model

Production swarms enforce a strict flow:

```
Agent Work → Quality Gate → Merge Queue → Main Branch
     │            │              │
     │            ├─ Lint pass   ├─ AI validation
     │            ├─ Type check  ├─ Conflict detection
     │            ├─ Test pass   └─ Ordering
     │            └─ Scope check
     │
     └─ Workers never push directly to main
```

**Workers never push directly to main.** Every change flows through a merge queue with automated validation. This is non-negotiable at scale—30 agents pushing directly to main produces merge conflicts, broken builds, and untraceable regressions.

*[2026-02-13]*: Overstory's merge queue implements 4-tier resolution escalation (Clean → Auto-Resolve → AI-Resolve → Re-Imagine) as a quality gate. Each tier validates correctness before advancing: Tier 1-2 validate syntactically (clean merge), Tier 3 validates semantically (AI understands intent), Tier 4 reimplements against current state (quality over merge archaeology). The escalation structure demonstrates that quality gates at scale require multiple resolution strategies, not binary pass/fail.

### Human Review at Scale: Sampling-Based

At 30 concurrent agents, reviewing every PR in detail is physically impossible. Production practice shifts to **sampling-based review**:

| Review Strategy | When to Apply |
|----------------|---------------|
| Full review | Architectural changes, security-sensitive code, new abstractions |
| Spot-check | Routine implementations that passed all quality gates |
| Skip (trust gates) | Mechanical transforms: formatting, imports, renames |
| Statistical sampling | Review 20-30% of PRs randomly to calibrate gate accuracy |

The goal of sampling isn't catching every defect—it's calibrating the automated gates. If sampled reviews consistently find issues that gates miss, the gates need tightening.

### Quality Feedback Loops

When quality gates reject work, the feedback must be structured for agent consumption:

```
Rejection:
  agent: builder-17
  task: ISSUE-342
  gate: type-check
  error: "Type 'string' is not assignable to type 'AuthToken'"
  file: src/auth/login.ts:47
  context: "AuthToken requires {token: string, expiry: Date}, received plain string"

Action: Rework with structured context
```

**Structured rejection context** prevents agents from flailing. "Type error on line 47" is less actionable than the full context of what was expected and what was provided. The quality gate produces the diagnosis; the agent implements the fix.

### Anti-Pattern: Trust Creep

The most dangerous failure mode at scale is **trust creep**—gradually relaxing quality gates because "agents usually get it right." The pattern:

1. Quality gates catch few issues (agents are working well)
2. Gates feel like unnecessary overhead
3. Gates get loosened or bypassed
4. Subtle quality degradation goes undetected
5. Systemic issues compound until a major failure

Resist this. Quality gates exist for the failure mode, not the success mode.

---

## Safeguards Across the Ecosystem

### Defense-in-Depth as Baseline

*[2026-02-11]*: Production multi-agent systems in early 2026 converge on layered security rather than single-point controls. Each layer catches what the layer above misses:

| Layer | Mechanism | What It Catches |
|-------|-----------|-----------------|
| OS-level sandbox | gVisor, Kata containers, macOS Seatbelt | Syscall-level violations, arbitrary code execution |
| Filesystem restrictions | Tiered access (none / read-only / read-write) | Unauthorized file modification |
| Command allowlists | Only permitted commands execute | Shell injection, unauthorized tooling |
| Agent config protection | Block writes to CLAUDE.md, .cursorrules, MCP configs | Configuration tampering by agents |
| Credential injection | Minimal at start, task-required only, short-lived tokens | Credential sprawl, persistent access |

**Application-level controls are insufficient alone.** Attackers use indirection—calling restricted tools through approved ones. An agent denied direct shell access might invoke a build tool that shells out internally. OS-level enforcement (intercepting syscalls before they reach the host kernel) catches these indirection attacks regardless of the execution path.

### The Command Allowlist Pattern

AutoForge implements a concrete example: a `security.py` file defines exactly which bash commands agents can execute—file inspection tools (`ls`, `cat`, `grep`), development tools (`npm`, `node`, `git`), and process management (`ps`, `sleep`). Everything outside the allowlist is blocked. This simple mechanism prevents entire categories of unintended agent behavior.

The pattern generalizes beyond AutoForge. Any multi-agent system benefits from an explicit command allowlist rather than relying on implicit trust. The allowlist acts as a capability boundary: agents operate freely within it and hit a hard wall outside it.

### Opt-In Communication as a Safeguard

*[2026-02-11]*: OpenClaw defaults agent-to-agent communication to OFF. Cross-talk requires explicit allowlisting per agent pair via configuration (`tools.agentToAgent: { enabled: false, allow: ['home', 'work'] }`). This treats inter-agent communication as a capability to be granted rather than a default to be restricted—a meaningful philosophical difference that prevents uncontrolled coordination between agents.

The principle extends beyond communication. Every capability an agent possesses—file access, network access, tool invocation, peer messaging—represents a potential attack surface or failure mode. Defaulting to off and requiring explicit enablement produces systems where the blast radius of any single compromise is bounded by the permissions actually granted.

### Prompt Injection Remains Unsolved

Every framework studied acknowledges prompt injection as an open problem. Mitigations include defense-in-depth (locked DMs, mention gating, instruction-hardened models), but no framework claims prevention. The practical stance across the ecosystem: assume injections will occur, limit blast radius, audit aggressively.

This has operational implications for swarm operators. Quality gates (see [Quality Gates at Scale](#quality-gates-at-scale) above) catch many consequences of injection—anomalous outputs, scope violations, unexpected file modifications—but they catch symptoms, not the injection itself. Layered safeguards ensure that even a successfully injected agent operates within constrained boundaries.

**Cross-reference**: See [The Multi-Agent Landscape: Safeguards](../6-patterns/10-multi-agent-landscape.md#safeguards--bounded-autonomy-as-standard) for the full ecosystem analysis of how production frameworks implement these layers.

---

## Incident Response for Agent Failures

### Agent Failure States

Agent failures don't look like traditional software crashes. Three distinct states require different responses:

| State | Symptoms | Diagnosis Challenge |
|-------|----------|-------------------|
| **Working** | Producing output, making progress | No issue (but verify quality) |
| **Stalled** | No output, but process is alive | Thinking deeply? Stuck in a loop? Waiting on input? |
| **Zombie** | Process alive, producing tokens, but output is meaningless | Appears active but context is degraded or hallucinating |

The **Stalled vs. Thinking** distinction is the hardest. An agent that has been silent for 3 minutes might be processing a complex task or might be stuck in an infinite reasoning loop. Mechanical checks ("is the process alive?") can't distinguish these cases. Intelligent checks ("has the agent made progress on its stated goal?") require understanding the task.

### Three-Tier Watchdog Architecture

Production swarms use layered monitoring with escalating intelligence:

```
Tier 1: Mechanical Watchdog (automated, continuous)
  │  ├─ Process heartbeat (is agent process alive?)
  │  ├─ Token flow (is agent producing/consuming tokens?)
  │  ├─ Time bounds (has agent exceeded task time limit?)
  │  └─ Resource limits (memory, CPU, disk)
  │
  ▼ Escalate if anomaly detected

Tier 2: AI Triage (intelligent, on-demand)
  │  ├─ Analyze agent's recent output (is it coherent?)
  │  ├─ Compare progress against task specification
  │  ├─ Classify failure state (stalled, zombie, confused)
  │  └─ Attempt automated recovery (restart, reprompt, context refresh)
  │
  ▼ Escalate if recovery fails

Tier 3: Human Intervention (expert, exceptional)
     ├─ Review triage analysis
     ├─ Decide: restart, reassign, or manual fix
     ├─ Update specifications if root cause is ambiguity
     └─ Adjust watchdog thresholds based on incident
```

**Tier 1 runs continuously** with negligible cost—process monitoring, heartbeat checks, time bounds. Most failures are caught here: crashed processes, timed-out agents, resource exhaustion.

**Tier 2 fires only on Tier 1 escalation.** An AI agent examines the failing agent's context and output. This costs tokens but catches the subtle failures that mechanical checks miss: agents stuck in reasoning loops, producing syntactically valid but semantically meaningless output, or operating on stale context.

**Tier 3 is the human.** The goal of the first two tiers is to minimize how often a human needs to intervene and to provide rich context when they do. A Tier 3 escalation arrives with: what happened, what was tried, what the triage agent concluded.

### Context Window Exhaustion

A failure mode unique to agent systems: the context window fills, and the agent degrades silently. Symptoms:

- Agent starts ignoring earlier instructions
- Output quality drops without obvious errors
- Agent "forgets" constraints it was given at session start
- Tool calls become less targeted (broader searches, redundant reads)

**Handoff protocol for context exhaustion:**

1. Detect: Token usage approaching window limit (>80% utilization)
2. Summarize: Extract current state, progress, and remaining work
3. Spawn: New agent with fresh context, receiving only the summary
4. Verify: New agent acknowledges task state before resuming
5. Terminate: Old agent completes gracefully

This is expensive—summarization and context transfer cost tokens. But it's cheaper than an agent operating on degraded context, producing work that fails quality gates and requires rework.

**See Also**: [Context Strategies: Context Window as Finite Resource](../4-context/2-context-strategies.md) — Managing context utilization thresholds

### Incident Post-Mortem Template

After significant agent failures, structured analysis prevents recurrence:

```markdown
## Incident: [Brief description]
**Date:** YYYY-MM-DD
**Agent(s):** [agent IDs]
**Duration:** [time from detection to resolution]

### What Happened
[Observable symptoms]

### Detection
[How was it caught? Which watchdog tier?]

### Root Cause
[What actually went wrong?]

### Resolution
[What fixed it?]

### Prevention
[What changes prevent recurrence?]
- [ ] Watchdog threshold adjustment
- [ ] Specification clarification
- [ ] Quality gate addition
- [ ] Infrastructure change
```

---

## Scaling from 1 to 30 Agents

### Scale Levels

Each scale jump requires new infrastructure, not just more agents. Adding agents without the corresponding infrastructure produces chaos, not throughput.

| Level | Agents | Human Role | Required Infrastructure |
|-------|--------|-----------|------------------------|
| **1-3** | 1 | Direct monitoring | Terminal, manual review |
| **4-6** | 3-5 | Active supervision | Basic logging, shared branch strategy |
| **7-8** | 5-10 | Supervisor pattern | Quality gates, merge queue, attribution |
| **9-10** | 10-30 | Factory operator | Watchdog tiers, automated routing, mail protocols |

### Level 1-3: Single Agent, Direct Monitoring

The practitioner watches agent output directly. Review is manual. Failures are caught by observation. This is where most practitioners start, and it works well:

- Terminal output provides real-time visibility
- Manual review catches quality issues immediately
- No coordination infrastructure needed
- Context is small enough to hold in human working memory

**Limitation:** Scales only as far as human attention. One person can meaningfully monitor one agent's stream of consciousness.

### Level 4-6: Small Team, Supervisor Pattern

At 3-5 agents, direct monitoring becomes impractical. The **supervisor pattern** emerges naturally:

- One agent (or human) coordinates task assignment
- Shared conventions replace direct oversight (CLAUDE.md, branch naming)
- Basic quality gates: lint, type check, test suite
- Agents work on separate files to avoid merge conflicts

**New infrastructure required:**
- Branch strategy (feature branches per agent)
- Merge process (PRs with basic CI checks)
- Task tracking (issues or structured task lists)
- Logging (who did what, when)

**Failure mode at this level:** Trying to scale to 10+ agents without quality gates or attribution. The resulting "works sometimes, breaks mysteriously" state is demoralizing and hard to debug.

### Level 7-8: Medium Team, Infrastructure Required

At 5-10 agents, the shift from "engineering team" to "operations" becomes real:

- Automated quality gates replace manual review for routine work
- Attribution becomes essential (which agent broke the build?)
- Merge queue prevents agents from stepping on each other
- Structured logging enables after-the-fact debugging

**New infrastructure required:**
- Merge queue with automated validation
- Structured agent identity and work logging
- Time-bound task execution (kill stalled agents)
- Human review shifts to sampling-based

### Level 9-10: Full Swarm, Factory Operations

At 10-30 agents, the practitioner operates more like a factory floor manager than a software engineer:

- Three-tier watchdog architecture monitors agent health
- Capability routing assigns tasks based on agent track records
- Mail protocols enable inter-agent coordination
- Convoy tracking groups related tasks across multiple agents
- Automated escalation chains handle routine failures without human intervention

**New infrastructure required:**
- Watchdog tiers (mechanical → AI triage → human escalation)
- Inter-agent communication protocols
- Cost monitoring and budget enforcement
- Automated context-window handoff
- Performance dashboards showing swarm throughput

*[2026-02-11]*: Gas Town production experience at this scale: cognitive load becomes "palpable stress" at 20+ parallel streams. Even with full infrastructure, the practitioner tracks active tasks, reviews escalations, queues new work, and monitors cost simultaneously. The infrastructure doesn't eliminate cognitive load—it makes it manageable.

### Scale Transition Checklist

Before adding agents beyond the current level, verify the infrastructure supports it:

**Moving from 1-3 to 4-6 agents:**
- [ ] Branch strategy defined and documented
- [ ] Basic CI pipeline running (lint, type check, tests)
- [ ] Task tracking system in use (issues, task list)
- [ ] Convention documentation exists (CLAUDE.md or equivalent)

**Moving from 4-6 to 7-10 agents:**
- [ ] Merge queue operational with automated validation
- [ ] Agent attribution logging in place
- [ ] Quality gates catching common issues automatically
- [ ] Human review strategy defined (what gets full review vs. spot-check)

**Moving from 7-10 to 10-30 agents:**
- [ ] Three-tier watchdog monitoring active
- [ ] Context-window handoff protocol implemented
- [ ] Cost monitoring with budget alerts
- [ ] Inter-agent communication protocol defined
- [ ] Escalation chain tested and documented
- [ ] Sampling-based review calibrated

---

## Operational Anti-Patterns

### Anti-Pattern: Scaling Without Infrastructure

**What it looks like:** Adding agents to increase throughput without adding quality gates, attribution, or monitoring.

**Why it fails:** Output increases but quality degrades undetectably. By the time issues surface, the codebase contains weeks of subtly wrong implementations from unmonitored agents.

**Better approach:** Scale infrastructure first, then add agents. Each scale level's infrastructure should be operational before adding agents beyond that level.

### Anti-Pattern: Homogeneous Model Assignment

**What it looks like:** Running all agents on the same frontier model regardless of task complexity.

**Why it fails:** Frontier models cost 10-20× more than mid-tier models. Mechanical tasks (formatting, migration, simple CRUD) don't benefit from frontier reasoning. The cost scales linearly while the quality return is flat.

**Better approach:** Match model to task. Orchestration and review tasks justify frontier models. Implementation tasks often perform equally well on mid-tier models. Mechanical tasks can run on fast, cheap models.

### Anti-Pattern: Design Starvation

**What it looks like:** A swarm of 20 agents sitting idle because the practitioner can't decompose work fast enough to keep them busy.

**Why it fails:** Idle agents still consume baseline tokens (heartbeat, polling). More importantly, batch-creating poorly specified issues to "keep agents busy" produces low-quality work that requires expensive rework.

**Better approach:** Right-size the swarm to match design throughput. 10 well-fed agents outperform 30 starving agents. Scale up when the design pipeline consistently produces a backlog.

### Anti-Pattern: Zombie Tolerance

**What it looks like:** Allowing agents in degraded states (stale context, looping behavior) to continue running because they're "technically still producing output."

**Why it fails:** Zombie agents consume tokens while producing work that fails quality gates. The rework cost exceeds the cost of terminating and restarting. Worse, zombie output that slips past gates introduces technical debt.

**Better approach:** Terminate aggressively, restart cheaply. Fresh context is almost always preferable to degraded context. The cost of a restart is bounded; the cost of zombie output is unbounded.

---

## Daily Operating Rhythm

A production swarm operation typically follows this daily cycle:

```
Morning:
  1. Review overnight results (quality gate reports, incidents)
  2. Triage failed work (reassign, respecify, or archive)
  3. Design and decompose new work (issue creation)

Active hours:
  4. Launch swarm on prepared issues
  5. Monitor dashboards (cost, throughput, quality metrics)
  6. Handle escalations from watchdog system
  7. Review sampled PRs to calibrate quality gates
  8. Queue additional work as agents complete tasks

End of day:
  9. Review day's metrics (cost, output, quality)
  10. Set up overnight batch work (low-priority, well-specified tasks)
  11. Adjust watchdog thresholds based on observed failures
```

**The practitioner's role is primarily operational.** The coding happens in the design phase (writing specifications) and the review phase (sampling output). The implementation phase is delegated to agents. Time allocation at production scale:

| Activity | Time Allocation |
|----------|----------------|
| Design and decomposition | 40% |
| Monitoring and escalation handling | 25% |
| Review and quality calibration | 20% |
| Infrastructure and tooling | 15% |

---

## Inter-Agent Coordination at Scale

### Communication Protocols

At 10+ agents, agents occasionally need to coordinate beyond the orchestrator. Direct agent-to-agent communication introduces complexity but solves problems that purely hierarchical coordination cannot:

| Protocol | Mechanism | Use Case |
|----------|-----------|----------|
| **Mail-based** | Agents read/write to shared message files | Asynchronous coordination, status updates |
| **Convoy tracking** | Groups of related tasks share a tracking ID | Multi-step features spanning multiple agents |
| **Broadcast** | One-to-many notification | "Build X is ready for integration" |
| **Point-to-point** | Direct agent-to-agent message | "Agent-12: I need the interface definition from your module" |

**Mail protocols** work well because they're asynchronous and persistent. An agent writes a message; the recipient reads it when ready. No real-time coordination required. The message persists if the recipient restarts.

**Convoy tracking** prevents lost work in multi-agent features. When a feature requires agents 3, 7, and 12 to each implement a component, the convoy ID links their work. If agent 7 fails, the convoy system identifies the incomplete feature and reassigns the work—rather than having agents 3 and 12's completed work sit orphaned.

### File Ownership as Coordination

The simplest coordination mechanism at swarm scale: **explicit file ownership**. Each agent owns specific files. No two agents modify the same file simultaneously.

```
agent-01: owns src/auth/*.ts
agent-02: owns src/api/*.ts
agent-03: owns src/database/*.ts
agent-04: owns tests/auth/*.test.ts
```

This eliminates merge conflicts at the cost of flexibility. When a task requires cross-cutting changes, it's assigned to a single agent or decomposed into per-module subtasks.

**See Also**: [Expert Swarm Pattern: File Ownership Coordination](../6-patterns/8-expert-swarm-pattern.md#implementation-pattern-file-ownership-coordination) — The pattern underlying this practice

---

## Practices Adopted Too Late

Production swarm operators consistently report the same regrets. These practices seem optional until their absence causes a crisis:

### Attribution from Day One

Every operator who added attribution retroactively wishes they'd started with it. Without attribution, debugging multi-agent failures becomes archaeology—reconstructing which agent did what from git history, timestamps, and educated guesses.

**Cost of delay:** Days of debugging time per incident that would take minutes with proper attribution.

### Cost Budgets with Hard Limits

Running without cost budgets leads to surprise bills. A runaway agent loop can consume hundreds of dollars before anyone notices. Set hard limits:

- Per-agent token budget (kill agent if exceeded)
- Per-task cost ceiling (reject tasks that estimate above threshold)
- Daily swarm budget (pause new work when daily limit approaches)

**Cost of delay:** A single runaway weekend can exceed a month's planned budget.

### Structured Incident Records

The first few agent failures are memorable. By the twentieth, patterns blur together. Without structured records, the same failure modes recur because the fixes aren't documented.

**Cost of delay:** Repeated incidents that were previously "fixed" but the fix was never captured.

### Quality Gate Calibration

Quality gates need tuning based on actual failure data. Gates that are too strict reject good work and waste tokens on rework. Gates that are too loose let defects through. Calibrate by comparing gate decisions against sampled human review.

**Cost of delay:** Either excessive false rejections (wasted tokens) or undetected quality degradation.

---

## Decision Framework: Is a Swarm Appropriate?

Not every multi-agent problem requires swarm-scale operations. Use this to determine the right approach:

```
Is the work decomposable into 10+ independent units?
├─ No → Use 1-5 agents with direct supervision (Level 1-6)
└─ Yes
   │
   Is the work time-sensitive enough to justify parallel execution?
   ├─ No → Queue work for sequential execution (cheaper)
   └─ Yes
      │
      Is attribution and quality gate infrastructure in place?
      ├─ No → Build infrastructure first (see Scale Transition Checklist)
      └─ Yes
         │
         Can the practitioner sustain design throughput?
         ├─ No → Right-size the swarm to match design capacity
         └─ Yes → Deploy full swarm
```

---

## Open Questions

- What's the theoretical ceiling on useful parallelism? Is there a point beyond 30 agents where coordination overhead exceeds throughput gains?
- How does agent capability routing evolve as models improve? Will the need for model-per-task optimization persist or collapse?
- Can design bottlenecks be addressed by design-focused agents, or is human architectural judgment irreducible?
- What inter-agent communication protocols emerge at 50+ agent scale? Do mail-based protocols hold, or does something new emerge?
- How do regulatory and compliance requirements adapt to agent-authored code at scale?
- What's the right balance between agent autonomy and human oversight at each scale level?

---

## Connections

- **To [Expert Swarm Pattern](../6-patterns/8-expert-swarm-pattern.md):** Expert Swarm provides the coordination architecture; this section covers the operational practices for running swarms in production over time.
- **To [Cost and Latency](3-cost-and-latency.md):** Economic analysis of agent operations. This section extends the cost discussion to swarm-scale economics where linear token scaling becomes the dominant constraint.
- **To [Production Concerns](4-production-concerns.md):** General production reliability practices. This section adds swarm-specific concerns: attribution, scale-dependent infrastructure, and multi-agent incident response.
- **To [Orchestrator Pattern](../6-patterns/3-orchestrator-pattern.md):** The orchestrator coordinates individual swarm runs; operating practices cover the ongoing management of repeated swarm deployments.
- **To [Workflow Coordination](5-workflow-coordination.md):** Structured metadata enables the task tracking and agent communication that swarm operations depend on.
- **To [Context Strategies](../4-context/2-context-strategies.md):** Context window management is critical at swarm scale, where context exhaustion is a primary failure mode requiring handoff protocols.

- **To [Production Multi-Agent Systems](../6-patterns/11-production-multi-agent-systems.md)**: Production Multi-Agent Systems defines the architectural patterns (persistent identity, watchdog chains, convoy tracking); this section covers the day-to-day operational practices that depend on those patterns.
