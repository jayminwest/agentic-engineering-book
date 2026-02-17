---
title: "Production Multi-Agent Systems"
description: "Patterns for operating multi-agent systems at scale, from persistent identity to watchdog chains"
created: 2026-02-11
last_updated: 2026-02-13
tags: [patterns, multi-agent, production, orchestration, gastown, overstory]
part: 2
part_title: Craft
chapter: 6
section: 11
order: 2.6.11
---

# Production Multi-Agent Systems

Running multi-agent systems in production differs fundamentally from running them in development. Development tolerates manual restarts, lost context, and occasional zombies. Production demands autonomous recovery, persistent tracking, and clean resource lifecycle. The patterns in this section address what happens after "it works on my machine."

**Production Evidence:** Gas Town and Overstory—two production multi-agent systems operating 10-30 parallel workers across different technology stacks (Go vs TypeScript/Bun) and coordination models (daemon-based vs session-based)—demonstrate these patterns under sustained load. Each pattern generalized here has been validated through implementations that handle failures as routine operational events, not exceptional circumstances.

---

## Core Questions

### Identity and State
- How do agents maintain continuity when sessions are disposable?
- What survives a crash, and what must be reconstructed?
- Where does agent identity live if not in the running process?

### Supervision and Recovery
- How does the system detect and recover from stuck agents without human intervention?
- What supervision architecture handles the full spectrum from process crashes to reasoning failures?
- When does automated recovery become more dangerous than the original failure?

### Coordination at Scale
- How do 20+ agents coordinate without shared memory or a central bus?
- What communication primitives survive crashes, restarts, and network partitions?
- How does work tracking persist across agent failures and handoffs?

### Resource Discipline
- What happens to agent resources when work completes (or fails)?
- How does the system prevent resource leaks from accumulating over time?
- When is "idle" a valid state versus a system failure?

---

## Your Mental Model

**Production multi-agent systems are factories, not workshops.** A workshop tolerates artisans managing their own tools and cleanup. A factory requires standardized stations, material tracking, quality control checkpoints, and—critically—the assumption that any station can break at any time. The factory continues operating because the system handles failures, not because individual stations are reliable.

The six patterns that follow address distinct factory concerns: identity (who works each station), supervision (who watches the stations), coordination (how stations communicate), cleanup (what happens when work completes), tracking (how management sees progress), and integration (how finished work reaches the product).

---

## Pattern 1: Persistent Identity, Ephemeral Sessions

Agents maintain durable identities across disposable execution sessions, solving context window exhaustion, session crashes, and model upgrades without losing accumulated expertise.

### The Problem

Agent sessions are inherently ephemeral. Context windows fill, processes crash, rate limits trigger, models upgrade. Any system that couples agent identity to a running session loses everything when that session ends.

### Core Structure

```
┌─────────────────────────────┐
│     Persistent Identity     │
│  ┌───────────────────────┐  │
│  │ CV / Work History     │  │    Survives across sessions
│  │ Domain Expertise      │  │    Stored on disk, not in memory
│  │ Accumulated Learnings │  │
│  └───────────────────────┘  │
└──────────┬──────────────────┘
           │
           │  "Seance" (context retrieval)
           │
┌──────────▼──────────────────┐
│     Ephemeral Session       │
│  ┌───────────────────────┐  │
│  │ Current task context  │  │    Disposable
│  │ Working memory        │  │    Can crash without loss
│  │ Tool state            │  │    Fresh context window
│  └───────────────────────┘  │
└─────────────────────────────┘
```

**Persistent layer** stores identity artifacts on the filesystem:
- **CV/Work History** — past tasks completed, quality assessments, domain specializations
- **Domain Expertise** — accumulated patterns, anti-patterns, decision heuristics
- **Session Logs** — queryable history of predecessor sessions

**Ephemeral layer** handles current execution:
- Fresh context window per session (no accumulated bloat)
- Current task assignment and working state
- Tool connections and temporary artifacts

### How It Works

1. **Identity creation**: Agent receives a persistent identity (name, role, expertise file path)
2. **Session start**: New session runs a priming command (e.g., `gt prime`) that loads relevant identity artifacts into context
3. **Seancing**: The session queries predecessor sessions for task-specific context—what was attempted, what failed, what succeeded
4. **Execution**: Agent works with full context window available for the current task
5. **Session end**: Results persist to disk; session terminates cleanly

**Evidence:** Gas Town workers ("polecats") maintain CVs across sessions. When a new session starts, `gt prime` restores identity context. A worker that crashed mid-task can be replaced by a new session that seances the crashed session's logs to understand what was attempted.

*[2026-02-13]*: Overstory demonstrates the same separation through TypeScript/Bun implementation. Agent identity persists in `.overstory/agents/{name}/identity.yaml` with work history in `work-history.md`. Sessions spawn via `overstory sling` with fresh context loaded through `overstory prime`, implementing the seancing pattern through different tooling. The cross-ecosystem validation (Go + TypeScript) suggests this pattern is language-agnostic and fundamental to swarm coordination.

### Implementation

The minimum viable implementation requires three artifacts:

```yaml
# identity.yaml — persists across all sessions
name: auth-specialist
role: Authentication and authorization implementation
created: 2026-01-15
sessions_completed: 47
expertise_path: /path/to/auth-expertise.yaml

# Key capabilities accumulated over time
strengths:
  - OAuth2 flow implementation
  - Token refresh edge cases
  - Rate limiting patterns
```

```markdown
# work-history.md — append-only log
## 2026-02-10 Session 34
- Task: Implement refresh token rotation
- Status: Complete
- Branch: auth/refresh-rotation
- Notes: Edge case discovered—concurrent refresh requests need mutex

## 2026-02-09 Session 33
- Task: Fix token expiry race condition
- Status: Complete
- Branch: auth/token-expiry-fix
- Notes: Root cause was missing atomicity in token store
```

```bash
# prime.sh — session initialization
#!/bin/bash
# Load persistent identity into session context
cat identity.yaml
echo "---"
# Load recent work history (last 5 sessions)
tail -n 50 work-history.md
echo "---"
# Load domain expertise
cat "$EXPERTISE_PATH"
```

### When to Use

**Good fit:**
- Long-running systems where agents operate across days or weeks
- Domains where expertise accumulates (each session builds on prior knowledge)
- Systems where agent sessions crash or expire regularly
- Environments with model upgrades (new model inherits old identity)

**Poor fit:**
- One-off tasks with no continuity requirement
- Systems where every task is independent (no expertise accumulation)
- Environments where session persistence adds complexity without benefit

### Trade-offs

| Dimension | Benefit | Cost |
|-----------|---------|------|
| Continuity | Expertise survives crashes and restarts | Requires filesystem persistence layer |
| Context freshness | Each session starts with clean context window | Priming adds startup latency (~5-10 seconds) |
| Model flexibility | Identity persists across model upgrades | Seancing logic must handle format differences |
| Debugging | Full history of agent behavior across sessions | History files grow; require periodic pruning |

---

## Pattern 2: Hierarchical Watchdog Chains

Multi-tier supervision architecture where each tier handles failures at a different speed and reasoning depth, from millisecond process checks to hourly strategic assessments.

### The Problem

No single supervision mechanism handles the full failure spectrum. A fast process monitor catches crashes but cannot reason about stuck agents. An AI supervisor reasons about complex failures but operates too slowly for health checks. Production systems need both—and the layers between.

### Core Structure

```
Tier 4: Strategic Patrol (AI, slow, deep reasoning)
  │     "Is the overall system making progress?"
  │     Runs: Every 30-60 minutes
  │
Tier 3: Per-Unit Supervisor (AI, medium speed)
  │     "Is this specific worker doing its job correctly?"
  │     Runs: Per-task or periodic check
  │
Tier 2: AI Triage (AI, fast reasoning)
  │     "What kind of failure is this and how should I respond?"
  │     Runs: On failure detection from Tier 1
  │
Tier 1: Mechanical Daemon (no AI, fastest)
        "Is the process alive? Is it responsive?"
        Runs: Every 5-30 seconds
```

### How It Works

Each tier has a distinct capability and speed:

| Tier | Speed | Reasoning | Handles |
|------|-------|-----------|---------|
| **Daemon** (Tier 1) | Seconds | None (mechanical) | Process crashes, unresponsive agents, resource exhaustion |
| **Triage** (Tier 2) | Seconds-minutes | Fast AI reasoning | Failure classification, restart decisions, escalation |
| **Supervisor** (Tier 3) | Minutes | Medium AI reasoning | Quality assessment, rework requests, task reassignment |
| **Patrol** (Tier 4) | 30-60 minutes | Deep AI reasoning | System-wide health, progress assessment, strategic replanning |

**Key insight:** The lowest tier (Daemon) cannot reason—it only detects mechanical failures. The highest tier (Patrol) reasons deeply but operates too slowly for health checks. The Triage tier bridges the gap: fast enough to respond to Daemon alerts, smart enough to classify failures and route responses.

**Evidence:** Gas Town implements this as Daemon → Boot (triage) → Deacon (patrol) → Witness (per-unit supervisor). The Daemon detects process-level failures in seconds. Boot classifies them and decides between restart, escalate, or ignore. The Deacon performs periodic sweeps of overall system health. Witnesses supervise individual workers during task execution.

*[2026-02-13]*: Overstory's tiered watchdog architecture parallels Gas Town but organizes layers differently:

| Tier | Overstory | Gas Town |
|------|-----------|----------|
| 0 (Mechanical) | Watchdog daemon (tmux/PID monitoring) | Daemon (Go process) |
| 1 (AI Triage) | Ephemeral triage agent (on-demand classification) | Boot (ephemeral) |
| 2 (Monitor) | Continuous patrol agent (persistent background) | Deacon (persistent) |
| 3 (Supervisor) | Per-project oversight agent (depth-1) | Witness + Refinery |

Both implement the same escalation pattern (mechanical → AI reasoning → strategic) through different tier labels and agent roles. The convergence validates the three-tier model as robust across implementations. Overstory's session-based coordination adds a distinct constraint: Tier 3 supervisors are active agents in the practitioner's orchestrator session, not independent daemons.

### Escalation Flow

```
Daemon detects: Worker 7 unresponsive (30s timeout)
    │
    ▼
Triage receives alert
    ├── Check 1: Is worker process alive? → No → Restart worker, reassign task
    ├── Check 2: Is worker stuck in loop? → Yes → Kill session, seance + retry
    └── Check 3: Is failure recurring? → Yes → Escalate to Patrol
                                                    │
                                                    ▼
                                              Patrol assesses:
                                              "Worker 7 has failed 3 times
                                               on the same task. Task may be
                                               impossible. Reassign to senior
                                               worker or flag for human review."
```

### Implementation Principles

1. **Each tier watches the tier below it**: Triage monitors Daemon health. Patrol monitors Triage responsiveness. This prevents single points of failure in the supervision chain.

2. **Mechanical checks first, reasoning second**: Process alive? Memory under limit? Disk space available? These checks cost nothing and catch 80% of failures before AI reasoning is needed.

3. **Escalation thresholds, not escalation defaults**: Most failures resolve at Tier 1-2. Only recurring or novel failures reach Tier 3-4. This prevents expensive AI reasoning on routine crashes.

4. **Supervisor independence**: Each tier operates on its own schedule. The Daemon runs every 30 seconds regardless of what Triage is doing. Patrol runs hourly regardless of current failures. This prevents cascading stalls.

### When to Use

**Good fit:**
- Systems with 10+ agents where manual monitoring is impractical
- Long-running operations (hours to days) where failures are inevitable
- Autonomous systems where human intervention should be the last resort
- Environments with diverse failure modes (crashes, stalls, reasoning failures)

**Poor fit:**
- Small systems (1-3 agents) where a single supervisor suffices
- Short-lived tasks where restart-from-scratch is cheaper than recovery
- Environments with reliable infrastructure (cloud functions with built-in retry)

### Trade-offs

| Dimension | Benefit | Cost |
|-----------|---------|------|
| Recovery speed | Sub-minute for mechanical failures | Four-tier architecture is complex to build |
| Failure coverage | Handles crashes through strategic failures | Each tier requires distinct implementation |
| Autonomy | Reduces human intervention to rare edge cases | Incorrect triage can amplify failures |
| Observability | Each tier produces structured logs | Log volume grows with agent count |

---

## Pattern 3: Mail-Based Agent Coordination

Agents coordinate through structured, typed messages with defined routes rather than shared memory or synchronous calls. Each message is a discrete, traceable event that survives crashes, restarts, and network partitions.

### The Problem

Shared memory coordination breaks at scale. When 20 agents read and write a shared state file, race conditions and corruption become routine. Synchronous calls create cascading failures—one slow agent blocks the entire chain. Production systems need coordination that is asynchronous, typed, and auditable.

### Core Structure

```
Worker A ──── TASK_COMPLETE ────► Merge Queue
Worker B ──── MERGE_READY ──────► Merge Processor
Merge Proc ── REWORK_REQUEST ──► Worker B
Worker C ──── HELP_REQUEST ────► Supervisor
Supervisor ── HANDOFF ─────────► Worker D
```

### Message Types

A production mail protocol needs a small set of well-defined message types:

| Type | Sender | Receiver | Purpose |
|------|--------|----------|---------|
| `TASK_COMPLETE` | Worker | Supervisor / Queue | Work finished, ready for review |
| `MERGE_READY` | Worker | Merge Processor | Branch ready for integration |
| `REWORK_REQUEST` | Reviewer | Worker | Quality gate failed, changes needed |
| `HELP_REQUEST` | Worker | Supervisor | Stuck, needs guidance or reassignment |
| `HANDOFF` | Supervisor | Worker | Reassigning task from failed/stuck worker |
| `STATUS_UPDATE` | Worker | Dashboard | Progress notification for visibility |

### How It Works

1. **Messages are files, not function calls.** Each message persists on the filesystem as a structured document. This survives crashes—a message sent before a crash is still readable after restart.

2. **Routes are explicit.** Each message type has a defined sender and receiver. No broadcast flooding, no implicit subscriptions. Workers know exactly where to send each message type.

3. **Processing is idempotent.** Receivers handle duplicate messages gracefully. If a worker crashes after sending `TASK_COMPLETE` but before receiving acknowledgment, the message can be re-processed without side effects.

4. **Order is per-route, not global.** Messages between Worker A and the Merge Queue are ordered. Messages across different routes have no ordering guarantee. This prevents global ordering bottlenecks.

**Evidence:** Gas Town implements a typed mail protocol with `POLECAT_DONE`, `MERGE_READY`, `REWORK_REQUEST`, and other message types. Each message follows a defined route (sender → receiver), creating an auditable trail of every coordination event across 20+ concurrent workers.

*[2026-02-13]*: Overstory implements typed mail protocol through custom SQLite database (`.overstory/mail.db`) with WAL mode for concurrent access. Message schema includes 8 protocol types (`status`, `question`, `result`, `error`, `dispatch`, `escalation`, `worker_done`, `merge_ready`) with ~1-5ms query latency. Hook integration (UserPromptSubmit) injects unread messages into agent context automatically. The SQLite approach demonstrates that file-based mail can achieve sub-10ms coordination suitable for production swarms, validating the pattern across both filesystem-based (Gas Town) and database-backed (Overstory) implementations.

### Implementation

Minimal mail implementation using filesystem:

```
mail/
├── outbox/
│   └── worker-7-task-complete-2026-02-11T14:30:00.json
├── inbox/
│   ├── merge-queue/
│   │   └── worker-7-merge-ready-2026-02-11T14:31:00.json
│   └── supervisor/
│       └── worker-3-help-request-2026-02-11T14:25:00.json
└── processed/
    └── (messages moved here after handling)
```

```json
{
  "type": "TASK_COMPLETE",
  "sender": "worker-7",
  "receiver": "merge-queue",
  "timestamp": "2026-02-11T14:30:00Z",
  "payload": {
    "task_id": "AUTH-142",
    "branch": "worker-7/auth-142",
    "files_changed": 4,
    "summary": "Implemented token refresh rotation"
  }
}
```

### Comparison with Alternatives

| Approach | Durability | Auditability | Scale Behavior | Failure Mode |
|----------|-----------|-------------|----------------|-------------|
| **Shared memory** (files/DB) | Low (race conditions) | Low (overwrites) | Degrades at 5+ writers | Corruption, lost updates |
| **Synchronous calls** | None (in-memory) | Low (no trace) | Cascading stalls | Blocked chains |
| **Event bus** | Medium (depends on bus) | High | Good (if bus scales) | Single point of failure |
| **Mail protocol** | High (filesystem) | High (every message persisted) | Linear (per-route) | Message processing delay |

### When to Use

**Good fit:**
- 10+ agents needing coordination without shared state
- Systems where crash recovery requires replaying coordination history
- Environments requiring audit trails of agent interactions
- Async workflows where agents operate at different speeds

**Poor fit:**
- Tight real-time coordination (sub-second responses needed)
- Simple two-agent systems where direct messaging suffices
- Environments where filesystem I/O is expensive or unreliable

---

## Pattern 4: Self-Cleaning Workers

Workers have exactly three states: Working, Stalled, and Zombie. There is no idle state. On completion, workers push their branch, submit to the merge queue, destroy their sandbox, and terminate their session. An idle worker is treated as a system failure, not a valid operational state.

### The Problem

Agent systems leak resources. A worker that finishes its task but remains running consumes compute, holds sandbox resources, and occupies a slot that could serve new work. Across 20 workers, resource leaks compound into significant waste. Worse, "idle" workers mask system failures—the system appears to have capacity it cannot actually use.

### Core Structure

```
┌─────────────────────────────────────────────┐
│              Worker Lifecycle                │
│                                             │
│   ┌──────────┐   completion   ┌──────────┐  │
│   │ WORKING  │──────────────►│ CLEANUP  │  │
│   └──────────┘               └────┬─────┘  │
│        │                          │         │
│        │ failure              1. Push branch│
│        ▼                      2. Send DONE  │
│   ┌──────────┐               3. Destroy     │
│   │ STALLED  │                  sandbox     │
│   └──────────┘               4. Terminate   │
│        │                          │         │
│        │ unrecoverable           ▼         │
│        ▼                    ┌──────────┐   │
│   ┌──────────┐              │ TERMINAL │   │
│   │  ZOMBIE  │              └──────────┘   │
│   └──────────┘                              │
│                                             │
│   ✗ No IDLE state exists                    │
└─────────────────────────────────────────────┘
```

**Three states, no more:**

| State | Meaning | System Response |
|-------|---------|-----------------|
| **Working** | Actively executing task | Monitor progress |
| **Stalled** | Detected failure, potentially recoverable | Triage → restart or reassign |
| **Zombie** | Unrecoverable failure, consuming resources | Kill process, reclaim resources |

### The Idle Worker Heresy

Treating "idle" as a valid worker state creates three problems:

1. **Resource waste**: Idle workers hold compute, memory, and sandbox resources for zero productive output
2. **Capacity illusion**: The system reports N workers available, but some fraction produces no work
3. **Failure masking**: An agent that stops working but remains "idle" looks healthy to monitoring

**The principle:** When a worker finishes its task, it either picks up new work immediately or terminates. There is no state between "productive" and "gone."

**Evidence:** Gas Town codifies this as the "Idle Polecat Heresy." Workers execute `gt done` on task completion, which pushes the branch, submits to the merge queue, destroys the sandbox, and terminates the session. The system never has idle workers—only working workers and cleanup-in-progress workers.

*[2026-02-13]*: Overstory implements the same pattern through worktree lifecycle management. Agent completion triggers `bd close <task-id>`, which pushes the branch, sends `worker_done` mail to orchestrator, and marks the task complete. The `overstory worktree clean` command removes the worktree directory, deletes the branch reference (if merged), and kills the tmux session. Logs are preserved (never auto-deleted) for debugging. The convergence validates that self-cleaning workers are implementation-agnostic—both Go and TypeScript implementations discover that idle state is an anti-pattern at scale.

### Cleanup Sequence

A well-defined cleanup sequence prevents partial resource release:

```
1. Push branch to remote
   └── Failure here: Branch preserved locally, supervisor notified

2. Send TASK_COMPLETE message
   └── Failure here: Supervisor detects via timeout, queries branch state

3. Destroy sandbox / working directory
   └── Failure here: Watchdog daemon reclaims after TTL expires

4. Terminate session
   └── Failure here: Process monitor kills orphaned process
```

Each step has a fallback. The cleanup sequence is designed so that failure at any step triggers the watchdog chain (Pattern 2) rather than leaving resources in an ambiguous state.

### When to Use

**Good fit:**
- Systems with 10+ workers where resource leaks compound
- Cloud environments where idle compute costs money
- Long-running operations where workers spin up and down frequently
- Any system where "how many workers are actually productive?" matters

**Poor fit:**
- Systems where worker startup is expensive (amortize by keeping workers alive)
- Environments with a fixed worker pool (workers wait for work by design)
- Development environments where manual lifecycle management is acceptable

---

## Pattern 5: Convoy-Based Work Tracking

A persistent tracking unit (convoy) wraps ephemeral execution (swarms of workers), providing visibility and continuity across worker failures, restarts, and handoffs.

### The Problem

When 20 workers operate across multiple repositories, tracking overall progress becomes difficult. Individual workers know their task status, but no single view shows: "Of the 15 tasks in this feature, 12 are complete, 2 are in progress, and 1 has failed twice." Workers are ephemeral—they crash, restart, get reassigned. The tracking system must outlive any individual worker.

### Core Structure

```
┌─────────────────────────────────────────┐
│              CONVOY                       │
│  (persistent tracking unit)              │
│                                          │
│  Feature: Implement Auth Module          │
│  Issues: AUTH-140, AUTH-141, AUTH-142     │
│  Repos: api-service, auth-lib, tests     │
│  Status: 12/15 complete                  │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │         SWARM (ephemeral)        │    │
│  │  Worker 1: AUTH-140 ✓            │    │
│  │  Worker 2: AUTH-141 (in progress)│    │
│  │  Worker 3: AUTH-142 (failed → ?) │    │
│  └──────────────────────────────────┘    │
│                                          │
│  History:                                │
│  - Swarm 1: Workers 1-8, 8/10 complete   │
│  - Swarm 2: Workers 9-12, 4/5 complete   │
│  - Swarm 3: Workers 13-15 (current)      │
└──────────────────────────────────────────┘
```

**Convoy** = persistent. Tracks the full scope of work, maintains completion state, records history across multiple swarm executions. Survives any individual worker failure.

**Swarm** = ephemeral. A batch of workers executing a subset of convoy tasks. Swarms start, execute, and terminate. The convoy spawns new swarms as needed.

### How It Works

1. **Convoy creation**: Define the full scope of work (issues, repos, acceptance criteria)
2. **Swarm dispatch**: Convoy spawns a swarm of workers for a batch of tasks
3. **Progress tracking**: Workers report completion; convoy updates aggregate state
4. **Failure handling**: If a worker fails, the convoy records the failure and includes the task in the next swarm
5. **Completion**: When all tasks reach done state, the convoy sends completion notification

### Key Properties

**Persistence across failures:**
```
Swarm 1: Workers A, B, C
  - Worker A: completes task 1 ✓
  - Worker B: crashes on task 2 ✗
  - Worker C: completes task 3 ✓

Convoy state: tasks 1,3 complete. Task 2 needs retry.

Swarm 2: Workers D, E
  - Worker D: completes task 2 (retry) ✓
  - Worker E: completes task 4 ✓

Convoy state: tasks 1-4 complete. Done.
```

Worker B's crash did not lose progress. The convoy tracked what completed and what needed retry.

**Cross-repository visibility:**
A single convoy can track work spanning multiple repositories, providing a unified dashboard that no individual repository's issue tracker can offer.

**Evidence:** Gas Town convoys group issues across repositories into a single tracking unit. Ephemeral worker swarms execute batches of tasks. When a worker fails, the convoy records it and includes the task in the next swarm. Dashboard visibility shows convoy-level completion, not per-worker status.

*[2026-02-13]*: Overstory implements batch coordination through task groups (`overstory group create <name>`). Groups track completion state across multiple beads (issues), auto-closing when all member issues complete. The persistent tracking survives worker failures—groups live in `.overstory/` filesystem state, not worker memory. While Gas Town's convoy pattern is more elaborate (multi-repo, multi-swarm), Overstory's task groups validate the core insight: persistent tracking units must outlive ephemeral worker executions.

### When to Use

**Good fit:**
- Multi-repo features requiring coordinated tracking
- Work that spans multiple swarm executions (too large for one batch)
- Systems where worker failures are expected and retries are routine
- Environments needing dashboard-level visibility into multi-agent progress

**Poor fit:**
- Single-task execution (tracking overhead exceeds benefit)
- Short-lived tasks where simple success/failure is sufficient
- Systems with reliable workers where retry logic is rarely needed

---

## Pattern 6: Merge Queue with AI Conflict Resolution

Workers never push directly to the main branch. A dedicated merge processor validates each submission, resolves conflicts, and—when conflicts become intractable—can re-implement the change against the current main branch state.

### The Problem

With 20 workers pushing branches in parallel, merge conflicts are not occasional—they are continuous. Two workers modifying adjacent code, shared configuration files, or test fixtures create conflicts that multiply with worker count. Manual resolution does not scale. Automated resolution must handle not just textual conflicts but semantic ones.

### Core Structure

```
Worker 1 ─── branch ──► ┌──────────────┐
Worker 2 ─── branch ──► │              │
Worker 3 ─── branch ──► │  Merge Queue │ ──► main branch
  ...                    │  (ordered)   │
Worker N ─── branch ──► │              │
                         └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │    Merge     │
                         │  Processor   │
                         │              │
                         │ 1. Validate  │
                         │ 2. Merge     │
                         │ 3. Resolve   │
                         │ 4. Re-imagine│
                         └──────────────┘
```

### Resolution Tiers

The merge processor operates through escalating resolution strategies:

| Tier | Strategy | When | Cost |
|------|----------|------|------|
| **Clean merge** | `git merge` succeeds | No conflicts | Seconds |
| **Auto-resolve** | Standard conflict resolution (accept theirs/ours by region) | Textual conflicts in non-overlapping regions | Seconds |
| **AI resolve** | AI reads both versions, produces merged result | Semantic conflicts requiring understanding of intent | Minutes |
| **Re-imagination** | AI re-implements the change against current main | Intractable conflicts where merge is harder than reimplementation | Minutes-hours |

**Re-imagination** is the critical innovation. When a worker's branch has drifted so far from main that merging is impractical, the merge processor understands the *intent* of the change (from the task description and branch diff) and re-implements it cleanly against current main.

**Evidence:** Gas Town's Refinery agent processes the merge queue. When standard merge fails, it escalates through AI resolution. For intractable conflicts—common when 20+ workers modify shared test fixtures—the Refinery re-imagines the implementation against current main state rather than attempting to merge divergent branches.

*[2026-02-13]*: Overstory's merge queue implements 4-tier resolution escalation:

1. **Clean merge** — `git merge` succeeds (seconds)
2. **Auto-resolve** — Standard conflict resolution by region (seconds)
3. **AI resolve** — AI reads both versions, produces merged result (minutes)
4. **Re-imagine** — AI re-implements change against current main (minutes-hours)

The "re-imagination" tier matches Gas Town's Refinery pattern, validating the approach across implementations. When merge drift exceeds auto-resolution capacity, reimplementation becomes cheaper than conflict archaeology. Both systems discover that Tier 4 is not a failure mode—it's a pragmatic escalation strategy when branch divergence is severe.

### Queue Properties

1. **FIFO ordering with priority override**: Normal submissions process in order. Critical fixes can jump the queue.
2. **Atomic commits**: Each merge produces one commit on main. The history remains linear and bisectable.
3. **Rejection with feedback**: Failed merges send `REWORK_REQUEST` back to the originating worker (or its successor, via Pattern 1's persistent identity).
4. **Idempotent processing**: Re-submitting the same branch to the queue produces the same result. Safe to retry after processor crashes.

### When to Use

**Good fit:**
- 5+ workers producing branches in parallel
- Repositories with high contention (shared files, configuration, tests)
- Systems requiring linear commit history (bisect-friendly)
- Environments where merge conflict resolution is a bottleneck

**Poor fit:**
- Single-worker systems (no contention)
- Repositories where workers modify completely disjoint files
- Environments where merge commits are acceptable (no linearity requirement)

---

## Pattern Composition

These six patterns compose into a coherent production architecture. No pattern operates in isolation—each addresses a specific concern while relying on others for complementary capabilities.

### Composition Map

```
                    Convoy (tracking)
                         │
                    ┌────▼────┐
                    │  Swarm  │ ◄── Persistent Identity
                    │ Dispatch│     (workers know who they are)
                    └────┬────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
         ┌────▼───┐ ┌───▼────┐ ┌──▼─────┐
         │Worker 1│ │Worker 2│ │Worker 3│
         └────┬───┘ └───┬────┘ └──┬─────┘
              │          │          │
              │    Mail Protocol    │ ◄── Coordination
              │    (typed messages) │
              │          │          │
         ┌────▼──────────▼──────────▼───┐
         │        Merge Queue           │ ◄── Integration
         └──────────────┬───────────────┘
                        │
                   main branch

         Watchdog Chain observes ALL layers
```

### Real-World Composition Examples

**Feature implementation across 3 repositories:**

1. **Convoy** created with 15 tasks across `api-service`, `auth-lib`, `test-suite`
2. **Swarm** of 8 workers dispatched, each with **persistent identity** (knows repo conventions from prior sessions)
3. Workers coordinate via **mail protocol** — `TASK_COMPLETE`, `HELP_REQUEST` messages flow between workers and supervisor
4. **Self-cleaning**: Each worker pushes branch + terminates on completion
5. **Merge queue** processes 8 branches, resolves 2 conflicts via AI, re-imagines 1 intractable conflict
6. **Watchdog chain**: Daemon detects Worker 5 stalled at 3 minutes. Triage restarts session. Worker 5v2 seances predecessor, completes task.
7. **Convoy** records: 8/15 complete. Dispatches Swarm 2 for remaining tasks.

**Bug hotfix with 20 workers already running:**

1. Hotfix branch submitted to **merge queue** with priority override
2. **Mail protocol** broadcasts `PRIORITY_MERGE` to all workers
3. Workers acknowledge—those modifying affected files pause and rebase
4. Merge queue processes hotfix first, then resumes normal queue
5. **Watchdog** monitors rebase status, flags workers that fail to rebase within timeout

---

## Anti-Patterns

### Anti-Pattern: The Idle Worker Heresy

**What it is:** Treating "idle" as a valid operational state for workers. Workers finish their task and wait for new assignments while holding resources.

**Why it fails:** Idle workers consume compute and sandbox resources while producing nothing. At scale, 5 idle workers out of 20 represent 25% wasted capacity. Worse, the system reports 20 "healthy" workers when only 15 are productive, masking true throughput.

**Better approach:** Self-Cleaning Workers (Pattern 4). On task completion, workers submit results and terminate. The orchestrator spawns new workers when new tasks arrive. Resources track demand, not headcount.

---

### Anti-Pattern: Batch-Closure Heresy

**What it is:** Closing work steps retroactively—marking tasks as done after moving on to subsequent work, or updating status in batches rather than at the moment of completion.

**Why it fails:** Retroactive status updates corrupt the timeline. When debugging a failure at 14:30, the investigation depends on knowing what was actually complete at 14:30. If tasks were batch-closed at 15:00, the timeline shows everything completing simultaneously, destroying the ability to reconstruct the failure sequence.

**Better approach:** Workers update status at the moment of state change. `TASK_COMPLETE` messages are sent when the task completes, not when the worker remembers to send them. Convoy tracking (Pattern 5) depends on real-time status for accurate progress dashboards.

---

### Anti-Pattern: Trust Without Verification

**What it is:** Skipping quality gates because the agent "usually gets it right" or because verification adds latency.

**Why it fails:** Agent reliability varies with task complexity, context state, and model behavior. A worker that produces correct code 95% of the time generates 1 defective output per 20 tasks. At 20 parallel workers processing 5 tasks each, that is 5 defective outputs per batch—enough to corrupt a release.

**Better approach:** Quality gates at every integration point. The merge queue (Pattern 6) validates before merging. Supervisors (Pattern 2, Tier 3) assess work quality. Trust accumulates through verified track record in the worker's persistent identity (Pattern 1), not through assumption.

---

## Pattern Selection Guide

Not every system needs all six patterns. Complexity should match scale.

### Scale Thresholds

| Pattern | Useful At | Essential At | Overkill Below |
|---------|-----------|-------------|----------------|
| Persistent Identity | 3+ agents | 10+ agents with turnover | Single-session tasks |
| Watchdog Chains | 5+ agents | 10+ autonomous agents | Supervised agents |
| Mail Protocol | 5+ agents needing coordination | 20+ agents | 2-3 agents with shared state |
| Self-Cleaning Workers | 5+ ephemeral workers | Any system paying for idle compute | Fixed worker pools |
| Convoy Tracking | 10+ tasks across sessions | Multi-repo multi-session work | Single-batch tasks |
| Merge Queue | 3+ parallel writers | 10+ parallel writers to same repo | Sequential workflows |

### Decision Framework

```
How many concurrent agents?
│
├── 1-3 agents
│   └── Persistent Identity (if multi-session)
│       └── Everything else is likely overkill
│
├── 3-10 agents
│   ├── Persistent Identity (if sessions expire)
│   ├── Merge Queue (if parallel branches)
│   ├── Self-Cleaning Workers (if cloud compute)
│   └── Simple supervisor (no full watchdog chain)
│
├── 10-20 agents
│   ├── All patterns become relevant
│   ├── Watchdog Chain (essential for autonomous recovery)
│   ├── Mail Protocol (shared state breaks at this scale)
│   ├── Convoy Tracking (for visibility)
│   └── Full Merge Queue with AI resolution
│
└── 20+ agents
    └── All six patterns are essential
        └── Without them: resource leaks, lost work,
            undetected failures, merge chaos
```

### Pattern Introduction Order

When building a production multi-agent system incrementally, introduce patterns in this order:

1. **Self-Cleaning Workers** — Prevents resource leaks from day one
2. **Persistent Identity** — Enables continuity as sessions start expiring
3. **Merge Queue** — Required as soon as parallel branches exist
4. **Mail Protocol** — Replaces shared state when coordination needs grow
5. **Convoy Tracking** — Added when work spans multiple sessions or repos
6. **Watchdog Chains** — Built when the system needs autonomous recovery

Each pattern builds on the previous. Self-Cleaning Workers is the foundation because resource discipline prevents the leaks that make every other problem worse.

---

## Connections

- **To [Expert Swarm](8-expert-swarm-pattern.md)**: Expert Swarm addresses *what* parallel agents work on (domain-consistent tasks with shared expertise). Production Multi-Agent Systems addresses *how* those agents operate reliably (identity persistence, supervision, coordination, cleanup). The two compose naturally: Expert Swarm provides task decomposition and expertise inheritance; these patterns provide the operational infrastructure.

- **To [Orchestrator Pattern](3-orchestrator-pattern.md)**: The orchestrator coordinates task assignment and synthesis. Production patterns extend this with supervision (watchdog chains), durable coordination (mail protocol), and integration (merge queue). An orchestrator without production patterns works in development; production requires both.

- **To [Autonomous Loops](4-autonomous-loops.md)**: Self-Cleaning Workers and Convoy Tracking apply the same resource discipline to autonomous loop execution. Long-running loops benefit from watchdog supervision and persistent identity to survive context window exhaustion.

- **To [Context Management](../4-context/2-context-strategies.md)**: Persistent Identity, Ephemeral Sessions is fundamentally a context management pattern—separating durable knowledge (identity) from ephemeral working memory (session context). The seancing mechanism implements selective context retrieval across sessions.

- **To [Workflow Coordination](../7-practices/5-workflow-coordination.md)**: Mail-Based Coordination and Convoy Tracking implement coordination practices at the infrastructure level. The typed message protocol provides the primitives that workflow coordination patterns build upon.

- **To [Production Concerns](../7-practices/4-production-concerns.md)**: These patterns directly address production concerns: resource management (Self-Cleaning), failure recovery (Watchdog Chains), integration safety (Merge Queue), and operational visibility (Convoy Tracking).

- **To [Operating Agent Swarms](../7-practices/7-operating-agent-swarms.md)**: Production Multi-Agent Systems defines the architectural patterns (persistent identity, watchdog chains, merge queues); Operating Agent Swarms covers the operational practices for running these systems day-to-day (cost management, scale transitions, incident response).

---

## Open Questions

- What is the optimal watchdog chain depth? Gas Town uses four tiers, but three or five tiers may suit different scale points. More tiers increase coverage but add supervision overhead.
- How should mail protocol message types evolve as the system grows? Starting with 6 types works, but 50-agent systems may need message subtypes or routing rules.
- When does re-imagination (merge queue Tier 4) produce worse results than asking the original worker to rebase manually? The cost crossover point is unclear.
- Can persistent identity transfer across different model providers? A worker's CV built with Claude may not map cleanly to Gemini's reasoning patterns.
- How do convoy boundaries interact with organizational boundaries? A convoy spanning teams introduces coordination costs beyond the technical patterns.
- What telemetry matters most for watchdog chains? False positive rates for stall detection, escalation frequency, and mean-time-to-recovery are candidates, but the right dashboard is undetermined.
- At what scale does the mail protocol need a dedicated message broker instead of filesystem-based persistence? Gas Town operates at 20 agents on filesystem; the scaling ceiling is untested.
- How do self-cleaning workers interact with spot instance preemption? Cloud providers terminate instances externally—does the cleanup sequence need a "preemption-aware" variant?
