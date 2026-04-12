---
title: Model Selection
description: How to choose the right model for agentic tasks
created: 2025-12-09
last_updated: 2026-04-11
tags: [models, selection, frontier, cost, slm, autoresearch, training]
part: 1
part_title: Foundations
chapter: 3
section: 1
order: 1.3.1
---

# Model Selection

Choosing the right model is simpler than it seems: default to frontier, downgrade only with evidence.

---

## Default to Frontier

For nearly every task, reach for the highest-capability model accessible to you. As of April 2026, that is Claude Opus 4.6 for most practitioners using the standard API.

Maximizing compute is the goal—frontier models consistently outperform smaller alternatives in reasoning depth, tool use, and complex instruction-following.

The gap between different labs' SOTA models (Gemini 3 Pro vs. Opus 4.6) has shrunk significantly over the past year. But frontier vs. non-frontier still matters enormously.

**When in doubt, use a frontier model.** Frontier should be your default. Only downgrade when there's evidence or clear logical reasoning to do so.

**Free-tier models are not slower frontier models — they are architecturally different.** Consumer free-tier access is optimized for chat interaction, not for accuracy or agentic task completion. Mollick (2026) characterizes the distinction as qualitative, not merely speed or cost: free-tier models underperform paid tiers by a measurable margin on complex tasks. For agent development, use frontier API access ($20+/month or direct API), not free chat interfaces.

> **Note — Capability tier vs. access tier:** "Frontier" means the highest-capability tier accessible to you, not the highest-capability model that exists. As of April 2026, Anthropic's Claude Mythos Preview (deployed under Project GlassWing) represents a capability tier above the standard API frontier but is accessible only through invitation-only programs for entities with critical infrastructure responsibilities. Standard-API practitioners should treat Opus 4.6 as their frontier ceiling. See [Capability-Gated Access Tiers](2-model-behavior.md#capability-gated-access-tiers) for the full pattern.

---

## What Capabilities Actually Matter

**Reasoning and tool use are the most important.**

Extended thinking and step-by-step reasoning directly address multi-step planning requirements. Instruction-following is important but is more of a prompting concern than a model selection concern—it's better framed as "how can I set up instructions so that any SOTA LLM can follow them?"

*[2026-04-11]*: **Evidence: The frontier gap is qualitative, not just quantitative.**

Claude Mythos Preview, assessed by Anthropic's Frontier Red Team (April 2026), demonstrated a step-change above Opus 4.6 on security-relevant agentic tasks:

| Task | Mythos Preview | Opus 4.6 |
|------|----------------|----------|
| Firefox 147 exploit attempts (of 200) | 181 successful | 2 successful |
| Tier-5 full control-flow hijacks (OSS-Fuzz) | 10 | 0 |
| USAMO mathematical olympiad | 97.6% | 42.3% |
| Autonomous zero-day discovery | CVE-2026-4747 (17-year-old FreeBSD NFS RCE) | — |

Expert validators confirmed results at 89% exact agreement (N=198). The gap is not incremental—it reflects qualitatively different capability on complex multi-step agentic tasks requiring reasoning across long dependency chains.

This is the strongest available illustration of why "default to frontier" is a reliability recommendation, not a cost-sensitivity preference. The table also illustrates that the capability gap is domain-dependent: it is widest for tasks requiring multi-step exploitation reasoning and mathematical problem-solving, both of which are structurally similar to complex multi-agent engineering tasks.

**Sources:** Anthropic Frontier Red Team ([red.anthropic.com/2026/mythos-preview/](https://red.anthropic.com/2026/mythos-preview/))

---

## When to Downgrade

Use smaller, faster models for specific task types where they excel:

- **Scouting agents**: Haiku 4.5 works well for investigative tasks—reading content quickly and distilling insights. These aren't computationally intensive tasks, so the speed/cost tradeoff makes sense.
- **Simple retrieval or filtering**: When the task is more about finding than reasoning.

Reliability compounds across a system—small model failures cascade through multi-step workflows. Don't mistake cost savings for optimization.

### Small Models Are RAG

*[2025-12-09]*: Small models function as RAG systems when embedded in an orchestrator pattern.

```
USER/trigger → Orchestrator → Delegation → retrieval-agent (Haiku, natural_language_query)
```

**Context Staging** for the retrieval agent:

| Component | Token Type |
|-----------|------------|
| `base.cc/` | Agent tokens (Claude Code base config) |
| `project_prompt` | Agent tokens |
| `tool_info` | Tool input tokens |
| `tool.call (query)` | Tool input tokens |
| `tool.result` | prompt/sys.info |
| `distillation synthesis` | prompt/sys.info |

The pattern: keep the small model's context minimal (base config, project prompt, tool info, query), let it retrieve and distill, then return results to the orchestrator for synthesis.

This explains why Haiku excels at scouting—it's not doing heavy reasoning, it's doing targeted retrieval with lightweight synthesis. The orchestrator handles the complex reasoning; the small model handles the fast, focused lookup.

**See Also**:
- [Orchestrator Pattern: Capability Minimization](../6-patterns/3-orchestrator-pattern.md#capability-minimization) — How tool restriction complements model selection
- [Context: Context Loading vs. Accumulation](../4-context/3-context-patterns.md#context-loading-vs-context-accumulation) — The "payload" mental model that makes small models work
- [Context Loading Demo](../../appendices/examples/context-loading-demo/README.md) — Demonstrates Haiku as retrieval agent with minimal context payloads (~800 tokens)

---

### When to Train Your Own

*[2026-04-11]*: A third pathway now exists between using a commodity SLM and staying on frontier: training a small model overnight specifically for your production task via an autonomous agent loop. This is distinct from prompt-tuning or hyperparameter search — the agent modifies model architecture, optimizer configuration, attention patterns, and regularization, and rewrites the training loop itself.

**The mechanism: autoresearch**

Autoresearch (Karpathy, 2026-03-06) is an autonomous agent loop that receives a training script (`train.py`) and a single validation metric (bits-per-byte on a held-out set), then iterates approximately 12 cycles per hour — running five-minute experiments and applying a binary keep/discard decision per cycle. An overnight run delivers roughly 100 experiments. The agent searches over architecture and training-procedure space, not hyperparameter space. The formal basis is gradient-free evolutionary program adaptation: GEPA (arxiv 2501.09361, ICLR 2026) formalizes this as evolutionary search over training programs where the validation metric is the fitness function.

Four constraints make unattended runs viable (independently identified by the HN community and embedded in Karpathy's implementation):

| Constraint | What it limits |
|------------|----------------|
| One-file scope | Changes confined to a single training script |
| One scalar metric | A single measurable validation signal drives all decisions |
| Time-boxed experiments | Each trial capped at ~5 minutes, preventing runaway compute |
| Git checkpoint per run | Every accepted change is committed; rollback is always available |

**Production evidence**

Two high-credibility results establish viability:

- **Karpathy (2026-03-07):** 700 experiments over two days on the GPT-2 / nanochat codebase. 20 real improvements accepted. 11% training speedup (2.02 → 1.80 hours). Single H100 GPU. Reproducible via the karpathy/autoresearch repository.
- **Lütke / Shopify (2026-03-10):** 37 experiments over 8 hours on internal query expansion data. A 0.8B-parameter trained-to-task model outperformed the prior manually-configured 1.6B model by 19% on the target task. The smaller model beat the larger model because the larger model was general-purpose; the smaller model was task-optimal.

Academic backing: Marquez Ayala et al. (KDD SKnowLLM Workshop 2025, arxiv 2505.24189) found fine-tuned SLMs outperform prompted frontier LLMs by 10% on structured domain tasks.

**When to use this pathway**

This pathway applies when all four conditions hold:

1. A high-volume, narrow-scope production task exists with stable requirements
2. A commodity SLM has been tested against that task and found insufficient
3. GPU compute for overnight training is available (single H100 or equivalent)
4. Task-specific training data exists in sufficient quantity

It does not apply to general-purpose workloads, tasks that change frequently, or prototypes where requirements are still being discovered.

**Note:** Autoresearch does not involve knowledge distillation or synthetic data generation. It is architecture and training-procedure search over real task data.

**See Also:**
- [Evaluation Inside Training Loops](5-model-evaluation.md#evaluation-inside-training-loops) — How validation loss functions as the fitness signal inside the autoresearch loop
- [Autonomous Loops](../6-patterns/4-autonomous-loops.md) — Structural parallel: autoresearch is an autonomous loop applied to model improvement
- [Cost and Latency](../7-practices/3-cost-and-latency.md) — Economics of SLM fine-tuning vs. frontier model API costs

**Sources:**
- Schmid, P. "How Autoresearch will change Small Language Models adoption." philschmid.de, ~2026-03-10. https://www.philschmid.de/autoresearch
- Karpathy, A. karpathy/autoresearch. GitHub, 2026-03-06. https://github.com/karpathy/autoresearch
- Karpathy, A. X post (700 experiments / 11% speedup). 2026-03-07. https://x.com/karpathy/status/2031135152349524125
- Lütke, T. X post (0.8B > 1.6B, 19% gain). 2026-03-10. https://x.com/tobi/status/2030771823151853938
- HN: "Autoresearch: Agents researching on single-GPU nanochat training automatically." 2026-03-08. https://news.ycombinator.com/item?id=47291123
- GEPA (arxiv 2501.09361). ICLR 2026. https://arxiv.org/abs/2501.09361
- Marquez Ayala et al. "Fine-Tune an SLM or Prompt an LLM?" KDD SKnowLLM Workshop 2025. https://arxiv.org/abs/2505.24189

---

## Capability vs. Latency vs. Cost

**Capability is most important.**

You should only downgrade from frontier models if you absolutely need to—and that's usually a cost-driven decision. Frontier models are significantly more expensive, but they're expensive for a reason.

Latency matters most in multi-agent architectures: a single orchestrator running a frontier model can coordinate several subagents that gather and synthesize information across domains. The orchestrator needs to be smart; the scouts can be fast.

---

## The Decision Framework

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Model Selection                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. Start with frontier model (Opus 4.6, etc.)                          │
│                         │                                                │
│                         ▼                                                │
│  2. Is the task working reliably?                                        │
│         │                    │                                           │
│        YES                   NO                                          │
│         │                    │                                           │
│         ▼                    ▼                                           │
│  3. Is cost a problem?    Debug prompt/context first                     │
│         │                                                                │
│        YES                                                               │
│         │                                                                │
│         ▼                                                                │
│  4. Test with smaller model                                              │
│         │                                                                │
│         ▼                                                                │
│  5. Does it still work reliably?                                         │
│         │                    │                                           │
│        YES                   NO                                          │
│         │                    │                                           │
│         ▼                    ▼                                           │
│     Use smaller     Is the task narrow + data available?                │
│                         │                    │                           │
│                        YES                   NO                          │
│                         │                    │                           │
│                         ▼                    ▼                           │
│                 Consider trained-to-  Stay with frontier                │
│                 task SLM via                                             │
│                 autoresearch loop                                        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

The key insight: only consider downgrading *after* you have a working solution with a frontier model. Optimizing cost before validating capability is premature optimization. If no existing smaller model meets requirements and the task is narrow with available data, a trained-to-task SLM via an overnight autoresearch loop is now a viable third option — not a research experiment.

---

## Cross-Provider Selection

*[2026-04-11]*: The frontier-first rule answers "which tier?" within a provider's model family. A second question arises when choosing where to build agentic systems: "which provider's frontier?"

Mollick (2026) identifies three distinct dimensions of this choice:

- **Model** — the underlying capability tier (Opus 4.6, GPT-5.2, Gemini 3 Pro are all frontier and have converged significantly in overall capability)
- **App** — the chat interface (claude.ai, chatgpt.com, gemini.google.com)
- **Harness** — the autonomous execution environment enabling multi-step task completion (Claude Code, OpenAI Codex, Google Antigravity)

**The harness matters more than the model within the frontier tier.** Claude Opus 4.6 in a chat window performs differently from Claude Opus 4.6 inside Claude Code — same model, different harness, dramatically different outcomes. Agent Psychometrics research (arXiv:2604.00594) quantifies this: agent success probability follows P(success) = σ(θ_LLM + θ_scaffold − β_difficulty), where model capability and scaffold quality are additively independent contributions. Improving the harness and improving the model yield comparable, separable gains.

### Harness Availability by Provider

| Provider | Frontier Model | Agentic Harness | Harness Maturity |
|----------|----------------|-----------------|------------------|
| Anthropic | Opus 4.6 | Claude Code | High — production-grade, coding and file ops |
| OpenAI | GPT-5.2 | OpenAI Codex | High — leading on terminal/agentic benchmarks |
| Google | Gemini 3 Pro | No direct equivalent | Limited — Gemini website is chat-optimized |

Google's Gemini 3 Deep Think is capable as a model; the harness gap is the constraint. For practitioners building autonomous agents, harness availability should precede model capability comparison.

### Task-Type → Provider Heuristics

When cross-provider choice is on the table, task type drives the selection:

| Task Type | Recommended Provider | Rationale |
|-----------|---------------------|-----------|
| Coding, file manipulation, agentic loops | Anthropic (Claude Code) or OpenAI (Codex) | Purpose-built harnesses; Claude: systematic caution, requests clarification before assuming; GPT-5.3-Codex: leads Terminal-Bench 2.0 at 77.3% |
| Statistical analysis, quantitative reasoning | OpenAI (GPT-5.2 Pro) | Described as superior for "complex statistical and analytical work" (Mollick, 2026) |
| Google Workspace, Docs, Sheets integration | Google (Gemini) | Near-flawless within Google ecosystem; agency currently restricted to that ecosystem |
| Research synthesis over large document sets | Google (NotebookLM harness) | Specialist harness for document synthesis; no Anthropic or OpenAI equivalent |

These heuristics reflect early 2026 harness availability and may shift as providers release new execution environments.

### Architecture Sensitivity by Model Type

AgentArch (arXiv:2509.10769) evaluated model-architecture pairings across enterprise benchmarks. Findings relevant to cross-provider selection:

- GPT-4.1 and Claude Sonnet 4 were most robust across agentic configurations (consistent performance regardless of architectural setup)
- o3-mini (reasoning-first model) showed extreme sensitivity to architectural choices: 1.3%–56.7% performance range across configurations
- Smaller models (GPT-4.1-mini at 67.1%) matched larger models (Sonnet 4 at 68.5%) with optimal configuration

**Implication:** Reasoning models (o-series, extended thinking) are not universally superior to standard frontier models in agentic systems — they are more sensitive to getting the harness configuration right. Standard frontier models offer more predictable baselines across configurations.

**See Also:**
- [Agent Frameworks](../9-practitioner-toolkit/4-agent-frameworks.md) — Framework comparison and harness capability tiers
- [Model Evaluation](5-model-evaluation.md) — Cognitive profiles across providers
- [Multi-Model Architectures](4-multi-model-architectures.md) — Cross-provider orchestration patterns

---

## Multi-Agent Model Selection

*[2026-01-30]*: Orchestration research reveals distinct model selection strategies for multi-agent systems. The frontier-first rule still applies, but spawn strategy changes based on model economics.

### Three-Tier Spawn Strategy

| Model | Cost | Speed | Primary Use Cases | Spawn Count |
|-------|------|-------|-------------------|-------------|
| **Haiku** | Lowest | Fastest | Information gathering, pattern finding, file discovery, grep-style searches | Many (5-10 parallel) |
| **Sonnet** | Medium | Medium | Well-defined implementation, established patterns, integration work, standard refactoring | Balanced (2-4 parallel) |
| **Opus** | Highest | Slowest | Architectural decisions, ambiguous problems, creative solutions, security review | Selective (1-2 serial) |

### Decision Matrix

**Use Haiku when:**
- Finding files or code patterns
- Fetching documentation quickly
- Simple data extraction tasks
- Quick exploration of multiple areas
- Cost/speed critical for parallel work
- Task has clear retrieval pattern

**Use Sonnet when:**
- Implementing features with clear specifications
- Refactoring with established patterns
- Writing tests from requirements
- Integration work between components
- Standard complexity reasoning
- Building on prior design decisions

**Use Opus when:**
- Designing system architecture
- Solving ambiguous problems without clear solution path
- Making security-critical decisions
- Creative problem-solving requiring novel approaches
- High-stakes decisions with broad impact
- Complex reasoning with many interdependencies

### Pipeline Escalation Pattern

**Common progression:** Start cheap for data gathering, escalate for decisions, balance for execution.

```
Haiku (Research) → Opus (Design) → Sonnet (Implement) → Haiku (Verify)
  ↓                  ↓                ↓                    ↓
Fast data         Creative         Efficient          Fast
gathering         decisions        execution       verification
```

**Rationale:**
- Research phase processes large volumes (many files, broad search) — cheap models work
- Design decisions require reasoning about trade-offs — expensive models excel
- Implementation follows established patterns — mid-tier models sufficient
- Verification is pattern matching against specs — cheap models adequate

### Economic Optimization

**The spawn count insight:** Cost per agent decreases with parallelism for cheaper models.

**Example calculation:**
- 10 Haiku agents (parallel) complete in ~30 seconds total
- 1 Opus agent completes in ~30 seconds
- Cost: 10 × $0.01 = $0.10 (Haiku) vs. 1 × $1.00 = $1.00 (Opus)
- Haiku delivers 10× breadth of exploration for 1/10 cost

**When to spawn many:**
- Independent search/analysis tasks
- Exploration requires coverage, not depth
- Fast feedback more valuable than perfect reasoning
- Failure of one agent doesn't block others

**When to spawn few:**
- Tasks require deep reasoning
- Work builds sequentially on prior decisions
- Context accumulation across agents costly
- High-stakes decisions warrant best model

### Orchestrator Model Selection

**The orchestrator itself typically uses Sonnet or Opus:**
- Coordination decisions require reasoning
- Synthesis across multiple agent outputs complex
- Workflow routing based on partial results
- Cost amortizes across many subagent spawns

**Rarely use Haiku for orchestration:**
- Weak at synthesis and decision-making
- May mis-route work or spawn wrong specialists
- Savings on orchestrator negligible vs. total workflow cost

### Practical Example: PR Review

```
Orchestrator (Sonnet):
├─ Spawn 3 parallel reviewers (single message):
│  ├─ Security (Opus): Deep analysis, high stakes
│  ├─ Performance (Sonnet): Standard profiling patterns
│  └─ Style (Haiku): Pattern matching against style guide
├─ Synthesis (Orchestrator Sonnet): Unified review
└─ Validation (Haiku): Verify review covers all files
```

**Cost breakdown:**
- 1 Opus (security): $1.00
- 2 Sonnet (performance + orchestrator): $0.50
- 2 Haiku (style + validation): $0.02
- **Total: ~$1.52** for comprehensive multi-perspective review

Compare to single Opus doing everything: $1.00 but serial execution (5× slower) and no parallelism benefits.

**Sources:** [cc-mirror orchestration patterns](https://raw.githubusercontent.com/numman-ali/cc-mirror/main/src/skills/orchestration/references/patterns.md), [Model selection strategy](https://raw.githubusercontent.com/numman-ali/cc-mirror/main/src/skills/orchestration/SKILL.md)

---

## See Also

- [Orchestrator Pattern](../6-patterns/3-orchestrator-pattern.md) - How model selection integrates with coordination patterns
- [Cost and Latency](../7-practices/3-cost-and-latency.md) - Economic optimization for multi-agent systems
- [Evaluation Inside Training Loops](5-model-evaluation.md#evaluation-inside-training-loops) — How validation loss functions as the fitness signal in autoresearch
