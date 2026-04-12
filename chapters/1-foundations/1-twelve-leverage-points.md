---
title: Twelve Leverage Points of Agentic Coding
description: A hierarchy of intervention points for improving agentic systems, from low to high leverage
created: 2025-12-08
last_updated: 2026-04-11
tags: [foundations, leverage-points, framework, agenticengineer, anti-patterns, engineer-behavior, failure-modes]
source: https://agenticengineer.com
part: 1
part_title: Foundations
chapter: 1
section: 1
order: 1.1.1
---

# Twelve Leverage Points of Agentic Coding

A framework for understanding where to intervene in agentic systems. The hierarchy follows Donella Meadows' "Places to Intervene in a System" pattern—lower numbers indicate higher leverage points that affect the entire system. Changes at the top (#1-#4) cascade throughout the system; changes at the bottom (#9-#12) produce local fixes.

> **Guiding philosophy:** "One agent, one purpose, one prompt."

---

## The Hierarchy

AI Developer Workflows (ADWs) define how work flows between agents in multi-agent systems—the highest leverage intervention point in the framework.

| # | Leverage Point | Core Question |
|---|----------------|---------------|
| 12 | [Context](#12-context) | What does the agent actually know? |
| 11 | [Model](#11-model) | What tradeoffs exist: cost, speed, intelligence? |
| 10 | [Prompt](#10-prompt) | Are instructions concrete and followable? |
| 9 | [Tools](#9-tools) | What actions can agents take, and in what form? |
| 8 | [Standard Out](#8-standard-out) | Can agents and operators see what's happening? |
| 7 | [Types](#7-types) | Is typing consistent and enforced? |
| 6 | [Documentation](#6-documentation) | Can agents navigate and trust the documentation? |
| 5 | [Tests](#5-tests) | Are tests helping agents or just theatre? |
| 4 | [Architecture](#4-architecture) | Is the codebase agentically intuitive? |
| 3 | [Plans](#3-plans) | Can agents complete tasks without further input? |
| 2 | [Templates](#2-templates) | Do agents know what good output looks like? |
| 1 | [ADWs](#1-adws-ai-developer-workflows) | How does work flow between agents? |

---

## Low Leverage (Local Fixes)

### 12. Context

What does the agent actually know? What's in the context window? Is it all necessary? Will the current context increase the likelihood of outputting the next token correctly?

**Key considerations:**
- How to audit what's actually in an agent's context at decision time
- The cost of irrelevant context and methods to measure it
- How to distinguish "necessary" context from "nice to have" information
- Processes for pruning context that isn't contributing to outcomes

**Example: Good vs. Poor Context Management**

Poor: Loading entire documentation suite into context for every task (500k+ tokens, most irrelevant).

Good: Loading only the relevant module documentation based on task requirements (15k tokens, high signal-to-noise ratio).

---

### 11. Model

What model is the system using? How capable is it? What tradeoffs exist? Cost vs speed vs intelligence.

**Key considerations:**
- How to map tasks to the right model tier
- Identifying cases where over-indexing on capability wastes resources
- Staying current on model capabilities without constant churn

**Example: Good vs. Poor Model Selection**

Poor: Using GPT-4o for simple text extraction tasks that GPT-3.5-turbo handles perfectly (10x cost overhead).

Good: Using Claude 3.5 Sonnet for complex code generation, GPT-3.5-turbo for text classification (matched capability to task requirements).

---

### 10. Prompt

What instructions does the agent have? Are they concrete? Can they be followed properly?

**Key considerations:**
- What makes a prompt "concrete" vs. "vague"
- How to test whether a prompt can actually be followed
- Setting quality bars for prompts before looking elsewhere for the problem

**Example: Good vs. Poor Prompting**

Poor: "Make this code better" (vague, no success criteria).

Good: "Refactor the authentication module to use dependency injection. Extract the token validation logic into a separate class. Maintain existing test coverage." (concrete, testable, clear success criteria).

---

### 9. Tools

What actions can agents take? What form are these tools available in? Internal tooling vs MCP vs CLI vs something else.

**Key considerations:**
- How to decide between internal tools, MCP servers, and CLI wrappers
- The tradeoff between tool flexibility and tool reliability
- When tool limitations become the bottleneck

---

## Medium Leverage (System Properties)

### 8. Standard Out

Can agents (and operators) actually SEE what the code is doing? Is it being output centrally? Is it self-documenting? Do logs have clear sources and descriptions?

**Key considerations:**
- How observable agent systems are
- What information is missing from current visibility
- How to balance verbose logging with signal-to-noise ratio
- What "self-documenting" output looks like in practice

---

### 7. Types

Is the codebase typing consistent? Are agents aware of it? To what extent is it being policed and are the agents informed if/when they make infractions?

**Key considerations:**
- How strong typing helps agents write correct code
- How to surface type errors to agents in actionable ways
- The relationship between type coverage and agent success rate

---

### 6. Documentation

Where is the documentation? What's in there? Can agents easily navigate? Is it out of date? Is it being updated constantly? Is it "self-improving"?

**Key considerations:**
- What makes documentation "agent-navigable"
- How to keep docs in sync with code when agents are making changes
- What "self-improving" documentation looks like in practice
- Where agents look for documentation, and whether that's where it lives

---

### 5. Tests

What are the tests doing and how do they help agents? Are agents conducting "testing theatre"? Is testing using mock implementations or running against actual code?

**Key considerations:**
- The difference between tests that help agents and tests that don't
- How to detect "testing theatre" (tests that pass but don't verify anything real)
- When mocks help and when they hide problems
- How test failures guide agent behavior

**Evaluation hierarchy and testing theatre.** Husain's three-level evaluation hierarchy maps directly to the testing theatre risk this leverage point names [Husain, 2024]. Level 1 evaluations—deterministic assertions per component, run on every code change—must be established before scaling to Level 2 (model-based evaluations using LLM-as-judge). Teams that skip Level 1 and build Level 2 evaluations on a shaky component foundation produce evaluations that cannot distinguish component failures from system failures. The result is exactly testing theatre: evaluations that run, produce numbers, and inform nothing. Establish per-component behavioral assertions first; model-based evaluations on top of a solid Level 1 foundation become actionable signal. See [Evaluation](../7-practices/2-evaluation.md) for the full error analysis and annotation methodology.

**Source:** Husain, H. "Your AI Product Needs Evals." hamel.dev/blog/posts/evals/ (2024-03-29)

---

## High Leverage (Structural Changes)

### 4. Architecture

What patterns does the codebase follow? How is it structured? Is it "agentically-intuitive"—following historically-popular structures with higher likelihood of existing in training data?

**Key considerations:**
- What makes an architecture "agentically intuitive"
- How to balance "what's in training data" with "what's right for the problem"
- What architectural patterns agents handle well vs. poorly
- How much codebase structure affects agent success

---

### 3. Plans

Plans are MASSIVE prompts passed to an agent with the expectation that no more user interaction must happen in that session for the agent to finish its task.

**Key considerations:**
- What makes a plan "complete enough" to run without human intervention
- How to scope a plan—what's too big, what's too small
- How to handle plans that fail partway through
- The relationship between plan quality and task success
- How to write plans that are robust to unexpected situations

---

### 2. Templates

Do agents know what docs, code, prompts, etc. should look like? Are prompts/plans reusable? Are templates structured consistently? Are all elements necessary?

**Key considerations:**
- What templates get used most frequently
- How to decide what goes in a template vs. what's generated fresh
- How to prevent templates from becoming bloated over time
- The relationship between template quality and output consistency

---

### 1. ADWs (AI Developer Workflows)

How does work carry between agents? How are multiple agents working together to accomplish a shared goal? To what extent are ADWs deterministic (based in code) vs. stochastic/agentic (using orchestrator agent, agents can invoke other agents, etc.)?

**Key considerations:**
- What ADWs have been built or used, and what worked
- How to decide between deterministic workflows and agentic orchestration
- The handoff problem between agents and how to solve it
- When multi-agent coordination helps vs. adds unnecessary complexity
- How to debug workflows that span multiple agents

---

## Connections

- **To [Core Four](_index.md):** The twelve leverage points expand on the four pillars. Context (#12), Model (#11), Prompt (#10), and Tools (#9) map directly to the core four. The higher leverage points (Plans, Templates, ADWs) represent system-level patterns built on top of the pillars.

- **To [Evaluation](../7-practices/2-evaluation.md):** Each leverage point requires different evaluation approaches. Low leverage points (context, model, prompt) can be evaluated per-task. High leverage points (architecture, templates, ADWs) require system-level metrics across multiple tasks.

- **To [Patterns](../6-patterns/_index.md):** ADWs (#1) and Plans (#3) directly correspond to the orchestrator and plan-build-review patterns. Templates (#2) enable self-improving expert patterns.

- **To [Prompt Maturity Model](../8-mental-models/2-prompt-maturity-model.md):** The Engineer Leverage Progression section maps practitioner framing stages to leverage levels. Stage 2 framing is what makes Plans (#3) and ADWs (#1) accessible; Stage 3 is what makes Tests (#5) and measurement infrastructure actionable. The anti-pattern catalog identifies the specific failure behaviors that occur when framing lags behind leverage ambition.

---

## Notes on the Source

This framework comes from [agenticengineer.com](https://agenticengineer.com). The hierarchy draws from Donella Meadows' "Leverage Points: Places to Intervene in a System" (1999), applying systems thinking to agentic engineering.

---

## Anti-Patterns by Leverage Level

*[2026-04-11]*: The per-level examples above describe correct use. This section catalogs named failure behaviors — practitioner-observed anti-patterns that corrupt specific leverage points regardless of individual prompt or tool quality. Four independent practitioners (Liu [1], Hamel [4][5][6], Willison [7]) converge on a common diagnosis: the gap between high- and low-leverage AI engineers is behavioral, not technical. These anti-patterns map that behavioral gap onto the existing hierarchy.

Organization follows the three leverage tiers from the hierarchy above.

### Low Leverage Anti-Patterns (#9–#12)

**Isolated Prompting** *(corrupts #12 Context)*

An engineer provides fragment context — meeting notes, a code snippet, a requirements section — without supplying architectural context or codebase integration. The model produces a syntactically coherent output (a plan, a design, a ticket set) that is semantically disconnected from the actual system.

Why it fails at #12: Context (#12) is the lowest leverage point, but corrupting it prevents all higher leverage points from functioning. A plan built on fragment context will fail execution regardless of ADW quality at #1. The anti-pattern is not visible at the prompt level — the prompt may be well-structured. The failure is upstream: what is provided to the prompt, not how the prompt is written.

*Distinguishing feature:* Not a prompt quality problem. Isolated Prompting can occur with a sophisticated L4-L5 prompt. The failure is the context selection decision that precedes the prompt.

*Source:* Liu [1] — the "David" example: full meeting transcript pasted without codebase context produces a plan disconnected from actual architecture. Contrast with the "Elena" example: full transcript plus codebase enables complete, executable workflow.

---

**Tool Proliferation** *(corrupts #9 Tools)*

An engineer launches parallel agent instances — Liu's example is 15 coding agents in 15 separate terminal windows — each operating independently on the same codebase without shared state, integration plan, or designated synthesis step.

Why it fails at #9: Tools (#9) only produce value when their outputs integrate. Parallel agent chaos generates the appearance of activity while producing unintegrated, often conflicting outputs. The reconciliation cost to the engineer frequently exceeds what a single well-directed agent would have required.

*Distinguishing feature:* Not an orchestration failure at #1 (ADWs) — there is no orchestration. The failure is the absence of a workflow design: treating parallel tool invocations as equivalent to a designed multi-agent workflow.

*Source:* Liu [1] — "15 coding agents in 15 separate terminal windows creating illusion of work rather than integrated output."

---

### Medium Leverage Anti-Patterns (#5–#8)

**Testing Theatre** *(corrupts #5 Tests)*

A team implements evaluation frameworks without domain expert alignment, then optimizes toward generic benchmark pass rates. Product-specific failure modes go undetected as aggregate scores improve.

Why it fails at #5: Generic evals create false confidence. Tests (#5) help agents when they surface real failure modes; they become theatre when optimized for a benchmark disconnected from product behavior. The tests run and pass — the failure is test selection, not test implementation.

*Distinguishing feature:* Not a test implementation failure. The tests execute correctly. The failure is the judgment about which targets the tests measure: high-ROI product-specific evals vs. low-value off-the-shelf benchmarks.

*Source:* Hamel [4] — "Create evals customized to your product that provide immediate value." The novice-to-expert gap in eval work centers on distinguishing high-ROI work from low-value busywork. Manual error analysis before automated optimization is the corrective.

---

**Metric Over-Aggregation** *(corrupts #5 Tests, measurement layer)*

Distinct failure types are lumped into a single aggregate metric. Two hallucination subtypes — fabricated facts versus invented user actions — collapsed into one "hallucination score" cause real failures to go undetected as the aggregate improves while a subtype worsens.

Why it fails at #5: Granularity is the skill in eval work. An aggregate metric that improves may mask a worsening subtype. The test infrastructure (#5) operates correctly but cannot surface the failure mode that matters when discrimination is insufficient.

*Distinguishing feature:* Not missing tests — tests exist and run. The failure is insufficient resolution in what the tests measure.

*Source:* Hamel [5] — coding agent eval example where hallucination types require separate metrics to catch regressions in individual subtypes.

---

### High Leverage Anti-Patterns (#1–#4)

**Design Delegation** *(corrupts #3 Plans and #4 Architecture)*

An engineer defers architectural decisions to AI during implementation, reasoning that "refactoring is cheap." The codebase accumulates structural choices the engineer never explicitly made and therefore cannot fully reason about. When the engineer attempts to write a plan (#3), the plan cannot be complete because the architecture (#4) was not consciously designed.

Why it fails at #3-#4: Plans (#3) require the engineer to understand the architecture (#4) well enough to write a complete, executable specification. Delegating architecture decisions during implementation forecloses both: plans cannot be complete without understood architecture, and architecture cannot be evaluated if its decisions were never consciously made.

*Distinguishing feature:* Not a plan quality failure — the engineer may not be writing plans at all. The failure is the framing: treating AI as architect rather than as implementer executing human-defined design.

*Source:* Willison [7] — "Deferring decisions corroded my ability to think clearly because the codebase stayed confusing." "Implementation has a right answer. Design doesn't." The corrective is explicit design authority: "lead on design, delegate on implementation."

---

**Post-Hoc Learning** *(corrupts #1 ADWs)*

Engineers acquire AI tooling knowledge exhausted, between meetings, through ad-hoc usage rather than structured learning programs. Point knowledge of individual tools accumulates without the workflow design literacy that connects tools into ADWs.

Why it fails at #1: ADWs (#1) represent how work flows between agents — the highest-leverage intervention. Point-tool knowledge does not produce workflow design capability. Engineers who learn tools in isolation can use individual tools competently but cannot design ADWs that compose them, because ADW design is a different skill than tool operation.

*Distinguishing feature:* Not a tool knowledge problem — the engineer may know the individual tools well. The failure is the absence of workflow design literacy, which structured learning (not ad-hoc usage) develops.

*Source:* Liu [1] — Post-Hoc Learning named as a specific anti-pattern; structured AI Coding Accelerator programs as the corrective.

---

**Automated Optimization Before Understanding** *(corrupts #2 Templates and #1 ADWs)*

Teams delegate prompt optimization to automated tools before developing manual understanding of failure modes. Automated hill-climbing on static metrics refines known failures but cannot discover new failure modes; practitioners never develop the judgment to determine whether the target metric is correct.

Why it fails at #1-#2: Templates (#2) and ADWs (#1) must encode practitioner judgment about what good output looks like. Automated optimization produces locally optimal outputs on the target metric; it cannot encode the judgment that determines whether the target metric is the right one. The prompt may improve by the metric while failing in ways the metric cannot detect.

*Distinguishing feature:* Not automation that is wrong in principle — automated optimization has its place in Stage 3 [Engineer Leverage Progression](../8-mental-models/2-prompt-maturity-model.md). The failure is sequencing: automation at Stage 1 forecloses the manual learning that produces judgment.

*Source:* Hamel [6] — "Good writing is good thinking. Prompt engineering requires active intellectual engagement; delegating early means never fully understanding requirements or failure modes." Manual prompt writing is the first stage, not an intermediate step to automate away.

---

| Anti-Pattern | Corrupts | Failure Type | Source |
|---|---|---|---|
| **Isolated Prompting** | #12 Context | Missing context, not missing prompt skill | Liu [1] |
| **Tool Proliferation** | #9 Tools | Parallel instances without integration plan | Liu [1] |
| **Testing Theatre** | #5 Tests | Generic evals optimized on wrong target | Hamel [4] |
| **Metric Over-Aggregation** | #5 Tests | Insufficient failure-mode discrimination | Hamel [5] |
| **Design Delegation** | #3-#4 Plans/Architecture | AI as architect, not implementer | Willison [7] |
| **Post-Hoc Learning** | #1 ADWs | Point-tool knowledge without workflow design | Liu [1] |
| **Automated Optimization Before Understanding** | #1-#2 ADWs/Templates | Judgment outsourced before developed | Hamel [6] |

The [Engineer Leverage Progression](../8-mental-models/2-prompt-maturity-model.md) section maps these anti-patterns to practitioner framing stages: Isolated Prompting and Tool Proliferation are Stage 1 behaviors applied to system components; Design Delegation and Post-Hoc Learning indicate Stage 1-2 framing applied to Stage 3 leverage points. Testing Theatre and Metric Over-Aggregation are treated in depth in the [Evaluation](../7-practices/2-evaluation.md) chapter.

**Sources**
- [1] Liu, Jason. "Do Your Engineers Know How to Leverage AI?" 2025-09-11.
- [4] Hamel Husain. "AI Evals For Engineers & Product Managers." 2026-01-23.
- [5] Hamel Husain. "Evals Skills for Coding Agents." 2026-04-11.
- [6] Hamel Husain. "Should I Stop Writing Prompts Manually?" 2025-09-19.
- [7] Willison, Simon. "Eight years of wanting, three months of building with AI." 2026-04-05.

