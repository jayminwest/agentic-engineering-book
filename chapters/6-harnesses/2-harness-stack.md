---
title: The Harness Stack
description: Raschka's six-component taxonomy of the execution environment surrounding a model
created: 2026-04-12
last_updated: 2026-04-12
tags: [foundations, harness, components, raschka, taxonomy, architecture]
part: 1
part_title: Foundations
chapter: 6
section: 2
order: 1.6.2
---

# The Harness Stack

The harness is not monolithic. It decomposes into six named components, each with distinct responsibilities, failure modes, and optimization targets.

Sebastian Raschka's taxonomy (2026-04-04) provides the canonical decomposition of the coding agent harness. Although Raschka's analysis focuses on coding agents specifically, the six components generalize: every agentic harness that manages multi-turn, multi-tool work requires some implementation of each component, even if the implementation is minimal or implicit.

---

## Component Overview

| Component | Responsibility | Common Implementation |
|-----------|--------------|----------------------|
| Workspace context | Stable facts about the working environment | CLAUDE.md, project instruction files, git metadata |
| Prompt shape | Stable/dynamic split for cache reuse | System prompt architecture, context window management |
| Tool access | Bounded, defined tool inventory | Tool schemas, MCP registrations, permission filters |
| Context management | Output clipping, deduplication, compression | Progressive disclosure, context compaction, token budgets |
| Session memory | Dual-layer state (working memory + full transcript) | Persistent memory files, auto-memory, session context |
| Subagent delegation | Bounded subtask spawning with inherited context | Task tool, sub-agent configuration, context scoping |

Raschka's core diagnostic implication: "The harness can often be the distinguishing factor" separating model capabilities from product performance. When a coding agent underperforms, the first audit target is the harness — specifically, this table — not the model.

---

## Component 1: Workspace Context

Workspace context is the stable, durable information about the working environment that the harness injects once rather than repeatedly per turn.

**Purpose:** Establish the agent's situational awareness at session start. The agent needs to know where it is (directory structure, repository layout), what the project is (purpose, conventions, constraints), and what state the project is in (current branch, recent commits, open issues) before any task begins. Assembling this information correctly at the start is cheaper than regenerating it per turn and more reliable than depending on the agent to discover it incrementally.

**What belongs in workspace context:**
- Repository layout (directory tree, key file locations)
- Project instruction files (CLAUDE.md, AGENTS.md, README excerpts)
- Git status (current branch, recent commit messages, uncommitted changes)
- Project conventions (naming, formatting, testing requirements)
- Technology stack (languages, frameworks, dependencies)

**What does not belong:** Dynamic session state (what was discussed this turn), user messages, tool outputs. These belong in the dynamic portion of the context.

**Reference implementation — the repo map pattern:** Aider implements workspace context as a symbolic repo map — tree-sitter parsing of the codebase extracts function signatures, class definitions, and symbol relationships, which are then graph-ranked by reference frequency and dependency depth. The default budget is 1,000 tokens via `--map-tokens`. The agent receives a compact, high-signal representation of the codebase rather than raw file content — the most referenced symbols surface automatically. This is the state of the art for coding agent workspace context: concise, ranked, navigable.

**Stable prefix caching economics:** Workspace context is a candidate for prompt caching. By placing it in the stable prefix — the part of the context that does not change between turns — practitioners pay the token cost once per session rather than once per turn. For a 2,000-token workspace context and 20 turns per session, caching eliminates 38,000 tokens of repeated input cost. The harness's responsibility is to enforce the stable/dynamic split correctly so that workspace context stays in the cached prefix.

**Common failure mode:** Workspace context that includes volatile information (e.g., a timestamp, the results of a tool call from the previous turn) breaks cache reuse. Every token placed in the stable prefix that changes between turns invalidates the cache for that prefix. The harness must enforce a strict boundary between stable workspace context and dynamic session state.

---

## Component 2: Prompt Shape and Cache Reuse

Prompt shape refers to the architectural decision about which portions of the context are stable across turns and which are dynamic — and the harness mechanism that enforces this distinction.

**The stable/dynamic split:** The context window at any turn consists of:
- **Stable prefix**: instructions, tool descriptions, workspace context summary — identical across all turns in the session
- **Dynamic session state**: user messages, agent responses, tool outputs — changes with every turn

This split is an explicit architectural decision that determines the economics of the entire session. When the split is designed correctly:
- Stable prefix is cached once (one-time cost)
- Dynamic state is priced per turn (incremental cost)
- Total session cost falls dramatically compared to treating all context as dynamic

When the split is wrong — volatile content placed in the stable prefix, or stable content regenerated per turn — every token is expensive.

**The harness as the enforcer:** The stable/dynamic split is meaningless without enforcement. The harness is the entity that:
1. Constructs the prompt with the stable prefix first
2. Appends dynamic session state after the prefix
3. Sends the combined prompt to the model with the correct caching directives
4. Ensures nothing added to the stable prefix changes between turns

**Aider's implementation:** Aider explicitly implements this pattern with four caching layers: (1) system prompt, (2) read-only files, (3) repository map, (4) editable files. All four are stable across turns and are cached. Individual user messages and responses are dynamic and priced per turn. Keepalive pings maintain cache freshness across long sessions.

**What belongs in the stable prefix:**
- Complete system prompt (role definition, task framing, behavioral constraints)
- Tool schema definitions (these change only when the tool inventory changes)
- Workspace context summary (the repo map or equivalent)
- Project conventions (language, formatting, testing requirements)

**What belongs in the dynamic state:**
- All user messages
- All agent responses
- All tool call inputs and outputs
- Any context that changes based on what the agent has done

**Common failure mode:** Practitioners frequently include tool call results in the stable prefix (e.g., "here is the current state of the codebase" reconstructed fresh each turn). This defeats caching entirely. The harness must enforce that tool outputs go into the dynamic session state, not the stable prefix.

Connect to [Context Patterns](../4-context/3-context-patterns.md) for comprehensive treatment of stable-prefix caching and context injection strategies.

---

## Component 3: Tool Access

Tool access is the harness component that defines what the agent can do — the bounded inventory of named operations available to the agent at any point in execution.

**The harness defines capability; the model executes within it:** Tool access is a harness concern, not a model concern. The model reasons about what tool to call and what arguments to provide. The harness decides whether the model is allowed to call that tool at all, validates the inputs before execution, and handles the output. Without the harness's tool access layer, the model has no tools — it is just a text generator.

**Bounded inventories vs. dynamic discovery:** Two architectural approaches to tool inventory management:

| Approach | Mechanism | When to Use |
|----------|-----------|------------|
| Bounded inventory | Fixed list of named tools defined at session start | Standard production agents; most use cases |
| Dynamic discovery | Tools discovered at runtime via registry or protocol | MCP-based systems; plugin architectures; rapidly evolving capability sets |

Raschka advocates for bounded inventories in coding agents: "a pre-defined list of named tools with clear inputs and boundaries." The emphasis is on validation and controlled execution over tool quantity. More tools are not better — more tools mean more surface area for mistakes and permission violations.

**Tool schemas as interface definitions:** Each tool in the harness has a schema that specifies:
- Input parameters (names, types, constraints)
- Required vs. optional parameters
- Description of what the tool does and when to use it
- Any restrictions on parameter values (e.g., paths must be absolute)

The schema is both documentation (for the model) and validation specification (for the harness). The harness validates inputs against the schema before execution and returns structured errors rather than allowing invalid calls to reach the underlying system.

**Permission filtering at the harness level:** The harness applies permission checks before any tool executes. These checks are the harness's responsibility, not the tool's. A tool that writes to the filesystem should not contain permission logic — the harness's pre-execution hook should prevent the call from reaching the tool if the target path is outside the allowed scope.

This separation matters for security: a compromised or poorly-written tool cannot bypass permissions by implementing them incorrectly. The harness enforcement is independent of the tool implementation.

**Common failure mode:** Tools that accept relative paths rather than absolute paths. Raschka cites Anthropic's SWE-bench work: converting relative to absolute filepaths reduced model errors measurably. The tool schema should require absolute paths; the harness should validate that the path exists and falls within the allowed scope before executing.

Cross-reference: [Tool Design](../5-tool-use/1-tool-design.md) for design principles; [Tool Restrictions](../5-tool-use/3-tool-restrictions.md) for restriction patterns. This chapter owns the access-control framing; those chapters own the design and restriction details.

---

## Component 4: Context Management

Context management is the harness component responsible for keeping the context window useful as the session accumulates tool outputs, agent responses, and intermediate results.

**The core problem:** Every tool call adds output to the context. Every agent response adds text. Without active management, the context window fills with redundant information, stale data, and verbose tool outputs — degrading the agent's ability to reason about its current task.

Raschka's insight: "much of apparent 'model quality' is really context quality." When an agent underperforms, practitioners often blame the model. The more common cause is context that has degraded through accumulation — the harness is showing the model too much noise and too little signal.

**Three primary context management operations:**

**Output clipping:** Long tool outputs truncated to relevant portions. A grep command returning 3,000 lines of matches is not useful; the harness clips to the first 100 matches or the most relevant matches. The clipping policy is a harness decision — the model never sees the unclipped output.

**Deduplication:** The same file read twice in the same session produces two identical blocks in the context. The harness detects and eliminates the duplicate, keeping only the most recent version. In long agent sessions, deduplication can recover 20-30% of context window space.

**Transcript compression:** As the session grows, earlier turns become less relevant to the current reasoning step. The harness applies recency-biased compression: recent turns are preserved in full; earlier turns are summarized or compressed. The agent's context shows the full current state with an abstract of the history.

**Compression timing:** Context management operates proactively, not reactively. The harness compresses when context fill reaches 40-60% of capacity — not at 95% when quality has already degraded. The context chapter's "frequent intentional compaction" principle applies here: the goal is to maintain a high-signal context, not to salvage a degraded one.

**What the harness decides vs. what the model decides:**
- The harness decides what gets clipped, deduplicated, and compressed
- The model reasons within the context the harness has shaped
- The model cannot observe that content has been removed (unless the harness explicitly notes it)

This asymmetry has implications for agent design: the agent's behavior depends on the harness's context management decisions. A poorly designed context management layer — one that clips the wrong outputs or compresses the wrong history — will cause agent failures that look like model failures.

Connect to [Context Strategies](../4-context/2-context-strategies.md) and [Context Patterns](../4-context/3-context-patterns.md) for comprehensive treatment of compression, injection, and retrieval strategies.

---

## Component 5: Session Memory

Session memory is the harness component responsible for maintaining useful state across turns within a session and, in persistent harnesses, across sessions.

**The dual-layer architecture:** Raschka identifies two distinct memory layers:

| Layer | Characteristics | Purpose |
|-------|----------------|--------|
| Working memory | Small, distilled, explicitly maintained, modified per turn | Active reasoning state |
| Full transcript | Complete, append-only, durable, never discarded | Auditability and recovery |

The duality is the design — these two layers serve different purposes and must not be conflated.

**Working memory** is curated. The harness maintains a small, high-signal summary of what the agent has learned, decided, and done. This summary is actively updated: when the agent makes a significant decision, the harness records it in working memory; when information becomes stale, the harness prunes it. Working memory is what the agent reasons from — it needs to be dense, current, and relevant.

**Full transcript** is archival. Every turn, every tool call, every agent response is appended to the transcript in its complete form. Nothing is removed. The transcript enables two critical capabilities that working memory cannot provide:
1. **Recovery**: if the session fails mid-task, the full transcript provides a complete record from which execution can resume
2. **Auditability**: the full transcript supports post-hoc review of what the agent did and why

**Persistent memory across sessions:** Claude Code's auto-memory mechanism extends working memory across session boundaries — the harness persists learned facts, project conventions, and task history to `.claude/memory` files that are loaded at session start. This means the agent begins each session with accumulated knowledge from previous sessions, not a blank slate.

**The trajectory capture insight (Schmid, ~2026-Q1):** The full transcript is not just an audit log — it is training data. Schmid's framing: "The Harness is the Dataset." Every session run through the harness produces sequences of observations, actions, and outcomes. These trajectories are:
- Training data for future model fine-tuning on domain-specific tasks
- Evaluation data for measuring harness improvement over time
- Documentation of edge cases and failure modes specific to the practitioner's context

The competitive advantage implication: organizations that instrument their harness to capture trajectories accumulate a dataset that no competitor can purchase — because it reflects their specific work, their specific failure modes, and their specific improvement history.

**Common failure mode:** Conflating working memory and full transcript. Some harness implementations use a single running context log — treating everything as equally important and equally persistent. This creates a context that grows without bound (no distillation) and loses audit fidelity (compression destroys history). The harness should maintain both layers separately, with explicit mechanisms for updating working memory and appending to the transcript.

---

## Component 6: Subagent Delegation

Subagent delegation is the harness component responsible for spawning bounded subtasks and integrating their results — the mechanism that transforms a single agent into a coordinated multi-agent system.

**What bounded means:** The qualifier "bounded" is the key. Unbounded subagent delegation — where subagents can touch any file, spawn more subagents without limit, or inherit the full context of the parent — produces unpredictable, expensive, and often broken behavior. Bounded delegation specifies:

1. **Scope**: which files, directories, or resources the subagent may access
2. **Context**: a scoped subset of the parent's context, not the full context
3. **Output format**: structured output that the orchestrating harness can integrate
4. **Stopping conditions**: when the subagent should stop, regardless of task completion state

**Why the harness owns delegation:** Spawning a subagent is not a tool call — it is an act of resource allocation, permission scoping, and state coordination. The model can decide that delegation is appropriate, but the harness executes the delegation, enforces the scope, and manages the integration of results. A model that could spawn subagents arbitrarily would be able to circumvent permission controls by delegating to a subagent with broader permissions.

**Anti-patterns in subagent delegation:**

| Anti-pattern | Symptom | Harness Fix |
|-------------|---------|------------|
| Duplicate file access | Subagents writing to the same file simultaneously | Scope isolation enforced at delegation |
| Recursive spawning | Subagent spawns more subagents without limit | Maximum delegation depth enforced by harness |
| Context flooding | Subagent inherits full parent context | Context scoping at delegation point |
| Result misintegration | Subagent output format varies; orchestrator cannot parse | Required output format specified in delegation |

**The scoped context principle:** When the harness delegates to a subagent, it should pass the minimum context the subagent needs to complete its task — not the parent's full context. Full context inheritance creates three problems: (1) the subagent spends tokens processing irrelevant context, (2) the subagent may take actions influenced by context it shouldn't have (security concern), and (3) large subagent context slows execution and increases cost.

**Connection to orchestration patterns:** Subagent delegation is how harnesses implement the Orchestrator Pattern (see [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md)) at the infrastructure level. The pattern describes the coordination logic; this component describes the execution mechanism. The harness makes the pattern operational.

---

## The Diagnostic Audit Sequence

When a coding agent underperforms, Raschka's six-component taxonomy provides the audit sequence. The order reflects the components most commonly responsible for failures:

```
1. Workspace context       → Is the agent aware of the correct environment?
2. Prompt shape            → Is the stable/dynamic split correct? Is caching active?
3. Context management      → Has context quality degraded through accumulation?
4. Session memory          → Is working memory accurate and current?
5. Tool access             → Does the agent have the tools it needs? Are permissions correct?
6. Subagent delegation     → Are subagents properly scoped and integrated?
```

The sequence begins with workspace context because environmental misawareness produces cascading failures that look like model errors. A model that has been told it is working in a Python project when it is actually working in a Go project will produce coherent but wrong outputs — and the failure looks like a model hallucination.

The sequence ends with subagent delegation because delegation failures are typically visible and diagnosable from the output. Context management and memory failures are subtler — they degrade output quality without producing obvious error signals.

**The "harness > model" principle in practice:** Before concluding that a model is insufficient for a task, a practitioner should run this audit sequence. The diagnostic question at each component: "Is this component functioning correctly, or does it represent the actual source of failure?" Most reported "model failures" are harness failures in disguise.

---

## Connections

- **[What Is a Harness?](1-what-is-a-harness.md)** — The foundational definition and formula
- **[Harness as Control System](4-harness-as-control-system.md)** — How these components function as guides and sensors
- **[Context Strategies](../4-context/2-context-strategies.md)** — Deep treatment of context management
- **[Context Patterns](../4-context/3-context-patterns.md)** — Stable prefix pattern and progressive disclosure
- **[Tool Design](../5-tool-use/1-tool-design.md)** — Tool schema design principles
- **[Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md)** — Coordination patterns implemented via subagent delegation
- **[Claude Code](../10-practitioner-toolkit/1-claude-code.md)** — Reference implementation of all six components
