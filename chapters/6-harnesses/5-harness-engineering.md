---
title: Harness Engineering
description: Hashimoto's methodology for iteratively improving the harness — the practice of making agent mistakes unrepeatable
created: 2026-04-12
last_updated: 2026-04-12
tags: [foundations, harness, engineering, hashimoto, iteration, improvement, methodology]
part: 1
part_title: Foundations
chapter: 6
section: 5
order: 1.6.5
---

# Harness Engineering

Harness engineering is a practice, not a one-time design decision.

Mitchell Hashimoto coined the term on February 5, 2026, to name a discipline that leading practitioners were already following without a shared vocabulary: when an agent makes a mistake, engineer the surrounding system so that mistake cannot recur. The naming matters because it shifts the practitioner's default response from prompt patching (adding warnings to the prompt) to structural improvement (changing the system).

---

## Hashimoto's Core Principle

"Anytime an agent makes a mistake, take the time to engineer a solution such that the agent never makes that mistake again."

This principle contains two non-obvious claims worth unpacking.

**"Take the time"** — harness engineering is a deliberate investment, not a reactive patch. The reflex after an agent failure is to add a sentence to the prompt warning the agent not to make that mistake. This is fast and often feels like a fix. Hashimoto's framing rejects this reflex. The prompt patch takes 30 seconds; the harness engineering takes 30 minutes. The prompt patch might reduce the frequency of the mistake; the harness fix makes the mistake structurally impossible.

**"Such that the agent never makes that mistake again"** — the target is structural prevention, not probability reduction. A prompt instruction can reduce the likelihood of a mistake. A harness constraint can make it impossible. These are categorically different outcomes for production systems.

### The Prompt Patching Reflex

Prompt patching is the dominant response to agent failure in the field because it is fast and because it sometimes works:

| Scenario | Prompt Patch | Harness Engineering |
|---------|-------------|-------------------|
| Agent writes to /tmp instead of /workspace | Add "always write files to /workspace/" to system prompt | Permission filter: intercept all writes, enforce /workspace/ prefix |
| Agent commits without linting | Add "always run linting before committing" to system prompt | Lint-before-commit hook: intercept commit tool call, run linter |
| Agent includes redundant file reads | Add "avoid reading the same file twice" to system prompt | Deduplication: harness detects duplicate reads, returns cached content |
| Agent produces inconsistent output format | Add format specification to system prompt | Schema validator: reject outputs that don't match required format |

Prompt patches are fragile for a structural reason: they depend on the agent consistently following the instruction, in every context, across every turn. Instructions compete with each other for influence. Long system prompts with many warnings dilute each individual warning. And because the agent is a probabilistic system, consistent instruction-following is never guaranteed.

Harness engineering is durable because it does not depend on instruction-following. The permission filter runs as code, not as a request. The lint hook executes before the commit tool reaches the filesystem. The deduplication logic operates at the context management layer, invisible to the agent. These constraints apply deterministically regardless of what the agent intends.

---

## The Harness Engineering Loop

Systematic harness engineering follows a six-step cycle:

### Step 1: Observe

An agent makes a mistake in production or evaluation. The observation captures:
- What the agent did (the action taken)
- What the agent should have done (the intended action)
- The context in which the mistake occurred (task type, turn number, prior actions)
- The consequences (cost, incorrect output, failed verification)

Observation quality determines the quality of the subsequent fix. Incomplete observations produce incomplete fixes that address the symptom rather than the cause.

### Step 2: Classify

Not every agent failure is a harness failure. Accurate classification is the difference between a productive fix and wasted engineering effort.

| Failure Type | Symptom | Correct Response |
|-------------|---------|-----------------|
| Model failure | Agent cannot reason about the task type regardless of harness quality | Model upgrade or task decomposition |
| Context failure | Agent has the capability but lacks the information to apply it | Improve workspace context or context management |
| Prompt failure | Agent misinterprets the instruction consistently and predictably | Prompt revision or additional examples |
| Harness failure | Agent succeeds in isolated testing but fails in the execution environment | Harness engineering |
| Tool failure | Agent reasons correctly but tool output is unreliable or malformed | Tool design revision |

The classification requires honest assessment: practitioners who default to "model failure" for every mistake will over-invest in model upgrades. Practitioners who default to "harness failure" will over-engineer the harness for problems that belong at the prompt layer.

**Classification heuristics:**
- If the failure is consistent across different models → likely prompt or harness failure
- If the failure disappears when the agent has more context → context failure
- If the failure disappears when the tool output is manually simplified → tool failure
- If the failure disappears in isolated tests but recurs in production → harness failure
- If the failure cannot be reproduced → probabilistic model failure

### Step 3: Locate

Within the harness, identify which component is responsible. The [six-component taxonomy](2-harness-stack.md) provides the diagnostic framework:

| Component | Failure Signature |
|-----------|-----------------|
| Workspace context | Agent acts on stale or incorrect environmental information |
| Prompt shape | Cache invalidation errors; token costs higher than expected |
| Tool access | Agent attempts to use tools outside its permission scope |
| Context management | Context degradation across turns; agent "forgets" earlier decisions |
| Session memory | Agent repeats completed work; working memory inconsistent with state |
| Subagent delegation | Subagent outputs not properly integrated; scope violations |

### Step 4: Engineer

With the component located, the engineering response maps to Fowler's guide/sensor framework:
- If the mistake is an action the agent should not take → add a guide (prevent the action)
- If the mistake is an error in reasoning after observing results → add a sensor (better feedback)
- If the mistake is missing information → improve the relevant stack component

The engineering choice should address the classified failure type, not just the immediate symptom. A context failure addressed by tightening permission filters (a harness fix) will recur when the context failure manifests in a different way. Fix the right component.

### Step 5: Verify

Confirm that the fix eliminates the failure:
- Reproduce the original failure case after the fix is applied
- Run the evaluation suite that covers the task category
- Check that the fix does not introduce new failure modes (regressions)

Verification prevents the accumulation of harness changes that each fix individual cases but collectively produce unexpected interactions. A harness that has accumulated 50 unverified fixes is brittle — any individual change might be correct, but the interactions are uncharted.

### Step 6: Generalize

Identify whether the fix applies to similar mistakes across the system. A permission filter added for filesystem writes may also be needed for network writes. A lint hook for Python commits may also be needed for TypeScript commits. A deduplication fix for file reads may also address other tool output deduplication gaps.

Generalization is what distinguishes systematic harness engineering from ad-hoc patching. Each individual fix is an opportunity to improve a pattern, not just resolve an instance.

---

## Concrete Before/After Examples

### Example 1: Unauthorized File Deletion

**Observation:** An agent deleted a configuration file during a refactoring task. The file was not in the task scope, but the agent inferred (incorrectly) that it was safe to remove.

**Classification:** Harness failure — the agent had permission to delete files within the project scope, and the configuration file was within that scope. The agent's reasoning was wrong, but the harness allowed the action.

**Prompt patch response:** "Always confirm with the user before deleting any file that is not explicitly in the task scope."

**Harness engineering response:** Add a computational guide — a pre-action classification prompt that triggers for all delete operations and requires explicit task-scope confirmation before the delete tool executes. For files matching a "configuration file" pattern (`.env`, `*.yaml`, `*.toml`, `*.json` at the project root), require human confirmation regardless of scope classification.

**Durability:** The prompt patch depends on the agent consistently applying the rule. The harness guide intercepts every delete call — the agent cannot bypass it by reasoning that the file is in scope.

### Example 2: Context Rot in Long Sessions

**Observation:** In sessions exceeding 40 turns, agents began producing outputs inconsistent with earlier decisions in the same session. An architectural decision made at turn 10 was contradicted by implementation choices at turn 35.

**Classification:** Context failure — the harness was allowing context to accumulate without compression. By turn 35, the early turns containing the architectural decision had been displaced from the effective reasoning window by accumulated tool outputs.

**Prompt patch response:** "Periodically review your earlier decisions to ensure consistency."

**Harness engineering response:** At 40% context fill, the harness triggers a context summarization step that:
1. Extracts key decisions from the full transcript
2. Produces a structured "decision log" summarizing decisions with their rationale
3. Injects this decision log into working memory
4. Compresses the transcript to recency-biased summary

**Durability:** The prompt patch depends on the agent voluntarily reviewing its history. The harness-triggered compaction ensures the decision log exists regardless of agent behavior. The architectural decision survives context compression because it was extracted and explicitly preserved.

### Example 3: Subagent Scope Violation

**Observation:** A subagent tasked with implementing a utility function modified the main application entry point — outside its assigned scope — because it determined the entry point needed updating for the utility to work correctly.

**Classification:** Harness failure — the subagent delegation was under-scoped. The subagent received a task but not explicit file-scope restrictions.

**Prompt patch response:** "Subagents should only modify files within their assigned scope."

**Harness engineering response:** Enforce scope at delegation time. Every subagent spawn includes explicit file-scope parameters. The harness intercepts all write tool calls from the subagent and verifies the target file is within the authorized scope. Out-of-scope write attempts are blocked and returned as structured errors with the scope constraints.

**Durability:** The prompt instruction could be overridden by the subagent's own reasoning about what is necessary. The harness enforcement cannot be overridden — write calls outside scope fail at the enforcement layer.

---

## Trajectory Capture as Competitive Advantage

Philipp Schmid's framing (~2026-Q1): "The Harness is the Dataset."

Every agent session run through a harness produces trajectory data — sequences of observations, actions, and outcomes. These trajectories are simultaneously:

- **Training data**: observation-action-outcome sequences for fine-tuning domain-specific models
- **Evaluation data**: before/after comparisons for measuring harness improvement over time
- **Edge case documentation**: records of failure modes specific to the practitioner's domain that no benchmark captures

**The compounding mechanism:** A well-instrumented harness run for six months produces more relevant training data than a curated benchmark dataset — because it captures the specific tasks, failure modes, and improvement history relevant to the practitioner's domain. The dataset accumulates while the model remains constant. Each harness engineering cycle adds a failure case and its resolution to the trajectory record.

**Practical trajectory instrumentation:**

| Instrumentation Layer | What to Capture | Storage |
|--------------------|----------------|--------|
| Tool call level | Every tool invocation: name, inputs, outputs, timestamp, success/failure | Append-only log |
| Session level | Task specification, completion status, duration, turn count | Session metadata file |
| Harness engineering level | Failure classifications, component diagnoses, engineering responses | Engineering log |
| Evaluation level | Before/after performance on relevant task categories | Evaluation results |

**Minimum viable trajectory capture:**
1. Log every tool call with inputs and outputs
2. Log session-level success/failure outcome
3. Tag trajectories with task type for later retrieval

The competitive advantage of trajectory capture is asymmetric: organizations that instrument their harness from the beginning accumulate a proprietary dataset. Organizations that add instrumentation later lose the early sessions that often contain the most valuable failure cases — the ones that occurred before the harness was well-engineered.

---

## The Harness as Institutional Infrastructure

The harness encodes organizational judgment about what correct behavior looks like.

Each harness engineering decision embeds a constraint that outlasts the conversation in which it was made. A permission filter that prevents filesystem access outside the working directory represents a judgment: agents in this organization do not operate on files outside the project scope. A lint-before-commit hook represents a judgment: this organization considers linting a prerequisite to code quality, not optional cleanup. These judgments survive individual conversations, personnel changes, and model upgrades — they are in the harness.

This framing has two implications for practitioners:

**1. Harness engineering is team infrastructure, not individual tooling.** A harness that one engineer has carefully engineered for their workflow is an asset to the whole team. The team that treats harness improvement as infrastructure investment — with review, documentation, and shared ownership — compounds the advantage. The team that treats it as individual configuration accumulates local expertise that cannot be shared.

**2. Harness design reflects organizational values about agent behavior.** The constraints in the harness are the organization's definition of acceptable agent behavior. An organization that has no permission enforcement in its harness is implicitly accepting that agents can operate on any system resource. An organization that has no verification in its harness is implicitly accepting agent outputs without quality checks. The harness is where stated values about safe, reliable agentic work are either implemented or not.

Connect to: [Software Factories](../9-mental-models/7-software-factories.md) for the organizational framing of agentic engineering infrastructure; [Design as Bottleneck](../9-mental-models/6-design-as-bottleneck.md) for the upstream specification dependency that harness engineering reduces but does not eliminate.

---

## Production Evidence

The SWE-bench production analysis (tianpan.co, 2026-04-09) provides calibrated evidence for the value of verification-driven harness engineering.

**The measurement paradox:** Production deployments of coding agents showed 98% more PRs merged but 91% increase in review time. Throughput increased while quality degraded — because the harness provided no verification layer. Agents completed tasks and submitted PRs without automated confirmation that the implementation was correct. Human reviewers absorbed the verification cost.

**The verification-driven finding:** The highest-leverage production practice is giving agents explicit, binary verification signals — failing tests they must pass, linter configurations to satisfy, integration test suites for self-evaluation. When the harness provides binary computational sensors, agents can self-correct without human intervention. When the harness provides no sensors, humans become the sensors.

**The METR controlled study:** METR found that AI tools made experienced developers 19% slower in controlled tasks — despite developers subjectively reporting 20% improvement. The pattern: narrow, mechanical tasks with clear verification (harness sensors available) succeed; cross-system architectural work fails. The success/failure distinction maps cleanly to harness sensor availability.

These findings support harness engineering as the highest-return investment in agentic systems. The systems that perform well in production are the ones with verification-driven harnesses, not the ones with the most capable models.

---

## Connections

- **[The Harness Stack](2-harness-stack.md)** — The six components that harness engineering acts on
- **[Harness as Control System](4-harness-as-control-system.md)** — Guides and sensors: the mechanisms harness engineering adds
- **[Security, Permissions, and Trust](6-security-permissions-trust.md)** — Security engineering as a category of harness engineering
- **[Software Factories](../9-mental-models/7-software-factories.md)** — Organizational framing for harness as institutional infrastructure
- **[Design as Bottleneck](../9-mental-models/6-design-as-bottleneck.md)** — Upstream specification dependency
- **[Debugging Agents](../8-practices/1-debugging-agents.md)** — Operational debugging practices that feed the engineering loop
