---
title: Pit of Success
description: Design context windows where the most likely output is also the correct output
created: 2025-12-08
last_updated: 2025-12-10
tags: [mental-models, context, prompts, transformers]
part: 3
part_title: Perspectives
chapter: 9
section: 1
order: 3.9.1
---

# Pit of Success

Design systems where the easiest path is also the correct path.

---

## The Core Idea

The term comes from Rico Mariani's work on .NET Framework design: make it easier to write correct code than incorrect code. Instead of guardrails that prevent mistakes, design the system so the natural path leads to the right outcome.

```
Traditional approach:        Pit of Success:

    ┌─────────┐                 ╲         ╱
    │ SUCCESS │                  ╲       ╱
    └────▲────┘                   ╲     ╱
         │ effort                  ╲   ╱
         │                          ╲ ╱
    ─────┴─────                   SUCCESS

    (climb to success)         (fall into success)
```

---

## The Context Window as Pit of Success

This is the holy grail of agentic engineering: **Shape the input tokens so the most probable output tokens are the correct ones.**

Transformers are next-token prediction machines. Every input token influences the probability distribution over output tokens. The context window isn't just "information for the model"—it's the gravitational field that pulls outputs toward certain regions of possibility space.

```
Input Tokens                    Output Distribution
┌─────────────────────┐
│ System prompt       │         ┌──────────────────┐
│ Examples            │  ───►   │ ████████ correct │  ◄── high probability
│ Relevant context    │         │ ██ plausible     │
│ Clear instruction   │         │ █ unlikely       │
└─────────────────────┘         └──────────────────┘

vs.

┌─────────────────────┐
│ Vague instruction   │         ┌──────────────────┐
│ Irrelevant context  │  ───►   │ ███ correct      │
│ No examples         │         │ ████ plausible   │  ◄── competing options
│ Contradictions      │         │ ███ unlikely     │
└─────────────────────┘         └──────────────────┘
```

When the context is a pit of success, the model "falls into" correct outputs without heroic effort. When it's not, you're relying on the model to climb uphill against its own probability landscape.

---

## What Makes Context a Pit of Success

### 1. Attention Flows to What Matters

Transformers use attention to weight which input tokens influence each output token. A well-designed context puts the most relevant information where attention will naturally land:

- **Recency bias**: Information near the end of the context gets more attention
- **Structural clarity**: Clear formatting helps attention patterns form correctly
- **Semantic proximity**: Related information grouped together creates coherent attention clusters

### 2. Examples Set the Distribution

Few-shot examples don't just "show the model what to do"—they shift the entire probability distribution. Each example is training data at inference time:

- Examples of correct format → correct format becomes most probable
- Examples of correct reasoning style → that style becomes the attractor
- Negative examples (what not to do) → those patterns become less probable

### 3. Context Primes Token Sequences

The tokens you put in prime specific continuations. "Let me think step by step" primes chain-of-thought. A code block primes code. Technical vocabulary primes technical output.

This is why prompt engineering works: you're not "instructing" the model in a human sense—you're setting up token sequences that make desired continuations statistically dominant.

---

## Designing for Input → Output

### Make the Correct Output the Obvious Continuation

```
# Weak: Model must infer what you want
"Handle this user request appropriately."

# Strong: The correct output is the only sensible continuation
"Respond to this user request with a JSON object containing:
- 'action': one of ['approve', 'deny', 'escalate']
- 'reason': a single sentence explanation

User request: {request}

Response:"
```

### Eliminate Competing Attractors

If your context contains multiple plausible paths, the model's probability mass splits between them. Remove ambiguity:

- Don't provide examples of things you don't want (they become attractors)
- Don't hedge instructions ("maybe do X or possibly Y")
- Don't include irrelevant context that could prime wrong directions

### Position Information by Importance

Given attention patterns and recency effects:

1. **System-level identity and constraints** → beginning (establishes frame)
2. **Task-specific context and data** → middle (bulk of working memory)
3. **Specific instruction and format** → end (highest attention, immediate priming)

---

## The Leverage

This mental model reframes prompt engineering from "writing good instructions" to "engineering probability distributions."

You're not convincing a mind. You're shaping a mathematical landscape where the peak of the probability mountain is exactly where you want the output to land.

When you get this right:
- Less token spend on elaborate instructions
- More consistent outputs across runs
- Robustness to model variations
- Agents that "just work" because the context makes correctness inevitable

---

## Connections

- **[Context](../4-context/_index.md)**: The mechanics of what goes in the window
- **[Prompt Structuring](../2-prompt/2-structuring.md)**: Tactical implementation of these ideas
- **[Twelve Leverage Points](../1-foundations/1-twelve-leverage-points.md)**: Context design as high-leverage intervention
