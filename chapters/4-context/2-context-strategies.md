---
title: Context Management Strategies
description: Practical techniques for handling context limits, compression, and structured context
created: 2025-12-10
last_updated: 2026-02-06
tags: [foundations, context, strategies, compression, retrieval]
part: 1
part_title: Foundations
chapter: 4
section: 2
order: 1.4.2
---

# Context Management Strategies

Practical approaches for managing context windows, balancing injection vs. retrieval, and structuring context for optimal performance.

---

## Managing Context Window Limits

Context availability serves as a rough proxy for remaining model capability. A model with 25% context utilization retains approximately 75% capability capacity. At 90% utilization, effective capability drops to roughly 10%.

### Current Practice: Boot Fresh Agents

*[2025-12-10]*: Context compaction via secondary agents is not a viable strategy in current implementations (December 2025). When agents near context limits or output quality degrades, boot a fresh instance rather than attempting salvage through compression.

Tooling support improves this workflow. Claude Code's hook system can automatically log agent actions—reads, writes, tool calls—enabling rapid reconstruction of relevant context for successor agents. This makes "boot fresh" practical even for complex workflows.

### When Compaction Makes Sense

See [Frequent Intentional Compaction](#frequent-intentional-compaction) below for the exception: proactive compression at 40-60% utilization as a deliberate quality maintenance strategy, not emergency salvage.

---

## Context Compression and Summarization

Compression works in conversational exchanges where information density matters less than continuity. Technical work like coding demands precision—context compression typically introduces gaps that manifest as "context rot" or "context bloat."

The "one agent, one task" principle applies: if an agent can't complete a task within its context budget, adjust the prompt or scope. Chaining multiple agents to compact and summarize context invites hallucination without guaranteeing quality preservation.

---

## Injection vs. Retrieval Balance

### Injection for Priming

Context injection typically occurs at session start to prime agent behavior. Common injection targets include:
- Base configuration (CLAUDE.md, project structure)
- Recent activity context (git history, previous session summaries)
- Domain knowledge (relevant documentation, coding standards)

### Retrieval for Discovery

Agent-driven retrieval works well for codebase exploration, particularly in large repositories. The agent discovers relevant files based on task requirements rather than receiving a predetermined payload.

### The Balance Point

Codebase scale determines the injection-retrieval balance. Small codebases tolerate more retrieval (agents find relevant files efficiently). Large codebases risk context bloat from excessive exploration—agents read tangential files searching for relevant content.

### Open Questions

*Areas requiring deeper exploration:*

- What constitutes optimal "priming" payload for typical coding sessions? CLAUDE.md alone? Recent git history? Documentation excerpts?
- When retrieval produces context bloat (irrelevant file reads), can mid-session intervention recover quality, or does it require agent restart?
- Does a codebase size threshold exist where injection-heavy approaches become mandatory due to retrieval unreliability?

---

## Structured Context Patterns

Structured context (markdown, JSON, XML) consistently outperforms unstructured prose. Structure provides:
- **Focus mechanisms** — Headers, sections, and delimiters guide attention
- **Parsing support** — Both models and humans parse structured content more reliably
- **Prompt engineering aid** — Templates and schemas assist both human developers and meta-prompting agents

Markdown offers the best balance: human-readable, model-friendly, and widely supported. JSON and XML work for machine-readable payloads where parsing guarantees matter.

---

## Understanding Context Rot

### Observable Symptoms

*Areas requiring characterization:*

Context rot manifests as quality degradation distinct from simple model errors. Symptoms likely include:
- Inconsistent outputs despite identical prompts
- Drift from original task specifications
- Increasing reliance on context window periphery
- Degraded reasoning chains compared to early-session outputs

### Reversibility Question

Whether context rot can be reversed within a session or requires agent restart remains uncharacterized. Current practice defaults to "boot fresh" when symptoms appear, suggesting low confidence in within-session recovery.

### Distinction from Model Confusion

Context rot differs from standard model errors or confusion. Model errors stem from capability limits or ambiguous prompts. Context rot results from degraded working memory—the model's capability remains intact, but its operational context has compromised output quality.

### Context Window as Finite Resource

*[2026-02-06]*: GSD project (12K-star open source tool) treats context window as a non-renewable resource with explicit quality relationship: **Quality ∝ 1/(% context used)**. This inverse relationship suggests quality degrades proportionally to context fill. Their mitigation: each plan execution gets fresh 200K context window, sized to remain <50% utilized. Main context never accumulates degradation. Context rot emerges from resource depletion—once context fills, quality cannot be recovered through compression, only through fresh allocation. This aligns with the "boot fresh agents" guidance above but adds quantified threshold: target <50% utilization for sustained quality, not just "avoid 95%."

---

## Context Window Percentage Monitoring

*[2026-01-17]*: Claude Code 2.1.9 introduced real-time context utilization percentage display, transforming context management from guesswork to data-driven decision-making.

### Display Format

The session header shows context usage as:

```
[45K/200K tokens] 22%
```

This updates in real-time as conversation progresses, accounting for both input and output tokens consumed.

### Operational Thresholds

| Range | Signal | Recommended Action |
|-------|--------|-------------------|
| 0-30% | Healthy | Continue normally |
| 30-60% | Monitor | Good checkpoint for intentional compaction |
| 60-80% | Caution | Consider fresh session if major phase ends |
| 80-95% | Warning | Begin graceful task wrap-up |
| 95%+ | Critical | Boot new agent immediately |

These thresholds complement the frequent intentional compaction strategy described below. The 30-60% "Monitor" range aligns directly with the 40-60% compaction trigger.

### Percentage-Driven Decision Framework

Instead of guessing when context nears capacity:

1. **Set mental alert at 60%**: "Should I compact or continue?"
2. **At natural break points**: Compact if above 50%
3. **At 80%**: Stop accepting new work, finish current task
4. **At 95%**: Force new session (no negotiation)

This framework removes ambiguity from the "when to compact" decision. The percentage provides objective data; the thresholds provide clear action triggers.

### Multi-Agent Context Tracking

In orchestrated workflows, track per-agent percentages to enable proactive scheduling:

```python
# Example subagent completion output
{
    "agent": "builder-agent",
    "context_used": 156000,
    "context_limit": 200000,
    "percentage": 78,
    "recommendation": "boot-fresh-next-task"
}
```

This enables capacity-aware task routing:
- Route new work to agents with headroom
- Reboot agents approaching limits before task assignment
- Predict whether an agent can complete a task within remaining capacity

### Integration with Hooks

PostToolUse hooks can monitor percentage progression automatically:

```python
def post_tool_use(event):
    percentage = event.get("context_percentage", 0)
    if percentage > 75:
        log_alert(f"Context at {percentage}%—consider compaction")
```

This provides operational awareness without manual monitoring, surfacing warnings at configurable thresholds.

### Practical Application

The percentage display transforms context management from reactive ("the agent seems confused") to proactive ("we're at 65%, compact before the next task").

Combined with the compaction strategies below, percentage monitoring enables:
- **Evidence-based compaction timing** — Compact at 50% rather than guessing
- **Predictable session planning** — Know when you'll need a fresh agent
- **Quality maintenance** — Intervene before degradation, not after

---

## Frequent Intentional Compaction

*[2025-12-10]*: Most teams compact context reactively—when the agent hits 95% capacity and auto-summarization kicks in as an emergency measure. By then, quality has already degraded. Frequent intentional compaction flips this: compact proactively at 40-60% utilization to maintain quality, not salvage it.

**The Pattern**: Instead of waiting for context limits to force compression, compact deliberately and frequently throughout a session. Target 40-60% context utilization as your compaction trigger, not 90%+.

**Optimization Priority Order**:
1. **Correctness** — Preserve factual accuracy above all else
2. **Completeness** — Ensure all critical information survives compaction
3. **Signal-to-noise** — Remove redundancy, keep high-value context
4. **Trajectory** — Maintain the narrative thread of what's been done and why

This ordering matters. Emergency compaction at 95% often sacrifices correctness for brevity. Intentional compaction at 50% can optimize all four dimensions without forced trade-offs.

### Concrete Technique: Status-to-Plan Compaction

One practical pattern is compacting status updates back into plan documents:

```markdown
# Before (context bloat)
## Plan
- Implement user authentication
- Add database schema
- Create API endpoints

## Status Updates
- [14:23] Started auth implementation
- [14:45] Auth working, moving to database
- [15:12] Database schema complete
- [15:30] Schema had bug, fixed
- [15:45] API endpoints half done
...
```

```markdown
# After (intentional compaction)
## Plan
- ✓ Implement user authentication — Complete, no issues
- ✓ Add database schema — Complete after fixing validation bug
- ⧗ Create API endpoints — In progress (3/5 complete)

## Current Work
Working on remaining API endpoints (POST /users, DELETE /users)
```

The compacted version preserves correctness (what's done), completeness (including the bug fix), signal (current state), and trajectory (what's next)—all in a fraction of the context.

### Why This Works

- **Proactive = Quality**: Compacting at 50% gives you room to optimize. At 95%, you're in triage mode.
- **Frequent = Fresh**: Small, regular compactions are easier to verify than massive emergency summarizations.
- **Intentional = Controlled**: You decide what to preserve based on priority order, not panic.

### The Trade-off

Requires active monitoring of context utilization and deliberate intervention. You can't set-and-forget. The investment is ongoing attention; the return is sustained quality.

**Real-World Results**: HumanLayer's ACE-FCA framework demonstrated this approach shipping 35,000 lines of code in 7 hours. The key wasn't speed—it was maintaining quality through aggressive proactive compaction. When you compact intentionally at 40-60%, you avoid the quality degradation that comes from emergency summarization at 95%.

### Contrast with Emergency Compaction

| Approach | Trigger Point | Quality Impact | Control |
|----------|---------------|----------------|---------|
| **Emergency Auto-Compact** | 95%+ capacity | Degrades (forced summarization) | Low (automatic) |
| **Frequent Intentional** | 40-60% capacity | Maintains (controlled compression) | High (deliberate) |
| **No Compaction** | Never (boot fresh agents) | High (fresh context) | Highest (manual restart) |

Our default advice remains "boot a new agent" for most cases. Frequent intentional compaction is for scenarios where agent continuity matters—long-running sessions, accumulated state, or workflows where restarting is expensive.

### When to Use

- Multi-hour coding sessions where restarting loses momentum
- Workflows that accumulate valuable learned context
- Situations where handoff overhead exceeds compaction cost
- Teams optimizing for sustained agent performance over time

### When to Boot Fresh Instead

- Short, focused tasks (< 30 minutes)
- Clear task boundaries where restart is natural
- Quality concerns outweigh continuity needs
- Agent output shows signs of degradation

---

## Connections

- **To [Context Fundamentals](1-context-fundamentals.md):** The "One Agent, One Task" principle this technique extends
- **To [Advanced Context Patterns](3-context-patterns.md):** How frequent intentional compaction relates to ACE's growing contexts
- **To [Patterns](../6-patterns/_index.md):** Emergency Context Rewriting anti-pattern demonstrates why reactive compaction fails

---

## Sources

ACE-FCA framework by HumanLayer — Demonstrated in production shipping 35K LOC in 7 hours using proactive compaction at 40-60% utilization
