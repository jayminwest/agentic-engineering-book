---
title: Orchestrator Pattern
description: Hub-and-spoke coordination of specialized sub-agents in a single workflow
created: 2025-12-08
last_updated: 2026-01-21
tags: [patterns, multi-agent, orchestration, coordination, tool-restriction]
part: 2
part_title: Craft
chapter: 6
section: 3
order: 2.6.3
---

# Orchestrator Pattern

A main coordinator invokes specialized sub-agents, synthesizes their outputs, and manages workflow transitions. Hub-and-spoke architecture for multi-agent systems.

---

## Core Structure

```
Orchestrator (Main Coordinator)
├── Phase 1: Scout Agent (read-only exploration)
├── Phase 2: Planning Council (parallel domain experts)
│   ├── Architecture Expert
│   ├── Testing Expert
│   ├── Security Expert
│   └── ... (domain-specific)
├── Phase 3: Build Agents (parallel batches, dependency-aware)
├── Phase 4: Review Panel (parallel experts + meta-checks)
└── Phase 5: Validation (execution)
```

---

## Key Mechanisms

### Single-Message Parallelism

All parallel agents must be invoked in **one message** for true concurrency. Sequential messages serialize execution.

```
In a single message, make multiple Task tool calls:
- Task(subagent_type="build-agent", prompt="[spec for file1]")
- Task(subagent_type="build-agent", prompt="[spec for file2]")
- Task(subagent_type="build-agent", prompt="[spec for file3]")
```

This is the critical insight: parallelism is achieved at the message level, not the agent level.

*[2025-12-09]*: This is the make-or-break implementation detail that most practitioners miss. If you invoke three Task tools across three separate messages, they execute sequentially—not in parallel. The orchestrator must emit all parallel Task calls in a single response. This explains why many "parallel" multi-agent systems actually run sequentially: developers assume agents will run concurrently by default, but the framework requires explicit single-message invocation. When debugging performance issues in multi-agent systems, check single-message parallelism first.

**How you scale multi-agent work**: Spawn all independent agents in a single message rather than sequential messages. This isn't just about orchestrators—it's the fundamental pattern for parallelizing any multi-agent work. Three Task calls in one message execute concurrently. Three Task calls across three messages execute sequentially. The difference compounds: 10 agents in parallel complete in roughly the same wall-clock time as 1 agent; 10 agents serialized take 10× longer.

**Sources**: [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system), [Subagents - Claude Code Docs](https://docs.claude.com/en/docs/claude-code/sub-agents)

### Dependency-Aware Batching

For files with dependencies:
- **Batch 1**: Files with no dependencies (parallel)
- **Batch 2**: Files depending on Batch 1 (wait, then parallel)
- **Batch 3**: Files depending on Batch 2
- etc.

Analyze the dependency graph, group into batches, parallelize within each batch.

### Spec File as Shared Context

A single artifact (spec file) flows through all phases:
- Scout outputs exploration findings
- Plan phase creates `docs/specs/<name>.md` with full context
- Build agents read spec file for implementation details
- Review agents reference spec file for compliance checking

This avoids passing massive context between agents—instead, they read from a shared artifact.

### Phase Gating

Mandatory prerequisites before transitions:
- **scout → plan**: Pass exploration findings
- **plan → build**: Spec file must exist (verify with `test -f`)
- **build → review**: Build must complete successfully
- **review → validate**: Always run validation after review

If a prerequisite fails, halt and provide remediation instructions.

---

## Context Passing

| Transition | What Flows |
|------------|------------|
| Scout → Plan | File locations, patterns, dependencies |
| Plan → Build | Spec file path, architecture patterns |
| Build → Review | Commit hashes, changed file paths |
| Review → Validate | Validation scope (based on review findings) |

Each agent receives complete context—they're stateless and assume nothing from prior calls.

---

## Context Isolation via Sub-Agents

*[2025-12-10]*: The primary rationale for delegation isn't just parallelism—it's **context hygiene**. Each sub-agent gets a fresh context window for its specialized task, preventing pollution of the orchestrator's decision-making context.

### Why Context Isolation Matters

When an orchestrator needs to search files, grep patterns, or summarize code, doing this work directly fills its context window with raw data:
- File listings with hundreds of paths
- Grep results with dozens of matching lines
- Full file contents for analysis

This raw data crowds out the orchestrator's primary job: workflow coordination and decision synthesis.

### The Pattern

**Deploy fresh context windows for search operations:**
```
Orchestrator (clean context):
├─ Task → Scout: "Find all Python files in src/"
│         Scout context: file paths, directory structure
│         Scout returns: "Found 47 Python files, organized in 3 modules..."
│
├─ Task → Analyzer: "Summarize authentication flow"
│         Analyzer context: auth files, dependencies
│         Analyzer returns: "Authentication uses OAuth2 with..."
│
└─ Orchestrator synthesizes summaries into decisions
    (never saw raw file listings or grep output)
```

**Sub-agents return synthesized summaries, not raw data:**
- Scout returns: "Found 47 files in 3 modules" (not 47 file paths)
- Grep agent returns: "Pattern appears in 12 locations, primarily in validation layer" (not 12 raw matches)
- Code analyzer returns: "Authentication flow uses OAuth2 with custom middleware" (not full code dump)

### The Trade-off

| Aspect | Direct Execution | Sub-Agent Delegation |
|--------|------------------|---------------------|
| **Token cost** | Lower (single context) | Higher (multiple contexts) |
| **Context clarity** | Polluted with raw data | Clean, summary-only |
| **Decision quality** | Degraded by noise | Focused on synthesis |
| **Parallelism** | Sequential | Concurrent |

**When the trade-off favors delegation:**
- Orchestrator needs to make complex decisions based on synthesized information
- Search/analysis operations produce large intermediate results
- Multiple independent investigations can run in parallel
- Workflow spans multiple phases requiring clean transitions

**When direct execution works:**
- Simple, single-phase tasks
- Orchestrator needs raw data for decision-making
- Token budget is constrained
- No subsequent decision synthesis needed

### Context Hygiene Best Practices

1. **Prompt sub-agents for synthesis**: "Summarize findings in 3-5 bullet points" rather than "return all matches"
2. **Filter before reporting**: Sub-agents should grep/analyze/filter, then report insights—not raw output
3. **Keep orchestrator context minimal**: Only workflow state, phase transitions, and synthesized findings
4. **Avoid echo chambers**: Don't pass large sub-agent outputs as-is to other sub-agents—summarize first

### This Is Why Orchestrators Delegate

The orchestrator pattern isn't just about parallelism—it's about maintaining clean separation between:
- **Discovery** (file finding, pattern searching, code reading) — sub-agent contexts
- **Synthesis** (decision-making, workflow coordination) — orchestrator context

Each sub-agent pollutes its own context with raw data, then returns clean summaries. The orchestrator never sees the mess.

**Mental model**: Sub-agents are expensive, disposable context buffers. They absorb the noise so the orchestrator can think clearly.

---

## Expert Synthesis

When multiple experts analyze in parallel:
1. Collect structured outputs from each expert
2. Identify cross-cutting concerns (mentioned by 2+ experts)
3. Synthesize into unified recommendations
4. Create priority actions and risk assessment

The orchestrator is responsible for synthesis—individual experts stay focused on their domain.

---

## Error Handling

### Graceful Degradation
- If an expert fails, note the failure and continue with available analyses
- Recommend manual review for failed expert domains
- Include recovery instructions in output

### Partial Success
- Commit successful changes before reporting failures
- Allow selective retry via phases parameter
- Never leave the workflow in an inconsistent state

---

## When to Use This Pattern

### Good Fit

**Multi-concern workflows requiring parallel analysis:**
- Complex tasks spanning multiple domains (architecture, security, testing, performance)
- Tasks that benefit from parallel execution (multiple independent analyses or builds)
- Workflows requiring explicit phase gates and artifacts for review
- Need for context isolation between exploration and decision-making

**Indicators you need this pattern:**
- Single agent's context window fills with search/analysis data
- Multiple independent tasks can run concurrently
- Workflow has clear phase transitions with checkpoints
- Need to synthesize insights from multiple domain experts

### Poor Fit

**Simple or tightly-coupled tasks:**
- Single-file changes with straightforward requirements
- Tasks where coordination overhead exceeds parallelism benefit
- Workflows requiring tight real-time interaction between agents
- When context pollution isn't a concern (simple, single-phase tasks)

---

## Anti-Patterns Discovered

- **Missing spec path validation**: Build phase must verify spec file exists before proceeding
- **Implicit environment assumptions**: Always detect and report environment in validation
- **Incomplete meta-reviews**: Reviews must include hygiene checks (commits, labels, linked issues)
- **Vague risk assessment**: Must be concrete with mitigation strategies, not generic warnings

---

## Questions This Raises

- How do you decide the right number of parallel agents? (Resource constraints vs. diminishing returns)
- When does the spec file become a bottleneck vs. a coordination aid?
- How do you handle agents that produce conflicting recommendations?
- What's the minimum viable orchestrator? (Probably: scout → build → validate)

---

## Capability Minimization

*[2025-12-09]*: Orchestrators work better when their subagents have **intentionally restricted capabilities**. This isn't just security—it's an architectural forcing function.

### Why Restrict Tools

1. **Reduces context overhead**: An agent with 3 tools maintains smaller context than one with 20
2. **Forces delegation**: A read-only scout *cannot* implement—it must report findings for others to act on
3. **Enables parallelization**: Agents with minimal scope can run more instances simultaneously
4. **Clarifies responsibility**: Tool restrictions make the agent's role unambiguous

### Tool Restriction Patterns

| Agent Role | Tools | Rationale |
|------------|-------|-----------|
| Scout | Read, Glob, Grep | Cannot modify—forces reporting back |
| Builder | Write, Edit, Read, Bash | Focused on implementation |
| Reviewer | Read, Grep, Bash (tests only) | Cannot fix what they find |
| Validator | Bash (run only), Read | Executes, doesn't implement |

### Scope Restriction Beyond Tools

Tool restriction is only half the pattern. **Scope restriction** achieves the same goal through workflow design:

- **One file per builder**: Each build-agent handles exactly one file, even though it *could* write many
- **One domain per expert**: Security expert doesn't comment on testing, even though it *could*
- **Orchestrator as spec writer**: Primary job becomes packaging comprehensive context, not doing work

A common pattern in multi-agent systems:
> "Never guess or assume context: Each build-agent needs comprehensive instructions as if they are new engineers"

This forces the orchestrator to be explicit about what each subagent needs, keeping each agent's context minimal and focused.

### The Meta-Principle

Default Claude Code behavior is to inherit all parent tools. The deliberate choice to *restrict* tools signals architectural intent:

```yaml
# Inherits everything (default)
tools: # omit entirely

# Read-only analysis specialist
tools: Read, Glob, Grep

# Implementation specialist
tools: Write, Edit, Read, Bash
```

When reviewing an agent definition, ask: "What can this agent NOT do, and is that intentional?"

### SDK-Level vs CLI-Level Enforcement

*[2025-12-09]*: True HEAD vs subagent tool differentiation requires SDK-level enforcement, not CLI configuration.

CLI tools like Claude Code apply tool restrictions uniformly—the HEAD agent and all subagents share the same allowed tools set. This is a known limitation. If you configure `allowedTools` in settings.json, those restrictions apply everywhere.

SDK-level orchestration solves this by passing different `allowed_tools` arrays when spawning each agent:

```python
# Orchestrator: management tools only, no implementation
orchestrator_tools = [
    "mcp__mgmt__create_agent",
    "mcp__mgmt__command_agent",
    "Read", "Bash"  # info-gathering only
]
# Excluded: Write, Edit, WebFetch, Task

# Build subagent: implementation tools
builder_tools = ["Write", "Read", "Edit", "Bash", "Glob", "Grep"]
```

The pattern emerges across three mechanisms:
- **Technical**: `allowed_tools` allowlist passed to SDK when spawning agents
- **Behavioral**: System prompts reinforce "let subagents do the heavy lifting"
- **Architectural**: Orchestrator gets management/coordination tools, not implementation tools

This explains why sophisticated multi-agent systems often build custom orchestration layers on top of the Claude Agent SDK rather than relying solely on CLI tools. The SDK gives you the primitives; CLI tools give you convenience with less granularity.

**Source**: [agenticengineer.com](https://agenticengineer.com) — Production orchestrator implementation demonstrating SDK-level tool restriction

### Tool Restriction as Coordination Forcing Function

*[2025-12-09]*: Tool restriction isn't just about limiting what agents can do—it's about enabling coordination patterns through deliberate capability differentiation.

When an orchestrator uses different tools than its subagents, it creates natural separation of concerns:

1. **Encourages parallel execution**: Orchestrator focuses on spawning multiple specialized agents
2. **Maintains separation of concerns**: Orchestrator manages workflow, builders implement
3. **Enables different tool sets per role**: Orchestrators get management tools, builders get implementation tools
4. **Reduces shortcuts**: Clear role definitions guide proper delegation

**Note**: Tool restriction is enforced through system prompt guidance and agent design, not through technical enforcement mechanisms. The effectiveness comes from clear role definitions and workflow design.

---

## Tool Assignment

Orchestrators should assign different tool sets to different subagent roles. For native tools, this follows least-privilege principles. For MCP tools, the same pattern applies—validators get browser tools, scouts get search tools, etc.

See [Tool Use: MCP Tool Declarations](../5-tool-use/_index.md#mcp-tool-declarations-in-frontmatter) for the frontmatter syntax and role-based assignment patterns.

---

## Workflow Primitives

*[2025-12-09]*: When orchestration patterns become routine, extract them as reusable primitives. Google ADK codifies this with SequentialAgent, ParallelAgent, and LoopAgent—workflow controllers that compose specialized agents without LLM overhead per coordination decision.

### The Three Primitives

| Primitive | Pattern | When to Use |
|-----------|---------|-------------|
| **Sequential** | Pipeline—A's output feeds B | Dependent phases, ordered transformations |
| **Parallel** | Fan-out/gather—concurrent execution, collected results | Independent analysis, embarrassingly parallel tasks |
| **Loop** | Iterate until condition met | Refinement cycles, retry-with-feedback |

### Why This Matters

Traditional orchestration requires an LLM call to decide "what next?" after each step. For deterministic workflows—where the pattern is always "run A, then B, then C"—that's wasted inference.

Workflow primitives eliminate this overhead:
- SequentialAgent knows it runs steps in order
- ParallelAgent knows it runs steps concurrently
- LoopAgent knows it runs until a condition

The orchestrator LLM focuses on decisions that require reasoning: which primitive to invoke, how to handle failures, when to escalate.

### Composition

Primitives compose naturally:
```
SequentialAgent([
    ParallelAgent([scout_a, scout_b, scout_c]),  # Fan-out exploration
    planning_agent,                               # Synthesize findings
    LoopAgent(build_agent, until=tests_pass),    # Iterate to completion
    ParallelAgent([reviewer_a, reviewer_b])      # Parallel review
])
```

The meta-principle: when orchestrators themselves become boilerplate, extract them as parameterizable primitives.

### Framework Implementations

- **Google ADK**: Native SequentialAgent, ParallelAgent, LoopAgent classes
- **LangGraph**: StateGraph with conditional edges
- **Claude Code**: Single-message parallelism + explicit sequencing (no native primitives)

Claude Code achieves similar patterns through discipline (single-message parallel Task calls) rather than framework primitives. ADK makes the patterns explicit and removes the possibility of accidental serialization.

**See Also**: [Google ADK](../9-practitioner-toolkit/2-google-adk.md#multi-agent-first-design) — Concrete implementation of workflow primitives

---

## See Also

- [Plan-Build-Review](1-plan-build-review.md) - simpler version without parallel experts
- [Self-Improving Experts](2-self-improving-experts.md) - how the domain experts evolve
- [Google ADK](../9-practitioner-toolkit/2-google-adk.md) - framework with native workflow primitives
- [Context: Multi-Agent Context Isolation](../4-context/_index.md#multi-agent-context-isolation) - the foundational context management strategy that makes orchestration viable
- [Context Loading Demo](../../appendices/examples/context-loading-demo/README.md) - minimal example showing context payload construction and verification layer
