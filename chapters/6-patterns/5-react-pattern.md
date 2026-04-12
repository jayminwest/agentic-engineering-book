---
title: ReAct Pattern
description: Interleaved reasoning and action for grounded, observable agent behavior
created: 2026-01-30
last_updated: 2026-04-11
tags: [patterns, reasoning, action, tool-use, grounding, coding-agent, execution-feedback, verification-driven, lint, test-runner, swe-bench]
part: 2
part_title: Craft
chapter: 6
section: 5
order: 2.6.5
---

# ReAct Pattern

An interleaved Reasoning + Acting loop where the agent explicitly reasons about observations before selecting actions, creating a trace of thought that grounds decisions in retrieved evidence.

---

## Core Structure

```
Thought: Analyze the current situation based on observations
Action: Select and invoke a tool
Observation: Record the tool's output
Thought: Analyze the new observation, decide next step
Action: Select next tool (or finish)
...
```

The cycle continues until the task completes or a termination condition triggers.

---

## How It Works

ReAct (Reasoning + Acting) alternates between two modes:

1. **Reasoning** - The model generates explicit thoughts about what it observes and what to do next
2. **Acting** - The model invokes a tool, producing new observations

This interleaving creates three key properties:

**Grounded decisions**: Each action follows explicit reasoning about current observations, reducing hallucination. The model cannot claim to have information it hasn't retrieved.

**Observable traces**: The thought-action-observation chain creates an audit trail. When debugging failures, the trace shows exactly where reasoning diverged from evidence.

**Adaptive behavior**: Each observation can redirect the plan. Unlike fixed multi-step plans, ReAct adjusts based on what tools actually return.

---

## Implementation

### The Basic Loop

```
System Prompt:
You solve tasks by interleaving Thought, Action, and Observation steps.

Thought: Reason about the current state and what to do next.
Action: Call exactly one tool. Format: tool_name(param=value)
Observation: [Tool output will appear here]

Continue until you can answer the task.

Task: {user_task}
```

### Execution Flow

1. Model generates a Thought (explicit reasoning)
2. Model generates an Action (tool call)
3. System executes tool, returns Observation
4. Model generates next Thought based on Observation
5. Repeat until task completes

### Structured Output Variant

For production systems, enforce structure:

```json
{
  "thought": "The user asked about error handling. I should search the codebase for exception patterns.",
  "action": {
    "tool": "grep",
    "params": {"pattern": "except|catch", "path": "src/"}
  }
}
```

The observation appends to context as a system message, triggering the next thought-action cycle.

---

## When to Use

### Good Fit

**Information gathering tasks requiring evidence:**
- Research questions needing multiple sources
- Debugging where the root cause is unknown
- Exploratory analysis across unfamiliar codebases
- Tasks where premature conclusions cause failures

**Task characteristics:**
- Unknown number of steps required
- Each step depends on what previous steps revealed
- Hallucination is high-risk (medical, legal, financial)
- Explainability matters (audit trails, user trust)

### Poor Fit

**Well-defined procedures with known steps:**
- Standard CRUD operations
- Template-based generation
- Tasks where the action sequence is predetermined
- High-throughput scenarios where reasoning overhead is unacceptable

**When simpler approaches work:**
- Single-tool tasks (no interleaving needed)
- Tasks where the model already knows the answer (no retrieval needed)
- Time-critical operations (ReAct adds latency per step)

---

## Trade-offs

| Aspect | ReAct | Direct Generation |
|--------|-------|-------------------|
| **Grounding** | Strong (every claim from observation) | Weak (hallucination risk) |
| **Latency** | Higher (multiple turns) | Lower (single generation) |
| **Token cost** | Higher (thoughts + observations accumulate) | Lower (single response) |
| **Explainability** | Excellent (full trace) | Poor (black box) |
| **Adaptability** | High (adjusts to observations) | Low (fixed plan) |

The trade-off: ReAct sacrifices speed for accuracy and observability.

---

## Comparison with Other Patterns

### ReAct vs. Chain-of-Thought

**Chain-of-Thought (CoT)**: Model reasons step-by-step but generates the full answer before any tool use. Thoughts are internal to a single generation.

**ReAct**: Thoughts interleave with tool calls. Each observation grounds the next thought.

```
CoT: Thought → Thought → Thought → Final Answer
ReAct: Thought → Action → Observation → Thought → Action → Observation → ...
```

CoT excels at reasoning tasks with sufficient in-context knowledge. ReAct excels when external information retrieval is required.

### ReAct vs. Plan-Build-Review

**Plan-Build-Review**: Separates planning from execution. Plan specifies all steps upfront, then build executes.

**ReAct**: No upfront planning. Each step emerges from the previous observation.

| Aspect | Plan-Build-Review | ReAct |
|--------|-------------------|-------|
| **Planning** | Explicit, upfront | Implicit, emergent |
| **Adaptability** | Follows spec | Adjusts per observation |
| **Coordination** | Multiple agents, checkpoints | Single agent loop |
| **Best for** | Complex, multi-phase projects | Exploratory, information-gathering |

Synthesis: Use Plan-Build-Review for tasks with known structure, ReAct for tasks where structure emerges from investigation.

### ReAct vs. Autonomous Loops (Ralph Wiggum)

**Ralph Wiggum**: Iteration-based, with git history as external memory. Fresh context per loop.

**ReAct**: Single context window with accumulated observations.

Ralph suits mechanical tasks with many iterations. ReAct suits reasoning tasks requiring observation synthesis within a session.

---

## Anti-Patterns

### Thought-Free Actions

**Problem**: Skipping the Thought step, invoking tools without explicit reasoning.

**Why it fails**: Without explicit reasoning, the model may invoke tools randomly or redundantly. The trace becomes useless for debugging.

**Solution**: Enforce thought generation before every action. Validate that thoughts reference prior observations.

### Observation Overload

**Problem**: Tools return massive outputs that fill context.

**Why it fails**: Large observations crowd out earlier thoughts and observations. Reasoning quality degrades as context fills.

**Solution**: Limit observation size. Summarize or truncate tool outputs. Consider spawning sub-agents for analysis (see Orchestrator Pattern).

### Infinite Loops

**Problem**: Agent cycles through the same thought-action pairs without progress.

**Why it fails**: Without termination conditions, the loop continues indefinitely, wasting tokens.

**Solution**:
- Limit maximum iterations (10-20 for most tasks)
- Detect repeated actions with same parameters
- Require explicit "Final Answer" action to terminate

### Reasoning Without Evidence

**Problem**: Thoughts that introduce claims not present in observations.

**Why it fails**: The grounding benefit of ReAct evaporates. Hallucination enters through the thought step.

**Solution**: Prompt for evidence-grounded reasoning. Validate that thoughts cite specific observations. Flag unsupported claims.

---

## Production Considerations

### Context Management

ReAct accumulates context rapidly. Each cycle adds thought + action + observation tokens.

**Mitigation strategies:**
- Summarize older observations as context fills
- Spawn fresh agent for new investigation threads
- Limit observation verbosity at the tool level

### Latency Budgeting

Each thought-action-observation cycle requires a model call plus tool execution time.

**For time-sensitive applications:**
- Set maximum iterations based on latency budget
- Parallelize independent tool calls within a single action step (if supported)
- Cache tool results for repeated queries

### Cost Tracking

ReAct's token cost scales with both depth (iterations) and breadth (observation size).

**Cost formula (approximate):**
```
Cost ≈ Σ(thought_tokens + action_tokens + observation_tokens) × iterations
```

For complex tasks requiring 10+ iterations with substantial observations, ReAct can exceed 10× the cost of direct generation. Budget accordingly.

---

## Coding Agent Specialization

*[2026-04-11]*: The observation step is the highest-leverage design surface in a coding agent's ReAct loop. Generic tool outputs (file content, search results) are weak feedback signals; coding-specific outputs (test results, lint errors, compiler diagnostics) are strong ones. The difference determines whether the loop converges or drifts.

### Observation Signal Taxonomy for Coding Loops

| Signal Type | Source | Feedback Quality | Why It Works |
|------------|--------|-----------------|--------------|
| **Test results** | Test runner (pytest, jest, cargo test) | Binary + localized | Pass/fail is unambiguous; failure messages identify the exact assertion |
| **Lint output** | Linter (ruff, eslint, clippy) | Binary + localized | Rule violations are precise; line numbers and rule IDs are actionable |
| **Compiler errors** | Compiler (tsc, gcc, rustc) | Binary + localized | Type errors and missing references cannot be argued with — they are facts |
| **Build output** | Build system | Binary + systemic | Catches integration failures that unit tests miss |
| **Generic tool output** | File reads, search results | Non-binary, interpretive | Requires model judgment; hallucination risk higher |

The practical design principle: prefer binary, localized feedback signals over interpretive ones. When the observation is "tests pass" or "lint: 0 errors," the model has an unambiguous success criterion. When the observation is "here is the file content," the model must judge whether the content satisfies the goal — introducing interpretive error.

### Verification-Driven Development

Verification-driven development gives the coding agent explicit, machine-checkable success criteria before the loop begins (Anthropic Building Effective Agents; tianpan.co SWE-bench production analysis, 2026-04-09):

- Provide failing tests the agent must pass
- Provide linter configuration the agent must satisfy
- Provide integration test suites for self-evaluation

**Why this works:** The agent's ReAct loop terminates when observations confirm success against objective criteria, not when the model judges its own work complete. The termination condition is external, not self-reported.

**Contrast with open-ended prompting:** "Fix the authentication module" gives the model no ground truth for loop termination. "Fix the authentication module so that `auth_test.py` passes with zero lint errors" gives the loop a binary exit condition tied to real system behavior.

### Execution Feedback Loop Structure (Coding Variant)

```text
Thought: Analyze the failing test output. The assertion error shows X is returning Y but expected Z.
Action: read_file("src/auth/token.py")
Observation: [file content]
Thought: The issue is in the token expiry calculation at line 47. Plan the fix.
Action: apply_edit("src/auth/token.py", search="...", replace="...")
Observation: Edit applied successfully.
Thought: Verify the fix by running the tests.
Action: bash("pytest auth_test.py -v")
Observation: 5 passed, 0 failed
Thought: Tests pass. Check lint.
Action: bash("ruff check src/auth/token.py")
Observation: All checks passed.
Thought: All verification criteria met. Task complete.
```

The coding variant differs from generic ReAct in one structural way: verification actions (run tests, run lint) are mandatory loop steps, not optional confirmation. Skipping verification produces agents that "complete" tasks without confirming correctness — the SWE-bench production gap in practical form.

### SWE-bench Evidence on Verification

SWE-bench analysis (tianpan.co, 2026-04-09) identifies the absence of verification-driven loops as the primary explanation for the gap between benchmark scores (80% on SWE-bench Verified, 23% on SWE-bench Pro) and production reliability. The pattern: agents that score well on narrow, verifiable tasks fail on cross-system work precisely because cross-system work lacks the binary feedback signals that ground convergent loops.

**METR controlled study finding (2026):** Experienced developers using AI coding tools were 19% *slower*, despite subjective assessments of ~20% improvement. The gap between perceived and measured productivity is consistent with agents producing outputs that appear correct but require more review time to validate.

**Sources:** [Building Effective Agents — Anthropic](https://www.anthropic.com/research/building-effective-agents), [Agentic Coding in Production: What SWE-bench Scores Don't Tell You — tianpan.co](https://tianpan.co/blog/2026-04-09-agentic-coding-production-swebench-gap) (2026-04-09), [Components of a Coding Agent — Raschka](https://magazine.sebastianraschka.com/p/components-of-a-coding-agent) (2026-04-04)

---

## Model Considerations

### Temperature Settings

*[2026-01-30]*: ReAct benefits from low temperature (0.0-0.3) for reliable reasoning chains. Higher temperature increases variance between thoughts, leading to inconsistent investigation paths.

Multi-step reliability degrades with temperature (see [Model Behavior: Temperature Effects](../3-model/2-model-behavior.md#temperature-effects-on-reliability)). For 10-step ReAct chains at temperature 1.0, reliability drops to approximately 60%.

### Model Selection

ReAct requires strong instruction-following to maintain the thought-action-observation structure. Frontier models (Opus 4.5, GPT-4o, Gemini 2.0 Pro) maintain structure reliably. Mid-tier models may drift from the format over many iterations.

**Observed pattern**: Smaller models (Haiku, GPT-3.5) often collapse the thought step or skip directly to final answers. Reserve ReAct for tasks where model capability justifies the pattern overhead.

---

## Implementation in Claude Code

Claude Code's tool-use system naturally supports ReAct-style interaction:

1. **Extended thinking** provides the "Thought" component
2. **Tool calls** map to "Action"
3. **Tool results** return as "Observation"

The pattern emerges without explicit prompting when extended thinking is enabled and tools are available. Claude reasons about tool selection, invokes the tool, then reasons about results.

For explicit ReAct traces, structure the system prompt to request `<thought>` blocks before tool calls:

```xml
Before each tool call, output your reasoning in <thought> tags:

<thought>
The user needs information about X. I should search for Y because Z.
</thought>

Then invoke the appropriate tool.
```

---

## Connections

- **To [Tool Use](../5-tool-use/_index.md)**: ReAct is a meta-pattern for tool orchestration. Tool design affects observation quality—verbose tools create context pressure, terse tools may lack information. See [Tool Design](../5-tool-use/1-tool-design.md) for principles.

- **To [Model Behavior](../3-model/2-model-behavior.md)**: Temperature and instruction-following reliability directly affect ReAct chain quality. Extended thinking modes complement ReAct by providing internal reasoning before action selection.

- **To [Plan-Build-Review](1-plan-build-review.md)**: Complementary patterns. Plan-Build-Review for known structure, ReAct for emergent investigation. Synthesis: use ReAct within the Research phase to discover information for planning.

- **To [Orchestrator Pattern](3-orchestrator-pattern.md)**: ReAct can operate within an orchestrator's sub-agents. Scout agents using ReAct gather grounded observations that the orchestrator synthesizes.

- **To [Context Management](../4-context/_index.md)**: ReAct's accumulated observations require careful context management. Progressive disclosure and observation summarization prevent context exhaustion.

- **To [Tool Design](../5-tool-use/1-tool-design.md#coding-agent-edit-formats):** Observation quality in coding loops depends on edit format selection. Binary feedback signals (test pass/fail, lint: 0 errors) converge faster than interpretive tool output.

---

## Origins and References

The ReAct pattern was introduced in:

**Yao et al. (2023)**: "ReAct: Synergizing Reasoning and Acting in Language Models"
[arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)

The paper demonstrated that interleaving reasoning traces with action execution outperforms both reasoning-only (Chain-of-Thought) and action-only approaches on tasks requiring information retrieval and multi-step reasoning.

**Key findings from the paper:**
- ReAct reduces hallucination by grounding reasoning in observations
- The trace improves human interpretability and error diagnosis
- Performance gains appear primarily on tasks requiring external knowledge

Subsequent work extended ReAct with reflection (Reflexion), planning (Plan-and-Solve), and tool-augmented variants across different domains.
