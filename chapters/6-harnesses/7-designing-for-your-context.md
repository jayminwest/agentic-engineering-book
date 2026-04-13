---
title: Designing for Your Context
description: Decision frameworks for harness design — when to use each category and how to build compound advantage
created: 2026-04-12
last_updated: 2026-04-12
tags: [foundations, harness, design, decision-framework, compound-advantage, trajectories]
part: 1
part_title: Foundations
chapter: 6
section: 7
order: 1.6.7
---

# Designing for Your Context

Harness design is not a one-size-fits-all decision.

The appropriate harness depends on the work type, team structure, security requirements, and scale of operation. The preceding sections of this chapter established what a harness is, what it consists of, what types exist, how it functions as a control system, and how it improves through engineering. This section translates the conceptual framework into actionable decisions for practitioners facing a concrete design problem.

---

## The Four Design Questions

Four questions determine the appropriate harness design for any given context. Answering them in order prevents the most common design mistake — selecting tools before understanding requirements.

### Question 1: What is the time horizon of the work?

Single-session work (a research query, a code review, a draft document) does not require persistent memory, phase discipline, or multi-session state. A chat interface or a raw coding agent session is sufficient.

Multi-session work (a software project spanning days or weeks, an ongoing research program, a recurring automated workflow) requires persistent state, context discipline, and a harness that survives session boundaries.

The time horizon determines the minimum memory architecture required:
- Single session → in-context working memory (no persistence required)
- Multi-session, defined lifecycle → workflow harness with explicit state files
- Multi-session, ongoing relationship → persistent personal runtime or full agentic harness with auto-memory

### Question 2: How many agents run in parallel?

A single agent running sequentially has no coordination requirements. The harness only needs to manage that agent's context, memory, and tool access.

A small number of agents (2–5) working in parallel on related tasks requires scope isolation (ensuring agents don't modify the same files simultaneously) and result integration (the orchestrator assembles subagent outputs into a coherent whole). A workflow harness or full agentic harness with orchestrator-pattern configuration handles this tier.

Twenty or more agents running in parallel requires infrastructure: git worktrees for isolated workspaces, merge queues for sequential result integration, supervisory oversight, and resource management. A workspace manager is the appropriate harness category.

### Question 3: What are the security requirements?

Read-only work on non-sensitive data requires minimal sandbox controls — the agent can only read, and the data is not sensitive.

Write access to production code, credentials, configuration files, or sensitive data requires explicit permission enforcement, sandbox containment, and observability. The harness must be able to prevent, detect, and log unauthorized actions.

Network access requirements add a third dimension: a harness that can make arbitrary outbound network calls is a data exfiltration risk for sensitive codebases. Network egress filtering becomes a requirement.

The security requirement level maps directly to the required harness complexity:
- Read-only, non-sensitive → minimal sandbox (directory jail sufficient)
- Write access to project files → full sandbox + permission hooks + observability
- Network access with sensitive code → whitelist egress + full sandbox + audit logs

### Question 4: What is the expected trajectory volume?

Trajectory volume determines whether trajectory capture is infrastructure or overhead.

Low volume (occasional, ad-hoc agentic sessions) — trajectory capture is optional. The data accumulated is unlikely to be sufficient for fine-tuning or systematic harness evaluation.

High volume (regular sustained production use, 10+ sessions per day, 100+ sessions per week) — trajectory capture is competitive infrastructure. The data accumulates into a proprietary dataset of the organization's specific tasks, failure modes, and improvement history. This dataset cannot be purchased; it must be grown.

*[2026-04-12]*: The trajectory capture question is frequently deferred because instrumentation feels like overhead. Schmid's framing — "The Harness is the Dataset" — reframes this as a strategic decision. Organizations that instrument from the beginning compound advantage; organizations that add instrumentation later lose the early trajectory data that often contains the most valuable failure cases.

---

## Decision Tree for Harness Selection

```
Is the work single-session and exploratory?
├─ Yes → Chat interface or raw coding agent
└─ No (multi-session or structured work)
   ├─ Does it require parallel agents at scale (20+)?
   │  ├─ Yes → Workspace manager
   │  └─ No
   │     ├─ Does it require persistent personal recall (single agent, ongoing relationship)?
   │     │  ├─ Yes → Persistent personal runtime (Hermes pattern)
   │     │  └─ No
   │     │     ├─ Does it involve defined phases with verification gates?
   │     │     │  ├─ Yes → Workflow harness (GSD pattern)
   │     │     │  └─ No → Full agentic harness (Claude Code or equivalent)
   └─ Is a custom multi-agent system with novel coordination required?
      ├─ Yes → Framework + runtime (LangGraph, CrewAI)
      └─ No → Full agentic harness (Claude Code or equivalent)
```

**Reading the tree:** Start at the top. Each decision narrows the category. The tree resolves most common cases; edge cases that fall between categories are addressed in the following section.

---

## Building the Harness Incrementally

Most practitioners should not design a harness from scratch. The investment required to build all six stack components — workspace context, prompt shape, tool access, context management, session memory, and subagent delegation — from nothing is substantial, and the failure modes of custom harnesses are difficult to anticipate.

The recommended sequence:

**1. Start with an existing full agentic harness.** Claude Code ships with Anthropic's defaults across all six components. The defaults work well for most software engineering tasks and can be configured rather than built. For teams without a compelling reason to build custom infrastructure, this is the starting point.

**2. Identify the first repeating failure mode.** Run the harness on real work. The first systematic failure — the mistake that recurs across different tasks, different sessions, or different team members — is the highest-value target for harness engineering. Don't engineer hypothetical failures; engineer the ones that actually occur.

**3. Apply harness engineering: add a guide or sensor.** The failure classification from [Harness Engineering](5-harness-engineering.md) determines whether a guide (prevent the mistake) or sensor (detect and correct it) is appropriate. The engineering change is specific and targeted.

**4. Verify the fix.** Reproduce the original failure case after the fix. Confirm that the fix doesn't introduce regressions. Confirm that it generalizes to similar cases.

**5. Repeat.** Each iteration adds a constraint that makes the harness more reliable for the specific class of work the team does. The harness becomes specialized through use.

**6. Graduate to a workflow harness when multi-session coordination is needed.** When the work begins to require explicit phase management, verification gates between phases, or parallel execution with context isolation, a workflow harness provides the coordination infrastructure that a full agentic harness does not.

**7. Add workspace manager tooling when parallel scale requires it.** When parallel execution reaches the point where 20+ agents are running simultaneously and manual supervision is infeasible, workspace manager infrastructure becomes necessary.

This sequence applies Hashimoto's methodology operationally: the harness grows from the bottom up, shaped by observed failures rather than anticipated ones. The harness that emerges is specialized for the team's actual work — which is more valuable than a general-purpose harness designed in advance.

---

## Evaluating an Existing Harness

For practitioners joining a team with an existing harness, or evaluating harnesses in the marketplace, the six-component audit sequence provides a structured evaluation framework:

| Component | Evaluation Question | Red Flag |
|-----------|-------------------|---------|
| Workspace context | Does the harness inject stable environmental facts at session start? | Agent discovers project structure through repeated tool calls |
| Prompt shape | Is the stable/dynamic split explicit? Is caching enabled? | All context treated as dynamic; no caching |
| Tool access | Is the tool inventory bounded and documented? Are permissions enforced at the harness layer? | Unbounded tool access; permission enforcement only in prompts |
| Context management | Does the harness proactively compress context before degradation? | No compression until context is nearly full |
| Session memory | Is working memory explicitly maintained separate from the transcript? | Single accumulating log used for both purposes |
| Subagent delegation | Are subagents spawned with explicit scope and context boundaries? | Subagents inherit full parent context |

Harnesses with red flags in multiple components require either configuration changes (if the harness supports them) or harness engineering (adding the missing components). A harness with no workspace context injection, no context management, and unbounded subagent delegation is a harness in name only — it is a raw API call with coordination logic.

---

## Compound Advantage Through Trajectory Capture

Schmid's framing ("The Harness is the Dataset") points to the mechanism by which harness quality compounds over time. The mechanism is trajectory accumulation.

A trajectory is a sequence: task specification → agent observations → agent actions → outcomes → human judgment (success/failure/correction). Each session run through an instrumented harness produces one or more trajectories.

**What trajectories enable:**

*Harness improvement:* Trajectories reveal systematic failure modes. A practitioner who reviews 100 trajectories from the past month will identify the 3–5 failure patterns that account for most failures — and can engineer targeted fixes for each. Without trajectory data, harness improvement is reactive (fix whatever the most recent complaint was) rather than systematic (fix the most common failure mode).

*Model fine-tuning:* Trajectories are training data in the format models understand: context → reasoning → action → outcome. A dataset of 1,000 high-quality trajectories from domain-specific work is more valuable for fine-tuning than a generic benchmark — because it represents the actual distribution of tasks the fine-tuned model will encounter.

*Capability measurement:* Before-and-after trajectory comparison measures harness improvement quantitatively. Run the same set of benchmark tasks before and after a harness engineering cycle; compare success rates and failure modes. Without trajectories, harness improvement is unmeasurable.

**Practical minimum for trajectory capture:**

| What to Log | Format | Retention |
|------------|--------|----------|
| Every tool call: name, inputs, outputs, timestamp | Append-only NDJSON | Indefinite (these are training data) |
| Session outcome: success/failure, task type, duration | Session metadata file | Indefinite |
| Harness engineering events: failure observed, component diagnosed, fix applied | Engineering log | Indefinite |

Adding trajectory instrumentation to an existing harness typically requires:
1. Pre/post hooks on all tool calls (write log entry on each invocation)
2. Session close handler (write session metadata on session end)
3. Storage that persists across sessions (local files, SQLite, or cloud storage)

The cost is low; the long-term value is high. The trajectory dataset is the organization's proprietary knowledge of how agents behave on its specific work.

---

## The Harness as the Fifth Pillar

The five pillars now form a complete picture of what determines agent capability in production:

| Pillar | What It Determines | Primary Lever |
|--------|-------------------|--------------|
| Prompt | What instruction the agent receives | Prompt engineering |
| Model | What reasoning the agent can perform | Model selection |
| Context | What information the agent can access | Context engineering |
| Tool Use | What actions the agent can execute | Tool design |
| Harness | What system orchestrates, constrains, and improves execution | Harness engineering |

No single pillar determines outcomes in isolation. The harness is the connective tissue: it manages context, enforces tool access, shapes prompts, creates feedback loops, and accumulates the trajectory data that drives improvement.

The key property that distinguishes the harness pillar from the others: it is the only pillar that compounds. Model capability does not improve while the model runs. Context does not improve while context fills. Prompts do not improve while the session proceeds. But the harness — through each engineering cycle, each trajectory captured, each guide added and sensor installed — improves over time. It is the agent's environment becoming more adapted to the agent's work.

This compounding property is the foundational argument for treating harness as a first-class engineering concern from the beginning of any agentic project. The early investment in harness engineering — selecting the right category, implementing trajectory capture, applying the engineering loop — produces returns that grow with use. The harness that has been in production and improved for six months outperforms a harness designed perfectly but used only once, not because of initial design quality, but because of accumulated adaptation.

---

## Summary

Harness design decisions in order:
1. Answer the four design questions (time horizon, parallelism, security, trajectory volume)
2. Use the decision tree to select the harness category
3. Start with an existing harness rather than building from scratch where possible
4. Apply the harness engineering loop to real failures as they occur
5. Instrument for trajectory capture from the beginning
6. Graduate to more capable harness categories as scale and requirements grow

The harness is not a peripheral concern. It is the system that makes the other four pillars operational. Investing in harness design and harness engineering is investing in the compounding foundation of every agentic system.

---

## Connections

- **[What Is a Harness?](1-what-is-a-harness.md)** — The foundational definition and Agent = Model + Harness formula
- **[The Harness Stack](2-harness-stack.md)** — The six components evaluated in the harness audit sequence
- **[Harness Categories](3-harness-categories.md)** — Category details and capability tiers
- **[Harness Engineering](5-harness-engineering.md)** — The iterative improvement methodology
- **[Foundations](../1-foundations/_index.md)** — The full five-pillar model this chapter completes
- **[Practitioner Toolkit](../10-practitioner-toolkit/_index.md)** — Specific tool evaluations for each harness category
- **[Workflow Coordination](../8-practices/5-workflow-coordination.md)** — Workflow harness patterns in production depth
