---
title: Advanced Context Patterns
description: Progressive disclosure, context loading vs. accumulation, and ACE framework for growing contexts
created: 2025-12-10
last_updated: 2026-02-06
tags: [foundations, context, patterns, progressive-disclosure, ace]
part: 1
part_title: Foundations
chapter: 4
section: 3
order: 1.4.3
---

# Advanced Context Patterns

Sophisticated patterns for managing context in complex scenarios: progressive disclosure for unlimited expertise, context loading for precise payloads, and the ACE framework for knowledge-intensive domains.

---

## Progressive Disclosure Pattern

*[2025-12-09]*: Progressive disclosure addresses context window limits by loading information in tiers based on relevance, enabling effectively unlimited expertise within fixed context budgets.

**The Pattern**: Information loads in three tiers:
1. **Metadata first** — Names, descriptions, summaries (~50-200 characters per item)
2. **Full content on selection** — Complete documentation when explicitly chosen (~500-5,000 words)
3. **Detailed resources on-demand** — Supporting files, source code, references (unbounded)

This creates a semantic index in the initial context, allowing the agent to navigate a vast information space without loading everything upfront.

### Concrete Example: Claude Skills

Claude Skills demonstrate this pattern in production:
- Initial load: ~50-200 chars per skill (description and when to use it)
- Activation: 500-5,000 words of expertise per selected skill
- References: Unlimited supporting files via Read tool when skill is active

With 10 skills at 100 chars each, the metadata costs ~1,000 characters. This buys semantic awareness of all available expertise. When a specific skill activates, its full context loads—but only that one, not all ten simultaneously.

### Cognitive Parallel

Humans don't memorize encyclopedias. We build indexing systems—file systems, bookmarks, tables of contents—for on-demand retrieval. Progressive disclosure mirrors this: maintain an index in working memory, fetch details when needed.

### Contrast with Alternatives

| Approach | Upfront Cost | Discoverability | Capacity |
|----------|--------------|-----------------|----------|
| **Eager Loading** | Massive (tens of thousands of tokens) | Perfect | Limited by context window |
| **Lazy Loading** | Zero | Poor (agent doesn't know what exists) | Theoretically unlimited |
| **Progressive Disclosure** | Small (metadata only) | Good (semantic index) | Effectively unlimited |

Anthropic's GitHub MCP integration illustrates the eager loading trap: "tens of thousands of tokens" consumed just to make repositories and issues accessible. Progressive disclosure would load repo names/descriptions first, then fetch specific repos on-demand.

### The Trade-off

Slight latency on selection (additional tool call to fetch full content) for dramatic capacity gains. A system with 100 items × 1,000 tokens each costs 100k tokens with eager loading, but only ~5k tokens with progressive disclosure (metadata + one activated item).

### When to Use Progressive Disclosure

- **Large knowledge bases** where most content won't be needed for any single task
- **Multi-domain expertise** where the agent needs awareness but not full activation
- **Tight context budgets** where capability breadth is essential but space is limited
- **Dynamic capability selection** where the agent should choose expertise based on task requirements

### When NOT to Use

- **Small, static knowledge sets** where eager loading costs less than infrastructure
- **Guaranteed access patterns** where you know exactly which content will be needed
- **Latency-critical paths** where additional tool calls are unacceptable
- **Simple retrieval** where a single Read or Grep suffices

### Implementation Patterns

- Tool descriptions as metadata layer (Read/Grep as on-demand fetchers)
- Structured indices with `description` fields (see [MCP Tool Declarations](../5-tool-use/3-tool-restrictions.md#mcp-tool-declarations-in-frontmatter))
- Skills systems (see [Claude Code: Skills](../9-practitioner-toolkit/1-claude-code.md))

*[2026-02-06]*: Federated knowledge architecture uses progressive disclosure to navigate multi-repository contexts. Instead of loading all repos upfront, load source metadata first (repo names, descriptions), then expand specific repos on-demand. See [Federated Knowledge Architecture](2-context-strategies.md#federated-knowledge-architecture) for distributed context management patterns.

---

## Context Loading vs. Context Accumulation

*[2025-12-09]*: Most LLM interaction patterns treat context as **accumulated**—chat history grows, tool results append, context fills passively until you hit limits. Context loading flips this: context is **curated**, deliberately constructed for each call.

### The Default Mental Model (Accumulation)

```
User message → append to context
Tool result → append to context
Agent response → append to context
... context fills until you hit limits
```

### Context Loading Mental Model

```
For this specific call:
├── Load: base config (always)
├── Load: project context (if relevant)
├── Load: tool definitions (only what this agent needs)
├── Load: query (the specific task)
├── Load: retrieved facts (verified, not raw)
└── Nothing else
```

The precision is the point. You're not asking "what has accumulated?" You're asking "what does this agent need for this exact call?"

### Why This Is Counterintuitive

Standard patterns assume context is a **log**—append-only, grows over time, summarize when full. Context loading treats context as a **payload**—constructed fresh, minimal, purpose-built.

This flips the default question:
- **Log model**: "What can I remove to fit?"
- **Payload model**: "What must I include to succeed?"

### Connection to Small Models

Context loading explains why small models work in orchestrator patterns. Haiku doesn't accumulate—it receives a curated payload (base config, project prompt, tool info, query) and returns a focused result. The orchestrator handles accumulation; scouts receive loads.

See [Model Selection: Small Models Are RAG](../3-model/1-model-selection.md#small-models-are-rag) for the context staging breakdown.

### When to Use Context Loading

- **Multi-agent systems** where orchestrators coordinate specialized scouts
- **High-precision tasks** requiring exact context composition
- **Small model deployments** where context budgets are tight
- **Quality-critical paths** where context noise degrades outputs
- **Stateless operations** where each call should be independent

### When Accumulation Works Better

- **Conversational interfaces** where continuity matters more than precision
- **Learning workflows** where context should grow with discoveries
- **Long-running sessions** where recomputing context is expensive
- **Debugging scenarios** where full interaction history provides diagnostic value

### See Also

[Context Loading Demo](../../appendices/examples/context-loading-demo/README.md) — Working implementation showing orchestrator → scout context staging with optional verification layer. Demonstrates payload construction, verification contract, and token economics.

### Open Questions

- Could a verification layer (like KotaDB) fact-check context before loading? Scout A says X, Scout B says Y—verify before the orchestrator loads either.
- What's the contract for verified context? Confidence scores? Source citations? Contradiction flags?
- Does this change agent architecture? Instead of scouts → orchestrator, maybe scouts → verification layer → orchestrator?

---

## Agentic Context Engineering (ACE)

*[2025-12-10]*: The ACE framework from Stanford/SambaNova challenges a core assumption in agent design: that context should shrink over time. Instead, ACE argues contexts should **grow**—comprehensive evolving playbooks outperform compressed prompts in complex domains.

### The Core Insight

Traditional optimization creates "brevity bias"—the assumption that shorter contexts are better. This leads to "context collapse" where critical learned information gets summarized away. ACE flips this: contexts should expand with learned knowledge, not compress it.

### The Tension with Frequent Intentional Compaction

This creates an interesting contrast with frequent intentional compaction. Both approaches reject **reactive** emergency compaction (waiting until 95% capacity). But they differ in philosophy:

| Approach | Philosophy | When to Use |
|----------|-----------|-------------|
| **Frequent Intentional Compaction** | Compress proactively at 40-60% | General-purpose coding, bounded tasks |
| **ACE (Growing Contexts)** | Expand deliberately with learned patterns | Knowledge-intensive domains, tool-heavy tasks |

The key: both are **proactive** strategies that beat reactive summarization. Choose based on task type, not as universal defaults.

### Three-Role Architecture

ACE organizes agents into three complementary roles:

1. **Generator** — Executes tasks using current playbook
2. **Reflector** — Analyzes outcomes and extracts learnings
3. **Curator** — Evolves the playbook based on reflections

This mirrors software development: execute code (generator), learn from errors (reflector), update documentation (curator). The context is the playbook—a living document that grows more comprehensive over time.

### Structured Playbook Format

Instead of prose instructions, ACE uses itemized bullets with metadata:

```markdown
## Authentication Patterns

- [AUTH-001] Use JWT tokens for stateless sessions
  Helpful: 12 | Harmful: 1

- [AUTH-002] Validate tokens on every API call
  Helpful: 15 | Harmful: 0

- [AUTH-003] Store refresh tokens in httpOnly cookies
  Helpful: 8 | Harmful: 2
  Reason harmful: Doesn't work with mobile clients
```

Each item has an ID for tracking, helpful/harmful counters from feedback, and explanations for anti-patterns. The structure makes it easy to add, update, or remove specific guidance without rewriting entire sections.

### Grow-and-Refine Principle

The playbook evolution follows a two-phase cycle:

1. **Growth Phase**: Add new learnings from reflections
   - Don't prune yet—accumulate insights
   - Capture both successful patterns and failures
   - Tag items with context (which tasks, which tools)

2. **Refinement Phase**: Semantic deduplication
   - Merge redundant items (AUTH-001 + AUTH-012 → AUTH-001-v2)
   - Remove contradicted patterns (harmful count exceeds helpful)
   - Consolidate related guidance into categories

The key insight: growth **then** refinement, not growth **versus** refinement. You need accumulation to see patterns before intelligent compression becomes possible.

### When to Use ACE

ACE shines in specific scenarios:

- **Knowledge-intensive domains**: Medical diagnosis, legal reasoning, scientific analysis where comprehensive playbooks matter
- **Complex tool use**: Multi-tool workflows (AppWorld benchmark) where learned tool patterns accumulate
- **Natural feedback loops**: Tasks with clear success/failure signals for helpful/harmful tracking
- **Long-running projects**: Where context grows across many sessions, not just one

### When NOT to Use ACE

- **Simple QA**: Factual lookup doesn't benefit from playbook evolution
- **Fixed-strategy problems**: If the approach is deterministic, no learning needed
- **Short-lived tasks**: Single-session work lacks the horizon for playbook growth
- **Unbounded domains**: Without natural categories, playbooks become unwieldy

### Performance Results

The Stanford/SambaNova paper demonstrates concrete gains:

- **+12.5% improvement** on AppWorld benchmark (complex multi-tool agent tasks)
- **82.3% latency reduction** compared to GEPA (Graph-Enhanced Planning Approach)
- **Better sample efficiency**: Fewer attempts needed to learn effective patterns

The latency reduction is particularly striking—growing contexts performed **faster** than compressed ones. The hypothesis: well-structured comprehensive playbooks reduce trial-and-error during execution. The generator doesn't need to rediscover patterns; they're already documented.

### Practical Implementation Pattern

A simplified ACE cycle for coding:

```markdown
## Session Start
Load: Base playbook (accumulated patterns from previous sessions)

## During Task Execution (Generator)
Agent executes using playbook guidance
Logs decisions and outcomes

## After Each Subtask (Reflector)
Analyze: What worked? What didn't?
Extract: New patterns worth capturing
Tag: Which tools, which contexts, which outcomes

## End of Session (Curator)
Review: All extracted patterns
Add: New items to playbook with IDs
Update: Helpful/harmful counts based on outcomes
Merge: Semantically duplicate items
Prune: Contradicted or obsolete guidance
```

The playbook grows session-over-session. Early sessions add rapidly; later sessions mostly increment counters and merge duplicates. Over time, you build a comprehensive knowledge base **in context**, not external to it.

### Connection to Other Patterns

ACE complements several existing patterns:

- **Persistent State vs. Ephemeral Context**: ACE playbooks are persistent state loaded into context. The playbook survives sessions; the working context does not.
- **Progressive Disclosure**: Playbook categories could use progressive disclosure—load category summaries first, expand specific sections on-demand.
- **Multi-Agent Context Isolation**: Each agent role (generator/reflector/curator) maintains separate context. Reflector accumulates learnings; curator synthesizes; generator receives refined playbook.
- **Context Loading vs. Accumulation**: ACE is deliberate accumulation—curated growth, not passive appending.

### The Mental Shift

Traditional context management asks: "How do I fit within limits?"

ACE asks: "How do I grow knowledge within structure?"

It's a shift from **context as constraint** to **context as knowledge base**. The context window isn't just working memory—it's the accumulated expertise of previous runs. This only works with structure (itemized bullets, IDs, counters) and discipline (grow-then-refine, not append-forever).

### Open Questions

- How large can playbooks grow before structure breaks down? Is there a practical limit to itemized guidance?
- Can helpful/harmful counters be tracked automatically via tool success/failure, or do they require human feedback?
- Does ACE work for domains without clear success signals? What replaces helpful/harmful in ambiguous tasks?
- Could reflector/curator roles be automated, or do they need human-in-the-loop validation?

---

## Connections

- **To [Context Strategies](2-context-strategies.md):** Frequent Intentional Compaction as complementary compression strategy
- **To [Multi-Agent Context](4-multi-agent-context.md):** How generator/reflector/curator roles maintain separate contexts
- **To [Tool Use](../5-tool-use/_index.md):** MCP tool declarations and progressive disclosure via tool metadata
- **To [Claude Code](../9-practitioner-toolkit/1-claude-code.md):** Skills implement progressive disclosure in production

---

## Sources

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic on progressive disclosure
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills) — Production implementation
- [Simon Willison: Claude Skills](https://simonwillison.net/2025/Oct/16/claude-skills/) — Practitioner perspective
- [Agentic Context Engineering: Enhancing AI Agents with Self-Evolving, Structured Contexts](https://arxiv.org/abs/2510.04618) — Stanford/SambaNova ACE framework paper
