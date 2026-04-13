---
title: Cost and Latency
description: Managing the economics and speed of agent systems
created: 2025-12-08
last_updated: 2025-12-10
tags: [practices, cost, latency, performance, economics]
part: 2
part_title: Craft
chapter: 8
section: 3
order: 2.8.3
---

# Cost and Latency

Agents that are too slow or too expensive don't ship. These constraints shape everything.

---

## Your Mental Model

**Frame cost as investment, not expense.** The question isn't "is this expensive?" but "what's the cost of NOT using it?" When agents ship 10× faster, the API bill becomes a productivity multiplier, not a line item.


---

## Connections

- **To [Model](../3-model/_index.md):** How does model choice affect cost/latency?
- **To [Context](../4-context/_index.md):** How does context size affect cost/latency?
- **To [Evaluation](2-evaluation.md):** How do you include cost/latency in your eval metrics?
- **To [Orchestrator Pattern: Capability Minimization](../7-patterns/3-orchestrator-pattern.md#capability-minimization):** Restricting subagent tools reduces context size → lower token cost per agent
- **To [Scaling Tool Use](../5-tool-use/4-scaling-tools.md):** Dynamic tool discovery (85% token reduction) and programmatic orchestration (37% token reduction) are production-tested optimization patterns

---

## The Economics of Agent-Assisted Development

### Real-World ROI: What $12K/Month Buys

*[2025-12-10]*: Concrete numbers from a 3-person engineering team running Claude Code in production:

**The Investment**:
- ~$12,000/month in API costs
- ~$4,000 per engineer per month

**The Return**:
- Week's worth of work shipped **daily** per engineer
- 10× productivity multiplier on feature delivery
- Tasks estimated at 3-5 days completed in 7 hours
- 35,000 lines of code generated and integrated in a single session

**The Reframing**:

Traditional question: "Can we afford $12K/month in API costs?"

Correct question: "Can we afford to ship 10× slower?"

For a team with loaded costs of ~$150K/year per engineer ($12.5K/month), spending $4K/month to 10× their output means:
- Effective cost per unit of work drops by 90%
- Each engineer delivers the output of 10 engineers
- The $4K API cost buys you $120K worth of equivalent engineering capacity

**Productivity Benchmarks**:

From actual development sessions:
- **Code generation**: 35K LOC in 7 hours (previously estimated at 3-5 days)
- **Feature velocity**: Daily shipping cadence vs. weekly estimates
- **Context switching**: Agents handle tedious implementation while engineers focus on architecture

### The Hidden Costs of NOT Using Agents

**Opportunity cost compounds**:
- Shipping 10× slower means 10× fewer customer deployments
- 10× fewer A/B tests run
- 10× fewer bugs found and fixed
- 10× less market feedback incorporated

**Engineering team economics**:
- Hiring cost for additional engineers: ~6 months to recruit, onboard, ramp
- Agent productivity: Available immediately, scales instantly
- Knowledge retention: Agents don't leave; patterns persist in prompts

### When Cost Actually Matters

**Cost becomes the constraint when**:
- You're running batch processing at massive scale (millions of documents)
- Latency requirements force you to over-provision for peak load
- You're building consumer products with thin margins

**Cost is rarely the constraint when**:
- Building internal tooling (developer time >> API costs)
- Shipping customer features (revenue impact >> API costs)
- Prototyping and validation (speed to learning >> API costs)

### Measuring What Matters

**Poor metrics**:
- Total API spend (no context on value delivered)
- Cost per token (optimizes the wrong thing)
- Agent invocation count (ignores quality)

**Better metrics**:
- Cost per feature shipped
- Cost per bug fixed
- API cost as % of engineer loaded cost
- Time-to-delivery improvement vs. baseline

**Best metric**:
- Revenue or value delivered per dollar of API cost

For a SaaS product, if agents help ship a feature that generates $50K/year in revenue, the $1K in API costs to build it is a 50× return.

---

## Optimization Techniques You've Used

*Document specific optimizations, what worked, what the tradeoffs were:*

### Multi-Agent: Trading Tokens for Quality

*[2025-12-09]*: Multi-agent architectures trade tokens for deterministic quality. The numbers from academic research are striking:

**What You Get**:
- 80× improvement in action specificity
- 100% actionable recommendation rate (vs. 1.7% for single-agent)
- 140× improvement in solution correctness in some domains
- Zero quality variance across trials—deterministic outcomes

**What You Pay**:
- ~15× more tokens than single-agent approaches
- Token usage explains 80% of performance variance

**The Surprising Finding**: Architectural value lies in deterministic quality, not speed. Both single and multi-agent achieve similar latency (~40s for complex research tasks). You're not parallelizing for speed—you're parallelizing for quality and reliability.

**Decision Framework**: Does your task benefit from parallel analysis by specialized experts AND require near-zero quality variance? Then multi-agent is worth the token cost. For simple tasks, it's overkill. The breakeven point is somewhere around "complex enough that a single agent would need multiple passes anyway."

**Anthropic's Production Numbers**: Their multi-agent research system showed 90.2% improvement over single-agent Claude Opus 4 on internal research evals, with 90% reduction in research time for complex queries. Lead agent spawns 3-5 subagents in parallel; each subagent uses 3+ concurrent tools.

**Sources**: [Multi-Agent LLM Orchestration Achieves Deterministic Decision Support](https://arxiv.org/html/2511.15755), [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)

### Token Cost Models by Feature Type

*[2025-12-09]*: Different agent feature types—tools, Skills, subagents, and MCP servers—have radically different token cost profiles. Understanding these profiles shapes architectural decisions.

**Cost Profiles by Feature Type**:

| Feature Type | Tokens per Invocation | Primary Cost Driver |
|--------------|----------------------|---------------------|
| Traditional Tools | ~100 tokens | Call overhead (parameters + results) |
| Skills | ~1,500+ tokens | Discovery metadata + execution context |
| Subagents | Full conversation history | Isolated context per subagent |
| MCP Servers | 10,000+ tokens | Rich integration schemas + persistent state |

**Frequency-Depth Trade-offs**:

The right choice depends on usage frequency and task complexity:

- **One-time actions**: Tools minimize cost. For a single database query or file read, ~100 tokens is the floor.
- **Weekly+ repeatable workflows**: Skills amortize higher per-invocation cost (~1,500 tokens) across multiple uses. If you invoke something 3+ times per week, Skills' progressive disclosure becomes cost-effective.
- **Parallel work streams**: Subagents' context isolation prevents main agent bloat. Instead of one agent with 100k tokens of mixed concerns, you get multiple 20k-token focused contexts.
- **Continuous data access**: MCP's upfront cost (tens of thousands of tokens for schema exchange) enables persistent connectivity. For integrations like GitHub that need sustained interaction, this beats repeated tool calls.

**Discoverability vs. Cost**:

There's a token cost hierarchy for discoverability:

- **No-overhead (Tools)**: Minimal cost but requires explicit invocation. The agent must know to call the tool.
- **Progressive disclosure (Skills)**: Metadata-based discovery costs tokens upfront (~1,500 per activation) but enables autonomous activation. The agent can discover "I should use this Skill" from context.
- **Eager loading (MCP)**: High upfront cost (full schema exchange) but complete discoverability. The agent sees all available operations at once.

**Decision Framework**:

Ask these questions when choosing feature types:

1. **How often will this be invoked?** One-time → Tool. Weekly+ → Skill. Continuous → MCP.
2. **Does the agent need to discover it autonomously?** No → Tool. Yes → Skill or MCP.
3. **Is context isolation valuable?** Yes → Subagent. No → Tool/Skill.
4. **How rich is the integration schema?** Simple → Tool. Complex → MCP.

**Real-World Example**:

For a code analysis workflow:
- File reading: Tool (~100 tokens/call, frequent, simple)
- Security audit: Skill (~1,500 tokens, weekly, needs discovery)
- Background research: Subagent (isolated context, parallel to main work)
- GitHub issue tracking: MCP (persistent connection, rich schema)

The token cost model directly reflects the architectural value provided. Tools are cheap because they're simple. Skills cost more because they're autonomous. Subagents burn tokens for isolation. MCP pays upfront for integration depth.

**Sources**: [Simon Willison: Claude Skills](https://simonwillison.net/2025/Oct/16/claude-skills/), [Claude Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/), [Claude Code: Skills Documentation](https://code.claude.com/docs/en/skills)

**See Also**:
- [Tool Use](../5-tool-use/_index.md) — Feature comparison and tool design patterns
- [Claude Code: Subagent System](../10-practitioner-toolkit/1-claude-code.md#subagent-system) — Implementation details for context isolation


