---
title: Harnesses
description: The execution environment that transforms model capability into agent performance
created: 2026-04-12
last_updated: 2026-04-12
tags: [foundations, harness, execution-environment, agent-formula]
part: 1
part_title: Foundations
chapter: 6
section: 0
order: 1.6.0
---

# Harnesses

The harness is everything around the model that makes it an agent.

```
Agent = Model + Harness
```

The four preceding foundational pillars — Prompt, Model, Context, Tool Use — answer what the agent says, what it can reason about, what it knows, and what it can do. The harness answers the fifth question: *what system orchestrates and constrains the agent's execution?* Without a harness, there is no agent — only a model being prompted. The harness is the connective tissue that makes multi-step, multi-tool, multi-session work possible.

*[2026-04-12]*: Practitioner consensus around this formula crystallized in early 2026. Martin Fowler, Ethan Mollick, Sebastian Raschka, Philipp Schmid, and Mitchell Hashimoto independently converged on the harness as the primary differentiator between model capability and product performance — not through coordination, but through simultaneous arrival at the same empirical observation.

---

## Chapter Overview

This chapter builds from definition through practical design guidance:

### [1. What Is a Harness?](1-what-is-a-harness.md)

The foundational definition, formula, and conceptual underpinnings. Establishes the horse harness metaphor (raw capability → useful work), presents definitions from five authoritative practitioners, distinguishes harness from scaffold, and traces the historical evolution from prompt engineering through context engineering to harness engineering.

**Key concepts:**
- Agent = Model + Harness formula
- Scaffold (pre-runtime) vs. harness (runtime) distinction
- The three eras: prompt engineering, context engineering, harness engineering
- Why harness is a foundational pillar, not a peripheral concern

### [2. The Harness Stack](2-harness-stack.md)

Raschka's six-component taxonomy of the execution environment. Each component has distinct responsibilities, failure modes, and optimization targets. Understanding the stack enables practitioners to audit agent failures at the right layer rather than defaulting to model blame.

**Key concepts:**
- Workspace context (stable facts, repo map)
- Prompt shape and cache reuse (stable/dynamic split)
- Tool access (bounded inventories, permission filtering)
- Context management (clipping, deduplication, compression)
- Session memory (working memory + full transcript duality)
- Subagent delegation (bounded spawning, context scoping)

### [3. Harness Categories](3-harness-categories.md)

The taxonomy of harness types from raw coding agents to full workspace managers. Includes Mollick's three-axis Model/App/Harness stack, capability tier comparison, and the distinction between frameworks (build-time) and harnesses (runtime with defaults). Provides a decision table linking problem characteristics to harness category.

**Key concepts:**
- Model/App/Harness as independent axes
- Harness capability tiers (full agentic / web-based / constrained)
- Framework vs. runtime vs. harness vocabulary
- Workflow harnesses, workspace managers, persistent personal runtimes
- Category selection decision table

### [4. Harness as Control System](4-harness-as-control-system.md)

Fowler's guides-and-sensors decomposition treats the harness as an active control system, not passive scaffolding. Guides intervene before agent actions (feedforward); sensors observe results and steer subsequent behavior (feedback). Each mechanism can be computational (deterministic) or inferential (model-based).

**Key concepts:**
- Guides (feedforward control) — computational and inferential
- Sensors (feedback control) — computational and inferential
- The cost tradeoff: computational vs. inferential mechanisms
- Agent Psychometrics formula: scaffold quality is additively independent from LLM capability

### [5. Harness Engineering](5-harness-engineering.md)

Hashimoto's methodology for systematic harness improvement — coined February 5, 2026. The core discipline: when an agent makes a mistake, engineer the surrounding system so that mistake cannot recur. Covers the six-step engineering loop, failure classification taxonomy, and Schmid's trajectory-capture competitive advantage thesis.

**Key concepts:**
- Harness engineering vs. prompt patching
- The six-step improvement loop
- Failure classification (model / context / prompt / harness / tool)
- Trajectory capture as competitive infrastructure
- Verification-driven development as harness engineering in practice

### [6. Security, Permissions, and Trust](6-security-permissions-trust.md)

The harness is the primary security boundary in an agentic system. The model does not enforce permissions — the harness does. Covers permission models (scope/operation/session dimensions), sandbox architecture, token-level vs. session-level access control, trust hierarchies in multi-agent systems, and observability requirements.

**Key concepts:**
- Harness enforcement vs. model enforcement
- Sandbox dimensions (filesystem, network, process, resource)
- Token-level vs. session-level access control
- Principle of least privilege in multi-agent trust hierarchies
- Observability surfaces: tool call logs, permission logs, session transcripts

### [7. Designing for Your Context](7-designing-for-your-context.md)

Translates the conceptual framework into actionable decisions. Presents the four design questions (time horizon, agent count, security requirements, trajectory volume), a decision tree for harness selection, the incremental build sequence, and the compound advantage that trajectory capture creates over time.

**Key concepts:**
- Four design questions for harness selection
- Decision tree: exploratory → workflow → workspace manager → custom
- Incremental build sequence starting from existing harnesses
- Compound advantage through trajectory capture

---

## Core Questions

This chapter explores:

- **Definition**: What distinguishes a harness from a prompt, a tool, or a framework?
- **Stack**: What components comprise a harness, and what does each one do?
- **Categories**: Which harness type fits which problem?
- **Control**: How does the harness steer agent behavior before and after actions?
- **Engineering**: How does the harness improve through observed failures?
- **Security**: How does the harness enforce permissions and trust boundaries?
- **Design**: How should practitioners make harness design decisions?

---

## The Short Version

**Default to an existing full agentic harness for most work.** Building a harness from scratch is a significant investment. Claude Code, Codex, and equivalent tools ship with Anthropic's harness defaults already set. Start there.

**When an agent fails repeatedly, the first audit target is the harness, not the model.** Raschka's diagnostic: "much of apparent model quality is really context quality." Context quality is a harness concern.

**Instrument the harness for trajectory capture from the beginning — this is infrastructure, not a nice-to-have.** Every session run through a well-instrumented harness produces training data, evaluation data, and edge-case documentation. The compounding effect accumulates over months.

---

## Connections

- **To [Foundations](../1-foundations/_index.md):** The harness is the fifth pillar, joining Prompt, Model, Context, and Tool Use. The Twelve Leverage Points chapter identifies execution-layer levers that map directly to harness components.
- **To [Prompt](../2-prompt/_index.md):** The harness manages prompt shape — the stable/dynamic split that enables cache reuse. Prompt engineering operates within constraints the harness establishes.
- **To [Model](../3-model/_index.md):** Harness and model capability are additively independent (Agent Psychometrics formula). Harness investment yields gains regardless of model selection.
- **To [Context](../4-context/_index.md):** Context management is a harness component. The context chapter covers strategies in depth; this chapter owns the architectural framing.
- **To [Tool Use](../5-tool-use/_index.md):** Tool access is a harness concern — the harness defines what the agent can do. Tool design principles operate within the access layer the harness enforces.
- **To [Patterns](../7-patterns/_index.md):** The Orchestrator Pattern, Autonomous Loops, and ReAct patterns all describe execution flows that harnesses implement. The harness is the runtime that makes patterns operational.
- **To [Practices](../8-practices/_index.md):** Debugging, cost management, and production operations are all harness-level concerns. The practices chapter provides operational depth; this chapter provides the conceptual foundation.
- **To [Practitioner Toolkit](../10-practitioner-toolkit/_index.md):** Specific harness implementations — Claude Code, Google ADK, LangGraph — are evaluated in the Toolkit chapter. This chapter provides the framework for evaluating them.
