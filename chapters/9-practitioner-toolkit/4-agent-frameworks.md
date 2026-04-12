---
title: Agent Frameworks
description: Comparing LangGraph, CrewAI, AutoGen, and Claude Agent SDK for multi-agent orchestration
created: 2026-01-30
last_updated: 2026-04-11
tags: [tools, frameworks, langchain, langgraph, crewai, autogen, claude-agent-sdk, multi-agent, orchestration, comparison, hermes, persistent-runtime]
part: 3
part_title: Perspectives
chapter: 9
section: 4
order: 3.9.4
---

# Agent Frameworks

The agent framework landscape has consolidated around four major approaches: graph-based workflows (LangGraph), role-based teams (CrewAI), conversational coordination (AutoGen/Microsoft Agent Framework), and harness-powered delegation (Claude Agent SDK). Each embodies fundamentally different philosophies about how agents should coordinate.

---

## Harness Capability Tiers

Before selecting an orchestration framework, practitioners face a more fundamental choice: which type of execution environment to build within.

Mollick (2026) identifies three distinct components in the AI selection stack:

- **Model** — the underlying capability (what it knows and reasons)
- **App** — the chat interface (how a human interacts with the model)
- **Harness** — the autonomous execution environment enabling multi-step task completion without continuous human input

These are independent axes. The same model produces dramatically different outcomes in different harnesses. Agent Psychometrics research (arXiv:2604.00594) formalizes this: P(success) = σ(θ_LLM + θ_scaffold − β_difficulty), where LLM capability and scaffold quality are additively independent. Harness investment yields separable gains from model investment.

### Current Harness Landscape

| Tier | Examples | Characteristics |
|------|----------|-----------------|
| **Full agentic harnesses** | Claude Code, OpenAI Codex | Autonomous file access, command execution, multi-step loops, tool-native design |
| **Web-based chat interfaces** | claude.ai, chatgpt.com | Optimized for conversation; limited autonomous execution; session-scoped context |
| **Constrained agentic environments** | Gemini website, Gemini in Workspace | Capable model with limited harness; agency bounded by platform |

**Practical implication:** Choosing a chat interface as the deployment environment for an agentic system is not a model choice — it is a capability ceiling. The frameworks discussed below (LangGraph, CrewAI, etc.) apply to the full agentic harness tier. If the deployment target is a chat interface, framework selection is moot.

**See Also:**
- [Model Selection: Cross-Provider Selection](../3-model/1-model-selection.md#cross-provider-selection) — Task-type to provider+harness mapping
- [Claude Code](1-claude-code.md) — Reference implementation of the full agentic harness tier

---

## Framework Comparison Matrix

| Framework | Philosophy | Best For | Learning Curve | Production Maturity |
|-----------|------------|----------|----------------|---------------------|
| **LangGraph** | Workflows as stateful graphs | Complex branching, conditional logic | Moderate | High (v1.0, enterprise adoption) |
| **CrewAI** | Agents as role-playing teams | Rapid prototyping, clear role separation | Low | Growing (60% Fortune 500) |
| **Microsoft Agent Framework** | Conversations between agents | Research, dynamic collaboration | Moderate-High | Transitioning (from AutoGen) |
| **Claude Agent SDK** | Tool-equipped autonomous harness | Deep tool integration, code execution | Low-Moderate | Maturing |

---

## LangGraph

*[2026-01-30]*: LangGraph reached v1.0 as the first stable major release in the durable agent framework space. Powering production applications at Uber, LinkedIn, Klarna, and JP Morgan.

### Core Architecture

LangGraph treats agent workflows as directed graphs with nodes (processing steps) and edges (transitions). State persists automatically across execution, enabling long-running workflows without custom database logic.

**Key v1.0 Capabilities:**
- **Durable state**: Agent execution state persists automatically
- **Built-in persistence**: Save and resume workflows at any point
- **Human-in-the-loop**: First-class API support for pausing execution for human review
- **Real-time streaming**: Token-by-token output showing agent reasoning

### Design Patterns

| Pattern | Implementation |
|---------|----------------|
| Sequential | Nodes chain linearly |
| Branching | Conditional edges based on state |
| Parallel | Fan-out/gather subgraphs |
| Cyclical | Loops for iterative refinement |

### Strengths

- **Granular control**: Every state transition is explicit and debuggable
- **Performance**: Lowest latency and token usage in benchmarks due to reduced redundant context passing
- **Enterprise adoption**: Production-proven at scale with observability built in
- **LangChain integration**: Seamless transition from chains to graphs when complexity requires

### Trade-offs

- **Scaling friction**: High parallelism and distributed execution require external systems
- **Missing batteries**: Retries, fallbacks, and observability often need external tooling
- **Verbose for simple cases**: Graph definition overhead for linear workflows

### When to Use

LangGraph fits when:
- Tasks require branching, error recovery, or conditional logic
- Long-running workflows need checkpoint and resume capabilities
- Production requirements demand durable execution guarantees
- Teams need visual representation of complex workflows

LangGraph does not fit when:
- Simple linear workflows suffice
- Rapid prototyping is the priority
- Teams lack graph-based thinking experience

---

## CrewAI

*[2026-01-30]*: CrewAI operates as a standalone framework (independent of LangChain) with 100,000+ certified developers and adoption by 60% of Fortune 500 companies.

### Core Architecture

CrewAI organizes agents into "crews" where each agent has a defined role, backstory, and tool access. Agents collaborate on tasks through structured delegation.

**Two Primitives:**
- **Crews**: Teams of autonomous agents with specialized roles
- **Flows**: Event-driven workflows that orchestrate crews and manage state

### Design Philosophy

Role-based design mirrors human team organization:

```
┌─────────────────────────────────────┐
│              Flow                   │
│  ┌───────────────────────────────┐  │
│  │           Crew                │  │
│  │  ┌─────────┐  ┌─────────┐    │  │
│  │  │Researcher│  │Developer│    │  │
│  │  │  Agent  │  │  Agent  │    │  │
│  │  └─────────┘  └─────────┘    │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### Strengths

- **Intuitive model**: Role-based design maps to natural team structures
- **Performance**: Executes 5.76x faster than LangGraph in certain task types
- **Standalone**: No dependency on LangChain, lighter resource demands
- **Enterprise-ready**: AWS partnership with Amazon Bedrock integration

### Trade-offs

- **Less granular control**: Role abstraction hides coordination details
- **Stochastic behavior**: Agent interactions can be unpredictable
- **Limited graph capabilities**: Complex conditional logic harder to express

### When to Use

CrewAI fits when:
- Teams have clear role separation (researcher, analyst, writer)
- Rapid prototyping is prioritized over fine-grained control
- Business users need to understand agent architecture
- Multi-agent collaboration is primary use case

CrewAI does not fit when:
- Workflows require complex branching logic
- Deterministic, reproducible behavior is critical
- Fine-grained state management is needed

---

## Microsoft Agent Framework (AutoGen Evolution)

*[2026-01-30]*: Microsoft merged AutoGen with Semantic Kernel into a unified Microsoft Agent Framework, targeting GA by end of Q1 2026. AutoGen enters maintenance mode—existing projects require migration planning.

### Core Architecture

The Agent Framework treats workflows as conversations between agents. Each agent can communicate with humans, tools, and other agents through natural language exchange.

**Orchestration Patterns:**
- **Sequential**: Step-by-step workflows
- **Concurrent**: Agents work in parallel
- **Group chat**: Agents brainstorm collaboratively
- **Handoff**: Responsibility transfers as context evolves
- **Magentic**: Manager agent coordinates via dynamic task ledger

### Unique Capabilities

**Request and Response**: Workflows can pause execution and wait for external input before continuing—enabling sophisticated human-in-the-loop patterns not present in AutoGen's Team abstraction.

**Enterprise Integration**: Native Azure integration, multi-language support (C#, Python, Java), production SLAs, and compliance guarantees (SOC 2, HIPAA).

### Strengths

- **Conversational flexibility**: Agents adapt roles based on context
- **Microsoft ecosystem**: Deep Azure AI Foundry integration
- **Research backing**: #1 accuracy on GAIA benchmark
- **A2A Protocol**: Agent-to-Agent interoperability across runtimes

### Trade-offs

- **Stochastic behavior**: Conversation-driven agents may loop or go off-track
- **Migration overhead**: AutoGen users must migrate to Agent Framework
- **Complexity**: Conversational dynamics harder to debug than explicit graphs
- **Azure-centric**: Full capabilities require Azure investment

### When to Use

Microsoft Agent Framework fits when:
- Azure is the target deployment environment
- Dynamic, context-dependent role assignment is valuable
- Enterprise compliance requirements exist
- Conversational multi-agent patterns match the use case

Microsoft Agent Framework does not fit when:
- Deterministic workflow behavior is required
- Teams want vendor-neutral solutions
- Simple delegation suffices over conversation

---

## Claude Agent SDK

*[2026-01-30]*: The Claude Agent SDK (renamed from Claude Code SDK in late 2025) provides the agent harness powering Claude Code, now available for custom agent development.

### Core Architecture

The SDK provides an agentic loop where Claude autonomously reads files, runs commands, searches the web, and edits code. Tools are first-class citizens in Claude's context window.

**Key Components:**
- **Built-in tools**: File operations, command execution, code editing
- **Custom tools via MCP**: In-process MCP servers without separate processes
- **Tool Search Tool**: Access thousands of tools without context bloat
- **Programmatic tool calling**: Invoke tools in code execution environment

### Advanced Tool Use (2026)

Three features for large-scale tool libraries:
- **Tool Search Tool**: Search over tools dynamically (85% token reduction)
- **Programmatic Tool Calling**: Reduces context window impact
- **Tool Use Examples**: Universal standard for demonstrating tool usage

Performance: Opus 4.5 accuracy improved from 79.5% to 88.1% with Tool Search enabled.

### Strengths

- **Tool-native**: Tools are the primary building blocks, not an afterthought
- **Same harness as Claude Code**: Battle-tested in production
- **Python and TypeScript**: SDK available in both languages
- **MCP ecosystem**: Access to growing tool library

### Trade-offs

- **Single-vendor**: Deep Anthropic coupling
- **Emerging patterns**: Multi-agent coordination still evolving
- **Subagent constraints**: Task tool unavailable to subagents (flat orchestration)

### When to Use

Claude Agent SDK fits when:
- Deep tool integration is the primary requirement
- Autonomous code execution and file manipulation are needed
- Anthropic models are the target
- Teams want Claude Code's proven patterns

Claude Agent SDK does not fit when:
- Multi-vendor model support is required
- Complex multi-agent coordination is primary use case
- Teams need established multi-agent patterns out of the box

---

## Decision Framework

### Quick Selection Guide

| If you need... | Consider |
|----------------|----------|
| Complex stateful workflows with checkpoints | LangGraph |
| Team-based collaboration with clear roles | CrewAI |
| Azure-native enterprise deployment | Microsoft Agent Framework |
| Deep tool integration with Anthropic models | Claude Agent SDK |
| Rapid prototyping | CrewAI |
| Production scale with durability | LangGraph |
| Conversational multi-agent dynamics | Microsoft Agent Framework |

### Decision Tree

```
Is model-agnostic critical?
├─ Yes → LangGraph or CrewAI
└─ No
   ├─ Azure ecosystem? → Microsoft Agent Framework
   └─ Anthropic models? → Claude Agent SDK

Is workflow complexity high?
├─ Yes (branching, loops, checkpoints) → LangGraph
└─ No (sequential, role-based) → CrewAI

Is rapid prototyping priority?
├─ Yes → CrewAI
└─ No → Match to deployment environment
```

### Hybrid Approaches

Frameworks are not mutually exclusive. Common patterns:

- **Prototype in CrewAI, productionize in LangGraph**: Use CrewAI's intuitive model for exploration, then implement in LangGraph for production durability
- **ADK tool wrappers**: Google ADK's `LangchainTool` and `CrewaiTool` enable cross-framework composition
- **MCP as integration layer**: Connect agents across frameworks via shared tool servers

---

## Framework Evolution Trends

### 2026 Landscape Shifts

- **Production focus**: Frameworks compete on enterprise features (durability, compliance, observability)
- **Agent interoperability**: A2A protocol and MCP enable cross-framework coordination
- **Consolidation**: Microsoft merged AutoGen and Semantic Kernel; expect similar consolidation
- **Multi-modal expansion**: Audio/video streaming becoming framework differentiator
- **Persistent personal runtimes**: A distinct category is emerging alongside orchestration frameworks — single-agent runtimes optimized for long-running personal use with persistent memory and self-improving skills rather than multi-agent coordination. Nous Research's [Hermes Agent](https://github.com/NousResearch/hermes-agent) (MIT, ~47K GitHub stars, February 2026) is the leading open-source example: multi-platform gateway (Telegram, Discord, Slack, CLI), SQLite-backed memory with full-text search, and a self-improvement loop where tasks generate procedural skills that refine during deployment. This category is not addressed by LangGraph, CrewAI, or AutoGen — those frameworks solve orchestration topology; persistent runtimes solve continuity and recall across sessions.

### Market Context

Agent-based systems now represent 86% of copilot spending ($7.2B). Framework selection has moved from technical evaluation to strategic platform decision.

---

## Connections

- **To [Claude Code](1-claude-code.md)**: Claude Agent SDK powers Claude Code's subagent system and tool execution
- **To [Google ADK](2-google-adk.md)**: ADK provides similar multi-agent coordination with Google Cloud integration
- **To [Orchestrator Pattern](../6-patterns/3-orchestrator-pattern.md)**: Frameworks implement orchestrator patterns differently—LangGraph via graphs, CrewAI via crews, AutoGen via conversations
- **To [Tool Use](../5-tool-use/_index.md)**: Framework tool integration patterns vary significantly—MCP support, tool filtering, context management
- **To [Multi-Agent Context](../4-context/4-multi-agent-context.md)**: Each framework handles context isolation differently

---

## Sources

- [LangChain and LangGraph v1.0 Milestones](https://www.blog.langchain.com/langchain-langgraph-1dot0/)
- [LangGraph Overview - LangChain Docs](https://docs.langchain.com/oss/python/langgraph/overview)
- [CrewAI - The Leading Multi-Agent Platform](https://www.crewai.com/)
- [CrewAI Introduction](https://docs.crewai.com/en/introduction)
- [Microsoft Agent Framework Overview](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview)
- [Introducing Microsoft Agent Framework - Azure Blog](https://azure.microsoft.com/en-us/blog/introducing-microsoft-agent-framework/)
- [AutoGen to Microsoft Agent Framework Migration Guide](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/)
- [Building agents with the Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk)
- [Advanced Tool Use - Anthropic](https://www.anthropic.com/engineering/advanced-tool-use)
- [Agent Orchestration 2026 - Iterathon](https://iterathon.tech/blog/ai-agent-orchestration-frameworks-2026)
- [CrewAI vs LangGraph vs AutoGen - DataCamp](https://www.datacamp.com/tutorial/crewai-vs-langgraph-vs-autogen)
- [Top AI Agent Frameworks - Turing](https://www.turing.com/resources/ai-agent-frameworks)
