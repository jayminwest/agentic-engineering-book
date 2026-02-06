---
title: Expert Swarm Pattern
description: Combining domain expertise with parallel swarm execution
created: 2026-02-02
last_updated: 2026-02-05
tags: [pattern, swarm, orchestration, expertise, parallel]
part: 2
part_title: Craft
chapter: 6
section: 8
order: 2.6.8
---

# Expert Swarm Pattern

Domain experts coordinate parallel workers that inherit shared expertise, achieving both scale and consistency.

---

## Core Problem

Traditional orchestration patterns face a fundamental tension: generic orchestrators achieve parallelism but lack domain context, while domain experts maintain consistency but execute sequentially. Expert Swarm resolves this by combining expertise inheritance with parallel execution.

**Production Evidence:** Commit 20500f1 (2026-01-30) demonstrates the pattern at scale—10 parallel agents generated 3,082 lines across 20 files in ~4 minutes. Quality remained consistent across all parallel work: Core Questions sections, Mental Model framing, trade-off tables, and cross-references all followed book conventions. The shared expertise.yaml provided consistent guidance without requiring the coordinator to micromanage each worker.

---

## Core Structure

```
Expert Lead (existing domain expert)
    │
    ├─── expertise.yaml (shared knowledge source)
    │
    ├─── Narrow Worker 1
    │    └─── EXPERTISE_PATH: /path/to/expertise.yaml
    │    └─── Task: Implement section A
    │
    ├─── Narrow Worker 2
    │    └─── EXPERTISE_PATH: /path/to/expertise.yaml
    │    └─── Task: Implement section B
    │
    ├─── Narrow Worker 3...N
    │
    └─── Join/Synthesis Phase
         └─── Aggregate results, verify consistency
```

**Key elements:**

1. **Expert as orchestrator** - Existing domain expert (plan/build agent) acts as coordinator
2. **Expertise inheritance** - Workers receive path to expertise.yaml, not full context copy
3. **Narrow scoping** - Each worker executes one focused task
4. **Path-passing protocol** - `EXPERTISE_PATH: /absolute/path/to/domain/expertise.yaml`
5. **Learning separation** - Workers execute; improve agents analyze afterward

---

## The Hybrid Approach

Expert Swarm differs from both traditional orchestration and pure swarm patterns by combining elements of each.

### Expert as Lead vs. Generic Orchestrator

The lead agent is not a generic coordinator—it's an existing domain expert with full context of patterns, anti-patterns, and decision heuristics accumulated through prior work.

**Expert Lead Characteristics:**
- Maintains domain expertise.yaml (typically 500-750 lines)
- Understands decomposition patterns specific to domain
- Knows which tasks require tight coupling vs. parallelization
- Can synthesize worker results using domain context

**Generic Orchestrator Characteristics:**
- No domain-specific knowledge
- Coordinates through generic patterns (scout → plan → build → review)
- Workers receive narrow specs but no shared mental model
- Synthesis relies on structural patterns, not domain heuristics

### Workers: Dynamically Spawned, Expertise-Enhanced

Workers are ephemeral agents spawned for single tasks. Unlike full experts, they:
- Execute one focused task only
- Inherit expertise via path-passing (not full context)
- Return summaries, not comprehensive documentation
- Do not update expertise.yaml (that's the improve agent's job)

---

## Expertise Inheritance Protocol

The core innovation: passing expertise.yaml location to workers rather than copying content into their context.

### Path-Passing Syntax

Workers receive expertise location and scope focus:

```markdown
EXPERTISE_PATH: /Users/jayminwest/Projects/repo/.claude/agents/experts/knowledge/expertise.yaml

Read this file and apply relevant patterns to your task.
Focus on these sections:
- Implementation Standards
- Content Structure Patterns
- Voice Implementation Patterns

Your Task: Create chapters/6-patterns/8-expert-swarm-pattern.md
```

### Why Not Context Copying?

| Approach | Pros | Cons |
|----------|------|------|
| **Copy expertise into context** | No file I/O; expertise visible in prompt | Pollutes context with 500-750 lines per worker; synchronization issues when expertise updates |
| **Pass expertise path** | Clean worker context; single source of truth; scales to 10+ workers | Requires file read; relies on workers reading relevant sections |

Path-passing keeps the orchestrator's context clean while ensuring all workers reference the same knowledge source. When expertise.yaml updates, all future workers automatically benefit.

### Scope Extraction

Workers don't need to read all 750 lines of expertise—they focus on sections relevant to their task:

- **Content generation tasks** → Read "Content Structure Patterns"
- **Style consistency tasks** → Read "Voice Implementation Patterns"
- **Cross-referencing tasks** → Read "Linking Strategy"

This selective reading reduces token overhead while maintaining consistency.

### Size Governance

From orchestration/expertise.yaml: "Target maximum ~750 lines for expertise.yaml to prevent context bloat when inherited by workers."

Size governance ensures expertise inheritance remains sustainable at scale. A 750-line expertise.yaml costs ~3,000 tokens per worker—manageable for 10 workers (~30,000 tokens total), unsustainable if expertise balloons to 2,000+ lines.

---

## Communication Patterns

Expert Swarm coordinates through two primary mechanisms: spec files and TeammateTool messaging.

### Spec-as-Artifact

The orchestrator creates specification files that workers reference:

```
Orchestrator creates: .claude/.cache/specs/knowledge/expert-swarm-pattern-spec.md
Workers read spec + expertise.yaml
Workers implement their section
Workers report summary back to orchestrator
```

Specs serve as coordination state—workers don't receive context directly from the orchestrator. This prevents context pollution and enables true parallelism (workers operate independently).

### Agent Teams Messaging (When Available)

*[2026-01-30]*: Agent teams (TeammateTool) provide richer coordination primitives beyond spec files. Currently experimental (accessible via `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), but enable:

- **Write**: Send message to specific worker
- **Broadcast**: Notify all workers of state change
- **Read**: Workers query messages without orchestrator mediation

**Expert Swarm + Agent Teams:**
```python
# Orchestrator broadcasts expertise update
SendMessage(type: "broadcast", content: "Expertise updated: new voice pattern added")

# Workers read messages and reload expertise
message = ReceiveMessage()
if "Expertise updated" in message:
    reload_expertise()
```

See [Agent Teams documentation](../../9-practitioner-toolkit/1-claude-code.md#agent-teams-native-multi-agent-coordination-experimental) for coordination primitives and patterns.

### No Context Passing Between Agents

Workers do not receive context from other workers. Coordination flows through:
- Spec files (explicit state)
- TeammateTool messages (notifications)
- Orchestrator synthesis (final aggregation)

This isolation prevents context leakage and keeps workers focused on narrow tasks.

---

## Learning Separation

Expert Swarm enforces strict separation between execution and learning phases.

### The Boundary

**Swarm agents (workers):**
- Execute ONE task only
- Read expertise.yaml for guidance
- Do NOT update expertise.yaml
- Return summary of work completed

**Improve agents:**
- Run AFTER swarm completion
- Analyze git history + changed files
- Extract patterns and learnings
- Update expertise.yaml for future swarms

### Why This Matters

Allowing workers to update expertise during parallel execution creates race conditions:

```
Worker 1 reads expertise.yaml (version A)
Worker 2 reads expertise.yaml (version A)
Worker 1 updates expertise.yaml → version B
Worker 2 updates expertise.yaml → version C (overwrites B)
Result: Worker 1's learnings lost
```

Sequential improve phase after swarm completion prevents conflicts:

```
Workers 1-10 execute in parallel (read-only access to expertise)
Swarm completes → commit
Improve agent analyzes all 10 worker outputs
Improve agent updates expertise.yaml once
Next swarm benefits from all learnings
```

### The Workflow

```
1. Expert lead spawns 10 workers (read expertise.yaml)
2. Workers execute tasks in parallel
3. Workers complete, orchestrator synthesizes
4. Commit changes
5. Run improve agent (analyzes git history)
6. Improve agent updates expertise.yaml
7. Next swarm uses improved expertise
```

This separation keeps workers focused on execution while ensuring knowledge accumulates over time.

---

## Scale Considerations

Expert Swarm achieves meaningful parallelism through single-message spawning and expertise size governance.

### Single-Message Parallelism

**Critical implementation detail:** All parallel workers must be invoked in one orchestrator message. Sequential messages serialize execution.

```markdown
# CORRECT: True parallelism
In a single message:
- Task(prompt="Worker 1: implement section A\nEXPERTISE_PATH: /path/to/expertise.yaml")
- Task(prompt="Worker 2: implement section B\nEXPERTISE_PATH: /path/to/expertise.yaml")
- Task(prompt="Worker 3: implement section C\nEXPERTISE_PATH: /path/to/expertise.yaml")
...10 workers total

Result: All workers execute concurrently (~4 minutes wall-clock)

# INCORRECT: Serialized execution
Message 1: Task(prompt="Worker 1...")
Wait for completion
Message 2: Task(prompt="Worker 2...")
Wait for completion
...

Result: Workers execute sequentially (~40 minutes wall-clock)
```

The difference compounds: 10 agents in parallel complete in roughly the same time as 1 agent; 10 agents serialized take 10× longer.

### Production Evidence at Scale

**Commit 20500f1 (2026-01-30):** 10-agent swarm implementing book content updates

**Metrics:**
- **Agents**: 10 workers + 1 orchestrator
- **Tasks**: 11 executed (10 workers + 1 synthesis)
- **Output**: 3,082 lines added
- **Files**: 7 created, 15 modified
- **Time**: ~4 minutes wall-clock
- **Consistency**: Zero voice drift; all entries followed structure standards

**Speedup calculation:**
- Sequential estimate: 10 tasks × 4 minutes = 40 minutes
- Actual parallel execution: 4 minutes
- **Speedup: 10×**

**Work completed:**
- New pattern entries: ReAct (320 lines), HITL (551 lines), Progressive Disclosure (307 lines)
- Expanded entry: Debugging Agents (250 → 1,030 lines, 312% growth)
- Cross-reference fixes: 15 broken links across 10 files
- Quality: Full structure maintained (Core Questions, Mental Model, Trade-offs, Connections)

### Token Economics

**Per-worker overhead:**
- Expertise.yaml: ~750 lines = ~3,000 tokens
- Task spec: ~100-200 tokens
- **Total**: ~3,200 tokens per worker for expertise inheritance

**10-worker swarm:**
- Expertise cost: 10 × 3,000 = 30,000 tokens
- Spec cost: 10 × 150 = 1,500 tokens
- **Total coordination overhead**: ~31,500 tokens

This is manageable because workers execute in parallel—the orchestrator's context window doesn't accumulate all worker contexts. Each worker operates independently with its own context budget.

### Orchestrator Context Hygiene

Workers return summaries, not full implementation details:

```markdown
# Worker report (good)
"Created chapters/6-patterns/8-expert-swarm-pattern.md (450 lines).
Sections: Core Structure, Expertise Inheritance, Communication Patterns,
Learning Separation, Scale Considerations. Cross-references: 5 entries.
Followed third-person voice, included production evidence from commit 20500f1."

# Worker report (bad - context pollution)
"Here's the full file I created:
---
title: Expert Swarm Pattern
description: ...
[450 lines of content dumped into orchestrator context]
```

The orchestrator needs confirmation of completion and high-level summary—not full artifacts. This keeps the orchestrator's context available for synthesis and coordination decisions.

---

## Implementation Examples

### Real Prompt: Spawning Expert Workers

From commit 20500f1, orchestrator spawning knowledge expert workers:

```markdown
Create chapters/6-patterns/8-expert-swarm-pattern.md

SPEC: .claude/.cache/specs/knowledge/expert-swarm-pattern-spec.md
EXPERTISE_PATH: .claude/agents/experts/knowledge/expertise.yaml

Read the spec and expertise file. Focus on:
- Implementation Standards
- Content Structure Patterns
- Voice Implementation Patterns

Implement the entry following the 12-section structure defined in the spec.
Include production evidence from commit 20500f1.
Return: Summary of what was built (sections, line count, cross-references).
```

**Key elements:**
1. Spec location (coordination state)
2. Expertise path (knowledge inheritance)
3. Focus guidance (which expertise sections matter)
4. Task scope (create one file)
5. Expected output (summary, not full content)

### Expert Lead Synthesis Pattern

After workers complete:

```markdown
Received reports from 10 workers:
- Worker 1: Created ReAct pattern (320 lines)
- Worker 2: Created HITL pattern (551 lines)
- Worker 3: Expanded debugging-agents (250 → 1,030 lines)
...

Synthesis tasks:
1. Verify all files exist and are well-formed
2. Check cross-references (are links valid?)
3. Update _index.md with new entries
4. Commit with descriptive message
5. Queue improve agent for post-analysis
```

The expert lead synthesizes worker outputs using domain knowledge—verifying cross-references, updating indexes, and ensuring consistency patterns are maintained.

---

## Trade-offs and Limitations

Expert Swarm optimizes for scale + consistency at the cost of coordination complexity.

### Comparison with Alternatives

| Dimension | Traditional Orchestrator | Expert Swarm | Model-Native Swarm |
|-----------|-------------------------|--------------|-------------------|
| **Scale** | 5-10 agents (SDK-limited) | 10-20 agents (SDK-limited) | 100+ agents (model-internal) |
| **Consistency** | Low (no shared expertise) | High (expertise.yaml) | Variable (training-dependent) |
| **Domain knowledge** | None (generic coordinator) | High (accumulated expertise) | Unknown (model-internal) |
| **Coordination mechanism** | Task tool + specs | Task tool + specs + expertise path | Model-internal orchestration |
| **Learning** | None | Separate improve phase | Implicit in training |
| **Debugging** | Explicit orchestration trace | Explicit + expertise trace | Opaque (model decisions) |
| **Infrastructure** | SDK orchestration only | SDK + shared expertise files | Requires model with native capability |
| **Setup complexity** | Simple (generic patterns) | Moderate (expertise.yaml governance) | Complex (model selection, training) |
| **Coordination cost** | Low (minimal prompts) | Medium (expertise loading per worker) | Low (trained behavior) |
| **Expertise updates** | N/A | Explicit improve agent | N/A |
| **Nesting** | Unreliable (Claude Code) | Unreliable (same limitation) | Unknown (model-dependent) |

### Limitations

**Flat Architecture (Current Constraint):**
Claude Code subagent nesting is unreliable—workers cannot reliably spawn sub-workers. This limits decomposition depth.

**Aspirational nested approach:**
```
Expert Lead
├─ Worker 1 (section A)
│  ├─ Sub-worker 1.1 (subsection A.1)
│  └─ Sub-worker 1.2 (subsection A.2)
├─ Worker 2 (section B)
│  └─ ...
```

Currently blocked by infrastructure. Document as future capability when nesting becomes reliable.

**Expertise Synchronization:**
The 750-line target requires discipline. Without governance, expertise.yaml grows unbounded, inflating token costs and diluting signal.

**Agent Teams Availability:**
Advanced messaging patterns (Write, Broadcast, Read) are currently experimental (requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` environment variable). Production systems should rely on spec-as-artifact coordination as the stable fallback until agent teams reach general availability.

**Token Overhead:**
Expertise path-passing adds ~3,000 tokens per worker (for 750-line expertise.yaml). At 10 workers, this is 30,000 tokens—manageable but not free. Expertise size governance directly impacts scaling economics.

---

## When to Use Expert Swarm

### Good Fit

**Multiple independent tasks within single domain:**
- Implementing 10 book chapters following the same structure
- Migrating 8 services to new API contract
- Expanding 12 test suites with consistent patterns

**Domain expertise is well-documented:**
- Expertise.yaml exists (500-750 lines)
- Patterns, anti-patterns, voice guidelines codified
- Decision heuristics clear enough for workers to apply

**Quality consistency valued over raw speed:**
- Voice drift would be costly (user-facing content)
- Anti-patterns must be avoided (security, compliance)
- Cross-reference integrity critical (documentation, books)

**Scale requirements exceed single-agent capacity:**
- 10+ similar tasks that would take 40+ minutes sequentially
- Time-sensitive delivery (parallel execution reduces wall-clock time)

### Poor Fit

**Simple single-file changes:**
- Overhead exceeds benefit
- Direct implementation faster than coordination

**Tasks require tight coupling:**
- Sequential dependencies between subtasks
- Each task depends on prior results
- Better served by sequential expert execution

**Domain expertise not yet codified:**
- Expertise.yaml doesn't exist
- Patterns still emerging through experimentation
- Generic orchestration sufficient until patterns stabilize

**Cross-domain work:**
- Each task requires different expertise.yaml
- Better served by multi-expert orchestration (council pattern)

**Learning required during execution:**
- Workers need to update expertise based on discoveries
- Conflicts with learning separation constraint
- Use sequential expert pattern with improve cycles instead

### Decision Framework

```
Multiple independent tasks? ─No─→ Use sequential Expert Pattern
          │
         Yes
          │
Within single domain? ─No─→ Use Multi-Expert Orchestrator (Council)
          │
         Yes
          │
Do teammates need to message ─Yes─→ Use Agent Teams (if available)
each other during execution?      Otherwise use Expert Swarm + spec files
          │
          No
          │
Expertise.yaml exists? ─No─→ Create expertise first OR use generic orchestrator
          │
         Yes
          │
Consistency critical? ─No─→ Generic orchestrator may suffice
          │
         Yes
          │
Scale justifies overhead? ─No─→ Sequential expert pattern
          │
         Yes
          │
   Use Expert Swarm Pattern
```

---

## Flat Team Coordination Archetypes

*[2026-02-05]*: Production agent teams operate under strict constraints: no nested teams, one team per session, flat hierarchy. Within this constraint, two coordination archetypes emerge: Implementation Pattern (file ownership) and Council Pattern (read-only analysis).

### The Flat Team Constraint

Agent teams systems enforce three architectural constraints:

1. **No nested teams**: Teams cannot spawn sub-teams
2. **One team per session**: Single active team coordination boundary
3. **Flat hierarchy**: All teammates are peers, no sub-coordinators

**Why these constraints:**
- Prevents coordination complexity explosion
- Keeps orchestration traceable (no hidden delegation layers)
- Simplifies mental model: lead coordinates N peers, peers collaborate

### Implementation Pattern: File Ownership Coordination

**Structure:**
```
Lead (orchestrator, coordination-only tools)
├── Build Agent 1: src/auth/*.ts (exclusive ownership)
├── Build Agent 2: src/api/*.ts (exclusive ownership)
├── Build Agent 3: src/utils/*.ts (exclusive ownership)
└── ... (each agent owns distinct file scope)
```

**Key characteristics:**
- Each teammate gets explicit file ownership scope
- No overlap: prevents write conflicts
- Build specialists with Write, Edit, Bash tools
- Parallel execution (no shared state)

**Use when:**
- Implementing multiple independent modules
- Clear file boundaries exist
- Work parallelizes cleanly across file scopes

**Trade-offs:**

| Advantage | Cost |
|-----------|------|
| Zero coordination overhead (no shared state) | Requires upfront decomposition |
| True parallelism (no blocking) | Rigid boundaries (hard to shift scope) |
| Conflict-free merging | Less effective for cross-cutting changes |

### Council Pattern: Independent Analysis

**Structure:**
```
Lead (orchestrator, aggregates findings)
├── Security Analyst: Read, Grep only
├── Performance Auditor: Read, Grep only
├── Code Quality Reviewer: Read, Grep only
└── ... (each analyst provides independent perspective)
```

**Key characteristics:**
- All teammates read-only (Read, Grep tools)
- No coordination needed (independent analyses)
- Parallel execution (no dependencies)
- Lead synthesizes perspectives

**Use when:**
- Multi-perspective review required (security + performance + quality)
- Independent expert opinions valued
- No implementation work needed

**Trade-offs:**

| Advantage | Cost |
|-----------|------|
| Diverse perspectives (independent analysis) | No implementation (analysis-only) |
| Embarrassingly parallel (no blocking) | Requires synthesis (lead aggregates) |
| Clean separation (analysts can't interfere) | Read-only limits (can't demonstrate fixes) |

### Serialization Within Parallel Teams: `addBlockedBy`

**The problem:** Sometimes teammates need sequential coordination:
```
Agent A: Implement database schema
Agent B: Implement API routes (DEPENDS ON schema)
```

**The solution:** `addBlockedBy` marks dependency without breaking parallelism:

```json
{
  "task_id": "implement-api-routes",
  "agent": "build-agent-2",
  "blocked_by": ["implement-schema"]
}
```

Orchestrator spawns both agents in parallel, but Agent B waits for Agent A completion before starting.

**When to use:**
- Sequential dependencies within parallel teams
- Batch 1 (no deps) → Batch 2 (depends on Batch 1) workflows
- Prevents agents from blocking on missing preconditions

### Hybrid Approach: Mixed Archetypes

Production teams often combine patterns:
```
Lead
├── Implementation Track (file ownership)
│   ├── Build Agent 1: auth module
│   └── Build Agent 2: api module
└── Council Track (read-only review)
    ├── Security Analyst
    └── Performance Auditor
```

**Workflow:**
1. Lead spawns implementation track (parallel builds)
2. Implementation agents complete work
3. Lead spawns council track (parallel review)
4. Lead synthesizes findings and addresses issues

This combines parallelism (multiple builders) with quality gates (council review).

### Connections

- **To [Orchestrator Pattern](3-orchestrator-pattern.md)**: Flat team constraint is architectural decision, not SDK limitation. Orchestrators coordinate flat teams; teammates execute.
- **To [Agent Teams](../../9-practitioner-toolkit/1-claude-code.md#agent-teams-native-multi-agent-coordination-experimental)**: Implementation and Council patterns map to TeammateTool coordination primitives
- **To [Workflow Coordination](../../7-practices/5-workflow-coordination.md)**: Flat team archetypes provide coordination patterns for structured metadata systems

**Sources:** Advanced external .claude/ implementation patterns, TeammateTool documentation analysis.

---

## Connections

- **To [Orchestrator Pattern](3-orchestrator-pattern.md)**: Expert Swarm extends generic orchestration by adding domain expertise inheritance. Single-message parallelism and context isolation principles still apply. The key difference: expert lead provides domain context that generic orchestrators lack.

- **To [Self-Improving Experts](2-self-improving-experts.md)**: Expertise.yaml becomes the shared knowledge source for swarm workers. The improve agent runs post-swarm to analyze collective execution and update expertise. This creates a feedback loop: swarms execute → improve analyzes → expertise grows → next swarm benefits.

- **To [Agent Teams](../../9-practitioner-toolkit/1-claude-code.md#agent-teams-native-multi-agent-coordination-experimental)**: Advanced messaging patterns (Write, Broadcast, Read) enable coordination beyond spec-as-artifact. Currently experimental but provides richer communication when available. Expert Swarm can layer agent teams (TeammateTool) messaging on top of expertise inheritance for peer-to-peer coordination between council members.

- **To [Multi-Agent Context](../../4-context/4-multi-agent-context.md)**: Path-passing protocol implements expertise sharing without context pollution. Workers read expertise.yaml themselves rather than receiving copied context from orchestrator. Maintains context isolation while enabling consistency.

- **To [Workflow Coordination](../../7-practices/5-workflow-coordination.md)**: Expert Swarm is an architectural pattern that enables workflow coordination practices at scale. The orchestration mechanics documented here support the coordination strategies in practices.

- **To [Model-Native Swarm](../../3-model/4-multi-model-architectures.md#model-native-swarm-orchestration)**: Alternative approach to swarm coordination. Model-native embeds orchestration within model reasoning (Kimi K2.5); Expert Swarm uses SDK-level coordination. Trade-off: SDK provides explicit control and debugging; model-native offers autonomous parallelization at 100+ agent scale.

---

## Open Questions

- How do expertise.yaml updates propagate when multiple swarms execute concurrently? (Potential race condition if two improve agents run simultaneously)
- What's the optimal expertise file size for path-passing? (Current target: 750 lines. Evidence needed for different scales)
- Can workers selectively read sections of expertise.yaml, or must they load entire file? (Scope extraction reduces tokens but adds complexity)
- How does expertise inheritance compose with agent teams Council pattern? (Multiple domain experts coordinating—each with their own expertise.yaml)
- What debugging patterns emerge for tracing expertise influence on worker decisions? (Observability into which expertise sections workers consulted)
- When nesting becomes reliable, how deep should expertise-inheritance chains go? (Worker → sub-worker → sub-sub-worker: at what depth does overhead exceed benefit?)
- Should workers report back which expertise sections they consulted? (Would enable expertise usage analysis and pruning of unused patterns)
- How do you version expertise.yaml for swarm executions spanning days/weeks? (Git-based versioning? Explicit version references in prompts?)
