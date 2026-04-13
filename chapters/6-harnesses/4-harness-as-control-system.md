---
title: Harness as Control System
description: Fowler's guides and sensors decomposition — feedforward and feedback control in agentic systems
created: 2026-04-12
last_updated: 2026-04-12
tags: [foundations, harness, control-theory, fowler, guides, sensors, feedforward, feedback]
part: 1
part_title: Foundations
chapter: 6
section: 4
order: 1.6.4
---

# Harness as Control System

The harness is not passive scaffolding — it is an active control system.

Martin Fowler's decomposition (~2026-Q1) treats the harness as a set of mechanisms that steer agent behavior before and after each action. Every agent action is bounded by mechanisms that intervene upstream (before the action, to constrain what the agent can do) and downstream (after the action, to observe results and adjust subsequent behavior). A harness with only one direction of steering is incomplete. A harness with no steering is a liability.

---

## Fowler's Guides/Sensors Framework

Fowler identifies two primary harness mechanisms:

- **Guides** (feedforward): intervene before the agent acts to constrain the action space
- **Sensors** (feedback): observe action results and steer subsequent agent behavior

Each mechanism can be implemented in two modes:

- **Computational**: deterministic code (lint rules, permission checks, schema validators)
- **Inferential**: model-based judgment (review agents, planning agents, self-reflection prompts)

This produces a four-quadrant taxonomy that maps most harness control mechanisms to a precise location:

| Mechanism | Computational | Inferential |
|-----------|--------------|------------|
| **Guide** (feedforward) | Permission filter, lint-before-commit hook, schema validator, path allowlist | Planning agent, spec review agent, pre-action classification prompt |
| **Sensor** (feedback) | Test runner results, CI signals, compiler output, linter output | Self-reflection prompt, error diagnosis agent, evaluation/judge agent |

Every harness control mechanism belongs to one of these four quadrants. Understanding which quadrant a mechanism occupies clarifies its cost, reliability, and appropriate use case.

---

## Guides in Practice

Guides are the harness mechanisms that shape agent behavior before any action executes. They are the preventive layer — they either prevent bad actions from occurring or shape the action space so that good actions are the path of least resistance.

### Computational Guides

**Permission filters:** The harness intercepts every tool call and checks whether the agent has permission to execute it. A permission filter for filesystem access might enforce: the agent may write to `/workspace/` but not to `/etc/` or any path outside the working directory. The filter executes as a pre-hook; if permission is denied, the tool call never reaches the tool. The agent receives a structured error and must adapt its approach.

Implementation note: permission filters must be stateless (checking only the call parameters, not session history) and fast (they execute synchronously on every tool call). Computational guides must not introduce latency that degrades agent throughput.

**Lint-before-commit hooks:** A computational guide that prevents commit until the code passes linting. The harness intercepts the commit tool call, runs the linter, and either permits the commit (lint passes) or blocks it and returns lint output as structured feedback. The agent cannot commit broken code — the structural guarantee comes from the harness, not from a prompt instruction.

This is the defining contrast with prompt-based constraints. A prompt instruction saying "always lint before committing" depends on the agent following instructions consistently. A computational guide intercepts the commit call regardless of what the agent intends — it is enforced, not requested.

**Schema validators:** Tool input schemas are enforced before execution. If the agent calls a file-write tool with a relative path, the schema validator rejects the call and returns a structured error explaining the requirement. The agent never reaches the filesystem with a malformed call. This implements the poka-yoke (mistake-proofing) principle at the tool interface: errors surface at the earliest possible point, before any side effects.

**Allowlists:** Only named tools may execute named commands. A harness might enforce that the shell tool may run `git`, `pytest`, `ruff`, and `python` but not `curl`, `wget`, or `ssh`. The allowlist is maintained in the harness configuration, not in the prompt. Adding a tool to the allowlist is a deliberate harness change with associated review; it cannot be circumvented by prompt manipulation.

### Inferential Guides

**Planning agents:** Before an execution agent begins a task, a planning agent generates a structured plan and validates it against the original specification. The execution agent receives the validated plan rather than the raw specification — the planning agent has already resolved ambiguities, checked feasibility, and confirmed alignment. This is an inferential guide because the planning agent uses model reasoning to produce the constraint rather than applying a deterministic rule.

**Spec review agents:** Before delegating implementation to a subagent, the orchestrator runs a spec review agent that checks whether the specification meets quality criteria: is it complete? Does it have sufficient detail for implementation? Are there internal contradictions? Specifications that fail the review are returned to the user for clarification before any implementation begins. This prevents the expensive failure mode of building the wrong thing.

**Pre-action classification:** Before executing a high-stakes action (deleting files, sending network requests, modifying configuration), a classification prompt asks the model to categorize the action's risk level and confirm that it is consistent with the user's original intent. Low-risk actions bypass classification; high-risk actions require explicit confirmation. The Pit of Success mental model (see [Pit of Success](../9-mental-models/1-pit-of-success.md)) frames this as making correct actions the path of least resistance — the classification prompt makes incorrect high-risk actions visibly wrong before they execute.

---

## Sensors in Practice

Sensors are the harness mechanisms that observe action results and feed observations back into the agent's reasoning. They are the corrective layer — they catch mistakes after they occur and provide the agent with information to self-correct.

### Computational Sensors

**Test results as binary truth:** The harness runs the test suite after each code modification and presents the results (pass count, fail count, error messages) to the agent as structured input. Pass/fail signals are the highest-quality feedback available for software tasks — they are unambiguous, deterministic, and directly actionable. An agent with failing tests knows precisely what to fix.

Verification-driven development (tianpan.co production analysis, 2026-04-09): the highest-leverage production practice is giving agents failing tests they must pass rather than natural language success criteria. A specification that says "implement user authentication that passes these 12 tests" produces dramatically better results than one that says "implement secure user authentication" — because the feedback signal is precise and the success criterion is verifiable.

**CI signals:** Build success, coverage thresholds, security scan results, and performance benchmarks fed back to the agent after each significant change. CI sensors transform the agent's execution into a verification-driven loop: implement → CI → observe → adjust → implement. The harness is responsible for invoking CI and routing its output to the agent.

**Compiler output:** Compiler errors and warnings are structured feedback from deterministic systems. A compilation failure is not ambiguous — the error message specifies the file, line, and nature of the problem. The harness presents compiler output as a sensor observation; the agent uses it to diagnose and fix the issue.

**Linter output:** Similar to compiler output but covering style, convention, and common error patterns. Linter sensors work alongside lint-before-commit guides: the guide prevents bad commits; the linter sensor provides detailed feedback to guide correction.

### Inferential Sensors

**Self-reflection prompts:** After completing a significant action, the harness prompts the agent to evaluate its own output against the original specification: "Review your most recent implementation against these acceptance criteria and identify any gaps." Self-reflection prompts are inferential sensors — they use model reasoning to produce feedback. They handle cases that deterministic sensors cannot: semantic correctness, specification alignment, design quality.

**Error diagnosis agents:** When an agent fails on a task, a specialized error diagnosis agent analyzes the failure: What went wrong? Was it a model failure, a context failure, a harness failure, or a tool failure? The diagnosis agent produces a structured failure report that informs both the immediate recovery attempt and longer-term harness engineering.

**Evaluation/judge agents:** Dedicated agents that score agent outputs against rubrics. An evaluation agent for code quality might assess: correctness (do tests pass?), completeness (are all requirements addressed?), maintainability (is the code readable and well-structured?), and security (are common vulnerabilities avoided?). The scores are sensors that feed back into the agent's reasoning about what to improve.

---

## The Complete Control Loop

An effective harness implements both guides and sensors in a continuous loop:

```
Specification
    ↓
[Guide: Plan agent validates specification]
    ↓
Agent reasons and selects action
    ↓
[Guide: Permission filter + schema validator]
    ↓
Action executes (tool call, code modification, file write)
    ↓
[Sensor: Test runner + linter output]
    ↓
Agent observes results and reasons about next step
    ↓
[Sensor: Self-reflection prompt if quality threshold not met]
    ↓
Next action ... (loop continues until stopping condition)
    ↓
[Sensor: Evaluation agent scores final output]
    ↓
Task complete or escalate to human
```

**Guides without sensors** produce agents that proceed confidently in the wrong direction. The permission filter prevents unauthorized actions, but without test results to confirm correctness, the agent may implement the right-looking wrong solution and never learn of the failure until human review.

**Sensors without guides** produce agents that take expensive wrong actions and then backtrack. Without a planning agent validating the approach upfront, the agent may spend 20 tool calls building the wrong architecture before test failures reveal the misalignment. Sensors catch the failure; guides prevent it.

**Neither guides nor sensors** produces agents that depend entirely on model capability and prompt quality for behavioral correctness — no structural guarantees, no feedback loops, no defense against systematic errors.

---

## Computational vs. Inferential — The Cost Tradeoff

Every guide and sensor mechanism involves a tradeoff between cost, speed, and coverage:

| Dimension | Computational | Inferential |
|-----------|--------------|------------|
| Cost per invocation | Low (microseconds to milliseconds) | High (LLM inference, seconds) |
| Reliability | Deterministic | Probabilistic |
| Coverage | Rules can express | Judgment can assess |
| Maintenance | Explicit rules must be updated | Prompts can be updated more fluidly |
| Failure mode | Rigid (cannot handle edge cases rules don't cover) | Inconsistent (results vary across invocations) |

**Design principle:** Use computational controls wherever the constraint can be expressed as a deterministic rule; use inferential controls for judgment calls that rules cannot anticipate.

Most permission enforcement, schema validation, and test verification should be computational — these concerns have clear, expressible rules and benefit from deterministic enforcement. Code quality assessment, specification alignment, and design evaluation should be inferential — these require judgment that varies by context.

**The inferential overhead**: Adding an inferential guide (planning agent) or inferential sensor (evaluation agent) to a harness adds LLM inference time and cost to each agentic cycle. The tradeoff is real: a planning agent that validates every subtask before execution adds overhead per task but reduces the expensive rework that follows misaligned implementations. The economic question is whether the reduction in failure-mode costs exceeds the increased per-task inference cost.

Production experience from SWE-bench analysis (tianpan.co, 2026-04-09): verification-driven development — providing binary computational sensors (failing tests) — consistently outperforms natural-language success criteria. This suggests that computational sensors are not just cheaper than inferential ones; they are often more reliable. Where a binary test can express the success criterion, it should be preferred over an inferential judge.

---

## Agent Psychometrics and the Harness Independence Principle

The Agent Psychometrics formula (arXiv:2604.00594, 2026-04) provides quantitative support for treating harness control quality as an independent variable:

```
P(success) = σ(θ_LLM + θ_scaffold − β_difficulty)
```

Where:
- `θ_LLM` = LLM capability (fixed for a given model)
- `θ_scaffold` = scaffold (harness) quality
- `β_difficulty` = task difficulty

The additive independence of `θ_LLM` and `θ_scaffold` is the critical insight. Improving harness control quality — adding better guides, adding better sensors, closing the feedforward/feedback loop — yields gains in P(success) independent of model capability. A practitioner cannot improve `θ_LLM` without changing the model. A practitioner can improve `θ_scaffold` through harness engineering.

This independence principle justifies treating the harness control system as a first-class optimization target. Harness improvements compound: each new guide prevents a class of failures; each new sensor catches a class of errors. Over time, a well-engineered harness control system raises P(success) for difficult tasks that would have been unreliable on the same model without the controls.

---

## Common Control System Failures

| Failure | Symptom | Root Cause | Fix |
|---------|---------|-----------|-----|
| No guides | Agent takes action outside intended scope | Missing permission filters or pre-action classification | Add computational permission enforcement |
| No sensors | Agent believes task is complete; output is wrong | No test runner, no evaluation agent | Add computational or inferential feedback |
| Over-gating | Every action requires multiple approvals; throughput collapses | Too many inferential guides in series | Replace inferential guides with computational guides where possible |
| Feedback loop delay | Agent cannot correct because sensor output arrives too late | Sensors run after multi-step sequences rather than per action | Move sensors earlier in the loop |
| Sensor noise | Agent receives contradictory signals from different sensors | Inconsistent evaluation rubrics across inferential sensors | Standardize rubrics; prefer computational sensors for binary criteria |

---

## Connections

- **[The Harness Stack](2-harness-stack.md)** — The six components that guides and sensors operate within
- **[Harness Engineering](5-harness-engineering.md)** — How observed failures become new guides and sensors
- **[Pit of Success](../9-mental-models/1-pit-of-success.md)** — Design philosophy for making correct actions structurally likely
- **[Tool Restrictions](../5-tool-use/3-tool-restrictions.md)** — Tool-level restriction design
- **[Human in the Loop](../7-patterns/6-human-in-the-loop.md)** — When human judgment replaces or augments inferential sensors
- **[Autonomous Loops](../7-patterns/4-autonomous-loops.md)** — Execution patterns that the control system enables and constrains
