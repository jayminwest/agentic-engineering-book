---
title: Practices
description: The craft of building, debugging, and running agents in the real world
created: 2025-12-08
last_updated: 2026-04-12
tags: [practices, craft, operations]
part: 2
part_title: Craft
chapter: 8
section: 0
order: 2.8.0
---

# Practices

Theory gets you started. Practice is where the learning happens.

---

## Practice Areas

| Practice | Focus |
|----------|-------|
| [Debugging Agents](1-debugging-agents.md) | Finding and fixing what went wrong |
| [Evaluation](2-evaluation.md) | Measuring agent performance systematically |
| [Cost and Latency](3-cost-and-latency.md) | Managing the economics and speed of agent systems |
| [Production Concerns](4-production-concerns.md) | Running agents reliably at scale |
| [Workflow Coordination](5-workflow-coordination.md) | Structured metadata for agent coordination |
| [Knowledge Evolution](6-knowledge-evolution.md) | Guidelines for updating knowledge base entries |
| [Operating Agent Swarms](7-operating-agent-swarms.md) | Running multi-agent systems at production scale |

---

## Questions to Answer Here

- What's the most important skill for someone building agents that isn't obvious at first?
- What practices do you wish you'd adopted earlier?
- How does the craft of agentic engineering differ from traditional software engineering?
- What does "experience" give you in this field? What can't be shortcut?

---

## Your Development Workflow

*How do you typically work when building or improving an agent?*

---

## High-Leverage Review

**Review research and plans, not code line-by-line.**

The most expensive mistakes in agentic systems happen at the conceptual level, not the implementation level. Multi-agent production systems demonstrate this pattern: agents operating from wrong assumptions execute flawed strategies flawlessly, propagating conceptual errors through thousands of generated lines. One misunderstood research conclusion causes more damage than any single code defect.

Traditional code review focuses on implementation details—checking syntax, catching edge cases, spotting potential bugs. That made sense when humans wrote every line. With agents generating code, the leverage point shifts dramatically upstream.

### Where Human Expertise Has Maximum Impact

**Early conceptual validation, not late implementation review:**

- **Research quality:** Did the agent understand the domain correctly? Are its assumptions sound?
- **Plan coherence:** Does the proposed approach make sense? Will it solve the actual problem?
- **Mental model alignment:** Is the agent thinking about this problem the right way?
- **Scope appropriateness:** Is it solving too much? Too little?

Spending an hour reviewing an agent's research findings catches misunderstandings before they propagate through thousands of lines of generated code. Spending that hour on line-by-line code review inspects the symptoms of conceptual errors without preventing them.

### The Anti-Pattern

Skimming the plan, then carefully reviewing every generated line of code. This inverts the leverage:

- Accepting flawed thinking at the cheapest intervention point (the plan)
- Catching its consequences at the most expensive intervention point (the implementation)
- The agent has already invested its context window in the wrong direction
- Wasting expertise on problems that static analysis or tests would catch

**The pattern:** Invest your review time where agents are weakest—conceptual reasoning, domain understanding, holistic design. Trust them for what they're strong at—consistent implementation of a well-defined plan.

### Practical Implementation

1. **Review the research phase thoroughly:** Read what the agent learned. Check its sources. Verify its conclusions.
2. **Review the plan critically:** Does this approach make sense? What could go wrong? What's it not considering?
3. **Spot-check the implementation:** Sample key sections to verify the plan is being followed. Don't read every line.
4. **Review by testing:** Run it. Does it behave as the plan specified? If not, was the plan wrong or the implementation?

Mental alignment matters more than individual line correctness. Correct thinking makes implementation debuggable. Flawed thinking means perfect code just automates misconceptions faster. This observation holds across production deployments: conceptual validation prevents more failures than implementation review catches.


