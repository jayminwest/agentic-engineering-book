---
title: Claude Code
description: Anthropic's CLI coding agent with subagents, Skills, and hooks
created: 2025-12-08
last_updated: 2026-02-05
tags: [tools, claude-code, coding-agent, anthropic, slash-commands, expert-pattern, skills, orchestration, tool-restriction, hooks, sandbox, security, multi-agent, teammatetool, coordination, agent-teams, session-memory, memory-management, persistent-memory, rules, path-scoping]
part: 3
part_title: Perspectives
chapter: 9
section: 1
order: 3.9.1
---

# Claude Code

Anthropic's official coding agent. A primary tool for agentic software development, offering subagent orchestration, progressive disclosure via Skills, and extensive customization through hooks.

---

## Mental Model for Claude Code

**Claude Code is a programmable orchestrator, not just a chatbot that writes code.** The value comes from configuring it as a multi-agent system where specialized subagents handle specific concerns. Hooks provide observability into agent behavior, Skills provide just-in-time expertise, and tool restrictions create forcing functions for better delegation.

The shift from "assistant" to "system" thinking changes how you use it—instead of asking Claude to do everything, design the environment where Claude coordinates specialists.


---

## Connections

- **To [Prompt](../2-prompt/_index.md):** Skills and slash commands demonstrate model-invoked vs user-invoked prompt patterns. See [Model-Invoked vs. User-Invoked Prompts](../2-prompt/_index.md#model-invoked-vs-user-invoked-prompts) for design trade-offs.
- **To [Context](../4-context/_index.md):** Skills implement the [Progressive Disclosure pattern](../4-context/3-context-patterns.md#progressive-disclosure-pattern) for context management
- **To [Context Fundamentals](../4-context/1-context-fundamentals.md#context-vs-memory):** Session memory provides persistent memory across sessions, addressing the ephemeral nature of context windows. The .claude/rules/ system enables path-scoped context injection.
- **To [Cost and Latency](../7-practices/3-cost-and-latency.md):** Token cost models for different feature types—understanding the economics of tools, Skills, subagents, and MCP
- **ReAct Loop:** How does Claude Code implement the ReAct pattern?
- **Human-in-the-Loop:** How do you stay in the loop while using Claude Code?

---

## Tips & Tricks

*Specific techniques that work well:*

*[2025-12-08]*: **The Three-Command Expert Pattern (Plan-Build-Improve)** - This pattern creates a learning triangle where production experience feeds back into prompts through mutable Expertise sections. Each expert has three slash commands: Plan (creates specifications), Build (implements the spec), and Improve (analyzes recent work and updates the Expertise sections in Plan and Build commands). The Improve command closes the loop by extracting learnings from actual production usage and updating the expert's knowledge base, making each iteration smarter than the last. This creates a self-improving system where the prompts themselves evolve based on real-world experience.

*[2026-01-11]*: **Real-Time Message Steering** - Claude Code 2.1.0 introduced the ability to send messages while Claude works, fundamentally changing the interaction model from "fire and wait" to "fire and steer."

**How it works:** Type and send messages during active execution. Claude receives the input and can adjust its approach mid-task—correcting misunderstandings, refining direction, or pivoting focus without waiting for completion.

**Use cases:**
- **Mid-task corrections:** "Actually, focus on the authentication module first"
- **Scope refinement:** "Skip the tests for now, just get the implementation working"
- **Context injection:** "The config file moved to /etc/app.conf last week"
- **Priority shifts:** "This is blocking, handle error cases before edge cases"

**Pattern: Progressive Refinement**

Rather than crafting a perfect initial prompt, start with a direction and refine as Claude works:

1. Issue broad instruction: "Refactor the payment module for testability"
2. Observe initial approach in real-time
3. Steer as needed: "Good direction, but preserve the async interface"
4. Continue steering: "Focus on the PaymentProcessor class specifically"

This pattern reduces upfront prompt engineering effort and enables adaptation based on what Claude actually does, not what was anticipated.

**Contrast with backgrounding:** Real-time steering requires the task to remain in foreground. Backgrounded tasks (Ctrl+B) run without steering capability—use backgrounding for well-defined tasks that need no mid-flight adjustment.

**Sources:** [Claude Code Changelog 2.1.0](https://code.claude.com/docs/en/changelog)

*[2026-01-11]*: **Clickable File Paths (OSC 8)** - Claude Code 2.1.2 added OSC 8 hyperlink support, making file paths in tool output clickable in supported terminals (iTerm2, WezTerm, Ghostty, Kitty, and others).

**What this means:** When Claude references files in output, paths render as clickable links that open directly in the default editor or file browser.

**Requirements for clickable paths:**
1. Terminal must support OSC 8 escape sequences
2. File paths must be absolute (relative paths don't resolve reliably)
3. Files must exist at the referenced location

**Practical implication:** Agents and skills that emit file paths should prefer absolute paths to enable clicking:

```markdown
# Less useful (not clickable)
See the config in ./config/app.yaml

# More useful (clickable in OSC 8 terminals)
See the config in /Users/dev/project/config/app.yaml
```

**Configuration tip:** Agents designed for navigation-heavy workflows benefit from instructions to emit absolute paths:

```markdown
When referencing files, always use absolute paths to enable terminal linking.
```

**Supported terminals:** iTerm2, WezTerm, Ghostty, Kitty, Windows Terminal, and others implementing the OSC 8 standard. Verify support in terminal documentation.

**Sources:** [Claude Code Changelog 2.1.2](https://code.claude.com/docs/en/changelog)

---

## Keyboard Customization

*[2026-01-17]*: Claude Code 2.1.7 added customizable keybindings for power-user workflows.

### Default Bindings

| Key | Action |
|-----|--------|
| `Ctrl+K` | New conversation |
| `Ctrl+Shift+L` | Clear context |
| `Ctrl+B` | Background task |
| `Ctrl+/` | Command palette |
| `Escape` | Cancel current operation |

### Custom Bindings

Configure in `~/.claude/keybindings.json`:

```json
{
  "bindings": [
    {
      "key": "ctrl+shift+b",
      "command": "background-task",
      "description": "Run current task in background"
    },
    {
      "key": "ctrl+shift+r",
      "command": "/review:clarity",
      "description": "Quick clarity review"
    }
  ]
}
```

### Power-User Patterns

- **Bind frequently-used slash commands** to muscle memory shortcuts
- **Create keyboard macros** for multi-step workflows
- **Organize by frequency**: most-used on easiest keys
- **Consider domain-specific bindings** for specialized work

The ability to bind slash commands directly to key combinations eliminates the friction of typing command names, enabling faster workflow execution for repeated patterns.

**Sources:** [Claude Code Changelog 2.1.7](https://code.claude.com/docs/en/changelog)

---

## Subagent System

*[2025-12-09]*: Claude Code's subagent system (part of the Claude Agent SDK, renamed from "Claude Code SDK" in late 2025) is the primary mechanism for multi-agent coordination. Understanding it is essential for building effective multi-agent workflows.

### Core Concepts

**Context Isolation**: Each subagent maintains a separate context window from the orchestrator. They return only synthesized, relevant information—not their full context. This prevents context pollution and enables parallel information processing without bloating the orchestrator's context.

**Parallelization**: Subagents can run truly concurrently when invoked via multiple Task tool calls in a single message. This is the foundation of parallel multi-agent workflows.

### Nesting Constraint and Workarounds

*[2025-12-11]*: Claude Code's Task tool is unavailable to subagents—they cannot spawn nested subagents. This is a hard architectural constraint as of December 2024, though the limitation will likely evolve as the tooling matures.

**Why the constraint exists**:
- Prevents infinite recursion scenarios where agents spawn agents indefinitely
- Avoids token budget runaway in deeply nested delegation chains
- Simplifies the execution model and debugging surface
- Keeps coordination patterns explicit rather than emergent

This creates a flat orchestration model: one HEAD agent coordinates multiple subagents, but subagents are leaf nodes that cannot delegate further.

**Workaround 1: Subprocess spawning via `claude -p`**

Subagents can invoke `claude -p <prompt>` through the Bash tool to spawn independent Claude Code processes:

```python
# In a subagent that needs to spawn another agent
result = bash("""
claude -p "agent-name" <<EOF
Perform specific task with inputs
EOF
""")
```

**Trade-offs**:
- **Gain**: Enables multi-level delegation when architecturally necessary
- **Cost**: Loses visibility—subprocess agents run in separate sessions
- **Cost**: No access to parent session state or shared context
- **Cost**: Adds 10+ second subprocess startup overhead per spawn
- **Cost**: Subprocess billing is separate (may complicate cost tracking)

**Workaround 2: MCP-based agent spawning**

Create an MCP server that exposes agent spawning as a tool. This maintains observability within the Claude Code session while enabling nested delegation:

```yaml
# MCP server definition exposing spawn_agent tool
tools:
  - name: spawn_agent
    description: Spawn a specialized subagent for delegation
    parameters:
      agent_name: string
      task: string
      context: object
```

Subagents invoke the MCP tool, which handles spawning and returns results within the session's execution context.

**Trade-offs**:
- **Gain**: Maintains session visibility and cost tracking
- **Gain**: Reusable across projects via MCP protocol
- **Cost**: Requires implementing and running an MCP server
- **Cost**: Additional architectural complexity
- **Cost**: MCP server itself must manage agent lifecycles

**Workaround 3: Document-driven coordination**

Instead of nested spawning, use shared document state for multi-level coordination:

1. Subagent writes task specification to a file (e.g., `tasks/pending/task-123.md`)
2. Subagent reports completion to orchestrator
3. Orchestrator reads pending tasks and spawns appropriate specialists
4. Specialists write results back to shared state

**Trade-offs**:
- **Gain**: Fully decoupled—no nesting required
- **Gain**: Natural audit trail via filesystem artifacts
- **Gain**: Enables asynchronous multi-agent workflows
- **Cost**: Requires explicit coordination protocol
- **Cost**: Orchestrator must poll or be notified of pending work
- **Cost**: Higher latency—tasks complete in subsequent orchestrator turns

**Future direction**: Nested spawning is a natural evolution as agent systems mature. The current constraint reflects Claude Code's focus on preventing runaway behaviors in early adoption. Expect capabilities to expand, possibly with:
- Depth limits on nesting (e.g., max 2-3 levels)
- Token budget controls across nested chains
- Enhanced observability for nested execution trees

For now, design architectures that work within the flat model. The workarounds exist for edge cases but should not be the default pattern.

**See Also**:
- [Orchestrator Pattern](../6-patterns/3-orchestrator-pattern.md) — Design patterns for flat orchestration
- [Multi-Agent Context](../4-context/4-multi-agent-context.md) — Managing context across agent boundaries

---

## Agent Teams: Native Multi-Agent Coordination (Experimental)

*[2026-02-05]*: Claude Code ships with agent teams, a multi-agent coordination layer enabling peer-to-peer messaging and shared task lists. This feature is experimental and disabled by default.

### Prerequisites

Agent teams are experimental and disabled by default. Enable them before use:

**Option 1: Environment Variable**
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

**Option 2: settings.json**
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Verification**: Start Claude Code. If agent teams are enabled, the team coordination tools become available.

**Warning**: Agent teams have known limitations around session resumption, task coordination, and shutdown behavior. See [Availability and Status](#availability-and-status) for details.

### Your Mental Model

**Agent teams enable peer-to-peer coordination between Claude Code sessions, unlike subagents which only report back.** Subagents (Task tool) run within a single session and return results to the caller. Agent teams (TeammateTool) run as independent Claude Code sessions that can message each other directly, share a task list, and coordinate without going through the lead.

### Core Capabilities

**Coordination Tools:**
- **SpawnTeam**: Create team with shared task list and messaging system
- **Spawn**: Create new teammate agents with role definitions
- **SendMessage**: Direct message to specific teammate OR broadcast to all
- **Shutdown**: Request teammate graceful exit (teammate can approve/reject)
- **Cleanup**: Remove shared team resources (requires all teammates stopped)

**Display Modes:**
- **in-process**: All teammates in main terminal (Shift+Up/Down to select, works any terminal)
- **split-pane**: Each teammate gets own pane (tmux or iTerm2, see all at once)

**Keyboard Controls (in-process mode):**
- `Shift+Up/Down`: Select teammate
- `Enter`: View teammate's session
- `Escape`: Interrupt teammate's current turn
- `Ctrl+T`: Toggle task list
- `Shift+Tab`: Cycle into delegate mode (lead coordination-only)

**Team Structure:**
- Team lead: Main Claude Code session that creates the team
- Teammates: Independent Claude Code instances
- Task list: Shared work items (pending/in-progress/completed states with dependencies)
- Mailbox: Messaging system for inter-agent communication

**Storage Locations:**
- Team config: `~/.claude/teams/{team-name}/config.json`
- Task list: `~/.claude/tasks/{team-name}/`
- Messages: Delivered automatically (no polling required)

### Getting Started

**1. Enable agent teams**:
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

**2. Create a team naturally**:
```
I'm designing a CLI tool that helps developers track TODO comments across
their codebase. Create an agent team to explore this from different angles: one
teammate on UX, one on technical architecture, one playing devil's advocate.
```

Claude creates the team, spawns teammates, coordinates work, and attempts cleanup when finished.

**3. Choose display mode**:

Default is `"auto"` (split panes if in tmux, in-process otherwise). Override in settings.json:
```json
{
  "teammateMode": "in-process"  // or "tmux"
}
```

Or per-session:
```bash
claude --teammate-mode in-process
```

**4. Control the team**:

Tell the lead in natural language what you want. It handles coordination, task assignment, delegation.

**5. Interact directly with teammates**:

- **In-process**: Shift+Up/Down to select, type to message
- **Split-pane**: Click into teammate's pane

### Advanced Features

**Delegate Mode:**

Press `Shift+Tab` to restrict the lead to coordination-only tools (spawning, messaging, shutting down teammates, managing tasks). The lead cannot implement tasks directly—forces pure orchestration role.

Use when: Lead should focus entirely on breaking down work, assigning tasks, and synthesizing results without touching code.

**Plan Approval for Teammates:**

Require teammates to plan before implementing. Teammate works in read-only mode until lead approves approach.

```
Spawn an architect teammate to refactor the authentication module.
Require plan approval before they make any changes.
```

When teammate finishes planning, it sends approval request to lead. Lead reviews and either approves (teammate proceeds) or rejects with feedback (teammate revises and resubmits).

**Influence lead's judgment**: Give approval criteria in prompt ("only approve plans that include test coverage").

**Task Dependencies:**

Tasks can depend on other tasks. Pending tasks with unresolved dependencies cannot be claimed until dependencies complete. System handles dependency resolution automatically.

**Task Claiming:**

- **Lead assigns**: Explicitly assign task to teammate
- **Self-claim**: After finishing, teammate picks up next unassigned, unblocked task automatically

Task file locking prevents race conditions when multiple teammates claim simultaneously.

### Coordination Patterns Enabled

Agent teams enable five coordination patterns documented in the implementation:

**1. Lead-Teammate**

```
┌──────────┐
│   Lead   │
└─────┬────┘
      │ Spawn N teammates
      ├─────────┬─────────┬─────────┐
      ▼         ▼         ▼         ▼
  Teammate1 Teammate2 Teammate3 Teammate4
      │         │         │         │
      │ Send results back to lead     │
      └─────────┴─────────┴─────────┘
                  │
                  ▼ Join (wait for all)
             ┌──────────┐
             │   Lead   │
             │ Synthesis│
             └──────────┘
```

Lead spawns N teammates, distributes work via SendMessage, collects results via Join. Common for parallel data processing, batch operations, or distributed search.

**2. Swarm**

```
┌──────────────────────────────────┐
│         Broadcast (all)          │
└──────────┬───────────────────────┘
           │
    ┌──────┼──────┬──────┬──────┐
    ▼      ▼      ▼      ▼      ▼
  Peer1  Peer2  Peer3  Peer4  Peer5
    │      │      │      │      │
    │  Local rules + emergent behavior  │
    └──────┴──────┴──────┴──────┘
```

N peers spawned simultaneously, Broadcast coordinates, emergent behavior from local rules. No central coordinator after initialization; agents self-organize based on shared state.

**3. Pipeline**

```
Agent A ──SendMessage──> Agent B ──SendMessage──> Agent C ──SendMessage──> Agent D
  │                        │                        │                        │
  └─── Process ────────────└─── Transform ──────────└─── Enrich ────────────└─── Finalize
```

Sequential processing where each agent completes before next begins. Agent A messages Agent B, B processes and messages C, etc. Classic ETL pattern for multi-stage transformations.

**4. Council**

```
         ┌──────────────┐
         │   Arbiter    │
         └──────┬───────┘
                │ Broadcast question
       ┌────────┼────────┬────────┐
       ▼        ▼        ▼        ▼
   Security  Perf   UX      Legal
   Expert   Expert Expert  Expert
       │        │        │        │
       │   Send responses back    │
       └────────┴────────┴────────┘
                │
                ▼ Join + synthesize
         ┌──────────────┐
         │   Arbiter    │
         │  Final rec   │
         └──────────────┘
```

Spawn domain experts, Broadcast question, collect responses, synthesize with arbiter. Enables multi-perspective analysis without requiring experts to coordinate directly.

**5. Plan Approval (HITL)**

```
┌──────────┐
│ Planner  │
└─────┬────┘
      │ Generate spec
      ▼
┌──────────────┐
│ Plan Approval│ ◄── Human reviews
│   (blocks)   │
└──────┬───────┘
       │ Approved
       ▼
┌──────────┐
│ Builder  │
└──────────┘
```

Planning agent generates spec, Plan Approval tool blocks execution until human reviews, execution proceeds after approval. Implements human-in-the-loop gate as a first-class coordination primitive.

### Official Use Cases

Agent teams excel at:

1. **Research and review**: Multiple teammates investigate different aspects simultaneously, then share and challenge findings
2. **New modules or features**: Teammates each own separate piece without stepping on each other
3. **Debugging with competing hypotheses**: Test different theories in parallel, converge faster
4. **Cross-layer coordination**: Changes spanning frontend, backend, tests—each owned by different teammate

**Anti-patterns**:
- Sequential tasks (use single session or subagents)
- Same-file edits (causes overwrites)
- Work with many dependencies (coordination overhead exceeds benefit)

**Sources**: Official docs "When to use agent teams", "Best practices"

### Agent Teams vs Subagents

|                   | Subagents (Task tool)                    | Agent teams (TeammateTool)                  |
| :---------------- | :--------------------------------------- | :------------------------------------------ |
| **Context**       | Own context; results return to caller    | Own context; fully independent              |
| **Communication** | Report results back to main agent only   | Teammates message each other directly       |
| **Coordination**  | Main agent manages all work              | Shared task list with self-coordination     |
| **Best for**      | Focused tasks where only result matters  | Complex work requiring discussion and collaboration |
| **Token cost**    | Lower: results summarized back to main   | Higher: each teammate is separate Claude instance |

**Sources**: Official docs comparison table

### Best Practices

**Give teammates enough context:**

Teammates load CLAUDE.md, MCP servers, and skills automatically, but not lead's conversation history. Include task-specific details in spawn prompt:

```
Spawn a security reviewer teammate with the prompt: "Review the authentication module
at src/auth/ for security vulnerabilities. Focus on token handling, session
management, and input validation. The app uses JWT tokens stored in
httpOnly cookies. Report any issues with severity ratings."
```

**Size tasks appropriately:**

- **Too small**: Coordination overhead exceeds benefit
- **Too large**: Teammates work too long without check-ins, increasing wasted effort risk
- **Just right**: Self-contained units producing clear deliverable (function, test file, review)

**Tip**: If lead isn't creating enough tasks, ask it to split work into smaller pieces. Having 5-6 tasks per teammate keeps everyone productive and enables work reassignment if someone stuck.

**Wait for teammates to finish:**

If lead starts implementing instead of waiting:
```
Wait for your teammates to complete their tasks before proceeding
```

**Start with research and review:**

If new to agent teams, start with tasks having clear boundaries that don't require code writing: reviewing PR, researching library, investigating bug. Shows parallel exploration value without coordination challenges.

**Avoid file conflicts:**

Two teammates editing same file leads to overwrites. Break work so each teammate owns different set of files.

**Monitor and steer:**

Check in on progress, redirect approaches not working, synthesize findings as they come. Letting team run unattended too long increases wasted effort risk.

**Sources**: Official docs "Best practices" section

### Troubleshooting

**Teammates not appearing:**
- In in-process mode: Press `Shift+Down` to cycle through active teammates
- Check task complexity warranted team (Claude decides based on task)
- For split panes: Verify tmux installed (`which tmux`) or iTerm2 + `it2` CLI available

**Too many permission prompts:**
Pre-approve common operations in permission settings before spawning teammates to reduce interruptions.

**Teammates stopping on errors:**
Check output (Shift+Up/Down or click pane), then either:
- Give additional instructions directly
- Spawn replacement teammate to continue work

**Lead shuts down before work done:**
Tell lead to keep going. Can also instruct lead to wait for teammates to finish before proceeding.

**Orphaned tmux sessions:**
If tmux session persists after team ends:
```bash
tmux ls
tmux kill-session -t <session-name>
```

**Sources**: Official docs "Troubleshooting" section

### Availability and Status

**Experimental Feature:**

Agent teams are disabled by default as of Claude Code 2.1.x. Enable via:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Or in `settings.json`:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Known Limitations** (as of 2026-02-05):

- **No session resumption with in-process teammates**: `/resume` and `/rewind` do not restore teammates. After resuming, lead may message teammates that no longer exist. Workaround: Spawn new teammates.
- **Task status can lag**: Teammates sometimes fail to mark tasks completed, blocking dependent tasks. Workaround: Check if work done and update task status manually or nudge teammate.
- **Shutdown can be slow**: Teammates finish current request/tool call before shutting down (can take time).
- **One team per session**: Lead can only manage one team at a time. Clean up current team before starting new one.
- **No nested teams**: Teammates cannot spawn their own teams or teammates. Only lead manages team.
- **Lead is fixed**: Session that creates team is lead for its lifetime. Cannot promote teammate to lead.
- **Permissions set at spawn**: All teammates start with lead's permission mode. Can change individual modes after spawning, but cannot set per-teammate modes at spawn time.
- **Split panes require tmux or iTerm2**: Default in-process mode works any terminal. Split-pane not supported in VS Code integrated terminal, Windows Terminal, Ghostty.

**Sources**: Official docs "Limitations" section

### When to Use Agent Teams vs Subagents

**Use subagents (Task tool) when:**

- Simple delegation suffices (spawn, wait, return)
- Agents work independently without coordination
- Official support and stability required
- Minimal coordination complexity preferred

**Use agent teams (TeammateTool) when:**

- Teammates need to message each other during execution
- Selective waiting (Join specific agents, not all)
- Human approval gates required (Plan Approval)
- Visual debugging needed (tmux, iTerm2 backends)
- Exploring cutting-edge coordination patterns

**Decision Framework:**

```
Do teammates need to message each other during execution? ──No──> Use subagents (Task tool) (simpler)
                │
               Yes
                │
Is human approval required? ──Yes──> Use agent teams (TeammateTool) (Plan Approval)
                │
               No
                │
Do agents run sequentially? ──Yes──> Consider Pipeline pattern
                │
               No
                │
Multiple perspectives needed? ──Yes──> Consider Council pattern
                │
               No
                │
Default: Use agent teams (TeammateTool) ──> Enable richer coordination
```

**Expert Swarm Variant**: Council pattern combines with expertise inheritance when domain experts coordinate. Each council member (agent teams teammate) inherits domain expertise.yaml via path-passing, enabling consistent multi-perspective analysis. See [Expert Swarm Pattern](../../6-patterns/8-expert-swarm-pattern.md) for combining orchestration with expertise consistency.

### Open Questions

**From previous analysis:**
- Which coordination operations beyond the 8 identified exist in the codebase?
- Can agent teams teammates spawn nested teams? (Official docs say no as of 2026-02-05)
- How does file-based messaging scale to 10+ teammates writing concurrently?
- What happens when inboxes fill—overflow behavior, message ordering guarantees?
- Will agent teams remain CLI-specific or migrate to Claude Agent SDK generally?
- How does Plan Approval integrate with the hooks system?
- What happens if a teammate crashes while holding unprocessed messages?
- Do backends (tmux, iTerm2) affect coordination semantics or just observability?
- What monitoring/observability tools exist for team coordination beyond filesystem inspection?

**From official docs:**
- How does task dependency resolution handle circular dependencies?
- What happens when teammate rejects shutdown request? (Official docs mention this capability)
- Can display modes be changed mid-session, or only at startup?
- How do permission prompts from teammates surface to lead in split-pane mode?
- What is the maximum recommended team size before coordination overhead dominates?
- How does lead's conversation context affect teammate spawning heuristics?

### Sources

- [Official Claude Code Agent Teams Documentation](https://code.claude.com/docs/en/agent-teams) (2026-02-05)
- [Claude Code codebase analysis](https://github.com/anthropic-ai/claude-code) (TeammateTool implementation)
- [Claude Agent SDK rename announcement](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk) (late 2025)
- Filesystem observations of `~/.claude/teams/` and `~/.claude/tasks/` structure

---

### Defining Subagents

Two approaches:

**Filesystem Definition** (`.claude/agents/*.md`):
```yaml
---
name: code-reviewer
description: Security-focused code review specialist
tools: [Read, Grep, Glob]
model: sonnet
---

You are a security-focused code reviewer. Analyze code for vulnerabilities,
code smells, and deviation from best practices.
```

Project-level agents go in `.claude/agents/`, user-level in `~/.claude/agents/`.

**Programmatic Definition** (SDK):
```typescript
const result = query({
  prompt: "Review the authentication module",
  options: {
    agents: {
      'code-reviewer': {
        description: 'Security-focused code review specialist',
        prompt: 'You are a security-focused code reviewer...',
        tools: ['Read', 'Grep', 'Glob'],
        model: 'sonnet'
      }
    }
  }
});
```

Programmatic definitions take precedence when names conflict.

### Tool Restrictions as Security Boundaries

Configure tool access per subagent to follow least-privilege principles:

| Subagent Role | Appropriate Tools | Rationale |
|---------------|-------------------|-----------|
| Reviewer/Analyzer | Read, Grep, Glob | Read-only, can't modify files |
| Test Runner | Bash, Read, Grep | Execute tests, read results |
| Builder/Implementer | Read, Edit, Write, Grep, Glob | Full modification access |
| Orchestrator | Task, Read, Glob | Routes work, minimal direct access |

This is production IAM thinking applied to agents: deny-all by default, allowlist only what's necessary.

For MCP (Model Context Protocol) tools, the same pattern applies with extended naming: `mcp__<server>__<tool>`. See [Tool Use: MCP Tool Declarations](../5-tool-use/3-tool-restrictions.md#mcp-tool-declarations-in-frontmatter) for the full pattern.

### Proactive Spawning

To encourage proactive subagent use, include phrases like "use PROACTIVELY" or "MUST BE USED" in the subagent description field. Native spawning eliminates subprocess overhead—agent creation is near-instantaneous rather than 10+ seconds.

**Sources**: [Subagents in the SDK - Claude Docs](https://platform.claude.com/docs/en/agent-sdk/subagents), [Building agents with the Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk), [Best practices for Claude Code subagents](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)

### Agent Template Standardization for Rapid Domain Creation

*[2026-02-05]*: As agent domain count scales (11 domains in this book), template-driven generation accelerates new domain creation while maintaining consistency.

**The pattern:**

Instead of manually authoring 4 agents + expertise.yaml for each domain, maintain standardized templates:

```
.claude/templates/
├── base-agent.md            # Common frontmatter + structure
├── plan-agent.md            # Specification creation template
├── build-agent.md           # Implementation template
├── improve-agent.md         # Learning analysis template
├── question-agent.md        # Q&A interface template
└── expertise-template.yaml  # 9-section schema
```

**Template parameterization:**

Templates use placeholders for domain-specific values:

```yaml
---
name: {{DOMAIN}}-plan-agent
description: Plans {{DOMAIN}} operations. Expects: USER_PROMPT (requirement), HUMAN_IN_LOOP (optional)
tools: Read, Glob, Grep, Write
model: sonnet
color: yellow
---

# {{DOMAIN}} Expert - Plan

You are a {{DOMAIN}} Expert specializing in analysis and planning.

## Instructions

- Read expertise from `.claude/agents/experts/{{DOMAIN}}/expertise.yaml`
- Apply {{DOMAIN}}-specific patterns
- Create specification in `.claude/.cache/specs/{{DOMAIN}}/`
```

**Generation workflow:**

1. Define new domain: `domain=api-security`
2. Substitute placeholders: `{{DOMAIN}}` → `api-security`
3. Customize expertise.yaml sections for domain
4. Generate 4 agent files + expertise.yaml
5. Update /do routing to include new domain

**Time savings:**

| Approach | Time Investment | Consistency |
|----------|----------------|-------------|
| **Manual authoring** | 4-6 hours per domain | Variable (drift over time) |
| **Template generation** | 30-60 minutes per domain | High (templates enforce structure) |

**What templates standardize:**

- Frontmatter structure (name, description, tools, model, color)
- Workflow sections (Understand → Apply → Formulate → Report)
- expertise.yaml schema (9 sections with consistent format)
- Cross-references between agents
- File naming conventions

**When templates accelerate work:**

- Adding 3+ new domains (upfront template investment pays off)
- Onboarding new contributors (templates are scaffolding)
- Maintaining consistency across large expert systems (11+ domains)

**When manual authoring makes sense:**

- First 2-3 domains (establishing patterns organically)
- Highly specialized domains requiring custom structure
- One-off expert agents (not part of domain system)

**Registry integration:**

Template-generated frontmatter ensures registry compatibility—all agents have machine-readable metadata for capability indexing. See [Agent Registry Pattern](../../6-patterns/2-self-improving-experts.md#agent-registry-pattern).

**Implementation note:** This book's 11 expert domains were created through iterative standardization (commits 39e7904, 5002a1c), not upfront templating. Templates capture the emerged pattern for future domains.

**Sources:** 4-agent pattern standardization commits (39e7904), bulk expertise updates (5002a1c), advanced .claude/ implementation template analysis.

---

## Skills System

*[2025-12-09]*: Claude Code's Skills system enables progressive disclosure of procedural knowledge—teaching agents specialized reasoning patterns without upfront context cost.

### What Skills Are

Skills are procedural knowledge modules stored as `SKILLS.md` files in `.claude/skills/` directories. Unlike subagents (which delegate work) or slash commands (which require explicit invocation), Skills activate autonomously when Claude's reasoning determines relevance.

Each skill consists of:
- **YAML frontmatter**: Name, description (~50-200 chars), optional tool restrictions
- **Markdown instructions**: Detailed guidance (500-5,000 words)
- **Optional resources**: Supporting files in `scripts/`, `references/`, `assets/`

Skills use three-tier loading for context efficiency (see [Progressive Disclosure pattern](../4-context/3-context-patterns.md#progressive-disclosure-pattern)):
1. **Metadata** loads into every prompt (descriptions only)
2. **Full instructions** load when the model determines relevance
3. **Supporting resources** load on-demand when referenced

### Model-Invoked Design

Skills represent a fundamental shift in how agents access specialized knowledge. The model decides when to activate a skill based on the task at hand—no explicit command required.

**Comparison**:
- **Tools** (function calling): *What* the agent can do—read files, run commands, call APIs
- **Skills**: *How* the agent should reason about specialized domains—temporary behavioral specialization
- **Subagents**: *Who* does the work—delegate to isolated agents with separate contexts
- **Slash commands**: *When* the user says—explicit invocation via `/command`

Skills solve the context efficiency problem differently than subagents. Subagents isolate work in separate contexts; Skills temporarily specialize the main agent's behavior within its existing context.

### Skills vs Other Features

**Skills vs Subagents**: Skills temporarily add expertise to the current agent. Subagents delegate work to separate agents with isolated contexts. Use Skills when you want the agent to *learn temporarily*; use subagents when you want to *divide work*.

**Skills vs Slash Commands**: Slash commands require users to know when specialized behavior is needed. Skills activate autonomously based on the agent's reasoning about the task. Use slash commands for user-driven workflows; use Skills for context-appropriate specialization.

*[2026-01-11]*: **Unified Mental Model (2.1.3):** Slash commands and Skills are now presented as a unified concept in Claude Code. The underlying behavior remains unchanged—slash commands still require explicit `/` invocation while Skills activate autonomously—but the UI and documentation treat them as variations of the same capability rather than separate features.

**What changed:**
- Single configuration surface for both
- Shared discovery mechanisms
- Unified documentation and help

**What stayed the same:**
- Invocation semantics (explicit vs. autonomous)
- Tool restriction patterns
- Progressive disclosure loading

**Practical implication:** When designing extensibility for Claude Code projects, think of slash commands and Skills as two invocation patterns for the same underlying primitive:

| Aspect | Slash Commands | Skills |
|--------|----------------|--------|
| Invocation | User types `/command` | Model determines relevance |
| Discovery | User knows command exists | Model finds during reasoning |
| Control | User-driven workflow | Context-appropriate activation |
| Definition | `.claude/commands/*.md` | `.claude/skills/SKILLS.md` |

The merge simplifies the mental model without changing when to use each pattern. Prefer slash commands when the user should explicitly trigger behavior; prefer Skills when behavior should activate based on task context.

**Sources:** [Claude Code Changelog 2.1.3](https://code.claude.com/docs/en/changelog)

**Skills vs MCP**: MCP connects agents to external data sources and services. Skills teach agents how to reason about and use that data. MCP says "here's a database"; Skills say "here's how to query databases effectively for this domain."

### Tool Restrictions in Skills

The `allowed-tools` field in SKILLS.md frontmatter enables temporary privilege escalation within defined scopes:

```yaml
---
name: database-migration
description: Safe database schema changes with rollback planning
allowed-tools: [Read, Write, Bash, mcp__supabase__execute_sql]
---
```

This implements "safe by default, permissive by context"—the skill grants additional capabilities only while active. Once the task completes, those permissions revert. This is more granular than subagent tool restrictions, which apply for the entire subagent lifecycle.

**Use case**: A general-purpose agent might not have database write access by default, but activating the `database-migration` skill temporarily grants those capabilities with appropriate guardrails baked into the skill's instructions.

### When to Use Skills

**Good fits**:
- Domain-specific reasoning patterns (security review, accessibility auditing, API design)
- Specialized workflows that recur but don't warrant dedicated agents
- Context-heavy procedures where progressive disclosure saves tokens
- Teaching agents about project-specific conventions or standards

**Poor fits**:
- Simple operations better handled by tools
- Work requiring full context isolation (use subagents)
- User-driven workflows with explicit triggers (use slash commands)
- Knowledge that should always be available (put in CLAUDE.md)

### Practical Example

A skill for API design might include:
- **Description**: "REST API design patterns and consistency checking"
- **Instructions**: Detailed guidance on endpoint naming, versioning, error handling, idempotency
- **Resources**: `references/api-standards.md`, `scripts/check-openapi-spec.py`
- **Allowed tools**: `Read, Grep, mcp__openapi__validate`

When the agent encounters a task involving API endpoints, it autonomously loads the skill, temporarily gaining specialized API design reasoning—without that knowledge consuming context tokens for unrelated tasks.

### Connections

- **To [Context](../4-context/_index.md):** Skills implement the [Progressive Disclosure pattern](../4-context/3-context-patterns.md#progressive-disclosure-pattern)—specialized knowledge loads only when needed, preserving context capacity for other work
- **To [Tool Use](../5-tool-use/_index.md):** Skills teach meta-patterns for tool usage; tools provide capabilities, Skills teach when and how to use them effectively
- **To [Prompt](../2-prompt/_index.md):** Skills are model-invoked rather than user-invoked—the agent determines relevance through reasoning, not explicit commands

**Sources**: [Skills - Claude Code Docs](https://code.claude.com/docs/en/skills), [Equipping Agents for the Real World with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills), [Claude Skills: progressive disclosure for agent workflows](https://simonwillison.net/2025/Oct/16/claude-skills/), [Claude Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/)

### Skill Development Workflow

*[2026-01-11]*: Claude Code 2.1.0 introduced automatic skill hot-reload. Skills in `~/.claude/skills` or `.claude/skills` become immediately available without restarting Claude Code. This enables an iterative development workflow where skill definitions evolve in real-time.

**The Workflow:**

1. **Create or edit** a skill file in `.claude/skills/`
2. **Save** the file
3. **Use** the skill immediately—no restart, no reload command

**Development Pattern: Iterative Refinement**

```
# Terminal 1: Claude Code session
> [Working on task, notice skill needs adjustment]

# Terminal 2 (or editor): Edit skill
$ vim .claude/skills/my-skill/SKILLS.md
# Save changes

# Terminal 1: Skill changes take effect immediately
> [Continue task with updated skill behavior]
```

**Testing Skills in Isolation:**

Combine hot-reload with sub-agent forking (`context: fork`) to test skill behavior without polluting the main conversation context:

```yaml
---
name: test-skill
description: Test skill in isolated context
context: fork
---

Test the target skill behavior with this specific scenario...
```

The forked context prevents test interactions from affecting the main session while hot-reload ensures the latest skill definition is used.

**Practical Benefits:**
- No context loss from restarts during skill development
- Immediate feedback loop on skill changes
- Reduced friction for experimental skill designs
- Enables "edit-save-test" cycles measured in seconds

**Sources:** [Claude Code Changelog 2.1.0](https://code.claude.com/docs/en/changelog)

---

## Configuration

### Tool Restriction as Forcing Function for Delegation

*[2025-12-09]*: The default Claude Code HEAD agent inherits all tools. This means it *can* delegate via Task, but nothing *forces* it to. The agent might do everything itself—creating bloated context, no parallelization, monolithic reasoning that's hard to debug.

**The insight**: If the orchestrator literally *cannot* read files, write code, or execute bash, it has no choice but to spawn subagents.

Two configuration layers shape orchestrator behavior:

**1. Claude Code Settings (`.claude/settings.json`)**

```json
{
  "permissions": {
    "allow": ["Task", "AskUserQuestion", "Read", "Glob", "TodoWrite"],
    "deny": ["Write", "Edit", "Bash*"]
  }
}
```

Restricts tools uniformly across HEAD and all subagents. The HEAD agent can explore (Read, Glob) and delegate (Task), but cannot implement. *Note: This also restricts subagents unless they explicitly declare tools in frontmatter.*

**2. Slash Commands as Workflow Boundaries**

Orchestrator commands (like `/orchestrators:knowledge`) embody this pattern structurally. The orchestrator has one job: spawn and coordinate specialists. The slash command defines the workflow, not the implementation.

**Why this matters**:

| Benefit | Mechanism |
|---------|-----------|
| Context isolation | Subagents return summaries, not raw tool output |
| Natural parallelization | Multiple Task calls = concurrent execution |
| Debugging clarity | Know which specialist failed, not "somewhere in 500 tool calls" |
| Cost optimization | Smaller specialists can use cheaper models |

**Minimum orchestrator tool set**:
- `Task` — spawn subagents
- `Read` — read specs, understand context
- `Glob` — file discovery for routing decisions
- `AskUserQuestion` — clarification when needed
- `TodoWrite` — track workflow progress

**The meta-principle**: Constraints enable architecture. The same way type systems prevent bugs by making invalid states unrepresentable, tool restrictions prevent bad orchestration by making direct implementation unavailable.

### Current Limitations

*[2025-12-09]*: Claude Code's permission system applies uniformly to HEAD and subagents—there is no built-in mechanism to restrict the HEAD agent while allowing subagents broader tool access.

**What this means:**
- Restrictions in `settings.json` affect all agents equally
- Subagents must explicitly declare tools to exceed HEAD restrictions
- The "HEAD can only delegate" pattern requires per-agent tool declarations

### Explicit Tool Declaration Pattern

To achieve orchestrator forcing functions today:

**1. Restrict settings.json to orchestration tools:**
```json
{
  "permissions": {
    "allow": ["Task", "Read", "Glob", "AskUserQuestion", "TodoWrite"]
  }
}
```

**2. Every subagent declares its tools:**
```yaml
---
name: build-agent
tools: Read, Write, Edit, Glob, Grep, Bash
---
```

**3. HEAD cannot implement (no Write/Edit/Bash)**

**4. Subagents opt-in to capabilities they need**

**Trade-offs:**
- Achieves forcing function for orchestration
- Makes capabilities explicit (good for documentation)
- Requires verbose tool declarations across all subagents
- Easy to forget when creating new agents

This is the recommended pattern until Claude Code adds HEAD/subagent permission distinction.

**Open questions**:
- ~~How do hooks determine HEAD vs. subagent context?~~
  *[2025-12-09]*: Hooks receive event data but no explicit HEAD/subagent flag. Detection would require heuristics based on session metadata. Not currently documented or confirmed reliable.
- What happens when subagents need to sub-delegate? (Currently not supported)
- How do you handle exploration that naturally leads to implementation?
- **What would native HEAD/subagent permission distinction look like?**
  Potential design: `permissionMode` extension or new `scope` field in permissions:
  ```json
  {
    "permissions": {
      "head": {
        "allow": ["Task", "Read", "Glob"]
      },
      "subagents": {
        "inherit": ["Write", "Edit", "Bash"]
      }
    }
  }
  ```
  Feature request candidate for Anthropic.

**See Also**:
- [Orchestrator Pattern: Capability Minimization](../6-patterns/3-orchestrator-pattern.md#capability-minimization) — The subagent side of this pattern
- [Tool Use: Tool Restrictions as Security Boundaries](../5-tool-use/3-tool-restrictions.md#tool-restrictions-as-security-boundaries) — Security framing

### Output Styles Deprecation

*[2026-01-11]*: Output styles are deprecated as of Claude Code 2.1.0. The feature continues to work but will be removed in a future version. Projects using output styles should migrate to one of the following approaches:

**Migration Options** (in order of preference):

| Approach | Scope | Use Case |
|----------|-------|----------|
| `CLAUDE.md` | Project | Style guidance checked into codebase, visible to all collaborators |
| `--system-prompt-file <path>` | Session | Load style instructions from file at startup |
| `--append-system-prompt <text>` | Session | Add to default system prompt without replacing it |
| `--system-prompt <text>` | Session | Replace default system prompt entirely |
| Plugins | Session | Most flexible; programmatic control over prompts |

**Recommended Migration Path:**

For agent definitions using `output-style` frontmatter, embed style requirements directly in the agent prompt body:

```yaml
---
name: research-agent
tools: Read, Grep, Glob, WebFetch
model: sonnet
# REMOVED: output-style: concise-reference
---

## Response Format
- Use bullet points for findings
- Include source citations
- Limit summaries to 3-5 sentences

[Rest of agent prompt...]
```

For project-wide style consistency, add guidance to `CLAUDE.md`:

```markdown
## Response Style

- Prefer bullet points over prose
- Include evidence for claims
- Use tables for comparisons
```

**Rationale:** `CLAUDE.md` already serves as the canonical source of project instructions. Embedding style guidance there (or directly in agent prompts) eliminates a separate abstraction layer and keeps all behavioral configuration visible in version control.

**Sources:** [Claude Code Changelog 2.1.0](https://code.claude.com/docs/en/changelog)

### Sandbox Mode vs. Permissions: Independent Security Layers

*[2025-12-09]*: A critical distinction that's easy to miss—Claude Code has two separate security mechanisms that operate independently:

**Sandbox Mode** (OS-level isolation):
- Uses bubblewrap (Linux) or Seatbelt (macOS) for container-like restrictions
- Restricts filesystem access to current working directory by default
- Routes network traffic through a proxy, allowing only approved domains
- Enforced at the OS level—even if Claude wants to escape, it physically cannot

**Permission System** (`--dangerously-skip-permissions`):
- Controls whether Claude asks before doing things
- Bypasses permission prompts (for CI, trusted environments, etc.)
- Does NOT disable sandbox boundaries

**The key insight**: Using `--dangerously-skip-permissions` does NOT disable sandboxing. You can run Claude Code unattended with the flag, and sandbox restrictions still apply at the OS level. Commands trying to write outside the working directory or reach non-whitelisted domains are blocked regardless of permission settings.

This is defense-in-depth: even if a prompt injection attack manipulates Claude into trying something malicious, the sandbox prevents actual damage. The permission flag just means "don't ask me," not "remove all restrictions."

**The escape hatch**: There's a `dangerouslyDisableSandbox` parameter on the Bash tool for commands that legitimately need to run outside sandbox (like Docker). Even this goes through normal permission flow—it's not automatic. Can be disabled entirely with `"allowUnsandboxedCommands": false` in settings.

| Layer | Controls | `--dangerously-skip-permissions` Effect |
|-------|----------|----------------------------------------|
| Sandbox | OS-level filesystem/network boundaries | None—still enforced |
| Permissions | Ask-before-doing prompts | Bypassed |
| Escape Hatch | Per-command sandbox override | Still requires permission |

This independence matters for security architecture: you can safely automate Claude Code in trusted environments without losing the OS-level protection against runaway behavior.

### Hook Context Injection

*[2026-01-17]*: Claude Code 2.1.9 enables PreToolUse hooks to inject context back to the model via `additionalContext`.

**The Problem**: Previously, hooks could only allow or block tool execution. This binary choice forced conservative blocking policies—hooks couldn't provide nuance.

**The Solution**: Hooks can now return context that influences the model's decision-making without hard-blocking:

```python
def pre_tool_use(event):
    tool = event.get("tool")
    params = event.get("parameters", {})

    if tool == "Write" and "/etc" in params.get("path", ""):
        return {
            "additionalContext": {
                "warning": "/etc is a system directory",
                "alternatives": ["/tmp", "~/.config"],
                "guidance": "Consider user-space paths for configuration"
            },
            "action": "allow"  # Let model decide with context
        }

    return {"action": "allow"}
```

### Use Cases

| Pattern | additionalContext Value |
|---------|------------------------|
| Cost warnings | Estimated API cost before expensive calls |
| Safety guardrails | Alternative approaches for risky operations |
| Style guidance | Project conventions before file writes |
| Dynamic permissions | Context-dependent allow/deny decisions |

### Comparison: Block vs. Context Injection

| Approach | When to Use |
|----------|-------------|
| Block (exit 2) | Hard security boundaries—never allow |
| additionalContext | Soft boundaries—model decides with information |

**The mental model**: Context injection enables *safer autonomy*. Models make better decisions when hooks provide visibility rather than just blocking. The hook becomes an advisor, not a gatekeeper.

**Security consideration**: `additionalContext` does not replace blocking for true security boundaries. System directories, credential files, and destructive operations should still use hard blocks. Context injection is for "you probably don't want to do this" cases, not "you must never do this" cases.

**Sources:** [Claude Code Changelog 2.1.9](https://code.claude.com/docs/en/changelog)

---

## Memory Management

*[2026-02-05]*: Claude Code 2.1.32 introduced automatic session memory and modular rules system, providing persistent memory across sessions for the first time.

### Your Mental Model

**Session memory bridges ephemeral context and persistent knowledge.** Context dies with each session; session memory automatically preserves critical insights across restarts. The `.claude/rules/` system organizes persistent knowledge modularly rather than monolithically.

### Session Memory: Automatic Recording & Recall

#### How It Works

Session memory operates as a background process that automatically summarizes conversation context at specific intervals. Unlike manual note-taking, this happens without user intervention—the system tracks conversation flow and writes summaries when certain thresholds are met.

**Triggering Conditions:**
- First summary written at approximately 10,000 tokens of conversation
- Subsequent summaries every 5,000 tokens OR every 3 tool calls (whichever comes first)
- Summaries capture key decisions, approaches, and context for future sessions

**Storage Architecture:**
```
~/.claude/projects/<project-hash>/<session-id>/session-memory/summary.md
```

Each session generates its own summary file. When starting a new session in the same project, Claude Code loads relevant summaries from previous sessions.

#### Observable Behavior

**Terminal Indicators:**
- "Recalled X memories" - Displayed at session start when loading previous session summaries
- "Wrote X memories" - Displayed during session when new summaries are written

**Expandable Details:**
Press `Ctrl+O` on the memory indicator to expand and view the full summary content. This shows exactly what context was preserved from previous sessions.

#### Feature Gate Status

**⚠️ Availability:** Session memory requires first-party Anthropic API access and is gated behind the `tengu_session_memory` feature flag. Not all Claude Code installations have this enabled by default.

**Verification:** Start a long session (10,000+ tokens) and watch for "Wrote X memories" indicators. If they don't appear, session memory is not enabled for your account.

### /remember Command: Curated Memory Management

The `/remember` command bridges automatic session memory and deliberate project knowledge. It reviews accumulated session memories, identifies recurring patterns across sessions, and proposes additions to `CLAUDE.local.md`.

**Workflow:**
1. Run `/remember` after working with session memory enabled
2. Review proposed memories extracted from session summaries
3. Approve or reject each proposed addition
4. Approved memories are written to `CLAUDE.local.md`

**Pattern:** Automatic collection → Manual curation → Permanent storage

This prevents session memory from remaining ephemeral. Important learnings surface through repeated mention across sessions, then graduate to permanent project knowledge through human approval.

### /compact Integration

Session memory pre-written summaries enable instant context compaction. Previously, `/compact` required approximately 2 minutes to generate summaries before compacting context. With session memory, summaries already exist—compaction becomes near-instantaneous.

**Use case:** When approaching context limits mid-session, `/compact` uses existing session summaries to compress conversation history without re-analysis delay.

### # Shortcut: Quick Memory Addition

The `#` shortcut provides a fast path for adding memories directly to CLAUDE.md files during conversation.

**Syntax:**
```
# Remember to use pytest fixtures for test setup
```

Starting a message with `#` flags it as a memory to be added to project documentation. This bypasses the `/remember` approval flow for quick knowledge capture during active work.

### .claude/rules/ System: Modular Path-Scoped Rules

The rules system enables modular, path-specific configuration separate from monolithic CLAUDE.md files.

#### Directory Structure

**Project-level:** `.claude/rules/*.md`
**User-level:** `~/.claude/rules/*.md`

Claude Code recursively discovers all `.md` files in these directories. Subdirectories are supported for organization:

```
.claude/rules/
├── testing/
│   ├── pytest-conventions.md
│   └── integration-tests.md
├── security/
│   └── auth-review.md
└── style-guide.md
```

#### Path-Scoped Rules

Rules with `paths` frontmatter field only activate when working with matching files.

**YAML Frontmatter Pattern:**
```yaml
---
paths:
  - "**/*_test.py"
  - "tests/**/*.py"
---

# Test File Conventions

When working with test files:
- Use pytest fixtures for shared setup
- Prefix test functions with test_
- Group related tests in classes
- Mock external dependencies
```

**Glob Pattern Support:**
- `**/*.py` - All Python files anywhere
- `src/**/*.ts` - TypeScript files in src tree
- `*_test.go` - Test files in current directory
- `docs/**/*.md` - Markdown in docs directory

#### Conditional Loading

Rules without `paths` frontmatter load for all files (project-wide). Rules with `paths` load only when the current file path matches one or more glob patterns.

**This enables:**
- Test-specific guidance only when editing tests
- Security review checklists only for auth modules
- Framework conventions only for relevant file types

#### Symlink Support

Rules can be symlinked across projects for shared conventions.

**Example: User-level Security Rules**
```bash
# Create user-level rule once
cat > ~/.claude/rules/security.md << 'EOF'
# Security Review Checklist

Before committing code:
- [ ] No hardcoded credentials
- [ ] Input validation on user data
- [ ] SQL injection prevention
- [ ] XSS protection in templates
EOF

# Symlink into project
ln -s ~/.claude/rules/security.md .claude/rules/security.md
```

Now the security checklist applies to this project without duplicating the file.

**User-Level Rules Default:**
All rules in `~/.claude/rules/` automatically apply to every project for that user. No symlinks required—these are global defaults.

### Complete Memory Hierarchy

Claude Code loads memory from seven tiers in order of precedence:

| Tier | Location | Scope | Automatic | Example Use |
|------|----------|-------|-----------|-------------|
| 1 | `/Library/.../CLAUDE.md` | Org-wide | ✗ | Corporate policies |
| 2 | `./CLAUDE.md` | Project | ✗ | Team conventions |
| 3 | `./.claude/rules/*.md` | Project/Path | ✗ | Modular rules |
| 4 | `~/.claude/CLAUDE.md` | User global | ✗ | Personal defaults |
| 5 | `./CLAUDE.local.md` | Project local | ✗ | Private overrides |
| 6 | `~/.claude/projects/.../summary.md` | Session | ✓ | Learned context |
| 7 | `@path/to/file` | Import | ✗ | Shared docs |

**Key additions in this hierarchy:**
- **Tier 3:** `.claude/rules/` system for modular, path-scoped organization
- **Tier 6:** Session memory for automatic persistence across sessions

**Loading behavior:** Later tiers can override or extend earlier ones. Tier 1 (org policy) loads first; Tier 7 (imports) loads last.

### Comparison: Memory System Features

| Feature | Automatic | Persistent | Modular | Path-Scoped | User Control |
|---------|-----------|-----------|---------|-------------|--------------|
| Session Memory | ✓ | ✓ | ✗ | ✗ | Low (via /remember) |
| .claude/rules/ | ✗ | ✓ | ✓ | ✓ | High (direct editing) |
| CLAUDE.md | ✗ | ✓ | ✗ (monolithic) | ✗ | High |
| CLAUDE.local.md | ✗ | ✓ | ✗ (monolithic) | ✗ | High |
| Imports | ✗ | ✓ | Semi | ✗ | High |

**Trade-offs:**
- Session memory maximizes convenience (automatic) at the cost of control
- .claude/rules/ maximizes modularity and path-scoping at the cost of setup complexity
- CLAUDE.md maximizes simplicity (single file) at the cost of modularity
- CLAUDE.local.md balances team visibility (not committed) with persistence

### Connection to Context Fundamentals

Session memory directly addresses the "Context vs Memory" distinction from [Context Fundamentals](../4-context/1-context-fundamentals.md#context-vs-memory):

> Memory would be persistent knowledge that survives restarts—but without external storage mechanisms, agents don't have this by default.

Session memory provides this external storage mechanism. Agents now have persistent memory that survives restarts, automatically maintained by the system.

### When to Use Each Memory Mechanism

**Use Session Memory when:**
- Automatic persistence desired (no manual maintenance)
- Cross-session learning needed (agent learns from past sessions)
- Working on long-running projects (weeks/months)
- Manual memory curation feels like overhead

**Use .claude/rules/ when:**
- Path-specific behavior needed (different rules for different directories)
- Modular organization preferred (separate concerns into files)
- Sharing rules across projects (via symlinks)
- Fine-grained control over rule application required

**Use CLAUDE.md when:**
- Project-wide conventions apply uniformly
- Team visibility required (checked into version control)
- Simple monolithic configuration preferred
- Single source of truth for project guidance

**Use CLAUDE.local.md when:**
- Personal preferences differ from team (local overrides)
- Private information shouldn't be shared (API keys, personal notes)
- Individual workflow customization needed
- Experimenting with guidance before committing to CLAUDE.md

### Decision Framework

```
Is memory automatic or manual?
│
├─ Automatic → Session Memory
│              (system maintains summaries)
│
└─ Manual → Is memory path-specific?
            │
            ├─ Yes → .claude/rules/
            │        (path-scoped YAML frontmatter)
            │
            └─ No → Is memory shared with team?
                    │
                    ├─ Yes → CLAUDE.md
                    │        (checked into version control)
                    │
                    └─ No → CLAUDE.local.md
                            (private, not committed)
```

### Practical Examples

#### Example 1: Path-Scoped Test Rules

`.claude/rules/test-conventions.md`:
```yaml
---
paths:
  - "**/*_test.py"
  - "tests/**/*.py"
---

# Test File Conventions

When working with test files:
- Use pytest fixtures for shared setup
- Prefix test functions with test_
- Group related tests in classes
- Mock external dependencies
```

This rule only applies when editing Python test files. Regular Python files don't see this guidance.

#### Example 2: Cross-Project Shared Rules

User-level rule (`~/.claude/rules/security.md`):
```markdown
# Security Review Checklist

Before committing code:
- [ ] No hardcoded credentials
- [ ] Input validation on user data
- [ ] SQL injection prevention
- [ ] XSS protection in templates
```

Applies to all projects for this user automatically. No per-project configuration needed.

#### Example 3: Session Memory → Permanent Memory Flow

**Session 1:** Agent works on authentication refactoring, generates session summary automatically

**Session 2:** Start new work in same project
```
> Recalled 1 memory
```

**Expand with Ctrl+O:**
```
Previous session refactored auth module, extracted JWT logic to
separate service. Token validation now happens in middleware layer
before route handlers execute.
```

**User action:** `/remember` → Review summaries → Approve addition to CLAUDE.local.md

**Result:** Temporary session knowledge becomes permanent project knowledge, available to all future sessions

### Open Questions

- How does session memory handle extremely long-running sessions (days/weeks)?
- What happens when session summaries conflict with CLAUDE.md instructions?
- Can session memory be disabled per-project for privacy-sensitive work?
- How are summaries merged across parallel sessions in same project?
- What's the maximum number of session memories retained before cleanup?
- Does session memory support manual editing of summary.md files?
- How do path-scoped rules interact with Skills system (which also uses paths)?
- Can .claude/rules/ subdirectories have their own scoping beyond path globs?
- What happens when multiple rules match same path—are they merged or does one win?

### Sources

- [Claude Code Changelog 2.1.32](https://code.claude.com/docs/en/changelog)
- [Official Memory Documentation](https://code.claude.com/docs/en/memory)
- Session memory file structure observed at `~/.claude/projects/<project-hash>/`

---

## Advanced: Feature Gate Reverse Engineering

*[2026-01-30]*: Claude Code ships with capabilities hidden behind server-side feature gates. Community research documents techniques for discovering and enabling these features through surgical patching.

**⚠️ Warning:** This is fragile, unsupported territory. The techniques below are observational research from the `claude-sneakpeek` project, not officially documented or recommended practices. Patches couple tightly to minified code structure and break with updates.

### Your Mental Model

**Feature gates are string anchors in minified JavaScript.** Claude Code's `cli.js` is minified and obfuscated—function names change between builds (`xK()`, `Yz()`), but string literals survive minification unchanged. Stable strings like `"tengu_brass_pebble"` (swarm mode gate) or `"TodoWrite"` (team mode gate) serve as anchors for locating surrounding gate logic.

### String Anchor Discovery Technique

**The pattern:**
1. Find stable string literal in minified code (e.g., `"tengu_brass_pebble"`)
2. Extract surrounding function using regex pattern
3. Dynamically capture minified function name
4. Replace gate logic with unconditional return

**Example: Swarm Mode Gate**

**Original minified code:**
```javascript
function xK(){
  if(Yz(process.env.CLAUDE_CODE_AGENT_SWARMS))return!1;
  return wQ("tengu_brass_pebble",!1)
}
```

**Detection regex:**
```javascript
const SWARM_GATE_FN_RE = /function\s+([a-zA-Z_$][\w$]*)\(\)\{if\([\w$]+\(process\.env\.CLAUDE_CODE_AGENT_SWARMS\)\)return!1;return\s*[\w$]+\("tengu_brass_pebble",!1\)\}/;
```

**Patched code:**
```javascript
function xK(){return!0}
```

### Regex Pattern Extraction Methodology

**Critical insight:** Patterns match exact minified structure, not semantic meaning.

**Anchor marker pattern:**
```javascript
const SWARM_GATE_MARKER = /tengu_brass_pebble/;
```

**Function extraction pattern:**
```javascript
const SWARM_GATE_FN_RE =
  /function\s+([a-zA-Z_$][\w$]*)\(\)\{if\([\w$]+\(process\.env\.CLAUDE_CODE_AGENT_SWARMS\)\)return!1;return\s*[\w$]+\("tengu_brass_pebble",!1\)\}/;
```

**Capturing groups:**
- `([a-zA-Z_$][\w$]*)` - Function name (group 1, dynamically extracted)
- Environment check pattern
- Statsig call pattern

**Replacement strategy:**
```javascript
const gate = content.match(SWARM_GATE_FN_RE);
const fnName = gate[1];
const patched = content.replace(
  gate[0],
  `function ${fnName}(){return!0}`
);
```

### Version Pinning Strategy

**Stability vs. staying current:**

| Approach | Trade-off |
|----------|-----------|
| **Pin to validated version** | Patches stable, features frozen, security updates delayed |
| **Track latest** | Latest features, patches break frequently, continuous maintenance |

**claude-sneakpeek strategy:** Pin to `@anthropic-ai/claude-code@2.0.76`

**Rationale:**
- Patches validated against known minified structure
- Feature gates at predictable locations
- Avoids breaking changes in newer versions
- Updates require deliberate validation

**When to update pins:**
- Critical security patches released
- Desired features only in newer versions
- Community confirms patches work for target version

### Three-State Detection

Feature gates exist in three observable states:

| State | Detection Method | Meaning |
|-------|------------------|---------|
| `disabled` | Gate function pattern matches | Original gate present, feature blocked |
| `enabled` | Marker absent + swarm code detected | Gate patched, feature unlocked |
| `unknown` | No marker, no swarm code | Version incompatible or feature removed |

**Detection implementation:**
```typescript
export const detectSwarmModeState = (content: string): SwarmModeState => {
  const gate = findSwarmGateFunction(content);
  if (gate) return 'disabled';

  if (!SWARM_GATE_MARKER.test(content)) {
    const hasSwarmCode = /TeammateTool|teammate_mailbox|launchSwarm/.test(content);
    if (hasSwarmCode) return 'enabled';
    return 'unknown';
  }

  return 'unknown';
};
```

### Fragility Considerations

**Why this is fragile:**

1. **Minified code coupling**: Regex patterns match exact minified JavaScript structure. Any minifier setting change breaks patterns.
2. **Function name volatility**: Minified function names change unpredictably. Dynamic capture required.
3. **String literal dependency**: Anthropic could rename `"tengu_brass_pebble"` or change gate structure entirely.
4. **No versioning contract**: Features can appear/disappear between releases without notice.
5. **Maintenance burden**: Each Claude Code update requires pattern re-validation.

**Mitigation strategies:**
- Version pinning (stability over features)
- Automated patch testing in CI
- Backup/restore mechanism for `cli.js`
- State detection before and after patching
- Clear documentation of which version patterns work for

### Example: TeammateTool Discovery

**String anchor:** `"TodoWrite"` variable assignment

**Multi-stage search:**
```typescript
const TODO_WRITE_MARKER = /(var|let|const)\s+[A-Za-z_$][\w$]*="TodoWrite";/;
const IS_ENABLED_FN_RE = /isEnabled\(\)\{return!([A-Za-z_$][\w$]*)\(\)\}/;
```

**Why more complex:** Team mode gate not directly adjacent to feature code. Requires windowed search:
1. Find `TodoWrite` marker
2. Search 8000-character window after marker
3. Extract `isEnabled()` function
4. Locate referenced gate function
5. Patch gate function

**Window-based search optimization:**
```typescript
const markerIndex = content.indexOf('"TodoWrite"');
const window = content.slice(markerIndex, markerIndex + 8000);
const match = window.match(IS_ENABLED_FN_RE);
```

### Observational Research vs. Recommended Practice

**Frame this as:**
- "Research documents how hidden features are discovered"
- "Community unlock for experimental access"
- "Observational analysis of feature gating"

**Not as:**
- "How to customize Claude Code in production"
- "Officially supported modification technique"
- "Stable API for feature enablement"

**Use cases where acceptable:**
- Research and learning about multi-agent coordination
- Prototyping workflows before official release
- Contributing feedback to Anthropic on gated features
- Understanding Claude Code's internal architecture

**Use cases where inappropriate:**
- Production systems requiring stability
- Commercial products depending on patched features
- Environments where maintenance burden unacceptable
- Situations where official support required

### Relationship to TeammateTool

Feature gates discovered through these techniques unlock capabilities documented in the TeammateTool section:
- Spawn, Join, Write, Broadcast operations
- File-based messaging (`~/.claude/teams/`)
- Five coordination patterns (Leader-Worker, Swarm, Pipeline, Council, Plan Approval)
- Backend selection (in-process, tmux, iterm2)

**See:** [TeammateTool: Native Multi-Agent Coordination](#teammatetool-native-multi-agent-coordination-hidden) for what these patches unlock.

### Sources

- [claude-sneakpeek swarm mode patching](https://github.com/mikekelly/claude-sneakpeek/blob/main/src/core/variant-builder/swarm-mode-patch.ts)
- [Feature gate discovery research](https://raw.githubusercontent.com/mikekelly/claude-sneakpeek/main/docs/research/native-multiagent-gates.md)
- [Patching techniques analysis](.claude/.cache/research/external/claude-sneakpeek-patching-techniques-2026-01-30.md)

---

*Your CLAUDE.md, settings, or setup notes:*


