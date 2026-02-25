---
title: "Pi: Minimal Core Agent Architecture"
description: "Case study of Mario Zechner's pi-mono — an aggressively extensible coding agent toolkit that inverts the batteries-included paradigm"
created: 2026-02-24
last_updated: 2026-02-24
tags: [case-study, agent-framework, extension-architecture, pi-mono, open-source, typescript]
part: 4
part_title: Appendices
chapter: 10
section: 5
order: 4.10.5
---

# Pi: Minimal Core Agent Architecture

Pi is a modular AI agent toolkit built by Mario Zechner (@badlogic)—creator of the libGDX game framework—as a TypeScript monorepo with ~15,900 GitHub stars and MIT license. Where most agent frameworks bundle orchestration, permissions, multi-agent coordination, and workflow management into a single product, Pi inverts the paradigm: the core remains minimal, every behavior is an extension, and the framework exists to enable customization rather than to encode it.

The project matters for this book because Pi demonstrates a coherent alternative to the batteries-included approach taken by Claude Code, GasTown, and Overstory. Pi's maintainer deliberately rejects sub-agents, MCP support, permission popups, plan mode, and to-do lists in the core—not from oversight, but from explicit philosophy. Understanding why this works reveals how architectural minimalism can enable diversity of use cases that monolithic frameworks struggle to support.

---

## Core Questions

### Architecture and Design
- How does a 20+ lifecycle hook extension system replace built-in features?
- What does "TypeBox schema validation for tools" provide that informal conventions miss?
- How does separating `AgentMessage` (internal) from LLM `Message[]` (provider) enable multi-provider portability?

### Extension Philosophy
- Where does the boundary between core and extension belong?
- What happens when extensions override built-in tool implementations?
- How does auto-discovery (`.pi/extensions/` and `~/.pi/agent/extensions/`) change the deployment model?

### Context Management
- How does cumulative file tracking survive multiple compaction events?
- What problem does "split turn" handling address that simpler compaction misses?
- How does branch summarization in session trees transfer context across navigation events?

### Contrasting Approaches
- What does Pi share with Claude Code, GasTown, and Overstory—and where do they diverge?
- When does radical minimalism outperform batteries-included frameworks?
- What does the "no permission system" stance reveal about trust models in agent design?

---

## Your Mental Model

**Pi treats the agent as a platform, not a product.** Claude Code, GasTown, and Overstory each make strong commitments about how agents should coordinate, manage permissions, handle multi-agent communication, and track work. Pi makes almost none of these commitments. Instead, Pi provides the runtime substrate—lifecycle hooks, tool registration, message types, session persistence—and expects users to compose the behaviors they need through extensions.

**The inversion is deliberate and maintained under pressure.** Pi's documentation explicitly rejects features that other frameworks include by default: MCP protocol support, sub-agent spawning, permission popups, plan mode, to-do lists. The maintainer's rationale: these belong in extensions, installed by users who need them, not in the core that everyone carries. This produces a framework that starts smaller and grows to fit each use case rather than imposing a comprehensive structure upfront.

---

## Philosophy

Pi's design emerges from three interrelated convictions that distinguish it from conventional agent frameworks.

### Radical Minimalism in Core, Radical Extensibility at Edges

The core Pi agent handles: receiving user input, calling the LLM, executing tool calls, managing the session, and persisting state. Everything else—permissions, multi-agent coordination, plan modes, task tracking—is optional and lives in extensions.

This distribution produces a different scaling curve than batteries-included frameworks. A minimal Pi installation handles simple coding assistance with low overhead. A Pi installation with a curated extension stack matches or exceeds the capabilities of more opinionated frameworks, but only where needed. The framework grows to fit the task; the task does not adapt to the framework.

**Architectural consequence:** extensions can override built-in implementations. If Pi's default `read_file` tool behavior conflicts with an extension's needs, the extension registers its own `read_file` and the built-in is replaced. This override capability is essential to the model—without it, extensions become additive-only, unable to modify core behaviors that don't fit the use case.

### No Permission System by Design

Pi contains no permission prompts, no approval gates, no "do you want to allow this action?" dialogs. The maintainer's stated rationale: "the user launched the agent, the agent should do its job."

This stance reflects a trust model opposite to Claude Code's approach. Claude Code layers permission checking at multiple levels—tool restrictions, `allowedTools` configuration, MCP server boundaries, hook-based enforcement. Pi treats the launch decision as the permission grant. A user who installs an extension and runs Pi has implicitly authorized whatever that extension does.

The trade-off is explicit and documented: Pi's security warning is blunt—"only install extensions you trust." There is no sandboxing, no capability limitation at the runtime level. Trust is placed in the extension selection process rather than in runtime enforcement.

### Anti-Slop Contribution Policy

Pi's AGENTS.md establishes contribution requirements that apply equally to human developers and AI agents: contributors must "understand your code." The file prohibits `no any` types, `no git add -A`, `no git reset --hard`. The "First Message Protocol" requires the agent to read READMEs before asking questions.

The shared human-agent rules pattern means Pi's own development practices are enforced on the AI agents that work on Pi itself. This creates consistency between how humans and agents are expected to work—both follow the same file, both face the same constraints. The framework that enforces agent behavior externally also enforces it internally.

---

## Architecture

### Extension System

Extensions are TypeScript modules that hook into Pi's lifecycle event system. The extension API spans 44KB of type definitions and 63KB of documentation—larger than many complete frameworks—indicating the depth of composition the system supports.

Auto-discovery loads extensions from two locations:

```
.pi/extensions/           # Project-local extensions (checked in, team-shared)
~/.pi/agent/extensions/   # User-global extensions (personal tooling)
```

Both directories are scanned at startup. Project-local extensions take precedence over global ones when naming conflicts occur. This two-tier discovery mirrors how Claude Code separates project-level `.claude/` configuration from user-level `~/.claude/` settings.

### Lifecycle Hook System

The extension lifecycle spans nine named events, each representing a point where extensions can inject behavior:

```
session_start
    │
    ▼
input (user message arrives)
    │
    ▼
before_agent_start
    │
    ▼
┌───────────────────────────────────┐
│           TURN LOOP               │
│  turn_start                       │
│      │                            │
│      ▼                            │
│  tool_call  (for each tool call)  │
│      │                            │
│      ▼                            │
│  tool_result                      │
│      │                            │
│      ▼                            │
│  turn_end                         │
└───────────────────────────────────┘
    │
    ▼
agent_end
    │
    ▼
session_shutdown
```

Each hook point gives extensions the ability to intercept and modify behavior:

| Hook | Extension Capabilities |
|------|------------------------|
| `session_start` | Load state, configure tools, inject system context |
| `input` | Validate, transform, or enrich user messages |
| `before_agent_start` | Pre-flight checks, resource allocation |
| `turn_start` | Inject messages, set turn-level context |
| `tool_call` | Block calls, modify arguments, redirect to alternative tools |
| `tool_result` | Transform results, summarize large outputs, inject metadata |
| `turn_end` | Checkpoint state, analyze conversation |
| `agent_end` | Persist session artifacts, trigger follow-up actions |
| `session_shutdown` | Cleanup, metrics collection, final state save |

The `tool_call` hook deserves specific attention: extensions can **block** tool calls entirely. This is the mechanism through which permissions can be implemented—not in the core, but in a permission extension that intercepts tool calls and applies whatever policy the user configures.

### Tool Registration and Override

Extensions register tools using TypeBox schema validation—the same JSON Schema standard that TypeScript-first tools use for type safety:

```typescript
// Extension registering a custom tool with TypeBox schema
extension.registerTool({
  name: "read_file",                           // Overrides built-in if same name
  description: "Read file contents with audit logging",
  parameters: Type.Object({
    path: Type.String({ description: "File path to read" }),
    encoding: Type.Optional(Type.String({ default: "utf8" }))
  }),
  handler: async ({ path, encoding }) => {
    auditLog.record({ action: "read", path, timestamp: Date.now() });
    return fs.readFile(path, encoding ?? "utf8");
  }
});
```

The TypeBox schema provides both runtime validation (malformed tool calls fail fast) and IDE autocomplete (extension authors get type safety). The override mechanism means extensions can silently replace built-in behavior—the agent continues calling `read_file`, but now routes through the extension's implementation.

### AgentMessage vs LLM Message Separation

Pi maintains a strict boundary between its internal message representation and the LLM-compatible format:

```typescript
// Internal representation (rich, extension-aware)
type AgentMessage = {
  id: string;
  role: "user" | "assistant" | "tool";
  content: MessageContent[];
  metadata: Record<string, unknown>;     // Extensions can attach data
  branchId?: string;                     // For session tree navigation
  summary?: string;                      // For compacted branches
};

// Provider-compatible representation (minimal, standardized)
type Message = {
  role: "user" | "assistant";
  content: string | ContentBlock[];
};

// Conversion happens at the API boundary only
function convertToLlm(messages: AgentMessage[]): Message[] { ... }
```

This separation enables:

1. **Extension metadata**: Extensions attach arbitrary data to `AgentMessage` without polluting the LLM request
2. **Branch-aware summaries**: Session tree navigation attaches branch summaries as `AgentMessage` entries that convert to injected context
3. **Custom message types**: TypeScript declaration merging lets extensions add new content types to `AgentMessage` without modifying the core type

The conversion boundary is the architectural checkpoint—whatever internal complexity extensions add, the LLM sees a clean, provider-compatible message array.

### Provider Unification

Pi's `@mariozechner/llm-tools` package unifies 25+ LLM providers behind a single interface, with an auto-generated 325KB model registry covering pricing, context windows, and capabilities. The registry updates automatically as providers publish new models.

```typescript
// Type-safe model selection with IDE autocomplete
const response = await llm.complete({
  model: "claude-sonnet-4-6",    // Autocompleted from registry
  messages: conversationHistory,
  tools: registeredTools
});

// Serializable Context enables provider switching mid-conversation
const context = llm.serializeContext(conversationHistory);
const resumedResponse = await differentProvider.complete({
  model: "gpt-4o",
  context: context               // Conversation history transfers
});
```

The serializable `Context` object enables a capability that most frameworks lack: switching providers mid-conversation. A conversation started with Claude can continue with GPT-4o or a local vLLM instance. This portability reduces vendor lock-in and enables cost-optimization strategies (use cheaper models for routine turns, expensive models for reasoning-heavy steps).

---

## Context Management

### Compaction with Cumulative File Tracking

Pi's context compaction algorithm addresses a gap in most implementations: what happens when compaction occurs multiple times across a long session?

Compaction triggers when `contextTokens > contextWindow - reserveTokens`, where `reserveTokens` defaults to 16,384 and `keepRecentTokens` defaults to 20,000. The algorithm:

1. Walk backward through messages, preserving the most recent `keepRecentTokens`
2. Summarize everything before the preservation boundary
3. Replace summarized messages with a compact summary message
4. **Record** which files were read or modified in the compacted segment

The innovation is step 4. Pi maintains a cumulative index of file operations across the entire session, including across multiple compaction boundaries. After three compaction events, the agent still knows:

- Which files it has read (enabling "check if already cached" logic)
- Which files it has modified (enabling "review prior changes" context)
- The sequence of file interactions (enabling reasoning about change order)

Most compaction implementations discard this operational history. Pi's cumulative tracking means the agent can reason about its own past behavior even when the messages that recorded that behavior have been summarized away.

**Split turn handling:** A single LLM turn can exceed the compaction budget. Pi handles this edge case by detecting "split turns" (incomplete turns at the compaction boundary) and ensuring the split turn is either fully included or fully summarized—never split across the boundary. This prevents the corruption that occurs when compaction cuts a tool call from its result.

### Session Tree with Branch Summarization

Pi persists sessions as JSONL files with tree structure, enabling in-place branching:

```
Session JSONL structure:
├── Turn 1 (linear)
├── Turn 2 (linear)
├── Turn 3 [branch point]
│   ├── Branch A (turns 4A, 5A, 6A)  ← User navigated away from this
│   └── Branch B (turns 4B, 5B)      ← Current branch
└── Turn 7 (linear, continuing from Branch B)
```

Two commands manage the tree:
- `/tree` — Navigate to a different branch in the session history
- `/fork` — Create a new session file (independent history)

When navigating with `/tree` from Branch A to Branch B, Pi performs **branch summarization**: Branch A's content is summarized and injected as context into Branch B. The agent arriving in Branch B knows what happened in Branch A without carrying Branch A's full token cost.

This contrasts with linear session management (where navigating back discards forward history) and branching without transfer (where the new branch has no knowledge of the abandoned branch). Pi's approach preserves knowledge while managing token cost.

---

## Human Interaction Model

### Steering and Follow-up

Pi provides two mechanisms for mid-session human-to-agent intervention:

**`steer()`** — Interrupts the agent after the current tool call completes and injects user messages immediately. The agent stops whatever sequence it was planning and responds to the steering input. This enables course correction without waiting for the current response to finish.

**`followUp()`** — Queues messages for delivery after the current turn completes. The agent finishes its current reasoning, delivers its response, then receives the queued messages as the next input. This enables prepared input without interrupting ongoing reasoning.

Both mechanisms support two delivery modes:

| Mode | Behavior | Use Case |
|------|----------|----------|
| `"all"` | Deliver all queued messages at once | Batch corrections, structured input |
| `"one-at-a-time"` | Deliver one message, wait for response, repeat | Sequential conversation, interactive debugging |

The combination of steering (interrupt) and follow-up (queue) with two delivery modes gives users fine-grained control over the human-agent interaction tempo without requiring Pi to implement a permission system.

### No Permission System

Pi's absence of permission gates reflects a specific stance on where agent trust belongs. The comparison with Claude Code illustrates the contrast:

**Claude Code permission model:**
- `allowedTools` configuration restricts available tools
- Hook-based enforcement blocks disallowed operations
- Interactive prompts for risky operations
- MCP server boundaries as capability limits

**Pi permission model:**
- Extension installation is the permission grant
- `tool_call` hook enables extension-implemented restrictions
- No built-in prompts or blocks
- Trust placed in user's extension selection

Neither model is universally superior. Claude Code's layered permissions provide defense-in-depth appropriate for shared environments and junior-user scenarios. Pi's trust-at-launch model provides lower friction for expert users who want the agent to execute without interruption.

Pi's documentation acknowledges the trade-off without apology: "only install extensions you trust." The framework documents the security model honestly rather than obscuring it behind reassuring UI patterns.

---

## Contrasting Philosophy

The four frameworks compared here represent distinct positions on the core-vs-extension spectrum:

| Dimension | Pi | Claude Code | GasTown | Overstory |
|-----------|-----|------------|---------|-----------|
| **Core size** | Minimal runtime | Batteries-included | Custom Go system | TypeScript/Bun stack |
| **Permissions** | None (extension) | Layered (config + hooks + prompts) | None (GUPP principle) | Hook-based mechanical enforcement |
| **Multi-agent** | Not built in (extension) | Task tool + TeammateTool | Role hierarchy (Mayor/Polecat/Witness) | Session-as-orchestrator + worktrees |
| **Context management** | Compaction with file tracking + session trees | Native compaction | Git-backed seancing | Fresh context per worker |
| **Provider support** | 25+ unified via registry | Claude only | Claude only | Claude only |
| **Extensibility mechanism** | 20+ lifecycle hooks + tool override | Hooks (5 types) + MCP servers | Custom Go plugins | Overlay CLAUDE.md + custom tools |
| **Persistence** | JSONL session tree | SQLite projects | Git-backed molecules/convoys | SQLite mail + worktrees |
| **Trust model** | User installs extensions | Layered per-tool configuration | Agent self-activates on hook state | Hook-enforced mechanical constraints |
| **Language** | TypeScript | Go | Go | TypeScript/Bun |
| **Stars (approx.)** | ~15,900 | N/A (commercial) | ~9,000 | N/A (private) |
| **Multi-provider** | Yes (25+ providers) | No | No | No |

### Where Pi Wins

Pi's multi-provider support is unique among the four frameworks. Production deployments that need cost optimization (route routine turns to cheaper models), redundancy (failover from one provider to another), or evaluation (compare outputs across providers) cannot achieve this with Claude-only frameworks.

Pi's extension override capability is also distinctive. Claude Code hooks can block and redirect operations, but extensions registered via hooks cannot replace the core tool implementation. Pi extensions can. This enables deeper customization—an extension can replace `read_file` with a version that performs caching, auditing, or transformation that the built-in cannot do.

### Where Pi Faces Challenges

Multi-agent coordination is absent from Pi's core. GasTown provides a complete role taxonomy (Mayor/Polecat/Witness/Refinery) with persistent identity, work queues, and quality supervision. Overstory provides depth-limited hierarchical delegation with SQLite-backed messaging. Pi provides... hooks that let users build it themselves.

For teams that need production-grade multi-agent coordination out of the box, Pi's minimalism becomes a liability. The extension model means someone must build and maintain the coordination layer, and that layer must be correct—distributed coordination bugs are hard to diagnose and harder to fix.

Similarly, Pi's no-permission stance limits deployment in shared or enterprise contexts. An administrator cannot configure Pi to prevent agents from accessing sensitive files without writing a custom blocking extension. Claude Code's `allowedTools` and MCP boundaries provide this without extension authoring.

### Architectural Implications

The core-vs-extension divide reflects a deeper disagreement about who knows best. Batteries-included frameworks (Claude Code, GasTown, Overstory) make strong assumptions about what most users need and encode those assumptions in the core. Pi assumes the core cannot know and requires users to compose their needs.

Both stances produce successful systems. The choice depends on: whether the use case fits the framework's assumptions (batteries-included wins), whether the use case is unusual enough to need deep customization (minimal core wins), and whether users have the capability to write extensions (technical teams can leverage Pi's flexibility; non-technical users cannot).

---

## Skills and Progressive Disclosure

Pi's skill system implements a specific form of progressive disclosure that differs from Claude Code's skill loading:

**Claude Code approach:** Skills (slash commands) load when invoked. The system prompt doesn't include skill content until the user calls the command. Discovery happens through `/help` listing and documentation.

**Pi approach:** At startup, the system prompt includes only skill **names and descriptions**. When the agent determines a skill is relevant, it uses the `read` tool to load the full `SKILL.md` content on demand. Discovery happens through in-context awareness—the agent sees skill names in its initial context and decides when to expand them.

```
Pi Skill Discovery Flow:

startup:
  system_prompt includes:
    - Skill: "python-env-setup" — Set up Python virtual environments
    - Skill: "git-workflow" — Standard git branching and commit patterns
    - Skill: "docker-compose" — Multi-container development setup
    [names + descriptions only]

when relevant task appears:
  agent calls: read_tool("~/.pi/agent/skills/python-env-setup/SKILL.md")
  system_prompt now includes full skill content
  [description expanded to full instructions]
```

This design keeps the initial context window small regardless of how many skills are installed. A Pi installation with 50 skills loads only ~50 lines of names/descriptions at startup. Claude Code with 50 slash commands also keeps startup context small (commands load on invocation), but the discovery mechanism differs: Pi agents proactively decide to load skills based on task relevance; Claude Code users explicitly invoke commands.

The `disable-model-invocation` frontmatter flag in `SKILL.md` files controls whether Pi should auto-invoke a skill when it determines relevance, or only load it when explicitly requested. This gives skill authors control over how aggressively their skill participates in conversations.

---

## Pattern Mappings

Pi's implementation patterns map to several book chapters:

| Pi Pattern | Book Chapter | Relationship |
|-----------|-------------|-------------|
| Extension lifecycle hooks | [Tool Use: Tool Design](../../chapters/5-tool-use/1-tool-design.md) | Hook attachment points are a meta-tool design pattern |
| Tool override via same-name registration | [Tool Use: Tool Restrictions](../../chapters/5-tool-use/3-tool-restrictions.md) | Override enables restriction without framework modification |
| Progressive skill disclosure | [Patterns: Progressive Disclosure](../../chapters/6-patterns/7-progressive-disclosure.md) | Name+description at startup, full content on demand |
| Steering and follow-up | [Patterns: Human-in-the-Loop](../../chapters/6-patterns/6-human-in-the-loop.md) | Mid-stream intervention without permission gates |
| AgentMessage vs LLM Message boundary | [Context: Context Fundamentals](../../chapters/4-context/1-context-fundamentals.md) | Separation prevents internal state from polluting LLM input |
| Cumulative file tracking through compaction | [Context: Context Strategies](../../chapters/4-context/2-context-strategies.md) | Operational history survives context window reduction |
| Session tree with branch summarization | [Practices: Workflow Coordination](../../chapters/7-practices/5-workflow-coordination.md) | Branching enables non-linear exploration with preserved context |
| AGENTS.md shared human-agent rules | [Practices: Production Concerns](../../chapters/7-practices/4-production-concerns.md) | Unified rule system reduces human-agent behavior drift |
| No permission system (trust at launch) | [Mental Models: Pit of Success](../../chapters/8-mental-models/1-pit-of-success.md) | Permission gates as climb-to-success; trust-at-launch as fall-into-success |
| Multi-provider unified API | [Model: Multi-Model Architectures](../../chapters/3-model/4-multi-model-architectures.md) | Provider abstraction enables cost and capability optimization |

---

## Implications for Practitioners

### When Minimal Core Wins

Pi's architecture produces the best outcomes when:

- **Use cases are non-standard.** Teams building coding agents for embedded systems, game development, scientific computing, or other specialized domains find that batteries-included frameworks impose assumptions that don't fit. Pi's extension model lets these teams build exactly what they need.

- **Multi-provider is required.** Cost optimization across providers, evaluation pipelines that compare outputs, and resilience through provider failover all require provider abstraction that Claude-only frameworks cannot provide.

- **Expert user base.** Teams with TypeScript expertise can write and maintain extensions. The extension API is well-documented (63KB of docs), but extension authoring requires technical capability.

- **Unusual permission models.** Standard permission frameworks impose specific trust hierarchies. Pi lets teams build whatever trust model fits—role-based, project-scoped, user-identity-aware—through extensions.

### When Batteries-Included Wins

Pi's minimalism creates friction when:

- **Multi-agent coordination is needed immediately.** GasTown's Mayor/Polecat/Witness hierarchy and Overstory's session-as-orchestrator model provide working multi-agent coordination. Pi requires building this through extensions.

- **Non-technical users will deploy the system.** Pi's security model requires users to evaluate extension trustworthiness. Claude Code's layered permissions provide safer defaults for less technical users.

- **Standard workflows cover the use case.** For typical coding assistance, code review, and documentation tasks, Claude Code's built-in features eliminate extension authoring overhead.

### Learning from Pi's Architecture

Three structural insights transfer to agent designs that don't use Pi directly:

**1. Separate internal state from LLM input.** Pi's `AgentMessage` vs `Message[]` separation enables richer internal bookkeeping without token cost. Practitioners building custom agents benefit from the same boundary—track metadata internally, send clean messages to the LLM.

**2. Track operational history through compaction.** Most compaction implementations discard operational context when summarizing. Pi's cumulative file index shows that specific high-value information (which files were touched) can survive compaction even when conversation content is compressed. Practitioners designing long-running agents should identify what operational facts must survive compaction and build explicit tracking for those facts.

**3. Design the extension boundary explicitly.** Pi's success comes partly from clear decisions about what belongs in the core versus in extensions. Practitioners building internal agent frameworks benefit from making this boundary explicit rather than allowing it to grow organically—the boundary determines what users can customize, what they must accept, and what the framework can guarantee about behavior.

---

## Open Questions

- How do extension conflicts get resolved when two extensions attempt to override the same tool with incompatible implementations?
- What does Pi's skill progressive disclosure reveal about optimal system prompt sizing—is there a measurable token cost at which expanding skill content improves outcomes?
- Can Pi's multi-provider portability survive provider-specific features (reasoning tokens, extended thinking, tool use formats) without diverging from a unified interface?
- How does cumulative file tracking perform as sessions extend to hundreds of file operations—does the index itself become a context cost?
- What would a permission extension look like that achieves Claude Code's layered protection without requiring users to write custom enforcement logic?
- Does the "no idle state" principle (from Gas Town's GUPP and Overstory's propulsion principle) apply to Pi sessions, or does Pi's trust-at-launch model require a different stance on agent forward motion?

---

## Connections

- **[Gas Town: Multi-Agent Workspace Manager](../gastown/_index.md)**: Gas Town and Pi share the no-permission stance (GUPP's "agent should do its job" mirrors Pi's "user launched the agent"), but implement it through entirely different mechanisms—Gas Town through hook-based self-activation, Pi through trust at extension installation.

- **[Overstory: Session-as-Orchestrator](../overstory/_index.md)**: Overstory and Pi share TypeScript/Bun heritage and zero-dependency architecture philosophy. Overstory adds multi-agent coordination that Pi omits; Pi adds multi-provider support that Overstory lacks.

- **[Progressive Disclosure Pattern](../../chapters/6-patterns/7-progressive-disclosure.md)**: Pi's skill system implements progressive disclosure at the context level—skill names in startup context, full content loaded on demand—demonstrating the pattern through a different mechanism than step-by-step agent capabilities.

- **[Context Strategies](../../chapters/4-context/2-context-strategies.md)**: Pi's cumulative file tracking through compaction events extends standard compaction patterns with operational history preservation, relevant to practitioners designing long-running agents that must reason about their own prior work.

- **[Tool Design](../../chapters/5-tool-use/1-tool-design.md)**: Pi's extension system—where tools are registered with TypeBox schemas and can override built-in implementations—represents a meta-tool design pattern where the tool registration mechanism itself is the primary API surface.

- **[Human-in-the-Loop](../../chapters/6-patterns/6-human-in-the-loop.md)**: Pi's `steer()` and `followUp()` primitives operationalize human-in-the-loop without permission gates, demonstrating that HITL coordination does not require blocking approval flows.
