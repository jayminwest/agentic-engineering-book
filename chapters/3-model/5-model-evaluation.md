---
title: Model Evaluation for Agents
description: How to evaluate models for agentic tasks—metrics, benchmarks, observability, and the compound error problem
created: 2025-12-10
last_updated: 2026-04-11
tags: [model, evaluation, benchmarks, observability, metrics, agentic, reliability-gap, reliability-dimensions, benchmark-validity, autoresearch, training-loop]
part: 1
part_title: Foundations
chapter: 3
section: 5
order: 1.3.5
---

# Model Evaluation for Agents

Agent evaluation differs fundamentally from chatbot evaluation. Single-turn response quality doesn't predict multi-step task completion. Standard benchmarks like MMLU measure knowledge retention, not tool use accuracy or reasoning consistency across dozens of steps.

---

## Core Questions

### Measurement Strategy
- How do you evaluate agentic capabilities versus conversational quality?
- What metrics actually predict production success?
- Which benchmarks test the capabilities that matter for agents?

### The Compound Error Problem
- Why does 99% per-step accuracy collapse over multi-step tasks?
- How does error compounding shape architecture decisions?
- What reliability threshold is needed for production deployment?

### Implementation Approaches
- How to evaluate without comprehensive test suites?
- When to use LLM-as-judge versus human evaluation?
- What observability infrastructure enables effective debugging?

---

## Why Agent Evaluation Differs

Evaluating agents requires different approaches than evaluating chatbots or code generators.

**Multi-step workflows amplify errors.** A chatbot makes one mistake per conversation turn. An agent executing a 20-step task can fail at any step, with each failure potentially cascading into subsequent steps. A task completion rate of 60% might reflect 98% accuracy at each individual step—high by chatbot standards, catastrophic for production agents.

**Tool use introduces new failure modes.** Correct tool selection, parameter formatting, and result interpretation each add potential failure points. Models can hallucinate tool parameters, call the wrong tool for a task, or misinterpret tool outputs even when the tool executes correctly.

**Task completion matters more than response quality.** A beautifully reasoned response that doesn't complete the task is worthless. An awkwardly worded response that achieves the goal is valuable. This inverts traditional evaluation metrics focused on fluency and coherence.

**Stochastic behavior requires statistical evaluation.** The same agent with the same inputs can produce different outputs due to sampling. Single test runs are misleading. Reliable evaluation requires multiple runs per test case to measure success rate distributions.

---

## The Compound Error Problem

*[2025-12-10]*: Error rates multiply across steps, not add. This shapes every architectural decision in agentic systems.

**The Math of Cascading Failures**

If each step in a workflow has accuracy `p`, then the probability of completing an `n`-step task successfully is `p^n`. This compounds errors exponentially:

| Per-Step Accuracy | 10 Steps | 20 Steps | 50 Steps | 100 Steps |
|-------------------|----------|----------|----------|-----------|
| 99% | 90.4% | 81.8% | 60.5% | 36.6% |
| 98% | 81.7% | 66.8% | 36.4% | 13.3% |
| 95% | 59.9% | 35.8% | 7.7% | 0.6% |
| 90% | 34.9% | 12.2% | 0.5% | 0.003% |

A 95% per-step accuracy—high by many standards—yields only 60% success over 10 steps and nearly 0% over 100 steps. This explains why agents that seem to work well in demos fail catastrophically in production multi-step workflows.

**Architectural Implications**

The compound error problem drives several core architectural patterns:

**Optimize per-step reliability above all else.** Small improvements in individual step accuracy yield dramatic improvements in task completion. Increasing per-step accuracy from 95% to 99% more than doubles the 50-step success rate (from 7.7% to 60.5%).

**Minimize workflow length.** Shorter workflows have exponentially higher success rates. A task that requires 100 steps at 95% accuracy (0.6% success) can be broken into five 20-step subtasks (35.8% success each) with human checkpoints between phases.

**Implement recovery mechanisms.** Retries, validation gates, and error correction reduce effective step count. A workflow with retry logic that catches and fixes 80% of errors behaves like a much shorter workflow.

**Measure and monitor per-step accuracy.** Production monitoring should track success rates at the step level, not just task level. A declining per-step accuracy of 99% → 98% predicts task completion dropping from 90% → 82% over 10 steps—a meaningful regression that overall metrics might miss initially.

---

## The Capability-Reliability Gap

*[2026-04-11]*: The compound error problem is a theoretical concern. The capability-reliability gap is its empirical confirmation.

Rabanser et al. define **capability** as mean task success rate — the standard benchmark metric — and **reliability** as a multi-dimensional property covering consistency, robustness, predictability, and safety that is independent of raw capability. The gap is the empirically measured divergence in how these improve over time. [Rabanser et al., arXiv:2602.16666, 2026]

"A highly capable system can be unreliable, and a less capable system can be highly reliable within its operating envelope. This separation is essential: improving capability does not automatically improve reliability." [Rabanser et al., 2026]

### Empirical Evidence

Across 14 frontier models (OpenAI, Anthropic, Google) evaluated over 18 months of releases on two benchmarks:

- On **GAIA** (general assistant tasks): accuracy improved steadily; reliability showed "barely any improvement, even among the latest models"
- On **τ-bench** (customer service simulation): reliability improved at **one-seventh** the rate of accuracy

The divergence is not uniform across reliability dimensions. Calibration (whether confidence aligns with actual performance) improved in recent models. Consistency (whether agents produce the same outcome on repeated identical runs) remained low across all models. Discrimination (whether confidence successfully separates successes from failures) "mostly worsened" on GAIA despite τ-bench improvements.

This means selecting a newer, higher-accuracy model does not reliably produce a more consistent or robust agent. Capability and reliability require independent measurement and independent improvement strategies.

### Why This Matters for Architecture

The compound error math (above) establishes why per-step reliability matters theoretically. The capability-reliability gap establishes that reliability does not improve as a byproduct of chasing benchmark scores.

Architectural implication: reliability dimensions must become explicit optimization targets, not expected byproducts of capability gains. The question "What reliability threshold is needed for production deployment?" (Core Questions, above) cannot be answered by benchmark accuracy scores alone — it requires measurement against the four reliability dimensions: consistency, robustness, predictability, and safety. These are operationalized in [Evaluation](../../7-practices/2-evaluation.md#reliability-dimensions-beyond-task-completion) and applied as deployment gate criteria in [Production Concerns](../../7-practices/4-production-concerns.md#reliability-thresholds-for-production-deployment).

**Source:** [Towards a Science of AI Agent Reliability](https://arxiv.org/abs/2602.16666) (Rabanser, Kapoor, Narayanan et al., Princeton, 2026); [AI agents are getting more capable, but reliability is lagging](https://fortune.com/2026/03/24/ai-agents-are-getting-more-capable-but-reliability-is-lagging-narayanan-kapoor/) (Fortune, 2026)

---

## Key Metrics for Agentic Tasks

Standard evaluation metrics miss what matters for agents. Task completion, tool accuracy, and error recovery predict production success better than BLEU scores or perplexity.

### Task Completion Rate

**What it measures:** Percentage of tasks where the agent achieves the defined goal, regardless of path taken.

**Why it matters:** This is the bottom line. An agent that completes 95% of tasks is production-ready. An agent that completes 70% of tasks is not, even if its reasoning is impeccable.

**How to measure:** Define clear success criteria before evaluation. Binary outcomes (task completed: yes/no) are more reliable than subjective quality ratings. For ambiguous tasks, use multiple human evaluators and report inter-rater agreement.

### Tool Calling Accuracy

**What it measures:** Percentage of tool calls where the agent selects the correct tool with correctly formatted parameters.

**Why it matters:** Tool calling failures are the most common reason agents fail to complete tasks. Models can reason correctly but choose the wrong tool or malform parameters.

**How to measure:** Log all tool calls with expected tool/parameters. Compare against ground truth or expert annotation. Break down into: correct tool selection rate, correct parameter formatting rate, and correct parameter values rate.

### Reasoning Relevancy

**What it measures:** Whether the agent's reasoning connects to the user's request and the available context.

**Why it matters:** Agents can produce plausible-sounding reasoning that's entirely disconnected from the actual task. This leads to confident failures—the agent proceeds down an irrelevant path without realizing it's off track.

**How to measure:** Human evaluation or LLM-as-judge comparing reasoning to task requirements. Look for grounding in provided context, acknowledgment of constraints, and logical connection between steps.

### Error Recovery Rate

**What it measures:** Percentage of times the agent successfully recovers from a failed step without human intervention.

**Why it matters:** Agents that gracefully handle failures achieve higher task completion than agents that abort on first error. Recovery mechanisms distinguish production-ready systems from prototypes.

**How to measure:** Inject controlled failures (wrong tool outputs, missing context, malformed inputs) and measure how often the agent detects the issue, adjusts its approach, and completes the task.

### Cost Per Successful Task

**What it measures:** Total token cost divided by number of successfully completed tasks.

**Why it matters:** An agent with 60% task completion that costs $0.10 per attempt has a real cost of $0.17 per success. An agent with 90% completion at $0.15 per attempt costs $0.17 per success—same effective cost, but better user experience.

**How to measure:** Track input tokens, output tokens, and tool call overhead. Divide total cost by successful completions. Monitor separately from average cost per attempt.

### Latency Distribution

**What it measures:** Not just average latency, but p50, p95, and p99 latencies for task completion.

**Why it matters:** Average latency hides tail behavior. An agent averaging 5 seconds might have a p99 of 45 seconds, creating terrible user experience for 1% of requests. Multi-step workflows amplify latency variance.

**How to measure:** Record end-to-end task completion time for each evaluation run. Calculate percentiles. Break down by task type to identify which workflows have high variance.

---

## Benchmarks for Agent Evaluation

Traditional NLP benchmarks don't measure agentic capabilities. Newer benchmarks designed for agents test multi-step reasoning, tool use, and task completion.

### τ-Bench (Tau-Bench)

**What it tests:** Real-world customer service and retail tasks requiring dynamic interaction between user simulation, agent, and tool systems.

**Why it matters:** Most agent benchmarks use static inputs and outputs. τ-Bench simulates realistic back-and-forth where the user's responses depend on the agent's prior actions. This tests whether agents can handle the dynamic, multi-turn nature of real tasks.

**Key insight:** Reveals whether agents can maintain coherent strategy across multiple turns of user interaction. Agents that score well on static benchmarks sometimes collapse when facing realistic conversational dynamics.

**Source:** [τ-Bench: A Benchmark for Tool-Augmented LLMs](https://www.sierra.ai/research/tau-bench) (Sierra, 2024)

### Terminal-Bench

**What it tests:** Command-line workflows requiring multi-step Bash interactions to accomplish systems administration tasks.

**Why it matters:** Terminal tasks require parsing command outputs, adapting to unexpected results, and maintaining state across multiple commands. Tests whether agents can handle real-world tool outputs (not sanitized API responses).

**Key insight:** Exposes brittleness in tool output parsing and context management. Agents must handle messy, unstructured command outputs rather than clean JSON responses.

### SWE-bench Family

**What it tests:** Real GitHub issues from open source projects. Agents must understand the codebase, locate relevant files, implement changes, and verify fixes.

**Variants:**
- **SWE-bench Lite:** 300 filtered issues, baseline difficulty
- **SWE-bench Verified:** Human-validated test cases, reduced contamination
- **SWE-bench Pro:** Significantly harder, addresses data contamination in original benchmark

**Why it matters:** Tests end-to-end software engineering workflows—understanding requirements, navigating codebases, writing code, running tests. Multi-step tasks requiring dozens to hundreds of tool calls.

*[2025-12-10]*: SWE-bench Pro revealed widespread contamination in the original benchmark. Top agents scored 70%+ on Verified but dropped to 23% on Pro, demonstrating that earlier results were inflated by training data leakage.

**Key insight:** Codebase navigation and multi-file reasoning are major bottlenecks. Agents often struggle more with finding the right location to make changes than with implementing the fix itself.

**Source:** [SWE-bench Pro: Addressing Real-World Contamination](https://www.scale.com/blog/swe-bench-pro) (Scale AI, 2025)

### Context-Bench

**What it tests:** Tasks requiring maintaining and using information across long conversations or documents (50k+ tokens).

**Why it matters:** Tests whether agents can effectively use large context windows—not just fit information in context, but actually retrieve and apply relevant details from thousands of tokens away.

**Key insight:** Context window size doesn't predict context utilization. Models with 200k token windows often fail to use information beyond the first and last 10k tokens effectively. This "lost in the middle" phenomenon affects multi-step agentic tasks.

### Current Benchmark Landscape

*[2025-12-10]*: 2025 agent benchmarks are significantly harder than 2024 benchmarks. Best-in-class agents score as low as 5% on some tasks. This reflects both reduced data contamination and more realistic task complexity.

Benchmark scores should be interpreted as lower bounds on capability, not predictions of production performance. Agents often perform better on domain-specific tasks than on generic benchmarks, especially when paired with well-designed tools and context.

---

## Cognitive Frameworks for Capability Assessment

*[2026-03-18]*: Google DeepMind's "Measuring Progress Toward AGI: A Cognitive Framework" (Burnell et al., 2026) offers a structured lens for understanding *what* an AI system can and cannot do—directly useful for agent architecture decisions.

### The Cognitive Taxonomy

The framework identifies 10 cognitive faculties that underpin general intelligence, drawn from decades of research in psychology, neuroscience, and cognitive science:

| Faculty | Definition | Agent Relevance |
|---------|-----------|-----------------|
| **Perception** | Extract and process sensory information | Multimodal input handling, image/document understanding |
| **Generation** | Produce outputs (text, code, actions) | Core LLM capability; tool call generation |
| **Attention** | Focus cognitive resources on specific aspects | Context window utilization, instruction following |
| **Learning** | Acquire new knowledge through experience | In-context learning, few-shot adaptation |
| **Memory** | Store and retrieve information over time | Context management, RAG integration |
| **Reasoning** | Draw valid conclusions via logical principles | Multi-step planning, debugging, chain-of-thought |
| **Metacognition** | Monitor and control own cognitive processes | Self-correction, knowing when to ask for help |
| **Executive Functions** | Planning, inhibition, cognitive flexibility | Workflow orchestration, task decomposition |
| **Problem Solving** | Find effective solutions (composite faculty) | End-to-end task completion |
| **Social Cognition** | Process and respond to social information | User intent understanding, collaborative behavior |

The taxonomy focuses on *what* the system can accomplish, not *how*—remaining agnostic to underlying mechanisms. This makes it applicable whether evaluating a single model or a multi-agent system with tools.

### Jagged Cognitive Profiles

A system's capabilities are not uniform across faculties. DeepMind's "cognitive profiles" (radar charts across all 10 faculties) reveal the **jagged frontier**: a model might score at the 99th percentile on generation and reasoning while falling below the human median on metacognition or social cognition.

This matters for agent design because:

- **Strengths inform delegation.** If a model excels at reasoning but struggles with memory, architecture should compensate—use external memory systems (RAG, structured context) rather than relying on in-context retention.
- **Weaknesses predict failure modes.** Low metacognition scores predict agents that don't know when they're failing. Low executive function scores predict poor multi-step planning. These map directly to the failure modes observed in production agents.
- **Profiles differ across model families.** Choosing between Claude, GPT, and Gemini isn't about "which is better" but about which cognitive profile fits the task's demands.

### System vs. Model Evaluation

The paper argues that evaluating the model checkpoint alone is increasingly insufficient—and potentially misleading:

> Modern AI systems are deployed with specific system instructions, have access to tools, can manipulate their environments via actions, and may even have the ability to make calls to other AI systems.

This directly validates the agent evaluation approach: measure the *system* (model + tools + prompts + orchestration), not just the model. A model that scores poorly on memory benchmarks might perform excellently when paired with a well-designed RAG system. Conversely, a model with strong reasoning might fail in practice when given poorly designed tools.

The analogy: evaluating a person's intelligence changes when you give them a calculator. For agents, the "calculator" is the entire tool and context infrastructure. Evaluate with it, not without it.

### Capabilities vs. Propensities

A critical distinction the paper raises: what a system *can* do versus what it *tends* to do. Propensities—risk tolerance, communication style, default problem-solving strategies—significantly affect deployment outcomes. This connects directly to model behavioral patterns (see [Model Behavior](2-model-behavior.md)): two models with identical capability profiles can behave very differently in production due to different propensities.

### Evaluation Protocol Implications

DeepMind's three-stage protocol maps to agent evaluation practice:

1. **Cognitive assessment** → Run broad capability benchmarks across multiple faculties (not just task completion)
2. **Human baselines** → Compare against human performance under the same conditions (same tools, same instructions)
3. **Cognitive profiles** → Build per-system capability maps that inform architecture decisions

The key insight: benchmark coverage gaps in metacognition, attention, learning, and social cognition mean current agent evaluations may miss critical capability dimensions. Agents that pass task-completion benchmarks might still fail on metacognitive tasks (knowing when to stop, when to ask for help) or social cognition tasks (understanding user intent in ambiguous requests).

**Source:** [Measuring Progress Toward AGI: A Cognitive Framework](https://storage.googleapis.com/deepmind-media/DeepMind.com/Blog/measuring-progress-toward-agi/measuring-progress-toward-agi-a-cognitive-framework.pdf) (Burnell et al., Google DeepMind, 2026)

---

## Observability Requirements

Production agents require observability infrastructure to enable debugging and performance monitoring. Logging is not optional—it's the only way to diagnose failures in multi-step workflows.

### What Tracing Must Capture

*[2025-12-10]*: Tracing consistently tops must-have lists in production agent surveys. This reflects the difficulty of debugging without visibility into decision paths.

**Token counts per step.** Input tokens, output tokens, and cumulative totals. Essential for cost analysis and identifying context window pressure.

**Tool calls with parameters.** Every tool invocation with full parameters and return values. Enables debugging tool selection failures and parameter formatting errors.

**Decision branches and reasoning.** The agent's reasoning at each step, including why specific tools were chosen or approaches selected. Critical for understanding failure modes.

**Timing data.** Latency at each step, not just overall task time. Identifies bottlenecks and timeout risks.

**Error states and retries.** Failed tool calls, retry attempts, and recovery paths. Separates transient failures from systemic issues.

### Visual Decision Graphs

Text logs are insufficient for understanding complex agent behaviors. Visual representations of decision trees reveal patterns invisible in linear logs.

**Decision graphs show:**
- Branch points where the agent chose between approaches
- Paths taken versus paths not taken
- Retry loops and recovery attempts
- Where in the workflow failures occur

These graphs help identify whether failures are concentrated at specific steps (suggesting prompt or tool issues) or distributed throughout (suggesting fundamental capability gaps).

### Alert Patterns for Quiet Failures

Some failures don't throw errors—they just waste resources or degrade quality silently.

**Retry spikes.** A sudden increase in retry frequency indicates environmental issues (API instability) or degrading model performance.

**Latency outliers.** The p99 latency spiking while p50 remains stable suggests specific task types or inputs are causing issues.

**Context window pressure.** Average context utilization climbing toward maximum warns of impending failures before they happen.

**Tool call distribution shifts.** Changes in which tools are called most frequently can indicate prompt drift or model behavior changes.

**Source:** [LangChain State of AI Agents Report 2024](https://www.langchain.com/stateofaiagents) reports 80% of production agent deployments consider tracing essential, with 40% still relying on human review as primary quality control.

---

## LLM-as-Judge Evaluation

Using language models to evaluate other language models scales better than pure human evaluation but introduces biases that must be understood.

### When LLM-as-Judge Works

**High agreement with humans when calibrated.** LLM judges can achieve 80%+ agreement with human evaluators on well-defined tasks when calibrated against human-annotated datasets.

**Pairwise comparison outperforms single output rating.** Asking a judge to compare two outputs ("Which response better answers the question?") produces more reliable results than asking for absolute quality ratings ("Rate this response 1-10").

**Structured rubrics improve consistency.** Providing the judge with explicit evaluation criteria (relevance, completeness, accuracy) yields more consistent results than open-ended quality assessment.

### Calibration Requirements

LLM judges are not objective. They must be calibrated against ground truth before deployment.

**Create a human-annotated calibration set.** Have humans evaluate 50-100 examples using the same rubric the LLM judge will use. Measure agreement between LLM and human ratings.

**Tune evaluation prompts based on disagreements.** When the LLM judge disagrees with humans, examine the reasoning. Often the evaluation prompt is ambiguous or missing key criteria.

**Re-calibrate when changing judge models.** Different models have different biases. Calibration for GPT-4 doesn't transfer to Claude or vice versa.

### Domain-Specific Judge Construction

Generic judge calibration—calibrate on 50–100 examples, measure agreement, deploy—misses the upstream work that determines whether the judge measures the right thing. Husain's structured process for building domain-specific judges starts with rubric design before any calibration begins [Husain, 2024b].

**Three rubric structuring dimensions** ensure evaluation coverage across realistic usage:

| Dimension | What It Covers | Examples |
|-----------|---------------|---------|
| **Features** | Specific functionalities the product provides | Order tracking, email summarization, code review, appointment scheduling |
| **Scenarios** | Problem contexts the agent encounters | Multiple matches, no match, ambiguous request, invalid data, system error, incomplete information, unsupported feature |
| **Personas** | User profiles with different needs | New users (high explanation need), expert users (low tolerance for verbosity), non-native speakers, busy professionals (prefer brevity) |

A rubric that covers all three dimensions produces evaluation cases that represent the product's actual usage distribution, not a developer's intuition about what matters.

**The principal domain expert principle.** For most teams, a single domain expert whose judgment defines acceptable performance produces more reliable calibration than a committee. Husain terms this the "benevolent dictator" approach: one person's standards govern, with clear accountability. The single-expert approach eliminates annotation conflicts and establishes a quality bar that the automated judge can be trained to replicate. This recommendation pairs with the annotation workflow guidance in [Evaluation](../../7-practices/2-evaluation.md#annotation-workflow-design).

**Seven-step calibration process:**

1. Identify a single principal domain expert—the person whose judgment defines product quality.
2. Create a diverse dataset covering the dimension combinations above (30+ examples minimum; aim for coverage across feature × scenario × persona combinations).
3. Domain expert makes binary pass/fail judgments on each example, with detailed written critiques explaining each decision.
4. Fix obvious product errors discovered during review—the calibration process often surfaces bugs before the judge is built.
5. Build the judge iteratively, using expert-labeled examples as few-shot demonstrations.
6. Measure human–judge agreement; refine until acceptable. Husain's documented example: >90% agreement achieved in three iterations.
7. Perform error analysis on remaining disagreements. If specific failure categories (e.g., all "ambiguous request" scenarios) show persistent low agreement, create specialized judges for those categories rather than forcing a single judge to handle all cases.

**Binary-over-Likert preference.** Binary pass/fail produces more consistent labeling than rating scales and requires fewer labeled examples to detect meaningful model differences. Husain states this directly: "Tracking a bunch of scores on a 1-5 scale is often a sign of a bad eval process" [Husain, 2024b]. Pairwise comparison (which response is better?) outperforms direct absolute scoring on both consistency and correlation with human preference—a finding corroborated by Eugene Yan's analysis of LLM evaluator effectiveness: pairwise comparison produces more stable results than direct scoring, and finetuned evaluators show catastrophic performance drops on cross-domain transfer, reinforcing the domain-specificity requirement [Yan, 2024].

**The process-value insight.** The iterative judge-building process produces value beyond the judge itself. Husain observes that "the judge is a hack to trick people into looking at their data"—the real benefit is the systematic examination of failures that the construction process forces. Teams that build judges by going through the calibration workflow develop product intuition that teams using off-the-shelf evaluators do not.

**Sources:**
- Husain, H. "Using LLM-as-a-Judge For Evaluation: A Complete Guide." hamel.dev/blog/posts/llm-judge/ (2024-10-29)
- Yan, E. "Evaluating the Effectiveness of LLM-Evaluators (aka LLM-as-Judge)." eugeneyan.com/writing/llm-evaluators/ (2024-08-18)

### Known Biases in LLM Judges

**Writing style preferences.** LLM judges favor responses that match their own generation style—verbose for verbose models, concise for concise models.

**Position bias in pairwise comparison.** Some models favor the first or second option more often than chance would predict. Mitigate by evaluating both orderings (A vs B, then B vs A) and averaging.

**Length bias.** Longer responses often receive higher ratings even when they don't contain more useful information. Control for this by including length limits in evaluation criteria.

**Recency bias.** Judges may favor more recent information or approaches mentioned later in the response. Test by reversing the order of information presented.

### Human Evaluation Still Essential

LLM judges scale evaluation but cannot replace human judgment entirely.

**Use humans for:**
- Calibration dataset creation
- Validating LLM judge reliability
- Evaluating subjective qualities (tone, appropriateness)
- Catching systematic biases in automated evaluation

**Use LLM judges for:**
- Large-scale comparative testing (A/B tests across hundreds of examples)
- Rapid iteration during development
- Regression testing (did this change hurt performance?)
- Initial filtering before human review

### Evaluation Inside Training Loops

*[2026-04-11]*: Autoresearch introduces a structurally distinct application of evaluation that does not appear elsewhere in this file. Existing coverage evaluates deployed agent outputs — task completion, tool accuracy, error recovery, LLM-as-judge quality ratings. The autoresearch pattern applies evaluation in a different role: as the fitness function inside an iterative model training loop.

**The structural distinction**

In standard agent evaluation, an eval suite measures output quality after deployment to guide prompt or architecture improvements. Evaluation is a gate on changes. In autoresearch, validation bits-per-byte (val_bpb) on a held-out set is measured after every five-minute training experiment to make a binary keep/discard decision. Evaluation is not a gate; it is the decision function inside a loop running approximately 100 iterations overnight. The loop modifies architecture, optimizer selection, attention configuration, and regularization — and keeps or discards each change based solely on whether val_bpb improves.

**The formal basis**

GEPA (gradient-free evolutionary program adaptation, arxiv 2501.09361, ICLR 2026) formalizes this pattern: the validation metric is the fitness function for evolutionary search over training programs. The eval infrastructure does not change; its role does. The same held-out set and scalar metric that measure a deployed model's quality can drive an autonomous model improvement loop.

**Why this matters for practitioners**

The prerequisite for autoresearch is a scalar quality metric computable in minutes. Practitioners who have already instrumented production eval pipelines — held-out sets, task completion metrics, domain-specific accuracy signals — are closer to autoresearch-capable than they may realize. The infrastructure investment they have made in evaluation supports both post-deployment monitoring and, if the task is narrow and compute is available, autonomous model improvement.

The compound error analysis elsewhere in this file establishes that small per-step accuracy improvements yield dramatic compounding reliability gains. Autoresearch applies this logic one level down: improving per-step training efficiency (via architecture and optimizer search) compounds within a fixed compute budget. The eval-in-the-loop pattern closes a feedback loop that compound error theory implies but does not close.

**Constraints for safe autonomous evaluation loops**

Four constraints identified independently by the HN community (2026-03-08) and embedded in Karpathy's implementation define the minimum viable guardrails for unattended evaluation-driven loops:

| Constraint | Why it matters |
|------------|----------------|
| One-file scope | Limits blast radius of any single experiment; prevents architectural drift across unrelated components |
| One scalar metric | Prevents gaming: a single metric forces genuine improvement rather than trade-off exploitation |
| Time-boxed experiments (~5 minutes) | Caps maximum compute waste per rejected experiment; enables 100 iterations overnight |
| Git checkpoint per accepted change | Every accepted modification is committed; full rollback available at any point |

These constraints map directly to observability requirements described earlier in this file: one metric to monitor, bounded execution time, and a complete audit trail of accepted changes.

**See Also:**
- [When to Train Your Own](1-model-selection.md#when-to-train-your-own) — Model selection context for trained-to-task SLMs
- [Autonomous Loops](../6-patterns/4-autonomous-loops.md) — Autoresearch as a specific application of the autonomous loop pattern
- [Evaluation (Practices)](../7-practices/2-evaluation.md) — Eval-driven model improvement workflows

**Sources:**
- Schmid, P. "How Autoresearch will change Small Language Models adoption." philschmid.de, ~2026-03-10. https://www.philschmid.de/autoresearch
- Karpathy, A. karpathy/autoresearch. GitHub, 2026-03-06. https://github.com/karpathy/autoresearch
- GEPA (arxiv 2501.09361). ICLR 2026. https://arxiv.org/abs/2501.09361
- HN: "Autoresearch: Agents researching on single-GPU nanochat training automatically." 2026-03-08. https://news.ycombinator.com/item?id=47291123

---

## Evaluation Strategy Progression

Effective evaluation starts simple and scales with the project. Waiting for comprehensive test suites delays learning and wastes time.

### Start Small: 3-5 Test Cases Immediately

The first evaluation should happen before the first deployment. Three to five carefully chosen test cases reveal more than zero test cases.

**Pick test cases that:**
- Cover the most common use case (happy path)
- Test the most likely failure mode
- Exercise the full workflow end-to-end

Run these manually. Observe the agent's reasoning, tool calls, and outputs. This reveals fundamental issues faster than building automation.

### Progress: Manual → User Feedback → Automated

**Phase 1: Manual tracing.** Run test cases by hand, reading logs and outputs. Identify obvious failures and iterate on prompts or tools.

**Phase 2: Online user feedback.** Deploy to limited users with feedback mechanisms. Track which tasks users report as failures. This reveals real-world failure modes missed in testing.

**Phase 3: Offline automated datasets.** Build regression test suites from production failures. Automate evaluation of known failure modes to prevent regressions.

### Component-Level Evaluation

Pipeline-level evaluations produce ambiguous failures: the agent failed, but which component failed? Component-level evaluation isolates each pipeline stage's behavior before composing it into system-level tests. This maps to Husain's Level 1 evaluation hierarchy [Husain, 2024]: deterministic assertions per component, run on every code change. Level 1 must be solid before scaling to Level 2 (model-based evaluations) because model-based evals on a fragile pipeline produce noise, not signal.

**Why the sequencing gate matters.** A RAG pipeline that returns wrong answers could fail at retrieval (wrong chunks returned), reranking (right chunks returned in wrong order), generation (right chunks, wrong synthesis), or format validation (correct content, wrong structure). A single pipeline-level eval catches that something is wrong. Component-level evals identify *which* component is wrong. That specificity is the difference between a debugging session that takes hours and one that takes minutes—see [Debugging Agents](../../7-practices/1-debugging-agents.md) for how failure attribution accelerates diagnosis.

**Canonical component list for RAG systems** (the most common pipeline architecture in deployed agents):

- **Retrieval relevance:** Are the retrieved chunks relevant to the query? Measure precision@k and recall@k against a labeled set of query–chunk pairs.
- **Reranking quality:** Does the reranker improve chunk ordering? Compare ranked vs. unranked chunk relevance scores on a held-out set.
- **Prompt adherence:** Does the generation follow the instructions in the system prompt? Check for constraint violations (format, length, scope restrictions).
- **Format correctness:** Does the output structure match the specification? Validate schema, required fields, and encoding.
- **Reasoning quality:** Does the response correctly use the retrieved information? Check for hallucinated claims not grounded in retrieved chunks.

**Generalization beyond RAG.** The component taxonomy extends to other architectures: routing accuracy for intent-classification agents, tool selection accuracy for function-calling agents, extraction precision for document-processing pipelines. The principle is architecture-agnostic—isolate and validate each component before trusting the composition.

**Sources:**
- Husain, H. "Your AI Product Needs Evals." hamel.dev/blog/posts/evals/ (2024-03-29)
- Husain, H. "LLM Evals: Everything You Need to Know (FAQ)." hamel.dev/blog/posts/evals-faq/ (2025)

### Build Regression Tests from Production Failures

Every production failure should become a test case.

**When an agent fails:**
1. Capture the exact input, context, and expected output
2. Add it to the regression test suite
3. Verify the fix prevents recurrence
4. Run the test on every subsequent change

This ensures fixes actually work and prevents regressions from reintroducing solved problems.

---

## Anti-Patterns in Agent Evaluation

Common evaluation mistakes waste time and produce misleading results.

### Waiting for Large Eval Suites Before Starting

**What it looks like:** Spending weeks building comprehensive test suites before evaluating the first agent implementation.

**Why it fails:** Requirements change as the agent develops. Half the test cases become irrelevant. Meanwhile, fundamental issues go undetected because evaluation hasn't started.

**Better approach:** Start with 3-5 representative test cases. Add more as patterns emerge. Build the test suite iteratively based on observed failure modes.

### Testing Only Final Outputs

**What it looks like:** Checking whether the agent completed the task without examining intermediate steps.

**Why it fails:** Agents can succeed for the wrong reasons (lucky guesses) or fail in ways that corrupt future tasks (incorrect intermediate state). Final output doesn't reveal whether the agent understood the task or just stumbled into the right answer.

**Better approach:** Log and evaluate reasoning at each step. Validate tool calls, intermediate outputs, and decision logic. This reveals fragile success patterns before they cause production failures.

### Relying Solely on LLM Judges

**What it looks like:** Using LLM-as-judge for all evaluation without human validation or calibration.

**Why it fails:** LLM judges have systematic biases. They can consistently misrate entire categories of outputs. Without human calibration, these biases go undetected and mislead development decisions.

**Better approach:** Use LLM judges for scale, but calibrate against human judgment. Validate judge reliability on a representative sample before trusting it on the full dataset.

### Ignoring Cost and Latency in Evaluation

**What it looks like:** Focusing exclusively on task completion rate while ignoring that successful tasks take 45 seconds and cost $2 each.

**Why it fails:** An agent that completes 95% of tasks at $2 per success is not production-viable for most use cases. Cost and latency constraints often matter as much as accuracy.

**Better approach:** Define acceptable cost and latency budgets before evaluation. Measure these alongside accuracy. Report cost-per-success and latency distributions, not just task completion rate.

---

## Connections

- **To [Evaluation (Practices)](../../7-practices/2-evaluation.md):** Practical evaluation workflows, tooling, and integration with development cycles.
- **To [Debugging Agents](../../7-practices/1-debugging-agents.md):** Evaluation metrics guide debugging by identifying where in the workflow failures occur. Observability infrastructure supports both evaluation and debugging.
- **To [Tool Use](../../5-tool-use/_index.md):** Tool calling accuracy is a critical evaluation metric. Tool design quality directly impacts agent performance on benchmarks.
- **To [Model Selection](1-model-selection.md):** Evaluation metrics determine which models are suitable for which tasks. Benchmark performance predicts (but doesn't guarantee) production capability.
- **To [Cost and Latency](../../7-practices/3-cost-and-latency.md):** Evaluation must include cost and latency metrics, not just accuracy. Multi-step workflows amplify both cost and latency beyond single-turn estimates.
- **To [Model Selection — When to Train Your Own](1-model-selection.md#when-to-train-your-own):** Evaluation infrastructure is the prerequisite for autoresearch. Practitioners with production eval pipelines can extend them to drive trained-to-task SLM development.

---

## Sources

- [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) (Anthropic): Evaluation drives design decisions
- [The compounding impact of AI agent errors](https://www.vellum.ai/blog/the-compounding-impact-of-ai-agent-errors) (Vellum): Mathematical analysis of compound error rates
- [Agents are a UX problem](https://wattenberger.com/thoughts/agents-ux) (Chip Huyen): Per-step accuracy compound effects (95%^100 = 0.6%)
- [LangChain State of AI Agents Report 2024](https://www.langchain.com/stateofaiagents): Production agent deployment patterns, observability requirements
- [τ-Bench: A Benchmark for Tool-Augmented LLMs](https://www.sierra.ai/research/tau-bench) (Sierra): Dynamic user/tool interaction evaluation
- [SWE-bench Pro](https://www.scale.com/blog/swe-bench-pro) (Scale AI): Data contamination in agent benchmarks, difficulty progression
- [Measuring Progress Toward AGI: A Cognitive Framework](https://storage.googleapis.com/deepmind-media/DeepMind.com/Blog/measuring-progress-toward-agi/measuring-progress-toward-agi-a-cognitive-framework.pdf) (Burnell et al., Google DeepMind, 2026): Cognitive taxonomy, capability profiling, system vs. model evaluation
- [How Autoresearch will change Small Language Models adoption](https://www.philschmid.de/autoresearch) (Schmid, 2026): Autoresearch mechanism, production evidence, Shopify case study
- [karpathy/autoresearch](https://github.com/karpathy/autoresearch) (Karpathy, 2026): Implementation of agent-driven training-loop search; 700-experiment results
- [GEPA: Gradient-Free Evolutionary Program Adaptation](https://arxiv.org/abs/2501.09361) (ICLR 2026): Formal basis for eval-as-fitness-function in autonomous training loops
