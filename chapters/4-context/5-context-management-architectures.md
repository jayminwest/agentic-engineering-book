---
title: Context Management Architectures
description: Architectural approaches to active context management — from passive accumulation to deterministic engines
created: 2026-03-09
last_updated: 2026-03-09
tags: [foundations, context, architecture, compaction, LCM, sapling]
part: 1
part_title: Foundations
chapter: 4
section: 5
order: 1.4.5
---

# Context Management Architectures

Most agent systems treat context as a passive log: messages accumulate, the window fills, and compaction happens as emergency surgery at 90-95% capacity. A new generation of systems treats context management as a **first-class architectural concern**—actively curating what the model sees between every turn, before degradation sets in.

This section surveys three points on the design spectrum:

- **Passive accumulation** (the default) — Messages pile up. Compact when forced.
- **Lossless preservation** (LCM) — Compress aggressively, but retain pointers to every original message. Nothing is ever truly lost.
- **Continuous curation** (Sapling) — Prune ruthlessly. A well-constructed 3-line summary of 15 stale turns outperforms 15 turns of noise.

The choice among these approaches mirrors an older engineering debate. Manual memory management (C) gives full control but requires discipline to avoid leaks. Garbage collection (Java/Go) automates reclamation at the cost of control. Ownership systems (Rust) enforce structure deterministically. Each trade-off is real; the right answer depends on task characteristics. LCM applies structured-programming discipline to context compaction. Sapling applies garbage-collection thinking: reclaim continuously, target a steady-state utilization, never let the heap fill up.

---

## The Baseline: Passive Accumulation

The passive accumulation model is what most deployed agent systems use today—Claude Code, Cursor, Windsurf, and similar tools all follow this pattern by default.

**How it works:**
1. Messages append to a flat list as the conversation progresses
2. Context window fills linearly with each turn
3. At some threshold (typically 90-95%), the system triggers emergency compaction: an LLM summarizes the entire conversation in one pass
4. Quality typically recovers briefly before the cycle repeats
5. When output quality degrades too far, the recovery strategy is to boot a fresh agent

The failure modes are well-characterized. Context rot—quality degradation correlated with context fill—is invisible until output worsens. Emergency summarization at 95% is uncontrolled and lossy, with no mechanism to prioritize what survives. The cliff-edge characteristic means the system is "fine" until it suddenly isn't. There is no signal ahead of the cliff.

The "boot fresh" default described in [Context Management Strategies](2-context-strategies.md) is a rational response to these failure modes. For short tasks, it remains the right answer. The passive model only becomes a liability for long-running sessions where the cost of repeated restarts is high.

**Key problems with passive accumulation:**
- Cliff-edge failure (no gradual degradation signal before quality drops sharply)
- Emergency summarization is uncontrolled—important details can disappear without trace
- No awareness of what is important vs. stale—all tokens treated equally
- Context rot is invisible until output quality degrades
- Recovery requires full agent restart, losing accumulated session state

---

## Lossless Context Management (LCM)

*[2026-03-09]*: Ehrlich and Blackman's Lossless Context Management (LCM) paper (February 2026, arXiv:submit/7269166) challenges the assumption that context compression inevitably loses signal. LCM introduces a deterministic, engine-managed architecture that makes compression viable by retaining pointers back to every original message.

### Architecture

LCM separates context into two distinct structures:

**Immutable Store** — Every message is preserved verbatim and never modified. The store is append-only. No compaction ever deletes from it. This is the source of truth.

**Active Context** — What the model actually sees. This is a curated view: recent messages appear in full; older messages are replaced by summary nodes. The active context is a derived view over the immutable store, not an independent record.

Summary nodes form a **hierarchical DAG** (directed acyclic graph). When a block of messages is compacted, a summary node replaces it in the active context. If the summary node itself grows too large, it can be summarized into a higher-level node. The DAG structure means each summary traces back to source messages through stable identifiers.

Two tools expose the lossless guarantee to the model:
- `lcm_grep` — Full-text search across all messages in the immutable store, including compacted content
- `lcm_expand` — Recover any compacted content by expanding a summary node back to its original messages

This means no message is ever permanently inaccessible. The model can retrieve any prior state if the task requires it.

Files are stored by reference (path + exploration summary), not by content. This prevents context from bloating with large file contents when only the exploration summary is needed.

### Compaction Mechanism

LCM uses a dual-threshold trigger with three-level escalation:

**Soft threshold (τ_soft)** — Triggers async compaction in the background. No user-facing latency. The model continues working while older blocks are summarized.

**Hard threshold (τ_hard)** — Triggers blocking compaction. The oldest block is summarized before the next model call proceeds.

Three-level escalation applies within each compaction event:

1. **Level 1: preserve_details** — Full narrative summary retaining key facts
2. **Level 2: bullet_points** — Compressed bullet summary of essential information
3. **Level 3: deterministic truncation** — Token-count reduction with no LLM involvement; guaranteed to converge

Level 3 guarantees convergence: regardless of content difficulty, the system always reduces token count. This addresses a real failure mode in LLM-based summarization, where a model asked to summarize dense content may produce an equally long result.

### Engine-Managed Iteration

LCM introduces two higher-order tools that shift iteration control from the model to the engine:

**`llm_map`** — A stateless per-item LLM call. The model specifies what to do for each item; the engine handles parallelism, retries, and context isolation. Token cost scales with item count, not with accumulated session history.

**`agentic_map`** — A full sub-agent per item, with tool access. The sub-agent gets a fresh context, bounded by a scope-reduction invariant: the sub-task scope must be strictly smaller than the parent task scope, preventing infinite delegation.

The analogy LCM draws explicitly is to Dijkstra's critique of GOTO. Model-written loops are stochastic—the model decides when to iterate, when to stop, whether to handle errors. Engine-managed iteration is deterministic: the engine controls execution; the model provides logic. This mirrors the shift from unstructured branching to structured for/while loops.

### Benchmark Results

On the OOLONG long-context benchmark (8K–1M tokens), LCM's Volt agent produced an average score of 74.8 vs Claude Code's average of 70.3. The gap widened at larger context lengths: at 512K tokens, Volt scored 42.4 vs Claude Code's 29.8. Below 32K tokens—where compaction is not triggered—the two systems performed comparably.

The benchmark results demonstrate a specific claim: LCM's compaction mechanism preserves enough information that long-session performance does not degrade relative to shorter sessions. This is the direct failure mode of passive accumulation that LCM addresses.

### Trade-offs

| Dimension | LCM Characteristic |
|-----------|-------------------|
| Information loss | None — every message recoverable via `lcm_expand` |
| Overhead (short tasks) | Zero — below soft threshold, no compaction occurs |
| Infrastructure | PostgreSQL dependency for immutable store |
| Summary nodes | Add indirection — model must reason about what's summarized |
| Model autonomy | Reduced — deterministic primitives replace model-written memory scripts |
| Best for | Long sessions, data-intensive aggregation, tasks with recall needs |

The infrastructure requirement is a real constraint. PostgreSQL means LCM is not a drop-in replacement for stateless agent deployments. The architecture is designed for persistent agent systems where session continuity has high value.

---

## Continuous Curation: Sapling

*[2026-03-09]*: Sapling (`@os-eco/sapling-cli`, CLI: `sp`) takes a different starting position. Where LCM asks "how do we preserve everything while managing what's visible?", Sapling asks "how do we keep only what currently matters?"

The target metric is 50-60% context utilization at steady state. Not "don't fill up"—*actively stay at half capacity*. This is the garbage-collector mindset: don't wait for the heap to fill; reclaim continuously so it never gets close.

Sapling is headless—it has no UI of its own and runs as a pipeline layer in front of the model. Inter-turn context management means the pipeline runs between every turn, not just when limits are hit.

### The Operation Model

Sapling's core abstraction is the **operation**: a group of semantically related turns. A typical coding operation might be: read file → analyze → edit file → run tests → observe results. These five turns form one operation because they share intent, files, and causal dependency.

Boundary detection uses a weighted heuristic applied at each turn boundary:

| Signal | Weight |
|--------|--------|
| Tool-type transition (e.g., read → bash) | 0.35 |
| File-scope change (different files accessed) | 0.30 |
| Intent signal (from message content) | 0.20 |
| Temporal gap between turns | 0.15 |

When the weighted sum exceeds threshold, Sapling opens a new operation. The active operation is always retained in full. Completed operations are scored and become candidates for compaction.

Each operation tracks: files touched, tools used, artifacts created, dependencies on other operations, and pending commitments.

### Five-Stage Pipeline

| Stage | Purpose | Key Mechanism |
|-------|---------|---------------|
| **Ingest** | Parse messages into semantic units | Boundary detection via weighted heuristics |
| **Evaluate** | Score each operation's relevance | Weighted signals across five dimensions |
| **Compact** | Summarize low-scoring operations | Template-based summaries; tool output truncation by type |
| **Budget** | Enforce token allocation by zone | Dynamic rebalancing across three zones |
| **Render** | Assemble final message array | Working memory in system prompt, retained operations as messages |

**Evaluation signals** (applied to completed operations):

| Signal | Weight | What it captures |
|--------|--------|-----------------|
| Recency | 0.25 | How recently the operation occurred |
| File overlap | 0.25 | Whether the operation's files are still in active use |
| Causal dependency | 0.25 | Whether later operations depend on this one |
| Outcome significance | 0.15 | Whether the operation produced important artifacts |
| Operation type | 0.10 | Inherent importance of the operation category |

Operations with low composite scores are compacted. High-scoring operations are retained in full. The active operation always scores maximum and is never compacted mid-operation.

**Budget zones** allocate context by function:

| Zone | Allocation | Contents |
|------|-----------|---------|
| System + archive | 25% | System prompt, long-term summaries, cross-session state |
| Active operations | 25% | Full-fidelity retained operations |
| Headroom | 50% | Target utilization ceiling; absorbs new turns |

Dynamic rebalancing adjusts zone sizes when pressure exceeds allocation. The headroom zone exists to prevent the cliff-edge characteristic of passive accumulation: 50% headroom means no operation is ever added in a context that is more than half full.

**Compaction** (MVP implementation) uses template-based summaries: structured text templates instantiated with operation metadata. This is intentionally fast and produces no additional LLM calls. The design document notes a planned upgrade path to LLM-based compaction (Haiku) post-MVP for higher-quality summaries.

**Render** assembles the final message array. Working memory—the high-level summary of what has been accomplished—goes in the system prompt. Retained operations appear as messages. Orphaned references (references to compacted operations without a summary) are sanitized to avoid confusing the model.

### Commitment Tracking

One distinguishing feature: Sapling extracts **commitments** from assistant messages. When the agent writes "I will edit foo.ts" or "Next I'll run the tests," Sapling records this as a pending commitment.

At operation completion, Sapling checks which commitments from that operation were fulfilled and which remain open. Unfulfilled commitments surface in the next system prompt. This addresses a specific failure mode: an agent that committed to a follow-up action loses track of that commitment when the relevant context is compacted.

### Composability

Sapling's stage registry allows swapping any stage in the pipeline. Custom evaluation signals can be added (e.g., a signal that weights operations involving security-critical files more heavily). Custom compaction strategies can replace the template approach. Tools can declare phase metadata and file paths, allowing the pipeline to reason about tool-specific compaction priorities.

The RPC interface over a Unix socket exposes pipeline state for external consumers. The Overstory orchestrator uses this interface to monitor agent context health, enabling orchestrator-level decisions about when to archive and restart agents.

### Trade-offs

| Dimension | Sapling Characteristic |
|-----------|----------------------|
| Information loss | Intentional — compacted operations lose detail permanently |
| Retrieval of old context | Not available — summary only, no `lcm_expand` equivalent |
| Overhead (short tasks) | Minimal — pipeline runs but mostly no-ops |
| External dependencies | None — all state in-memory operation registry |
| Extra LLM calls | None (MVP: template-based; post-MVP: Haiku) |
| Process restart | State does not survive restart (design doc addresses archive persistence) |
| Best for | Medium-length coding sessions, workflows with clear operation boundaries |

The lossy-by-design decision is the key philosophical difference from LCM. Sapling's premise is that a well-constructed 3-line summary of 15 stale turns is *more useful* than the 15 turns themselves—because the summary surfaces only what remains relevant, while the 15 turns require the model to re-read and re-evaluate. The information loss is controlled (template-based summaries follow a predictable structure) rather than uncontrolled (emergency compaction produces unpredictable results).

---

## The Design Spectrum

The three approaches form a coherent design spectrum with clear trade-off dimensions:

| Dimension | Passive Accumulation | LCM (Lossless) | Sapling (Curated) |
|-----------|---------------------|-----------------|-------------------|
| **Philosophy** | Let it fill | Preserve everything, compress views | Keep only what matters |
| **Compaction trigger** | 90-95% (emergency) | Soft/hard thresholds (proactive) | Every turn (continuous) |
| **Information loss** | Uncontrolled at compaction | None (lossless pointers) | Controlled, intentional |
| **Retrieval of old context** | Gone after compaction | Always recoverable via `lcm_expand` | Gone — summary only |
| **Overhead (short tasks)** | None | None (zero-cost continuity) | Minimal (pipeline runs but no-ops) |
| **Target utilization** | Fill to capacity | Managed via thresholds | 50-60% steady state |
| **External dependencies** | None | PostgreSQL | None |
| **Extra LLM calls** | 1 at compaction | Async summarization calls | None (template-based MVP) |
| **Iteration model** | Model writes loops | Engine-managed (`llm_map`) | N/A (single-agent focus) |
| **Best for** | Short tasks | Long sessions, data-heavy tasks | Medium sessions, coding tasks |

The Dijkstra analogy from the LCM paper maps cleanly onto the spectrum:

- **Passive accumulation** = GOTO (unstructured, flexible, prone to failure)
- **LCM** = structured programming (for/while — constrained, deterministic, reliable)
- **Sapling** = garbage collection (automatic, continuous, reclaims without explicit management)

### When to Use Each Approach

**Passive accumulation with "boot fresh":**
- Short tasks (under 30 minutes) where context budget is unlikely to fill
- Tasks with natural restart points where session continuity adds minimal value
- Teams prioritizing simplicity over long-session performance

**LCM (lossless preservation):**
- Long sessions where recall of earlier conversation state may be required
- Data-intensive tasks that process many items — `llm_map` handles unbounded datasets without accumulating history
- Workflows where any information loss is unacceptable
- Multi-day work where cross-session continuity matters

**Sapling (continuous curation):**
- Medium-length coding sessions (30 minutes to several hours)
- Tasks with clear operation boundaries where completed operations have declining relevance
- Deployments where infrastructure simplicity is a priority (no external store)
- Agents running in orchestrated swarms where the orchestrator monitors context health via RPC

### Composition

The approaches are not mutually exclusive. A system could apply Sapling-style continuous curation for the active session while persisting compacted operation summaries to an LCM-style immutable store for cross-session recovery. The time horizons differ: Sapling optimizes the current turn; LCM optimizes the full session history. A system that composes both addresses both horizons.

---

## Implications for Practitioners

**Context management is becoming a first-class discipline.** The "boot fresh" default works for short tasks and remains sound. For long-running sessions, active context management is the next frontier—both LCM and Sapling demonstrate that the passive model leaves performance on the table.

**The passive baseline has known failure modes with engineering solutions.** Cliff-edge compaction, invisible context rot, and uncontrolled information loss are not inherent properties of context windows. They are consequences of passive accumulation. Both LCM and Sapling address them through different architectural choices.

**Choose the trade-off consciously:**
- If recall of any prior state may be required, LCM's lossless architecture is the only option
- If simplicity and sharp focus matter more than recall, Sapling's continuous curation delivers steady-state quality without infrastructure overhead
- If tasks are short enough that context fill is not a concern, passive accumulation with "boot fresh" remains the pragmatic default

**Both architectures converge on a key principle:** the engine should manage context, not the model. LCM moves iteration control to the engine via `llm_map`. Sapling moves compaction decisions to the engine via the five-stage pipeline. The model focuses on reasoning; the engine handles memory management. This division of responsibility is likely to become standard as context management matures.

### Open Questions

- What is the optimal compaction granularity? Per-message, per-operation, per-phase produce different trade-offs.
- Can lossless and lossy approaches compose effectively across session boundaries?
- How does active context management interact with model capability improvements? Better models may tolerate higher context fill without quality degradation, changing the economics.
- What benchmarks beyond OOLONG capture context management quality for realistic agentic tasks?
- How should orchestrators make cross-agent context management decisions when some agents use LCM, others use Sapling, and others use passive accumulation?

---

## Connections

- **To [Context Fundamentals](1-context-fundamentals.md):** LCM challenges the "boot fresh" default by showing that lossless compaction can maintain continuity across long sessions. Sapling operationalizes the capability capacity model by targeting 50% utilization as a steady state, not a warning threshold.
- **To [Context Strategies](2-context-strategies.md):** Both architectures formalize frequent intentional compaction into deterministic systems. Passive accumulation with emergency compaction is the failure mode that proactive compaction strategies (and these architectures) address.
- **To [Advanced Context Patterns](3-context-patterns.md):** Progressive disclosure appears in both architectures: LCM uses exploration summaries that expand on demand; Sapling places operation summaries in the system prompt as a progressive view over compacted history.
- **To [Multi-Agent Context](4-multi-agent-context.md):** LCM's `agentic_map` creates context-isolated sub-agents as a first-class engine primitive. Sapling's RPC interface enables orchestrator-level monitoring of agent context health across a swarm.
- **To [Tool Design](../5-tool-use/1-tool-design.md):** LCM's `llm_map` and `agentic_map` are tools that relocate iteration logic from the stochastic model layer to the deterministic engine layer — a design pattern applicable beyond context management.
- **To [Design as Bottleneck](../8-mental-models/6-design-as-bottleneck.md):** Both architectures embody design as bottleneck — investing in infrastructure to eliminate agent failure modes rather than accepting them as inherent limitations.

---

## Sources

- Ehrlich, C. & Blackman, T. (2026). "LCM: Lossless Context Management." Voltropy PBC. arXiv:submit/7269166.
- Sapling context pipeline v1 design document. `os-eco/sapling/docs/context-pipeline-v1.md`.
- Hong, K., Troynikov, A., & Huber, J. (2025). "Context Rot: How context degradation affects LLM performance."
- Zhang, A. L., Kraska, T., & Khattab, O. (2026). "Recursive Language Models." arXiv:2512.24601.
