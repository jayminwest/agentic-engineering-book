---
title: Multi-Model Architectures
description: When and how to use multiple models in agent systems—orchestrator patterns, cascades, routing strategies, and planning versus execution separation
created: 2025-12-10
last_updated: 2026-04-11
tags: [model, multi-model, orchestrator, cascade, routing, agentic, swarm, advisor-strategy, model-pairing]
part: 1
part_title: Foundations
chapter: 3
section: 4
order: 1.3.4
---

# Multi-Model Architectures

Most agent systems use a single model for everything. Multi-model architectures optimize cost, latency, and capability trade-offs by routing different tasks to different models.

The pattern emerges when single-model constraints become apparent. A frontier model like Opus delivers exceptional reasoning but costs 10-20× more than smaller models. Using Opus for every task—including simple instruction-following—wastes budget on capability that isn't needed.

Multi-model systems route tasks based on complexity, capability requirements, and economic constraints. They introduce coordination overhead but unlock substantial cost reduction and latency improvement when designed correctly.

---

## Core Questions

### Architecture Decisions
- When does a single model suffice versus requiring multiple models?
- How do orchestrator-specialist patterns differ from model cascades?
- What routing strategies exist and when does each apply?

### Trade-off Management
- What overhead does multi-model coordination introduce?
- How do cost savings compare to coordination complexity?
- When does planning-execution separation improve outcomes?

### Implementation Patterns
- How do quality gates work in cascade architectures?
- What intermediate representations enable clean model handoffs?
- How does model family consistency affect integration complexity?

---

## When Single Model Suffices

Start with a single frontier model. Multi-model architectures add complexity that only pays off at scale.

**Single model works when:**
- Task volume is low (hundreds of queries per day, not thousands)
- Task complexity is homogeneous (all planning, all execution, or all simple)
- Prototyping and validation phase (optimize for learning speed, not cost)
- Total API spend is below economic threshold ($500-1000/month)

The default should be "one model until proven otherwise." Premature optimization toward multi-model introduces coordination bugs, testing complexity, and architectural overhead that outweighs cost savings.

**Indicators that single model is breaking down:**
- Monthly API costs exceed team budget allocations
- Simple tasks wait behind complex tasks in queue (latency suffers)
- Quality variance increases (model struggles with mixed complexity)
- Token costs for simple operations become obviously wasteful

At this point, multi-model becomes an architecture decision worth exploring.

---

## Orchestrator-Specialist Pattern

A strong reasoning model decomposes tasks and coordinates execution by weaker, cheaper models running in parallel.

### The Pattern

```
┌─────────────────────────────────────┐
│   Orchestrator (Opus/GPT-4o)        │
│   - Decompose task into subtasks    │
│   - Route to specialists            │
│   - Synthesize results              │
└─────────────────────────────────────┘
           │
           ├──────────────┬──────────────┬──────────────┐
           ▼              ▼              ▼              ▼
    ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
    │ Specialist│   │ Specialist│   │ Specialist│   │ Specialist│
    │  (Sonnet) │   │  (Sonnet) │   │  (Haiku)  │   │  (Haiku)  │
    │  Task A   │   │  Task B   │   │  Task C   │   │  Task D   │
    └──────────┘   └──────────┘   └──────────┘   └──────────┘
           │              │              │              │
           └──────────────┴──────────────┴──────────────┘
                          ▼
           ┌─────────────────────────────────────┐
           │   Orchestrator synthesizes outputs   │
           └─────────────────────────────────────┘
```

The orchestrator handles complex reasoning—task decomposition, dependency analysis, result synthesis. Specialists handle straightforward execution—following structured instructions, applying patterns, implementing solutions.

### Evidence: NVIDIA ToolOrchestra

*[2025-12-10]*: NVIDIA's ToolOrchestra research demonstrated that an 8B parameter orchestrator coordinating 1B specialist models outperformed larger single models on tool-use benchmarks.

**Results:**
- 8B orchestrator + 1B specialists: 78.2% accuracy at $9.20 per 1000 queries
- 70B single model: 72.5% accuracy at $17.80 per 1000 queries
- Better performance at half the cost

The finding challenges assumptions about model size. Orchestration allows smaller models to punch above their weight by separating planning from execution.

**Source:** [ToolOrchestra: Multi-Agent Collaboration via Tool-Oriented Orchestration](https://arxiv.org/abs/2501.01678)

### Token Overhead: The Economic Trade-off

Anthropic's multi-agent research revealed the cost of coordination: ~15× token overhead compared to single-agent approaches.

**Where the tokens go:**
- Orchestrator context: task description + specialist outputs
- Specialist contexts: decomposed subtasks + execution instructions
- Coordination messages: routing decisions, synthesis prompts
- Result aggregation: combining parallel outputs into coherent response

The 15× multiplier means this pattern only justifies itself for high-value use cases. If a task costs $0.10 with a single agent, multi-agent orchestration costs $1.50. The economic threshold depends on what quality improvement or speed gain that $1.40 buys.

**When the overhead pays off:**
- Complex research tasks requiring parallel domain expertise
- Code generation with architecture, implementation, and testing phases
- Multi-step workflows where specialists can run concurrently
- Quality-critical tasks where deterministic outcomes justify token cost

**When it doesn't:**
- Simple queries answerable by a single model pass
- Low-budget prototyping where cost discipline matters
- Tasks with low parallelization potential (sequential dependencies)

**Source:** [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)

### Quality Gains: What the Tokens Buy

The overhead buys deterministic quality. Anthropic's production system showed 90.2% improvement over single-agent Opus on internal research evaluations.

Academic research on multi-agent legal systems found:
- 80× improvement in action specificity
- 100% actionable recommendation rate (vs. 1.7% for single agent)
- 140× improvement in solution correctness
- Zero quality variance across trials

The determinism matters more than the absolute quality gain. Single agents can achieve high quality with prompt engineering, but variance across runs creates reliability problems in production. Multi-agent orchestration with clear decomposition and synthesis reduces variance to near-zero.

**Source:** [Multi-Agent LLM Orchestration Achieves Deterministic Decision Support](https://arxiv.org/abs/2511.15755)

---

## Model Cascades

Route requests to a fast, cheap model first. Escalate to expensive model only when the cheap model fails quality gates.

### The Pattern

```
Request → Cheap Model (Haiku/GPT-3.5)
            │
            ├─ Quality Gate Pass → Return Response
            │
            └─ Quality Gate Fail → Expensive Model (Opus/GPT-4o)
                                      │
                                      └─ Return Response
```

Cascades optimize for the common case. If 70-80% of requests are simple enough for a cheap model, routing them there first cuts costs dramatically while maintaining quality for complex requests.

### Evidence: 40-85% Cost Reduction

*[2025-12-10]*: Research on cascade architectures shows 40-85% cost reduction when cheap models successfully handle 70-80% of queries.

**Representative numbers:**
- Query mix: 70% simple (Haiku handles), 30% complex (requires Opus)
- Haiku cost: $0.001 per query
- Opus cost: $0.010 per query

Single model (all Opus): 100 queries × $0.010 = $1.00
Cascade (70 Haiku, 30 Opus): (70 × $0.001) + (30 × $0.010) = $0.37

62% cost reduction with no quality loss, assuming perfect quality gates.

The savings scale with the cheap model success rate. If 80% resolve at cheap tier, savings approach 70%. If only 50% resolve, savings drop to 35%.

**Source:** [CascadeFlow: Optimizing LLM Model Selection](https://arxiv.org/abs/2408.08948)

### Quality Gate Implementation

Quality gates determine when to escalate. Poor gates waste money by routing everything to expensive models. Overly permissive gates sacrifice quality by accepting inadequate responses.

**Common quality gate strategies:**

**Confidence thresholds:**
Model outputs confidence scores with responses. Responses below threshold (e.g., 0.7) escalate automatically.

```python
response = cheap_model.generate(prompt)
if response.confidence < 0.7:
    response = expensive_model.generate(prompt)
return response
```

This requires models that output calibrated confidence scores. Many models over-report confidence, making threshold tuning critical.

**Semantic validation:**
Check response against expected structure or content requirements. Escalate if validation fails.

```python
response = cheap_model.generate(prompt)
if not validate_json_schema(response) or missing_required_fields(response):
    response = expensive_model.generate(prompt)
return response
```

Works well for structured outputs where validation rules are clear. Less effective for open-ended generation.

**Classifier-based routing:**
Train a lightweight classifier on historical data: features from request → cheap vs. expensive model.

```python
complexity = classifier.predict(prompt_features)
model = cheap_model if complexity == "simple" else expensive_model
response = model.generate(prompt)
```

Requires labeled training data and regular retraining as query distribution shifts. Adds latency (classifier inference) but enables predictive routing before generation.

**Hybrid approach:**
Combine static classification with dynamic confidence checks.

```python
if classifier.predict(prompt) == "simple":
    response = cheap_model.generate(prompt)
    if response.confidence < 0.7:
        response = expensive_model.generate(prompt)
else:
    response = expensive_model.generate(prompt)
```

Avoids wasting cheap model invocations on obviously complex queries while maintaining quality gates for borderline cases.

---

## Planning vs. Execution Separation

Complex reasoning (planning) uses an expensive model. Instruction-following (execution) uses a cheaper model.

### The Pattern

```
Phase 1: Planning (Expensive Model)
├─ Understand requirements
├─ Decompose into subtasks
├─ Define execution instructions
└─ Output: Structured plan

      ↓ (Handoff via structured intermediate representation)

Phase 2: Execution (Cheap Model)
├─ Read structured plan
├─ Execute each subtask
├─ Report results
└─ Output: Completed work
```

The expensive model never executes. The cheap model never plans. Separation is enforced through intermediate representations—structured data formats that planning outputs and execution consumes.

### Evidence: 41.1% → 96.3% Success Rate

*[2025-12-10]*: A study using GPT-4o for planning and GPT-3.5 for execution showed task success rates increased from 41.1% (single model) to 96.3% (separated planning and execution).

The improvement came from specialization. Planning models focused on decomposition and correctness without execution distractions. Execution models followed precise instructions without second-guessing the plan.

The cost savings came from volume. Execution steps outnumber planning steps 5-10×. Using a cheap model for the high-volume phase cut costs while preserving quality through better planning.

**Source:** [Planning-Execution Separation in LLM Workflows](https://arxiv.org/abs/2406.12582)

### The Advisor Strategy: Inverted Orchestration

*[2026-04-11]*: Anthropic's officially named "advisor strategy" is a structural variant on planning-execution separation that inverts the orchestrator-specialist topology described above. In standard orchestrator-specialist architectures, the expensive model orchestrates — it decomposes tasks, routes to specialists, and synthesizes results. In the advisor strategy, the small model drives end-to-end: it calls tools, iterates, and produces output. The expensive model advises only — it receives queries from the executor, returns guidance, and never calls tools or produces user-facing responses.

This inversion changes where expensive model tokens are consumed. The executor handles the high-frequency, low-reasoning steps (tool calls, iteration loops, output formatting). The advisor handles only the low-frequency, high-reasoning steps (architectural decisions, ambiguous judgment calls). Advisor tokens are tracked separately from executor tokens, enabling precise cost attribution at the decision-type level.

**API primitive:** Anthropic ships the advisor role as a first-class platform feature. The `max_uses` parameter caps advisor calls per request, functioning as a hard cost gate rather than a soft prompt instruction.

**Evidence (Anthropic, 2026-04):**

- Sonnet (executor) + Opus (advisor): +2.7 percentage points on SWE-bench Multilingual vs. Sonnet alone, at −11.9% cost per task
- Haiku (executor) + Opus (advisor) on BrowseComp: 41.2% vs. 19.7% Haiku solo (>2× improvement), 85% cheaper than Sonnet solo at equivalent or higher accuracy

**When this pattern applies:**
- Tasks require occasional high-stakes decisions embedded in a routine execution loop — the executor handles most steps; the advisor unblocks specific judgment calls
- Cost observability per decision-type is required — `max_uses` limits enforce spending at the architectural level, not prompt level
- The executor model is capable enough to drive tool use and iteration but bottlenecks at specific reasoning quality thresholds

**Source:** [The Advisor Strategy](https://claude.com/blog/the-advisor-strategy) — Anthropic official blog, ~2026-04, retrieved 2026-04-11

---

## Model-Native Swarm Orchestration

*[2026-01-30]*: Moonshot AI's Kimi K2.5 introduced model-native swarm orchestration—a 1 trillion parameter MoE model that spawns and coordinates multiple subagents internally rather than through external framework code.

Unlike SDK-based orchestration patterns (LangGraph, AutoGen, Claude Code's Task tool), Kimi K2.5 embeds coordination logic within the model's reasoning itself. The model decides when to parallelize, spawns subagents dynamically, and synthesizes results—all as trained behavior rather than prompted instructions.

This architectural shift moves orchestration from application layer to model capability. Instead of writing coordination code, developers provide tasks and the model handles decomposition, parallel execution, and synthesis autonomously.

### The Pattern

```
┌─────────────────────────────────────────────────┐
│   Main Agent (Kimi K2.5)                        │
│   - Receives task                               │
│   - Decides: sequential or parallel?            │
│   - Spawns subagents if beneficial              │
│   - Synthesizes results                         │
└─────────────────────────────────────────────────┘
                     │
    ┌────────────────┼────────────────┬────────────┐
    ▼                ▼                ▼            ▼
┌─────────┐    ┌─────────┐    ┌─────────┐   ...  ┌─────────┐
│Subagent │    │Subagent │    │Subagent │        │Subagent │
│   #1    │    │   #2    │    │   #3    │        │  #100   │
└─────────┘    └─────────┘    └─────────┘        └─────────┘
    │                │                │                │
    └────────────────┴────────────────┴────────────────┘
                          ▼
           ┌─────────────────────────────┐
           │   Main Agent synthesizes     │
           │   all subagent outputs       │
           └─────────────────────────────┘
```

The model spawns up to 100 concurrent subagents for complex tasks. Each subagent executes independently with its own tool access. The main agent coordinates execution, tracks dependencies, and combines outputs—all without external orchestration framework.

### Evidence: PARL Training and Critical Steps

Kimi K2.5's orchestration capability comes from Parallel-Agent Reinforcement Learning (PARL)—a training approach that rewards parallelization only when it reduces wall-clock completion time.

**The Serial Collapse Problem:**
Models trained on sequential demonstrations default to single-agent execution even when parallel execution would be faster. They learn "solve step A, then B, then C" rather than "identify independent steps and execute concurrently."

PARL addresses this through the **Critical Steps** metric:

```
CriticalSteps = Σ(Smain(t) + max_i(Ssub,i(t)))
```

Where:
- `Smain(t)` = steps taken by main agent at time t
- `max_i(Ssub,i(t))` = maximum steps taken by any subagent at time t
- Sum across all timesteps in task completion

This metric counts only steps along the critical path—the longest sequential dependency chain. Parallelization reduces Critical Steps when independent subtasks execute concurrently. Sequential execution keeps Critical Steps high even with many subagents.

**Training Reward:**
The model receives higher reward for solutions with lower Critical Steps count. This incentivizes identifying parallelizable work and spawning subagents appropriately, while avoiding unnecessary parallelization when tasks have strict sequential dependencies.

### Performance: 80% Runtime Reduction

*[2026-01-30]*: Moonshot AI reports Kimi K2.5 achieves 3-4.5× wall-clock speedup on complex tasks through model-native swarm orchestration.

**Quantified results:**
- **Runtime reduction:** Up to 80% on tasks with high parallelization potential
- **Wall-clock improvement:** 3-4.5× faster completion vs. sequential execution
- **Scalability:** 100 concurrent subagents demonstrated
- **Tool calls:** 1500+ parallel tool invocations in complex workflows

**Example scenario (complex research task):**
- Sequential execution: 45 minutes (single agent, 120 steps)
- Model-native swarm: 10 minutes (1 main agent + 40 subagents, 28 Critical Steps)
- Speedup: 4.5× faster

The gains scale with task parallelizability. Tasks with strict sequential dependencies see minimal improvement. Tasks with many independent subtasks (multi-source research, parallel analysis, bulk processing) benefit most.

**Source:** Moonshot AI technical documentation (January 2026)

### Comparison: SDK Orchestration vs. Model-Native Swarm

| Dimension | SDK Orchestration | Model-Native Swarm |
|-----------|-------------------|-------------------|
| **Coordination Logic** | External framework code (LangGraph, AutoGen) | Model reasoning (trained behavior) |
| **Subagent Spawning** | Explicit API calls or Task tool invocations | Model decision during generation |
| **Parallelism Limit** | Framework/infrastructure constraints (5-10) | Model capacity constraints (100+) |
| **Developer Control** | High (explicit routing, handoffs) | Medium (prompt guidance, not code) |
| **Debugging** | Trace through coordination code | Inspect model reasoning traces |
| **Infrastructure** | Requires orchestration layer | Single model API endpoint |
| **Latency Overhead** | Framework coordination + network calls | Model-internal coordination only |
| **Token Overhead** | Context duplication across agents | Shared context in model memory |
| **Adaptability** | Fixed coordination logic | Dynamic parallelization decisions |
| **Implementation** | Write coordination code | Write task prompts |

**Key distinction:** SDK orchestration separates planning from execution through code. Model-native swarm embeds both in trained model behavior. This trades explicit control for learned efficiency.

### Trade-offs

**Autonomy vs. Control:**
The model decides when and how to parallelize. Developers provide guidance through prompts but cannot enforce specific coordination strategies. This autonomy enables adaptive parallelization but reduces determinism for workflows requiring precise execution order.

**Debuggability:**
SDK orchestration exposes coordination logic in traceable code. Model-native swarm embeds coordination in model reasoning, making it harder to inspect decision-making. Debugging requires analyzing model traces rather than stepping through framework code.

**Token Efficiency:**
Model-native swarm potentially reduces token overhead by sharing context internally rather than duplicating across API calls. However, coordination still consumes tokens—the model must reason about task decomposition, subagent allocation, and result synthesis.

**Infrastructure Simplicity:**
Eliminates orchestration framework, reducing deployment complexity. Trade-off: couples orchestration capability to specific model families. Applications cannot switch models without rebuilding coordination layer.

**Quality Variance:**
Trained coordination behavior may vary across similar tasks. SDK orchestration produces identical coordination decisions for identical inputs. Model-native swarm may decompose tasks differently based on subtle prompt variations.

### When to Use Model-Native Swarm

**Good fit:**
- Model supports native swarm capability (currently limited to Kimi K2.5)
- Tasks benefit from dynamic parallelization (model determines optimal decomposition)
- Willing to trade explicit control for autonomous coordination
- High parallelization potential (research, analysis, bulk processing)
- Trust model's coordination decisions based on validation

**Poor fit:**
- Requires deterministic coordination (regulatory, safety-critical)
- Complex handoff logic between subtasks (stateful workflows)
- Need to switch models frequently (tight coupling to model family)
- Low parallelization potential (inherently sequential tasks)
- Debugging and observability are critical requirements

**Economic threshold:**
Model-native swarm makes sense when parallelization speedup (3-4.5×) exceeds the cost of running larger model with swarm capability versus smaller model with SDK orchestration. Calculate total cost including model inference and infrastructure.

### Open Questions

- How do model-native swarms compare to SDK orchestration on identical tasks for cost, latency, and output quality?
- What debugging patterns emerge for inspecting coordination decisions embedded in model reasoning?
- How does prompt engineering change when orchestration is a trained capability rather than instructed behavior?
- Will future models (GPT-5, Claude Opus 5) adopt model-native swarm, or is this approach model-family-specific?
- What's the optimal abstraction for developers: expose swarm controls via API parameters, or treat as internal model behavior?
- How does Critical Steps metric extend to workflows with probabilistic dependencies or conditional branching?

---

### Structured Intermediate Representations

Clean model handoffs require structured data. Unstructured planning outputs force execution models to interpret intent, reintroducing the complexity separation was meant to avoid.

**JSON task definitions:**
```json
{
  "tasks": [
    {
      "id": 1,
      "action": "read_file",
      "parameters": {"path": "config.yaml"},
      "output_to": "config_data"
    },
    {
      "id": 2,
      "action": "validate_schema",
      "parameters": {"data": "config_data", "schema": "config_schema"},
      "depends_on": [1]
    }
  ]
}
```

The planning model outputs this structure. The execution model reads it as unambiguous instructions. No interpretation needed.

**Dependency graphs:**
```python
{
    "nodes": {
        "A": {"action": "fetch_data", "params": {...}},
        "B": {"action": "transform", "params": {...}},
        "C": {"action": "validate", "params": {...}}
    },
    "edges": {
        "A": ["B"],  # B depends on A
        "B": ["C"]   # C depends on B
    }
}
```

Execution engine traverses the graph, executing nodes when dependencies are satisfied. Planning model focuses on correctness of decomposition. Execution model focuses on reliable execution.

**Domain-specific languages:**
For workflows with recurring patterns, define a DSL that planning outputs and execution interprets.

```yaml
workflow:
  - step: data_extraction
    source: database
    query: "SELECT * FROM users WHERE active = true"
    output: active_users

  - step: transformation
    input: active_users
    function: anonymize_pii
    output: safe_users

  - step: export
    input: safe_users
    destination: s3://exports/users.json
```

The DSL constrains planning to valid operations, reducing ambiguity. Execution models interpret DSL without complex reasoning.

---

## Routing Strategies

How systems decide which model handles which request.

### Static Routing

Map task types to models at design time. No runtime complexity detection.

```python
ROUTING_TABLE = {
    "simple_qa": "haiku",
    "code_generation": "sonnet",
    "research": "opus",
    "summarization": "haiku"
}

model = ROUTING_TABLE[task_type]
```

**Advantages:**
- Zero overhead—no classification inference
- Deterministic and testable
- Easy to optimize per task type

**Disadvantages:**
- Requires accurate task classification upfront
- Cannot adapt to within-category complexity variance
- Brittle when task types don't map cleanly to requests

Static routing works when task types are clearly distinguishable and complexity within each type is homogeneous.

### Dynamic Routing

Classify complexity at request time. Route based on detected characteristics.

```python
complexity = analyze_request(prompt)
if complexity > THRESHOLD_HIGH:
    model = "opus"
elif complexity > THRESHOLD_MEDIUM:
    model = "sonnet"
else:
    model = "haiku"
```

**Complexity signals:**
- Prompt length (longer often means more complex)
- Keyword presence (technical terms, domain jargon)
- Structural markers (nested lists, multi-part questions)
- Historical patterns (similar requests routed to which tier)

**Advantages:**
- Adapts to actual request complexity
- Handles within-category variance
- Can route unfamiliar request types

**Disadvantages:**
- Classification overhead (latency + cost)
- Requires calibration of thresholds
- Misclassification sends simple requests to expensive models

Dynamic routing works when requests vary significantly in complexity and classification overhead is small relative to generation cost.

### Learned Routing

Train a model to predict optimal routing based on historical data.

```python
# Training: label historical requests with actual model performance
training_data = [
    (prompt_features, "haiku", quality_score),
    (prompt_features, "sonnet", quality_score),
    ...
]

router = train_classifier(training_data, target="best_model")

# Inference: route new requests
model = router.predict(extract_features(prompt))
```

**Feature engineering:**
- Embedding similarity to known query types
- Syntactic complexity (parse tree depth, clause count)
- Semantic complexity (domain-specific vocabulary density)
- User context (enterprise tier vs. free tier)

**Advantages:**
- Learns patterns from actual performance data
- Improves over time as training data accumulates
- Can optimize for custom objectives (cost, latency, quality)

**Disadvantages:**
- Requires labeled training data (expensive to collect)
- Must retrain as query distribution shifts
- Adds inference latency and complexity

Learned routing works when you have sufficient historical data and query distribution is stable enough for trained models to generalize.

### Hybrid: GPT-5 Real-Time Router

*[2025-12-10]*: OpenAI's GPT-5 system includes a real-time router that selects between fast-but-cheap and slow-but-capable models based on request analysis.

The router uses a lightweight classifier trained on millions of production queries. It extracts features from the request, predicts expected quality for each model tier, and routes to the minimum-capability model that meets quality threshold.

**Key design decisions:**
- Router runs on separate infrastructure (doesn't consume model capacity)
- Optimizes for latency—routing decision completes in <50ms
- Continuously retrains on production feedback (quality vs. predicted quality)
- Includes manual override rules for known high-stakes query types

The system demonstrates that routing becomes infrastructure-level concern at scale. Early systems embed routing in application logic. Production systems extract it as a dedicated service.

**Source:** [GPT-5 System Card](https://openai.com/research/gpt-5-system-card)

---

## Cost/Latency/Capability Trade-offs

Multi-model architectures enable optimization along multiple dimensions. Choosing which to optimize depends on use case constraints.

| Pattern | Cost | Latency | Capability | Coordination Complexity |
|---------|------|---------|------------|------------------------|
| Single Model (Frontier) | High | Medium | Maximum | None |
| Single Model (Small) | Low | Low | Limited | None |
| Orchestrator-Specialist | High | Medium | High | High |
| Model Cascade | Medium | Low (common case) | High (fallback) | Medium |
| Planning-Execution | Medium | Medium | High | Medium |
| Static Routing | Low-Medium | Low | Varies by task | Low |
| Dynamic Routing | Medium | Medium | Adapts | Medium |
| Learned Routing | Medium | Medium | Optimizes over time | High |

### Cost-Optimized: Model Cascade

When budget is the primary constraint, cascades deliver maximum cost reduction with minimal quality sacrifice.

**Configuration:**
- Tier 1: Smallest model that can handle structured tasks (Haiku)
- Tier 2: Frontier model for complex reasoning (Opus)
- Quality gate: Strict validation with low tolerance for ambiguity

**Expected savings:** 50-70% cost reduction if 70-80% of queries resolve at Tier 1.

**Trade-off:** Adds latency to requests that escalate (two model calls instead of one).

### Latency-Optimized: Static Routing + Small Models

When response time is critical, route aggressively to fast models and accept capability limitations.

**Configuration:**
- Default: Fastest available model (Haiku, GPT-3.5-Turbo)
- Exceptions: Small set of query types hard-coded to frontier model
- No quality gates (accept first response)

**Expected latency:** 200-500ms for simple queries (vs. 1-3s for frontier models).

**Trade-off:** Quality degradation on complex requests that get routed to small models.

### Capability-Optimized: Orchestrator-Specialist

When quality is paramount and budget permits, orchestrate multiple frontier models.

**Configuration:**
- Orchestrator: Largest reasoning model (Opus)
- Specialists: Mix of frontier models based on subtask requirements
- No cascade fallback (start with best model)

**Expected quality:** Near-zero variance, deterministic outcomes, maximum capability.

**Trade-off:** 10-15× token overhead compared to single-model approaches.

---

## Family-Level Consistency Benefits

Using models from the same family (Claude 3.5 Sonnet + Haiku, GPT-4o + GPT-3.5) reduces integration complexity.

**Behavioral consistency:**
Models in the same family share training data, instruction-following patterns, and output formatting. A prompt tuned for Sonnet often works well for Haiku with minimal modification.

**Example:**
```python
# Same prompt works across Claude family
PLANNING_PROMPT = """
Analyze this task and output a JSON plan:
{task_description}

Format:
{
  "subtasks": [...],
  "dependencies": {...}
}
"""

# Works for both models with similar output quality
plan_opus = claude_opus.generate(PLANNING_PROMPT)
plan_sonnet = claude_sonnet.generate(PLANNING_PROMPT)
```

**Tool calling compatibility:**
Models in the same family support identical tool schemas. Routing between them doesn't require translation layers.

**Prompt portability:**
Chain-of-thought patterns, few-shot examples, and output format instructions transfer cleanly within families. Cross-family routing often requires prompt adaptation.

**Testing simplification:**
Test prompt engineering on expensive model (Opus), deploy on cheap model (Haiku). Behavior remains predictable. Cross-family deployment breaks this workflow.

---

## Anti-Patterns

### Over-Engineering Simple Use Cases

Implementing multi-model routing for prototypes or low-volume applications wastes engineering time on premature optimization.

**Why it fails:** Coordination complexity exceeds cost savings. Building, testing, and maintaining routing logic costs more than running all queries through a single frontier model.

**Better approach:** Start with single model. Track cost and latency. Migrate to multi-model only when metrics justify the complexity.

### Ignoring Coordination Overhead

Counting only model inference costs while ignoring routing, quality gates, and result synthesis.

**Why it fails:** A cascade might save 50% on model costs but add 200ms per request for quality gate validation. For latency-sensitive applications, the overhead negates the savings.

**Better approach:** Measure end-to-end latency and cost including all coordination components. Optimize the whole system, not just model selection.

### Premature Cost Optimization Before Validating Capability

Routing to cheap models before confirming they can handle the task.

**Why it fails:** Saves money initially but produces low-quality outputs that require rework, debugging, and eventual migration back to frontier models. The rework costs more than using the right model from the start.

**Better approach:** Validate capability first with frontier models. Optimize cost once quality is proven. Downgrade models only with empirical evidence that cheaper options maintain quality.

### Cascade Without Quality Gates

Assuming cheap models will self-report failures or users will catch errors.

**Why it fails:** Models rarely indicate low confidence explicitly. Incorrect responses get returned to users. Manual error detection becomes quality assurance bottleneck.

**Better approach:** Implement automated quality gates—schema validation, confidence thresholds, or semantic checks—before returning cascade responses.

---

## Connections

- **To [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md):** Multi-model architectures often use orchestration patterns. The orchestrator coordinates specialist models just as it coordinates specialist agents. Single-message parallelism applies to multi-model calls—invoke multiple models concurrently rather than sequentially. Model-native swarm orchestration represents a shift from SDK-based coordination to trained model behavior.

- **To [Cost and Latency](../8-practices/3-cost-and-latency.md):** Multi-model architectures are cost optimization strategies. The 15× token overhead in orchestrator-specialist patterns trades tokens for quality. Model cascades trade latency for cost. Model-native swarm orchestration reduces coordination overhead by embedding parallelization in model reasoning rather than external framework calls.

- **To [Context Management](../4-context/_index.md):** Context window size varies across model tiers. Haiku supports shorter context than Opus. Multi-model routing must account for context limits—complex requests with large context may require frontier models regardless of task complexity. Model-native swarm shares context internally rather than duplicating across API calls, potentially reducing total token consumption.

- **To [Model Selection](1-model-selection.md):** Single-model selection focuses on capability matching. Multi-model architectures add routing logic that selects models dynamically based on request characteristics, enabling optimization across cost, latency, and capability dimensions simultaneously. Model-native swarm capability introduces a new selection criterion: does the model support autonomous parallel orchestration?

- **To [Evaluation](../8-practices/2-evaluation.md):** Multi-model systems require evaluation at the architecture level, not just model level. Test routing accuracy, quality gate precision, and end-to-end performance across model transitions. Model-native swarm systems require evaluating coordination quality—whether the model parallelizes appropriately and synthesizes results correctly.
