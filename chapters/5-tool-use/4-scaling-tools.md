---
title: Scaling Tool Use
description: Patterns for managing large numbers of tools without overwhelming context or degrading selection accuracy
created: 2025-12-10
last_updated: 2026-04-07
tags: [scaling, dynamic-discovery, orchestration, mcp, deployment]
part: 1
part_title: Foundations
chapter: 5
section: 4
order: 1.5.4
---

# Scaling Tool Use

Patterns for managing large numbers of tools without overwhelming context or degrading selection accuracy.

---

## Dynamic Tool Discovery

*[2025-12-09]*: When tool definitions consume significant tokens (>10K), dynamic discovery trades search overhead for context efficiency.

**The Problem**: Tool definitions create substantial token overhead before conversation even begins. A typical MCP setup with GitHub, Slack, Sentry, Grafana, and Splunk consumes ~55K tokens in tool schemas alone—and can exceed 100K+ with additional services.

**The Solution**: Mark rarely-used tools with `defer_loading: true`. The agent searches for relevant tools dynamically, receiving references that expand into full definitions only when discovered.

**Results** (Anthropic internal testing):
- 85% reduction in token usage while maintaining full tool library access
- Improved selection accuracy: Opus 4 from 49% → 74%, Opus 4.5 from 79.5% → 88.1%

**Best Practices**:
- Use when tool definitions consume >10K tokens total
- Employ clear, descriptive tool names and descriptions (discovery depends on search)
- Keep 3-5 most-used tools always loaded (no discovery overhead for common operations)
- Compatible with prompt caching—deferred tools remain excluded from cache

**Decision Point**: If you're managing <20 simple tools, eager loading is fine. If you're connecting MCP servers with dozens of endpoints each, dynamic discovery becomes essential.

**See Also**:
- [Context: Progressive Disclosure](../4-context/3-context-patterns.md#progressive-disclosure-pattern) — The underlying pattern for tiered information loading
- [Cost and Latency](../7-practices/3-cost-and-latency.md) — Token cost implications of discovery vs. eager loading

**Source**: [Advanced Tool Use - Anthropic](https://www.anthropic.com/engineering/advanced-tool-use)

---

## Programmatic Tool Orchestration

*[2025-12-09]*: For complex workflows, let the agent write orchestration code instead of calling tools directly.

**The Problem**: Traditional tool calling creates context pollution. Processing 2,000+ expense line items requires results to re-enter the agent's context, consuming 50KB+ and requiring a full inference pass per tool invocation.

**The Inversion**: Instead of agent → tool → agent → tool sequences, the agent writes Python code that orchestrates multiple tool calls. Results process in-place within a sandboxed environment, never re-entering the agent's context until final output.

**Implementation Pattern**:
- Mark tools with `allowed_callers: ["code_execution_20250825"]`
- Agent generates async Python using `asyncio.gather()` for parallel execution
- Code pauses at tool calls, processes results, continues without model intervention
- Only final output appears in agent's view

**Results** (Anthropic internal testing):
- 37% token reduction (43,588 → 27,297 average)
- Eliminates 19+ inference passes in 20-tool workflows
- Improved accuracy on knowledge retrieval: 25.6% → 28.5%

**When to Use**:
- Multi-step workflows with 3+ dependent tool calls
- Parallel operations where results need filtering/transforming
- Large result sets that would bloat context if returned directly

**When NOT to Use**:
- Simple single-tool invocations (overhead isn't worth it)
- Tasks requiring agent reasoning between each tool call

**See Also**:
- [Context: Multi-Agent Context Isolation](../4-context/4-multi-agent-context.md#multi-agent-context-isolation) — Related pattern for preventing context pollution through agent boundaries
- [Cost and Latency: Token Cost Models](../7-practices/3-cost-and-latency.md#token-cost-models-by-feature-type) — How programmatic orchestration compares to other approaches

**Source**: [Advanced Tool Use - Anthropic](https://www.anthropic.com/engineering/advanced-tool-use)

---

## MCP Deployment Architecture

*[2025-12-09]*: MCP deployment isn't one-size-fits-all. The transport mechanism determines scaling characteristics and operational constraints.

**Three Deployment Patterns**

| Pattern | Mechanism | Scaling | Best For |
|---------|-----------|---------|----------|
| **stdio** | Local server proxying remote services | Doesn't scale horizontally—single process | Local development, simple integrations |
| **Streamable HTTP** | SSE-based, independent process | Stateless replication possible | Cloud services, production APIs |
| **Sidecar** | Kubernetes co-located container | Lifecycle-coupled with main app | Orchestrated container environments |

**The Statefulness Challenge**

MCP connections are stateful. This creates real constraints:
- Load balancing requires session affinity
- Horizontal scaling needs sticky sessions or connection migration
- Failure handling must account for lost connection state

For simple deployments, this doesn't matter. At scale, it determines your architecture.

**Pattern Selection Guide**

```
Local dev only?
  └── Yes → stdio (simplest, no scaling needed)
  └── No → Cloud deployment?
           └── Container orchestration (K8s)?
                └── Yes → Sidecar (lifecycle coupling)
                └── No → Streamable HTTP (stateless scaling)
```

**Google ADK's MCP Integration**

ADK supports both directions:
- **Client mode**: ADK agents consume tools from external MCP servers via `McpToolset`
- **Server mode**: Expose ADK tools as MCP servers for other clients (Claude Code, Cursor)

The bidirectional nature enables framework interoperability—ADK tools accessible from Claude Code, MCP servers accessible from ADK.

**Tool Filtering at Runtime**

`tool_filter` in McpToolset enables dynamic capability restriction:

```python
mcp_toolset = McpToolset(
    server_config=StdioServerConfig(command="npx", args=["mcp-server-github"]),
    tool_filter=lambda tool: tool.name in ["list_issues", "create_issue"]
)
```

This serves two purposes:
1. **Security**: Expose only needed capabilities per agent
2. **Model focus**: Fewer tools means better tool selection (less overwhelm)

**See Also**: [Google ADK: MCP Integration](../9-practitioner-toolkit/2-google-adk.md#mcp-integration) — Concrete implementation patterns

---

## MCP Auto-Selection Mode

*[2026-01-17]*: Claude Code 2.1.7 introduced automatic MCP tool selection, enabled by default. This changes how tools surface in agent workflows.

**Default Behavior**

Starting with version 2.1.7:
- MCP tools auto-discover based on task context relevance
- Tool descriptions load automatically without explicit configuration
- System uses keyword matching against tool names and descriptions

**Threshold Configuration (2.1.9+)**

Control when tool descriptions defer vs. load upfront:

```json
// In .claude/settings.json or project config
{
  "mcpConfig": {
    "toolSearch": "auto:15"  // Defer when >15% of context window
  }
}
```

**Threshold values:**
- `auto:5` — Conservative: defer early, preserve context
- `auto:10` — Default: balanced approach
- `auto:15-20` — Aggressive: load more tools upfront
- `auto:0` — Disable auto mode (manual tool management)

**When to Adjust Threshold**

| Scenario | Recommended Threshold |
|----------|----------------------|
| Few MCP servers, small descriptions | Default (`auto:10`) |
| Many servers, large descriptions | Lower (`auto:5`) |
| Context-constrained workflows | Lower (`auto:5`) |
| Discovery-heavy exploration | Higher (`auto:20`) |

**Trade-offs**

Benefits of auto mode:
- Fewer "tool not available" errors
- Reduced manual configuration burden
- Dynamic tool discovery adapts to task

Costs:
- Slower startup if many tools load unnecessarily
- Context overhead for tool descriptions
- Less predictable tool availability

**Disabling Auto Mode**

For deterministic workflows where tool availability must be explicit:

```json
{
  "mcpConfig": {
    "toolSearch": "manual"
  }
}
```

Manual mode requires explicit tool declarations in prompts or configuration. Use this when reproducibility matters more than convenience.

**See Also**:
- [Dynamic Tool Discovery](#dynamic-tool-discovery) — The underlying pattern auto mode builds on
- [Tool Selection](2-tool-selection.md) — How selection accuracy changes with auto-discovery

---

## Leading Questions

- How do you monitor tool usage patterns to optimize discovery configurations?
- When should you cache tool definitions vs. re-fetch them?
- How do you handle versioning when tools are dynamically discovered?
- What's the right balance between eager loading and discovery overhead?
- How do you test programmatic orchestration patterns?

---

## Connections

- **To [Tool Selection](2-tool-selection.md):** Discovery changes selection dynamics
- **To [Tool Restrictions](3-tool-restrictions.md):** MCP deployment intersects security
- **To [Context](../4-context/_index.md):** Progressive disclosure and context management
- **To [Cost and Latency](../7-practices/3-cost-and-latency.md):** Token cost models by feature type
- **To [Google ADK](../9-practitioner-toolkit/2-google-adk.md):** MCP implementation patterns
- **To [Enterprise Codebase Context Tools](../9-practitioner-toolkit/6-enterprise-context-tools.md):** For tool selection in enterprise brownfield contexts, the MCP deployment patterns documented here (transport mechanism, auto-selection thresholds, tool filtering) apply directly to tools like Serena MCP, Augment's Context Engine MCP, and Sourcegraph's MCP server surveyed there.
