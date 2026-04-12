---
title: Model
description: Understanding and leveraging the capabilities of foundation models in agentic systems
created: 2025-12-08
last_updated: 2026-04-11
tags: [foundations, models, capabilities]
part: 1
part_title: Foundations
chapter: 3
section: 0
order: 1.3.0
---

# Model

The model is the engine. Its capabilities and limitations define what's possible.

---

## Sections

- [Model Selection](1-model-selection.md) — How to choose the right model for agentic tasks
- [Model Behavior](2-model-behavior.md) — How models behave in agentic contexts—variance, consistency, temperature effects, and behavioral patterns
- [Model Limitations and Workarounds](3-model-limitations.md) — Common model limitations in agentic contexts and practical workarounds—math, hallucination, context limits, instruction drift, and upgrade breakage
- [Multi-Model Architectures](4-multi-model-architectures.md) — When and how to use multiple models in agent systems—orchestrator patterns, cascades, routing strategies, and planning versus execution separation
- [Model Evaluation](5-model-evaluation.md) — How to evaluate models for agentic tasks—metrics, benchmarks, observability, and the compound error problem

---

## Core Questions

This chapter explores:

- **Selection**: How do you choose the right model for a task? What capabilities matter most?
  - **Cross-provider selection**: When does provider choice matter for agentic systems? Which harness ecosystem fits which task type?
- **Behavior**: What model behaviors help or hinder agentic work? How do you account for variance?
- **Limitations**: What can't models do? How do you work around constraints?
- **Architecture**: When do multi-model systems make sense? How do you route between them?

---

## The Short Version

**Default to frontier models.** The capability gap between SOTA and everything else still matters more than cost optimization in most cases. Downgrade only when you have evidence that a smaller model works reliably for your specific task.

**Access tier is distinct from capability tier.** As of April 2026, the highest-capability model (Claude Mythos Preview) is not accessible via the standard API — it is restricted to enrolled programs. For standard-API practitioners, Opus 4.6 is the frontier ceiling. See [Capability-Gated Access Tiers](2-model-behavior.md#capability-gated-access-tiers).

**Within the frontier tier, harness ecosystem matters more than model capability differences.** Top frontier models (Opus 4.6, GPT-5.2, Gemini 3 Pro) have converged in overall capability. The harness — the autonomous execution environment — is the primary differentiator for agentic task outcomes. Choose provider based on harness availability for your task type before comparing model benchmarks.

**Reasoning and tool use are the capabilities that matter most** for agentic work. Context length, speed, and cost are secondary — important for architecture decisions, but not for the core question of "can this model do the job?"

---

## Connections

- **To [Prompt](../2-prompt/_index.md):** Instruction-following is shaped more by prompt quality than model selection. Different models respond differently to the same prompt, but SOTA models are generally more forgiving.
- **To [Context](../4-context/_index.md):** Context length limits shape architecture decisions. Frontier models tend to have larger windows, but the capability-capacity tradeoff still applies.
- **To [Tool Use](../5-tool-use/_index.md):** Some models are better at tool-use than others. This is a core selection criterion for agentic work.
- **To [Practitioner Toolkit](../9-practitioner-toolkit/_index.md):** Harness availability varies by provider. Model selection and harness selection are coupled decisions for agentic systems. See [Agent Frameworks](../9-practitioner-toolkit/4-agent-frameworks.md) for harness capability tiers.
