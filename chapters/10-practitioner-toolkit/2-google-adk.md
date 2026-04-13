---
title: Google ADK
description: Google's Agent Development Kit - multi-agent-first framework for production AI agents
created: 2025-12-09
last_updated: 2025-12-10
tags: [tools, google-adk, multi-agent, orchestration, gemini, mcp, workflow-primitives]
part: 3
part_title: Perspectives
chapter: 10
section: 2
order: 3.10.2
---

# Google ADK

Google's open-source Agent Development Kit. A code-first framework for building production AI agents with native multi-agent coordination.

---

## Multi-Agent First Design

*[2025-12-09]*: ADK's defining characteristic is that multi-agent coordination is baked in, not bolted on. Where LangChain retrofitted multi-agent support and CrewAI added it as a specialized layer, ADK designed hierarchical agent systems from day one.

### Agent Types

**LlmAgent**: Dynamic, model-driven decision-making. The workhorse for tasks requiring reasoning and tool selection.

**Workflow Agents**: Structured execution without LLM overhead per coordination decision:
- **SequentialAgent**: Pipeline execution—agent A's output feeds agent B
- **ParallelAgent**: Fan-out/gather—multiple agents run concurrently, results collected
- **LoopAgent**: Iteration until completion condition met

**Custom Agents**: Developer-defined implementations for specialized coordination patterns.

The insight: workflow agents eliminate LLM calls for coordination decisions that don't require reasoning. When your orchestration logic is deterministic, don't pay for intelligence you don't need.

### Three Coordination Mechanisms

1. **Shared Session State**: Agents read/write to common state with scoped prefixes (`user:`, `app:`, `session:`, `temp:`)
2. **LLM-Driven Delegation**: `transfer_to_agent()` hands control based on model reasoning
3. **Explicit Invocation**: `AgentTool` wrappers make agents callable as tools

These aren't mutually exclusive—production systems often combine all three.

### The Single-Instantiation Constraint

"An agent instance can only be added as a sub-agent once."

This prevents circular dependencies and keeps architecture comprehensible. It's a hard constraint that forces explicit design decisions about agent relationships.

---

## State and Memory Architecture

*[2025-12-09]*: ADK distinguishes what Claude Code's context management doesn't—persistent state vs. ephemeral context.

### State Prefixes

| Prefix | Lifetime | Use Case |
|--------|----------|----------|
| `session:` | Current conversation | Working memory, intermediate results |
| `user:` | Across sessions for this user | Preferences, accumulated knowledge |
| `app:` | Application-wide | Shared configuration, global state |
| `temp:` | Single turn | Scratch space, discardable |

The elegance: prefix determines persistence lifetime without configuration overhead. Namespace by lifecycle, not by feature.

### Three-Tier Persistence

1. **InMemorySessionService**: Development/testing. No persistence—data dies with the process.
2. **VertexAiSessionService**: Production. Google-managed persistence and scaling.
3. **Database-backed**: SQLite or custom. For persistence outside Google Cloud.

### Memory Services

- **InMemoryMemoryService**: Keyword-based retrieval. Simple, fast, no vector dependencies.
- **VertexAiMemoryBankService**: Managed vector storage. Semantic search over accumulated knowledge.

This is the persistent state layer that Claude Code's context management doesn't have—state that survives sessions rather than reconstructed context.

---

## MCP Integration

*[2025-12-09]*: ADK supports MCP in both directions—as client and server.

### Two Integration Modes

**Using MCP Servers in ADK**: ADK agents consume tools from external MCP servers via `McpToolset`:

```python
mcp_toolset = McpToolset(
    server_config=StdioServerConfig(command="npx", args=["mcp-server-github"]),
    tool_filter=lambda tool: tool.name in ["list_issues", "create_issue"]
)
```

**Exposing ADK Tools via MCP**: Build MCP servers that wrap ADK tools for consumption by other MCP clients (Claude Code, Cursor, etc.).

### Deployment Patterns

| Pattern | Mechanism | Scaling Characteristics |
|---------|-----------|------------------------|
| **stdio** | Local server proxying remote services | Doesn't scale horizontally—single process |
| **Streamable HTTP** | SSE-based, independent process | Stateless replication possible |
| **Sidecar** | Kubernetes co-located container | Lifecycle-coupled with main app |

Match deployment architecture to target environment. Stdio for local dev, HTTP for cloud services, sidecar for orchestrated containers.

### The Statefulness Challenge

MCP connections are stateful—session affinity required for load balancing at scale. This is a real production constraint that affects horizontal scaling strategies.

### Tool Filtering for Security

`tool_filter` in McpToolset restricts exposed capabilities per agent:
- Reduces model overwhelm during tool selection (fewer options = better decisions)
- Implements least-privilege at the tool level
- Dynamic filtering at runtime vs. static configuration

---

## Production Lessons

*[2025-12-09]*: Real deployment experiences from practitioners.

### Common Gotchas

**Environment Variable Conflicts**: Generic names like `MODEL` conflict with system variables. Use prefixed names like `GEMMA_MODEL_NAME`.

**Cloud Run API Permissions**: Enable Cloud Run Admin API before deployment—missing permissions fail with cryptic errors.

**Asyncio Requirements**: Both ADK and MCP Python libraries use asyncio extensively. Tool implementations should be async functions. Sync code blocks the event loop and serializes parallel operations.

### Deployment Journey

1. **Local**: `adk web` development UI for testing
2. **Containerize**: Docker with proper environment setup
3. **Deploy**: Cloud Run or Vertex AI Agent Engine
4. **Observe**: Cloud Trace integration for request tracing

### The "Build Once, Interact Anywhere" Mental Model

The agent executed on the local command line is the same one debugged in the web UI, and the same one called from a live, user-facing application. This portability assumption helps teams reason about agent behavior across environments.

### Production-Ready for Whom?

Official sources emphasize production-readiness ("powers Google's Agentspace"). Practitioners note early-stage developer experience outside Google's ecosystem.

**Resolution**: ADK is production-ready *for Google Cloud users*. Outside that context, it's still maturing. The model-agnostic claims are accurate but friction increases outside Google's stack.

---

## Framework Comparison Context

*[2025-12-09]*: Practitioners don't evaluate frameworks in isolation—they compare.

### Positioning

| Framework | Strength | Trade-off |
|-----------|----------|-----------|
| **ADK** | Native multi-agent, Google Cloud integration | Smaller community, ecosystem lock-in risk |
| **LangChain** | Massive community, flexibility | Production orchestration at scale is harder |
| **CrewAI** | Rapid prototyping, role-playing abstraction | Less control for complex workflows |
| **Claude Code SDK** | Deep Anthropic integration, subagent isolation | Single-vendor assumption |

### Practitioner Pattern

A common approach: prototype ideas in CrewAI, then productionize in LangGraph—or in Google ADK when tight GCP integration is needed. Framework selection is context-dependent, not dogmatic.

### What ADK Does Differently

- **Multi-agent by design**: Hierarchical coordination as first-class architecture, not an afterthought
- **Workflow primitives**: Deterministic orchestration without LLM overhead
- **State persistence built-in**: Session services and memory banks as core concepts
- **Bidirectional streaming**: Audio/video streaming (unique among these frameworks)
- **Framework interoperability**: `LangchainTool` and `CrewaiTool` wrappers for cross-framework composition

### Framework Interoperability Pattern

ADK's tool wrappers enable composing frameworks:

```python
from google.adk.tools import LangchainTool, CrewaiTool

# Wrap a LangChain agent as an ADK tool
langchain_tool = LangchainTool(my_langchain_agent)

# Use CrewAI's role-playing in an ADK workflow
crewai_tool = CrewaiTool(my_crewai_agent)
```

This enables best-of-breed composition: prototype with CrewAI's abstractions, productionize with ADK's deployment infrastructure.

---

## Connections

- **To [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md):** ADK's SequentialAgent/ParallelAgent/LoopAgent are concrete implementations of the orchestrator pattern's workflow primitives
- **To [Context](../4-context/_index.md):** ADK's state prefixes distinguish persistent state from ephemeral context—a distinction worth adopting more broadly
- **To [Tool Use](../5-tool-use/_index.md):** ADK's MCP integration and tool filtering extend the patterns in MCP Tool Declarations
- **To [Production Concerns](../8-practices/4-production-concerns.md):** Deployment gotchas and scaling considerations
- **To [Claude Code](1-claude-code.md):** Different approaches to multi-agent: ADK's workflow agents vs. Claude Code's subagent isolation

---

## Sources

- [Agent Development Kit - Google](https://google.github.io/adk-docs/)
- [ADK Python GitHub](https://github.com/google/adk-python)
- [Multi-agent systems in ADK](https://google.github.io/adk-docs/agents/multi-agents/)
- [Session and Memory Management](https://google.github.io/adk-docs/sessions/)
- [MCP Integration](https://google.github.io/adk-docs/mcp/)
- [Google ADK vs LangGraph - ZenML](https://www.zenml.io/blog/google-adk-vs-langgraph)
- [Framework Comparison - Medium](https://medium.com/@prabhudev.guntur/choosing-your-ai-agent-framework-google-adk-vs-autogen-langchain-crewai-a-deep-dive-c0f07e3a9d13)
- [Production Deployment Journey - Medium](https://medium.com/@jackchem2003/from-prototype-to-production-my-journey-deploying-an-adk-agent-on-google-cloud-run-8a0ac0bc1468)
- [ADK meets MCP - Google Cloud Blog](https://medium.com/google-cloud/adk-meets-mcp-bridging-worlds-of-ai-agents-1ed96ef5399c)
