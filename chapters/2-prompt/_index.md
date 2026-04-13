---
title: Prompt
description: The art and science of instructing agentic systems
created: 2025-12-08
last_updated: 2025-12-10
tags: [foundations, prompt-engineering]
part: 1
part_title: Foundations
chapter: 2
section: 0
order: 1.2.0
---

# Prompt

The prompt is how you communicate intent to an agent. This is the interface between human goals and machine action.

*[2025-12-10]*: Unlike traditional programming where you write explicit instructions, prompts operate in a space of *interpreted intent*. The same words can produce different behaviors across models, contexts, and even runs. This makes prompt engineering both more accessible (natural language) and more subtle (emergent behavior).

---

## Core Concepts

### The Prompt Is Not Just Text

A prompt is everything that shapes the model's behavior before it generates output:

- **System instructions** - Persistent behavioral directives
- **User messages** - The immediate request or conversation
- **Injected context** - Files, tool outputs, prior conversation
- **Structural cues** - Formatting, examples, templates that guide output shape

In agentic systems, the "prompt" often spans multiple files and injection points. Understanding this distributed nature is key.

### Static vs. Dynamic Prompts

Prompts exist on a spectrum from completely fixed to highly adaptive:

| Type | Description | When to Use |
|------|-------------|-------------|
| Static | Same text every time | Simple, predictable tasks |
| Parameterized | Text with variable slots | User-specific customization |
| Conditional | Branches based on context | Multi-path workflows |
| Contextual | Pulls from external sources | Knowledge-intensive tasks |
| Composed | Invokes other prompts | Complex orchestration |

See [Prompt Types](1-prompt-types.md) for the full maturity model.

### The Prompt-Model Contract

Every prompt implicitly makes assumptions about the model:
- What it knows (training data, knowledge cutoff)
- What it can do (reasoning depth, tool use, output formats)
- How it interprets instructions (literal vs. inferred intent)

When prompts fail, it's often because the contract was violated—either the prompt assumed capabilities the model lacks, or the model interpreted instructions differently than intended.

---

## Principles

### 1. Clarity Over Cleverness

The model will interpret what you write, not what you meant. Explicit, unambiguous instructions outperform clever shortcuts:

```
# Fragile
"Handle errors appropriately"

# Robust
"When an error occurs:
1. Log the error message and stack trace
2. Return a user-friendly message (never expose internals)
3. Continue execution if possible, exit gracefully if not"
```

### 2. Structure Reduces Variance

Formatting matters. Consistent structure helps the model parse intent and produce predictable output:

- Use headers to delineate sections
- Use lists for sequences or options
- Use code blocks for structured data
- Use explicit markers like `## Instructions` or `## Output Format`

### 3. Constraints Enable Creativity

Paradoxically, adding constraints often improves output quality. Without boundaries, models may:
- Over-explain or under-explain
- Choose arbitrary formats
- Hallucinate structure

Good constraints: output format, length limits, required sections, explicit non-goals.

### 4. Examples Beat Explanations

When possible, show don't tell. Few-shot examples establish:
- Expected input/output format
- Tone and style
- Edge case handling

But beware: examples can also anchor the model too strongly. Use them for format, not for content creativity.

---

## Agent-Specific Considerations

### Tool Documentation

Agents need to know *when* and *how* to use tools. Effective tool prompts include:

```markdown
## Available Tools

### search_codebase
Use when: You need to find where something is defined or used
Parameters:
  - query (string): The search term
  - file_type (optional): Limit to specific extensions
Returns: List of matching file paths with line numbers

Do NOT use for: Reading file contents (use read_file instead)
```

The "Do NOT use for" section prevents common misuse patterns.

### Stopping Conditions

Agents can loop indefinitely without clear stopping criteria:

```markdown
## When to Stop

Stop and report results when ANY of these are true:
- The requested change has been verified working
- You've attempted 3 fixes for the same error
- You need information only the user can provide
- You've exceeded the scope of the original request
```

### Uncertainty Handling

Agents need guidance on when to act vs. when to ask:

```markdown
## Handling Uncertainty

ASK the user when:
- The request is ambiguous and multiple interpretations are valid
- The change has significant consequences (data loss, breaking changes)
- You're missing critical context

PROCEED with best judgment when:
- The choice is stylistic, not functional
- You can easily verify your assumption
- The action is reversible
```

---

## Iteration & Debugging

### When the Prompt Is the Problem

Signs that prompt issues (not model limitations) are causing failures:
- Inconsistent behavior across runs
- Model following instructions literally but missing intent
- Correct reasoning, wrong output format
- Model explaining why it can't do something it actually can

### Debugging Approach

1. **Isolate** - Test the prompt with minimal context
2. **Verbose** - Ask the model to explain its interpretation
3. **Simplify** - Remove sections until behavior changes
4. **Contrast** - Compare working vs. failing cases

### Version Control for Prompts

Treat prompts like code:
- Store in version control
- Document changes with rationale
- Test changes against known inputs
- Consider A/B testing in production

---

## Anti-Patterns

| Anti-Pattern | Why It Fails | Better Approach |
|--------------|--------------|-----------------|
| "Be smart about this" | Vague, unactionable | Specify what "smart" means |
| "Do whatever's best" | No criteria provided | Define success criteria |
| Walls of text | Information overload | Structure with headers |
| Implicit assumptions | Model may not share them | State assumptions explicitly |
| Over-constraining | Blocks valid solutions | Constrain output, not process |

---

## Connections

- **To [Model](../3-model/_index.md):** Different models require different prompting styles. *[2025-12-10]*: Claude tends to follow instructions literally; adjust verbosity accordingly.
- **To [Context](../4-context/_index.md):** The prompt defines how context should be used. Without guidance, injected context may be ignored or misinterpreted.
- **To [Tool Use](../5-tool-use/_index.md):** Tool descriptions are prompts themselves. Poor tool docs lead to misuse regardless of the main prompt quality. Tool restrictions define what agents can do—a form of capability prompting.
- **To [Prompt Types](1-prompt-types.md):** Understanding prompt maturity levels helps you choose the right complexity for your use case.
- **To [Prompt Language](3-language.md):** Specific linguistic choices—verb mood, constraint framing, structural delimiters—significantly impact how models interpret and execute prompts.
- **To [Claude Code](../10-practitioner-toolkit/1-claude-code.md):** Skills (model-invoked) and slash commands (user-invoked) exemplify the invocation pattern trade-offs in practice.

---

## One-Shot vs. Conversational Agents

A critical design decision: is the agent meant to complete a task autonomously, or to collaborate through dialogue?

*[2025-12-10]*: **One-shot agents** should receive everything they need upfront. The initial prompt is the entire interface—if it's insufficient, the task fails. This favors:
- Comprehensive instructions over brevity
- Explicit constraints and stopping conditions
- Self-contained context (no assumptions about follow-up)

**Conversational agents** operate differently. They can clarify, iterate, and adapt. This allows:
- Lighter initial prompts that evolve through dialogue
- More tolerance for ambiguity (can be resolved interactively)
- Progressive disclosure of context as needed

Mixing these paradigms causes problems. A one-shot prompt that asks clarifying questions wastes the user's time. A conversational prompt that tries to do everything at once overwhelms the interaction.

---

## Model-Invoked vs. User-Invoked Prompts

A fundamental design question for prompt systems: should the prompt activate automatically, or require explicit user invocation?

### The Distinction

**Model-invoked prompts** activate autonomously based on semantic matching. The model analyzes the user's request, compares it to available capabilities, and selects relevant prompts to activate. Examples:
- Skills in Claude Code (semantic matching via description field)
- Subagent delegation (orchestrator chooses which agent to spawn)
- Context-triggered behaviors (auto-loading documentation when relevant)

**User-invoked prompts** require explicit triggering by name or alias. The user must know what's available and choose to activate it. Examples:
- Slash commands (`/knowledge:capture`, `/review:clarity`)
- Direct instructions referencing a specific capability
- Menu-driven or autocomplete-triggered workflows

### Trade-offs

**Model-invoked prompts enable proactive specialization without user knowledge:**

Strengths:
- User doesn't need to memorize command names
- New capabilities integrate transparently
- Natural language request → appropriate specialist
- Reduces cognitive load on user

Costs:
- Requires rich descriptions with trigger terms for semantic matching
- Harder to debug ("Why didn't it use the specialist I wanted?")
- Less predictable—same request might trigger different behaviors
- Can feel like "magic" when matching is opaque

**User-invoked prompts offer precise control but demand awareness:**

Strengths:
- Explicit, predictable activation
- User chooses exactly which capability to engage
- Easier to debug (explicit invocation in logs)
- Clear boundaries between different workflows

Costs:
- User must know what's available
- Requires learning command names/syntax
- Doesn't scale well past ~20 commands
- Friction in discovery of new capabilities

### Design Implications

**For model-invoked prompts:**

The description field is critical. It needs:
- **Trigger terms** the model can semantically match against user requests
- **Scope boundaries** so the model knows when NOT to activate it
- **Differentiation** from similar capabilities

Example (from Claude Code Skills):
```yaml
---
name: security-reviewer
description: >
  Security-focused code review specialist. Use when analyzing code for
  vulnerabilities, authentication/authorization issues, injection attacks,
  or security best practices. NOT for general code quality or style review.
---
```

The trigger terms ("vulnerabilities", "authentication", "injection attacks") help the model match relevant requests. The negative constraint ("NOT for general code quality") prevents over-activation.

**For user-invoked prompts:**

Naming and brevity matter most:
- **Short, memorable names**: `/capture` beats `/knowledge:add-new-entry`
- **Clear prefixes for namespacing**: `/knowledge:*`, `/review:*`, `/build:*`
- **Brief descriptions for autocomplete**: "Capture a learning" not "This command allows you to capture insights and learnings by..."

The user will read the name/description in a menu or autocomplete dropdown. Optimize for scanability.

### When to Use Which

**Use model-invocation for repeatable, semantic workflows:**
- The capability will be used 3+ times per week
- The trigger condition can be described semantically (not just syntactically)
- The user benefit from not having to remember to invoke it
- You can write a rich description with clear boundaries

**Use user-invocation for exploratory or preference-sensitive tasks:**
- The task is rare or one-off
- The user needs to consciously choose when to engage it
- Multiple valid approaches exist and the user should decide
- The workflow is still being refined

**Hybrid approach:**

Many systems benefit from both. Example from this knowledge base:
- **Model-invoked**: Subagents for research, analysis, implementation (Skills pattern)
- **User-invoked**: Workflow orchestrators, manual review commands (slash commands)

The model-invoked layer handles delegation to specialists. The user-invoked layer provides explicit control over high-level workflows.

### Implementation Examples

**Claude Code Skills** (model-invoked):
Skills are defined in `.claude/agents/*.md` with frontmatter containing `name` and `description`. The orchestrator reads the description and decides whether to spawn the subagent based on semantic matching. See [Claude Code: Subagent System](../10-practitioner-toolkit/1-claude-code.md#subagent-system) for details.

**Slash Commands** (user-invoked):
Commands are defined in `.claude/commands/**/*.md` and appear in the autocomplete menu when the user types `/`. The user sees the command name and brief description, then chooses to invoke. See [Claude Code documentation](https://code.claude.com/docs/en/slash-commands) for implementation details.

**Comparison:**

| Aspect | Skills (Model-Invoked) | Slash Commands (User-Invoked) |
|--------|------------------------|-------------------------------|
| Activation | Automatic via semantic match | Explicit via `/command` |
| Discovery | Transparent to user | Must browse or remember |
| Description | Rich, with trigger terms | Brief, for scanability |
| Predictability | Lower (model decides) | Higher (user decides) |
| Best for | Recurring specialized tasks | Explicit workflow control |

### Practical Guidance

**Improving model-invoked matching:**
- Include synonyms and domain terms in descriptions
- Add "Use when..." and "Do NOT use when..." sections
- Test with varied phrasings of the same request
- Monitor activation logs to see false positives/negatives

**Improving user-invoked discoverability:**
- Namespace commands by domain (`/knowledge:`, `/review:`)
- Keep names short and action-oriented
- Provide rich help text via `/help` commands
- Consider a `/discover` command that suggests relevant capabilities

---

## Measuring Prompt Quality

Beyond "did it work?", prompt quality has multiple dimensions:

| Dimension | What It Measures |
|-----------|------------------|
| **Structure** | Is it well-organized and parseable? |
| **Reusability** | Can it be adapted for similar tasks? |
| **Templating** | Are the variable parts clearly delineated? |
| **Breadth** | Does it handle the full scope of expected inputs? |
| **Depth** | Does it handle edge cases and nuances? |
| **Maintainability** | Can someone else understand and modify it? |

A prompt can succeed at its task while scoring poorly on these dimensions—but it won't scale or evolve well.

---

## Open Questions

- **Instruction-to-example ratio:** Unknown optimal balance. Warrants empirical research. Current hypothesis: the initial prompt should be sufficient; examples are for format clarification, not capability extension.

- **Diminishing returns threshold:** Highly situational. Signs you've hit it: marginal prompt changes produce marginal improvements, or you're working around model limitations rather than guiding model behavior.

- **Future exploration:** How do retrieval-augmented prompts change these dynamics? Does the rise of longer context windows change the calculus on prompt comprehensiveness? 
