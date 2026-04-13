---
title: Harness Categories
description: The taxonomy of harness types from workflow harnesses to full agentic runtimes
created: 2026-04-12
last_updated: 2026-04-12
tags: [foundations, harness, categories, workflow, workspace-managers, frameworks]
part: 1
part_title: Foundations
chapter: 6
section: 3
order: 1.6.3
---

# Harness Categories

Not all harnesses are the same.

The term "harness" spans a spectrum from lightweight workflow structuring to full agentic execution environments with filesystem access, command execution, and persistent multi-session memory. Understanding which category fits a given problem is a prerequisite to framework and tool selection — and selecting the wrong category creates a capability ceiling that no amount of prompt engineering can overcome.

---

## The Three-Axis Stack

Ethan Mollick identifies three independent axes in the AI selection decision (~2026-Q1):

| Axis | What It Selects | Examples |
|------|----------------|---------|
| Model | The underlying reasoning capability | Claude Opus, GPT-5, Gemini Pro |
| App | The human-facing interface | claude.ai chat, IDE integration, CLI |
| Harness | The autonomous execution environment | Claude Code, LangGraph, raw API |

These axes are genuinely independent. The same model deployed in different harness categories produces categorically different outcomes — not marginally different outputs, but fundamentally different capabilities.

**The capability ceiling implication:** Choosing a chat interface as the harness for an agentic system is not a model choice — it is a capability ceiling. A frontier model in a web chat interface cannot maintain state across sessions, cannot execute commands, cannot manage a multi-agent workflow, and cannot apply harness engineering improvements. The harness selection determines the upper bound of what the agent can do; the model selection operates within that bound.

*[2026-04-12]*: This observation from Mollick's analysis has significant practical implications. Many practitioners who report that AI tools "cannot handle complex tasks" are operating frontier models inside chat interfaces — harnesses that were not designed for autonomous, multi-step execution. The limitation is architectural, not model-level.

---

## Harness Capability Tiers

*[2026-04-12]*: The three-tier taxonomy below reflects the production landscape as of April 2026. Boundaries between tiers are shifting as vendors add agentic capabilities to chat interfaces.

| Tier | Examples | Characteristics |
|------|----------|----------------|
| Full agentic harnesses | Claude Code, OpenAI Codex | Autonomous file access, command execution, multi-step loops, tool-native design, hooks for enforcement, persistent memory across sessions |
| Web-based chat interfaces | claude.ai, chatgpt.com | Optimized for conversational interaction; limited autonomous execution; session-scoped context; no persistent filesystem access |
| Constrained agentic environments | Gemini website, Gemini in Google Workspace | Capable model with limited harness; agency bounded by platform design; some tool access but not full autonomy |

**Practical implication for practitioners:** Framework selection is moot if the deployment target is a chat interface. A practitioner who selects LangGraph as their orchestration framework but deploys agents through a chat interface has applied the wrong tool to the wrong problem. The harness category must be selected before framework selection begins.

**The capability gap within tiers:** Not all full agentic harnesses are equivalent. Claude Code ships with Anthropic's defaults already configured — system prompt conventions, permission models, memory mechanisms, and hooks are pre-built. A raw API call with a custom Python script is also a "full agentic harness" in principle but requires building all six stack components from scratch. The tier defines what is possible; the specific implementation determines what is practical.

---

## Framework vs. Runtime vs. Harness

Three terms that practitioners frequently conflate but that refer to distinct layers of the stack:

| Term | Definition | Examples | When It Operates |
|------|-----------|---------|-----------------|
| Framework | Build-time libraries for constructing agent workflows | LangChain, CrewAI, AutoGen | Build-time (development) |
| Runtime | Production execution with state persistence and observability | LangGraph, LangSmith, Weave | Build-time + runtime |
| Harness | Opinionated runtime with defaults, batteries-included | Claude Code, OpenAI Codex | Runtime (production) |

**The key distinction:** A framework is what a developer uses to build an agent. A harness is what the agent runs inside during execution. These are different concerns at different times.

LangGraph is a runtime framework — it provides the execution infrastructure but requires developers to make all architectural decisions: which tools, which memory model, which prompt shape, which permission enforcement. It is a blank canvas with powerful primitives.

Claude Code is a harness — it ships with Anthropic's architectural decisions already made. The system prompt conventions, the memory mechanisms, the permission model, the hooks system — these are included with defaults that work for most use cases. Developers configure and extend; they do not build from scratch.

**Choosing between framework and harness:**

| Situation | Recommendation |
|-----------|---------------|
| Novel multi-agent coordination pattern | Framework (LangGraph, AutoGen) — need full control |
| Standard coding/development work | Harness (Claude Code) — defaults work; start immediately |
| Web-based AI product | Framework + custom runtime — harness defaults won't fit product requirements |
| Research and experimentation | Framework — flexibility needed for exploration |
| Production agentic workflow at standard tasks | Harness — faster deployment, maintained defaults |
| Enterprise with specific compliance requirements | Framework + custom runtime — compliance-specific controls |

---

## Harness Category Taxonomy

### Category 1: Full Agentic Harnesses

Full agentic harnesses are purpose-built execution environments that provide all six stack components out of the box. They are the reference implementation of the harness concept.

**Defining characteristics:**
- Autonomous filesystem access (read, write, navigate directories)
- Command execution (shell, compilers, test runners)
- Multi-step execution loops with stopping conditions
- Persistent memory across sessions
- Permission enforcement via hooks or equivalent mechanisms
- Tool-native design (tools are first-class, not bolted on)

**Canonical examples:** Claude Code (Anthropic), OpenAI Codex

**When this category applies:** Software development and engineering tasks, code review, codebase navigation and modification, automated testing, and any task requiring repeated read-write-execute cycles on a filesystem. The full agentic harness is the natural environment for the Orchestrator Pattern, Autonomous Loops, and ReAct execution patterns.

**The Claude Code reference implementation:** Claude Code ships with CLAUDE.md convention support (workspace context), auto-memory (session memory), hooks (computational guides), skills (progressive context disclosure), and sub-agents (bounded delegation). All six stack components are implemented with Anthropic's defaults — practitioners configure rather than build.

### Category 2: Web-Based Chat Interfaces

Web-based chat interfaces are conversational environments that prioritize human-AI dialogue over autonomous execution. They are not harnesses in the production engineering sense — they are apps that use harnesses internally.

**Defining characteristics:**
- Optimized for back-and-forth conversation
- Session-scoped context (typically no cross-session persistence unless explicitly provided)
- Limited tool access (file upload, web search, code execution via sandbox — not filesystem access)
- No hooks or enforcement mechanisms accessible to the developer
- No programmatic control over context management

**When this category applies:** Exploratory work, single-session research, drafting and editing, tasks where the human remains the execution environment (the model advises; the human acts).

**The capability ceiling is real:** A practitioner attempting to run a multi-phase software project through a chat interface will hit the ceiling quickly: no persistent memory, no autonomous execution, no enforcement mechanisms, no trajectory capture. The limitation is not model capability — it is harness design.

### Category 3: Workflow Harnesses

A workflow harness is a structured layer above raw agent interactions that imposes phase discipline, context engineering, and state persistence on multi-step human-AI work. It is the middle tier between chat interfaces and full agentic runtimes.

*[2026-04-12]*: Workflow harnesses are a recently named category (2026-Q1) that fills the gap between single-session chat and full autonomous execution. The GSD (get-shit-done) tool is the canonical open-source example.

**Defining characteristics:**
- Meta-prompting system that structures human-AI workflows
- Phase decomposition (break large work into defined phases with verification gates)
- Context engineering (fresh context per executor, no context rot across phases)
- State persistence (state files as coordination layer, not model memory)
- Wave-based parallelism (simultaneous execution of independent tasks within a phase)

**The GSD pattern:** The GSD workflow harness implements a five-phase lifecycle for complex software projects: specification → design → implementation → verification → integration. Each phase uses a fresh executor with scoped context, preventing context rot that accumulates when a single agent carries full conversation history across all phases. State is persisted in explicit files (not model memory) so each phase has access to previous phase outputs without inheriting previous phase context.

**Tooling hierarchy:**
```
Scale and abstraction:
  Raw coding agent       → Single-session, ad-hoc interaction
  Workflow harness (GSD) → Multi-phase structured lifecycle, context-engineered
  Workspace manager      → 20+ agent infrastructure (worktrees, merge queues, supervision)
```

The workflow harness occupies a practical niche: it handles projects too complex for a single agent session but not large enough to require full workspace manager infrastructure.

### Category 4: Workspace Managers

Workspace managers are the high end of the harness spectrum — infrastructure for coordinating 20 or more agents simultaneously with shared filesystem coordination, merge queues, and active supervision.

**Defining characteristics:**
- Multi-worktree management (each agent operates in an isolated git worktree)
- Merge queue coordination (sequential integration of parallel agent outputs)
- Supervision layer (orchestrating agent monitors and corrects subagent behavior)
- Shared state coordination (distributed state accessible to all agents without conflicts)
- Scale designed for sustained parallel throughput, not occasional multi-agent use

*[2026-04-12]*: Workspace manager tooling is an emerging infrastructure category (2026-Q1). Specific tools in this category include multi-agent workspace coordinators that manage git worktrees, parallel execution queues, and supervisory oversight. The category is evolving rapidly.

**When this category applies:** Large-scale parallel software development, enterprise codebase refactoring, sustained autonomous engineering workflows where human supervision of individual agents is not feasible.

### Category 5: Persistent Personal Runtimes

An emerging category (2026) optimized for single-agent long-running personal use with persistent memory and self-improving skills — not multi-agent coordination.

**Defining characteristics:**
- Cross-session memory (SQLite-backed or equivalent persistent storage)
- Self-improvement loop (tasks generate procedural skills stored for future reuse)
- Multi-platform integration (connects to calendar, email, documents, external services)
- Single-agent design (continuity and recall, not orchestration topology)

**Canonical example:** Hermes Agent (Nous Research, ~47K GitHub stars, Feb 2026) — multi-platform gateway, SQLite-backed memory, self-improvement loop where tasks automatically generate procedural skills. The harness accumulates knowledge about the user's preferences, workflows, and domain-specific conventions across all sessions.

**The distinction from full agentic harnesses:** Full agentic harnesses are optimized for task execution within development environments. Persistent personal runtimes are optimized for continuity of relationship between a single agent and a single user across all domains of that user's work.

### Category 6: Custom Framework + Runtime

The full-flexibility option: practitioners build agent infrastructure using framework primitives (LangChain, CrewAI, AutoGen) and deploy on runtime infrastructure (LangGraph, LangSmith) without using an opinionated harness.

**When this applies:**
- Novel coordination patterns not supported by existing harnesses
- Enterprise compliance requirements that prohibit opinionated defaults
- Research contexts requiring full control over every component
- Products that embed AI agents within larger systems with specific integration requirements

**The cost:** All six stack components must be built explicitly. There is no equivalent of Claude Code's auto-memory, hooks system, or skill dispatch — the team must implement each component or accept the risk of operating without it.

---

## Decision Table

| Problem Type | Harness Category |
|-------------|-----------------|
| Single-session, exploratory work | Chat interface or raw coding agent |
| Multi-session projects with defined phases | Workflow harness (GSD pattern) |
| Parallel execution across 20+ agents | Workspace manager |
| Long-running personal assistant with persistent recall | Persistent personal runtime |
| Custom multi-agent system from scratch | Framework + runtime |
| Standard software development with Anthropic models | Full agentic harness (Claude Code) |
| Research with novel coordination patterns | Framework + runtime |
| Compliance-sensitive enterprise deployment | Framework + runtime with custom controls |

---

## When Category Selection Matters Most

Category selection matters most at the beginning of a project, when the deployment environment is still open. Once a chat interface is chosen and workflows are established around it, the switching cost to a full agentic harness is significant — not in technical terms but in organizational terms. Teams develop habits and expectations around the interface they use.

The correct order of selection decisions:
1. **Define the problem type** (single-session vs. multi-session, exploratory vs. systematic)
2. **Select the harness category** based on the decision table
3. **Select specific tools** within the chosen category (evaluated in [Practitioner Toolkit](../10-practitioner-toolkit/_index.md))
4. **Select the model** based on capability requirements and harness compatibility

Reversing this order — selecting the model first, then discovering that the preferred interface doesn't support the required harness capabilities — is a common source of architectural rework.

---

## Connections

- **[What Is a Harness?](1-what-is-a-harness.md)** — Foundational definition and the Agent = Model + Harness formula
- **[The Harness Stack](2-harness-stack.md)** — Six-component decomposition used to evaluate harnesses within categories
- **[Designing for Your Context](7-designing-for-your-context.md)** — Decision frameworks for choosing between categories
- **[Practitioner Toolkit](../10-practitioner-toolkit/_index.md)** — Specific tool evaluations within each category
- **[Workflow Coordination](../8-practices/5-workflow-coordination.md)** — GSD case study and workflow harness patterns in depth
- **[Agent Frameworks](../10-practitioner-toolkit/4-agent-frameworks.md)** — Framework-specific capability tiers and selection guidance
