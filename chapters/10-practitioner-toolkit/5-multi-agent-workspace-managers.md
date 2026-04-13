---
title: "Multi-Agent Workspace Managers"
description: "Tools for orchestrating large-scale agent swarms, with Gas Town and Overstory as primary examples"
created: 2026-02-11
last_updated: 2026-02-13
tags: [toolkit, multi-agent, gastown, overstory, workspace-management, orchestration]
part: 3
part_title: Perspectives
chapter: 10
section: 5
order: 3.10.5
---

# Multi-Agent Workspace Managers

As agent counts scale from 2-3 subagents to 20-30 parallel workers, a new category of infrastructure emerges: **multi-agent workspace managers**. These tools solve problems that frameworks and coding agents were never designed to handle—persistent agent identity, automated merge coordination, work attribution across dozens of simultaneous branches, and supervisory hierarchies that recover from failures without human intervention.

The relationship to existing tools is analogous to Docker's emergence alongside virtual machines. VMs handled single-server isolation; Docker handled container orchestration at scale. Similarly, coding agents (Claude Code, Cursor) handle single-agent workflows; workspace managers handle the infrastructure layer when the bottleneck shifts from implementation capacity to coordination overhead.

---

## Your Mental Model

**Workspace managers are infrastructure, not agents.** The distinction matters. A coding agent (Claude Code, Copilot) generates code. A framework (LangGraph, CrewAI) coordinates agent logic. A workspace manager operates one layer below both—it provisions isolated environments, routes work assignments, merges outputs, and supervises health across an entire fleet of coding agents. The workspace manager does not write code itself; it ensures that 20 agents writing code simultaneously do not destroy each other's work.

This is the difference between a developer and a DevOps platform. The developer writes features; the platform ensures those features deploy safely. Workspace managers are the DevOps layer for agent swarms.

---

## The Problem Space

### Why Existing Tools Break at Scale

Single-agent tools assume a model where one agent works in one repository at a time. This assumption creates cascading failures when agent count increases:

| Problem | At 1-3 Agents | At 20-30 Agents |
|---------|---------------|-----------------|
| **Branch conflicts** | Rare, manually resolved | Constant, blocks all progress |
| **Merge coordination** | Developer handles | Requires automated queue |
| **Context loss** | Restart and re-explain | 20 agents lose context simultaneously |
| **Work attribution** | Obvious (one agent did it) | Unclear which agent produced what |
| **Failure recovery** | Restart the agent | Need supervision to detect and restart |
| **Cost tracking** | Single billing stream | Dozens of parallel billing streams |

### The Coordination Tax

*[2026-02-11]*: At small scale, coordination overhead is negligible. A developer managing 2-3 Claude Code subagents via Task tool spends minimal time on coordination. At 20+ agents, coordination becomes the dominant cost:

```
Agent Count:     1    3    5    10    20    30
Useful Work:   95%  85%  75%   60%   40%   25%  (without workspace manager)
Coordination:   5%  15%  25%   40%   60%   75%
```

Workspace managers exist to invert this ratio—keeping useful work above 70% even at 20+ agent scale by automating coordination that would otherwise consume human attention.

---

## Gas Town: Primary Example

**Gas Town** is an open-source multi-agent workspace manager created by Steve Yegge (former Google, Amazon, Grab engineer). Released December 2025, MIT licensed, built in Go, with 9,000+ GitHub stars as of February 2026.

Gas Town provides two CLI tools:
- **`gt`** — Workspace and agent management (provisioning, assignment, completion, context priming)
- **`bd`** — Git-backed issue tracking and workflow state (bead management, step progression)

### Architecture Overview

```
~/gt/                          Town root (all projects live here)
├── .beads/                    Town-level beads (hq-* prefix)
├── mayor/                     Mayor config (global coordinator)
├── deacon/                    Background supervisor daemon
│   └── dogs/boot/             Health triage subsystem
│
└── <rig>/                     Per-project container
    ├── mayor/rig/             Canonical clone (source of truth)
    ├── refinery/rig/          Merge queue processor
    ├── witness/               Per-rig supervisor
    ├── crew/<name>/           Human workspaces
    └── polecats/<name>/       Worker worktrees (one per agent)
```

**Key architectural decisions:**

- **Git worktrees as isolation**: Each agent operates in its own worktree (`polecats/<name>/`), sharing the same Git history but maintaining independent working directories. No filesystem conflicts between agents.
- **Canonical clone separation**: The `mayor/rig/` directory holds the canonical repository state. Agent worktrees branch from it. This prevents any single agent from corrupting the source of truth.
- **Supervisory hierarchy**: The `deacon` daemon monitors agent health. The `witness` per-rig supervisor detects stalled or failed agents. Recovery happens automatically without human intervention.
- **Bead-based state**: All workflow state persists as "beads"—Git-backed artifacts that survive agent restarts. No ephemeral state means no context loss on failure.

### Core Workflows

#### Receiving Work

```bash
gt hook    # Check what work is assigned to this agent
```

The `hook` command is the agent's entry point. It reads the current bead assignment and returns the task specification. Agents call this at session start (typically via a SessionStart hook in Claude Code) to understand their current assignment.

#### Completing Work

```bash
gt done    # Complete work, push branch, submit to merge queue, cleanup
```

`gt done` handles the entire completion sequence atomically:
1. Stage and commit changes
2. Push the agent's branch
3. Submit to the merge queue (refinery)
4. Clean up the worktree for reuse

This eliminates the common failure mode where agents commit but forget to push, or push but fail to signal completion.

#### Context Restoration

```bash
gt prime   # Restore context for new sessions
```

Agent sessions are ephemeral—context dies when the session ends. `gt prime` reconstructs working context from Git-backed beads, enabling agents to resume work after restarts without human re-explanation. This maps directly to the [Context vs Memory](../4-context/1-context-fundamentals.md#context-vs-memory) distinction: beads serve as persistent memory that survives session boundaries.

#### Workflow Progression

```bash
bd mol current              # Check current position in workflow
bd close <step> --continue  # Close step and auto-advance to next
```

The `bd` tool manages workflow state through a bead-based issue tracker. Steps close explicitly, and `--continue` auto-advances to the next step in the sequence. This prevents agents from stalling between workflow steps.

### The Merge Queue (Refinery)

The **refinery** is Gas Town's automated merge coordinator. When an agent completes work and calls `gt done`, the branch enters the merge queue:

```
Agent A completes ──> Refinery Queue ──> AI-Assisted Merge ──> Canonical Clone
Agent B completes ──>                                          (mayor/rig/)
Agent C completes ──>
```

**How it works:**
1. Completed branches queue in order of submission
2. The refinery attempts automated merge against the canonical clone
3. If conflicts arise, an AI agent resolves them (not the original worker)
4. Successful merges update the canonical clone
5. Other agents' worktrees rebase against the updated canonical

This decouples work completion from merge resolution. Agents never block on merge conflicts—they move to the next task while the refinery handles integration.

**Trade-off**: AI-assisted merge introduces risk of incorrect conflict resolution. The refinery prioritizes throughput over perfect accuracy, relying on subsequent review to catch errors.

### Agent Identity and Persistence

Gas Town assigns persistent identities to agents through the `polecats/` naming convention. Each agent has:

- **A named worktree**: `polecats/agent-name/` persists across sessions
- **A work history**: Git commits attributed to the agent
- **Accumulated context**: Beads record what the agent has worked on

This contrasts with ephemeral agent models (Claude Code subagents, LangGraph nodes) where agents have no identity beyond a single invocation. Persistent identity enables:

- **Skill routing**: Assign agents to work matching their prior experience
- **Accountability**: Trace which agent produced which output
- **Context efficiency**: Agents familiar with a codebase area need less context loading

### Supervision Architecture

```
┌────────────────────────────────────┐
│            Deacon                   │
│  (Town-level supervisor daemon)    │
│                                    │
│  ┌──────────┐  ┌──────────┐       │
│  │ Witness A │  │ Witness B │  ... │
│  │ (Rig 1)  │  │ (Rig 2)  │       │
│  └─────┬────┘  └─────┬────┘       │
│        │              │            │
│  ┌─────┴─────┐  ┌────┴─────┐     │
│  │ dogs/boot │  │ dogs/boot│      │
│  │ (triage)  │  │ (triage) │      │
│  └───────────┘  └──────────┘      │
└────────────────────────────────────┘
         │                │
    ┌────┴────┐      ┌───┴────┐
    │Polecats │      │Polecats│
    │(Agents) │      │(Agents)│
    └─────────┘      └────────┘
```

Three supervision layers operate independently:

1. **Deacon** (town-level): Background daemon monitoring all rigs. Detects infrastructure-level failures (disk space, network, process crashes).
2. **Witness** (rig-level): Per-project supervisor. Detects agent-level failures (stalled work, repeated errors, timeout).
3. **Dogs/boot** (triage): Health triage subsystem that classifies failures and routes recovery actions.

This layered approach means a single agent failure does not cascade. The witness detects the failure, dogs/boot classifies it, and recovery proceeds while other agents continue working.

---

## Overstory: TypeScript/Bun Workspace Manager

*[2026-02-13]*: Overstory represents the session-as-orchestrator alternative to Gas Town's daemon-based architecture. Built in TypeScript using Bun runtime (~31K LOC, 912 tests), Overstory demonstrates that workspace management patterns transcend language ecosystems and coordination models.

### Architecture Overview

```
.overstory/                         Target project root
├── config.yaml                     Project configuration
├── agent-manifest.json             Agent registry
├── hooks.json                      Central hooks config
├── agents/                         Agent state
│   └── {name}/
│       ├── identity.yaml           Persistent identity (CVs)
│       └── work-history.md         Append-only work log
├── worktrees/                      Git worktrees (gitignored)
│   └── {agent-name}/
├── specs/                          Task specifications
│   └── {bead-id}.md
├── logs/                           Agent logs (gitignored)
│   └── {agent-name}/{timestamp}/
├── mail.db                         SQLite mail (gitignored, WAL)
└── metrics.db                      SQLite metrics (gitignored)
```

**Key Architectural Decisions:**

- **Session-as-orchestrator**: Your active Claude Code session coordinates agents via `overstory` CLI. No separate daemon process.
- **Bun runtime**: Zero runtime dependencies—only Bun built-ins (sqlite, spawn, file I/O).
- **Hook integration**: SessionStart/UserPromptSubmit/PreToolUse hooks provide coordination points.
- **SQLite messaging**: WAL mode enables concurrent agent messaging at ~1-5ms latency.
- **Two-layer definitions**: Base agent .md files (HOW) + dynamic overlay CLAUDE.md (WHAT).

### Core Workflows

#### Spawning Workers

```bash
overstory sling --task bd-abc --capability builder --name auth-login \
  --spec .overstory/specs/bd-abc.md --files src/auth/login.ts,src/auth/types.ts
```

Creates worktree, writes overlay CLAUDE.md, deploys hooks, starts tmux session running Claude Code.

#### Communication

```bash
overstory mail send --to orchestrator --subject "Build complete" \
  --body "Implemented login flow. Tests passing." --type result
```

Messages persist in `.overstory/mail.db` (SQLite). Hook automatically injects unread messages into agent context on next prompt submission.

#### Merge Queue

```bash
overstory merge --branch overstory/auth-login/bd-abc
```

4-tier resolution: Clean → Auto-Resolve → AI-Resolve → Re-Imagine. Escalates automatically until conflicts resolve.

#### Monitoring

```bash
overstory status       # Show all active agents, worktrees, beads state
overstory watch        # Start watchdog daemon
overstory metrics      # Show metrics summary
```

### Comparison with Gas Town

| Dimension | Overstory | Gas Town |
|-----------|-----------|----------|
| **Runtime** | TypeScript/Bun | Go |
| **Orchestrator** | Session-as-orchestrator (your active Claude Code session) | External daemon (Mayor) |
| **CLI** | overstory (17 commands) | gt/bd dual CLIs |
| **Messaging** | Custom SQLite mail (~1-5ms) | Typed mail protocol |
| **Cost Model** | Subscription (fixed monthly cost) | API tokens (~$100/hr) |
| **Dependencies** | Zero (Bun built-ins only) | Go stdlib |
| **Agent Definition** | Base .md + overlay CLAUDE.md | Similar pattern |
| **Merge Resolution** | 4-tier (Clean/Auto/AI/Re-Imagine) | Refinery with AI resolution |
| **Supervision** | Tiered (Daemon → Triage → Monitor → Supervisor) | Daemon → Boot → Deacon → Witness |

**Convergence:** Both implement persistent identity, worktree isolation, typed messaging, tiered health monitoring, and merge queue infrastructure. The architectural alignment across different technology stacks (Go vs TypeScript) and coordination models (daemon vs session) validates these patterns as fundamental to swarm coordination.

### When to Choose Overstory

**Good fit:**
- TypeScript/Bun ecosystem preferred
- Subscription cost model acceptable (fixed monthly vs per-token)
- Session-based coordination workflow natural (human stays engaged)
- Zero-dependency deployment valued
- Hook-based mechanical enforcement sufficient

**Poor fit:**
- Multi-language polyglot teams (Go ecosystem familiarity in Gas Town)
- API token budget model required (pay-per-use vs subscription)
- External daemon coordination preferred (orchestrator survives session crashes)
- Production validation maturity critical (Gas Town has more operational history as of early 2026)

---

## Workspace Manager Selection Framework

When choosing between workspace managers, consider scale requirements, ecosystem preferences, and coordination models:

```
Scale and requirements:
  1-5 agents → Claude Code Agent Teams (built-in)
  5-15 agents → Overstory (session-based) OR Gas Town (daemon-based)
  15-30 agents → Gas Town (proven at scale as of early 2026)

Ecosystem preference:
  TypeScript/Bun → Overstory
  Go → Gas Town
  Language-agnostic → Either (patterns converge)

Cost model:
  Subscription (fixed) → Overstory
  API tokens (pay-per-use) → Gas Town
  Either acceptable → Choose by ecosystem

Coordination model:
  Session-as-orchestrator → Overstory
  External daemon → Gas Town
```

---

## When Workspace Managers Make Sense

### Good Fit

- **10+ parallel agents**: The coordination overhead threshold where manual management becomes the bottleneck
- **Multi-repository projects**: Complex merge coordination across repositories benefits from automated queuing
- **Attribution requirements**: Regulated environments or teams requiring clear traceability of agent-produced code
- **Human design capacity as bottleneck**: When there are more implementation tasks than agents to assign them to—workspace managers maximize agent utilization
- **Long-running projects**: Persistent agent identity and bead-based state compound value over weeks and months

### Poor Fit

- **1-5 agents**: Claude Code subagents or agent teams handle this scale with far less overhead. The infrastructure cost of a workspace manager exceeds its coordination benefit.
- **Small projects**: If the codebase fits in a single agent's context, workspace management adds unnecessary complexity.
- **Early-stage practitioners**: Workspace managers target practitioners at prompt maturity Level 7-8 (see [Prompt Maturity Model](../9-mental-models/2-prompt-maturity-model.md)). Understanding single-agent workflows is prerequisite.
- **Cost-sensitive environments**: Gas Town's operational model consumes approximately $100/hour in aggregate token costs across 20-30 agents. Budget must justify the throughput gain.
- **Exploratory work**: When requirements are unclear and iteration speed matters more than parallelism, single-agent exploration is more efficient than coordinated swarms.

### The Scale Decision

```
Agent Count Decision Framework:

1-2 agents  ──> Single session or subagents (Task tool)
                No workspace manager needed.

3-8 agents  ──> Agent teams (TeammateTool) or framework orchestration
                Workspace manager optional, likely overkill.

10-20 agents ──> Workspace manager becomes valuable.
                 Coordination overhead exceeds manual capacity.

20-30 agents ──> Workspace manager essential.
                 Without one, coordination consumes >60% of effort.

30+ agents  ──> Workspace manager + custom scaling infrastructure.
                 Current tools approaching upper limits.
```

---

## Comparison with Other Approaches

| Dimension | Gas Town | Overstory | Claude Code Agent Teams | Google ADK | LangGraph |
|-----------|----------|-----------|------------------------|------------|-----------|
| **Scale target** | 20-30 agents | 10-15 agents | 2-8 agents | Varies | Varies |
| **Primary abstraction** | Workspace/worktree | Worktree + session | Teammate session | Agent/workflow | Graph node |
| **Persistence** | Git-backed beads | Filesystem + SQLite | Session-only | State store | Checkpoints |
| **Agent identity** | Persistent (named CVs) | Persistent (identity.yaml) | Ephemeral | Configurable | Ephemeral |
| **Coordination model** | Mail protocol + beads | SQLite mail (~1-5ms) | Message passing + tasks | Shared state | Graph edges |
| **Orchestrator model** | External daemon (Mayor) | Session-as-orchestrator | Built-in (flat) | Configurable | Explicit graph |
| **Merge strategy** | Automated AI refinery | 4-tier escalation | Manual | N/A | N/A |
| **Supervision** | Deacon/Witness/Dogs | Daemon/Triage/Monitor/Supervisor | Flat (lead monitors) | Configurable | External |
| **Failure recovery** | Automatic (supervisor chain) | Tiered (mechanical → AI) | Manual (respawn) | Configurable | Checkpoint restore |
| **Cost profile** | ~$100/hr (20-30 agents) | Subscription (fixed monthly) | Lower (2-8 agents) | Lower | Lower |
| **Maturity** | Early (December 2025) | Early (February 2026) | Experimental | Production | Production (v1.0) |
| **Language** | Go (CLI tools) | TypeScript/Bun | TypeScript (SDK) | Python | Python |
| **Dependencies** | Go stdlib | Zero (Bun built-ins) | Node + npm packages | Python + pip packages | Python + pip packages |
| **License** | MIT | MIT | Proprietary | Apache 2.0 | MIT |

**Key differentiators:**

- **Git-native persistence**: Gas Town's bead system and Overstory's filesystem + SQLite persistence both ensure state survives crashes. Other approaches use ephemeral or application-specific state.
- **Merge automation**: Gas Town and Overstory automate merge conflict resolution through AI-assisted tiers. At 20+ agents producing concurrent branches, merge becomes the primary bottleneck without automation.
- **Supervisory depth**: Three-layer supervision (Gas Town: deacon/witness/dogs; Overstory: daemon/triage/monitor/supervisor) provides fault isolation missing from flat coordination models. A failed agent is detected and recovered without affecting peers.
- **Coordination model**: Gas Town's daemon-based Mayor vs Overstory's session-as-orchestrator represent fundamental architectural trade-offs. Daemon provides crash independence; session provides infrastructure simplicity.

---

## Implementation Patterns

### Integration with Coding Agents

Gas Town is agent-agnostic at the worker level—any coding agent that can operate in a Git worktree can serve as a polecat. The integration surface is three commands:

```bash
# SessionStart hook (in .claude/settings.json or equivalent)
gt prime    # Load context from beads

# During work
gt hook     # Read current assignment

# On completion
gt done     # Push, submit to refinery, cleanup
```

For Claude Code specifically, these map to:

| Gas Town Command | Claude Code Integration Point |
|-----------------|-------------------------------|
| `gt prime` | SessionStart hook |
| `gt hook` | Agent prompt preamble |
| `gt done` | Post-completion workflow step |
| `bd mol current` | Status check within agent loop |

### Bead-Based State Management

Beads are Gas Town's unit of persistent state. Every workflow step, task assignment, and completion record is a bead stored in Git:

```
.beads/
├── hq-project-setup           # Town-level bead
├── hq-architecture-review     # Town-level bead
└── task-implement-auth         # Task bead assigned to polecat
```

**Properties of beads:**
- **Versioned**: Every bead change is a Git commit
- **Attributable**: Beads record which agent created or modified them
- **Recoverable**: Git history enables rollback to any prior state
- **Portable**: Beads survive agent restarts, machine changes, and session boundaries

This maps to the [Context as Code](../9-mental-models/4-context-as-code.md) mental model—treating agent state as version-controlled artifacts rather than ephemeral memory.

### Work Distribution Pattern

```
┌──────────────┐
│    Mayor     │  (Global coordinator)
│  Assigns →   │
└──────┬───────┘
       │
  ┌────┴──────────────────────────┐
  │         Bead Queue            │
  │  [task-1] [task-2] [task-3]   │
  └────┬──────┬──────┬────────────┘
       │      │      │
       ▼      ▼      ▼
   Polecat  Polecat  Polecat
   (Alice)  (Bob)    (Carol)
       │      │      │
       ▼      ▼      ▼
   gt done  gt done  gt done
       │      │      │
       ▼      ▼      ▼
  ┌────┴──────┴──────┴────────────┐
  │         Refinery              │
  │  Merge queue + AI resolution  │
  └───────────────────────────────┘
       │
       ▼
  Canonical Clone (mayor/rig/)
```

The mayor assigns tasks as beads. Polecats (worker agents) pull assignments via `gt hook`, execute in isolated worktrees, and submit via `gt done`. The refinery merges outputs in submission order, resolving conflicts with AI assistance. No agent waits for another agent's merge to complete.

---

## Anti-Patterns

### Anti-Pattern: Workspace Manager for Small Teams

Deploying Gas Town for 2-3 agents introduces infrastructure overhead (deacon daemon, refinery process, worktree management) that exceeds the coordination benefit. Claude Code agent teams or subagents handle this scale with zero infrastructure.

**Better approach:** Start with agent teams. Migrate to a workspace manager when coordination overhead visibly limits throughput—typically at 8-10 concurrent agents.

### Anti-Pattern: Treating Workspace Managers as Frameworks

Workspace managers and agent frameworks serve different layers. Attempting to use Gas Town as a replacement for LangGraph or CrewAI conflates infrastructure with logic. Gas Town manages *where* agents work; frameworks manage *how* agents reason.

**Better approach:** Use workspace managers alongside frameworks. Gas Town provisions the worktree; Claude Code (or another agent) operates within it.

### Anti-Pattern: Ignoring the Cost Curve

At $100/hour for 20-30 agents, workspace-managed swarms consume significant resources. Running a full swarm on exploratory work or unclear requirements wastes budget on parallel execution of potentially discarded work.

**Better approach:** Use single-agent exploration to clarify requirements. Deploy the swarm only for well-specified implementation tasks where parallelism yields clear throughput gains.

### Anti-Pattern: Skipping the Maturity Ladder

Jumping directly to 20-agent workspace management without experience at lower scales creates operational risk. Practitioners unfamiliar with single-agent failure modes will struggle to diagnose swarm-level failures.

**Better approach:** Progress through the scale ladder: single agent (1-2) to agent teams (3-8) to workspace manager (10+). Each tier builds operational intuition for the next.

---

## The Emerging Category

Gas Town represents the first visible entry in what may become a recognized tool category. The pattern is familiar from infrastructure evolution:

| Era | Problem | Category That Emerged |
|-----|---------|----------------------|
| 2000s | Managing many servers | Configuration management (Chef, Puppet, Ansible) |
| 2010s | Managing many containers | Container orchestration (Docker, Kubernetes) |
| 2020s | Managing many microservices | Service mesh (Istio, Linkerd) |
| 2025+ | Managing many agents | Workspace management (Gas Town, ...) |

**Signals this category is real:**

- **Scale demand exists**: Production systems already run 10-30 agents (Claude Code agent teams, custom frameworks). The tooling gap is observable.
- **Shared problems recur**: Merge coordination, agent supervision, state persistence, and work attribution are not Gas Town-specific—any system at this scale faces them.
- **Infrastructure layer is distinct**: Workspace management is neither agent logic (framework layer) nor code generation (agent layer). It occupies a recognizable infrastructure niche.

**Signals this category is premature:**

- **Single entrant**: Gas Town is effectively the only tool in this space as of February 2026. Categories need competition to validate.
- **Early maturity**: Gas Town launched December 2025. Production hardening takes years.
- **Model capabilities may eliminate the need**: If models improve enough, fewer agents working smarter may outperform many agents coordinated by infrastructure.
- **Cost barrier**: $100/hour operational cost limits adoption to well-funded teams and high-value projects.

---

## Open Questions

- How does merge quality degrade as branch divergence increases across 20+ concurrent agents?
- What is the practical upper limit on agent count before the refinery becomes a bottleneck?
- Can workspace managers integrate with non-Git version control systems, or is Git-native state fundamental?
- How does persistent agent identity interact with model updates (agent "personality" may shift between model versions)?
- What monitoring and observability patterns emerge for workspace-managed swarms?
- How do workspace managers handle multi-repository dependencies (agent in repo A needs changes from agent in repo B)?
- Will model improvements (larger context, better reasoning) reduce the need for high agent counts?
- What security model governs agent access within worktrees (can a compromised agent affect other worktrees)?
- How does bead-based state compare to database-backed state for operational query patterns?
- What happens when the refinery's AI merge resolution produces subtle bugs that pass tests?

---

## Connections

- **To [Claude Code](1-claude-code.md)**: Gas Town's polecats can run Claude Code as the underlying coding agent. Integration happens through SessionStart hooks (`gt prime`) and completion workflows (`gt done`). Claude Code agent teams operate at smaller scale (2-8 agents) compared to Gas Town's target (20-30).
- **To [Agent Frameworks](4-agent-frameworks.md)**: Workspace managers operate at a different layer than frameworks. LangGraph coordinates agent *logic*; Gas Town coordinates agent *infrastructure*. The two are complementary, not competitive.
- **To [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md)**: Gas Town's mayor implements an orchestrator pattern, but at infrastructure level rather than prompt level. The mayor assigns work and monitors completion; it does not reason about task decomposition.
- **To [Expert Swarm Pattern](../7-patterns/8-expert-swarm-pattern.md)**: The expert swarm pattern describes agent coordination logic. Workspace managers provide the infrastructure substrate that makes swarms practical at scale—isolation, merge automation, supervision.
- **To [Context Fundamentals](../4-context/1-context-fundamentals.md)**: Gas Town's bead system directly addresses the context vs. memory gap. Beads provide persistent memory that survives session boundaries, enabling agents to resume work without re-explanation.
- **To [Workflow Coordination](../8-practices/5-workflow-coordination.md)**: Gas Town's `gt done` + refinery pattern automates the commit-push-merge workflow that manual coordination handles at smaller scale.

---

## Sources

- [Gas Town GitHub Repository](https://github.com/steveyegge/gastown) (MIT License, 9k+ stars)
- [Steve Yegge - Gas Town announcement and documentation](https://github.com/steveyegge/gastown/blob/main/README.md)
- [Gas Town CLI reference](https://github.com/steveyegge/gastown/tree/main/docs)
- Steve Yegge's background: Former Google, Amazon, Grab principal engineer; creator of Gas Town (December 2025)
