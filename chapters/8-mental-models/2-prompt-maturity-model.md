---
title: Prompt Maturity Model
description: Seven levels of prompt sophistication from static to meta-cognitive
created: 2025-12-08
last_updated: 2026-01-21
tags: [prompts, mental-models, maturity]
part: 3
part_title: Perspectives
chapter: 8
section: 2
order: 3.8.2
---

# Prompt Maturity Model

A framework for understanding and designing prompts at different levels of sophistication. Each level builds on the previous, adding new capabilities and complexity.

## The Seven Levels

### Level 1: Static

**What defines it**: Hardcoded instructions with no variation. The prompt is the same every time it runs.

**When to use it**: For simple, repeatable tasks that never need customization. Quick utilities where the overhead of parameters isn't worth it.

**Example**: A command that always formats code the same way, or always runs the same test suite.

```markdown
# format-code.md
Run prettier on all TypeScript files in src/
```

**Trade-offs**:
- Pros: Simplest to write and understand. No state to manage.
- Cons: Inflexible. Requires creating new commands for variations.

---

### Level 2: Parameterized

**What defines it**: Uses `$ARGUMENTS` or other variables to accept input at runtime.

**When to use it**: When you need the same logic with different inputs. The behavior is consistent but the data varies.

**Example**: From the knowledge base examples, commands that take a file path or topic as input.

```markdown
# review-file.md
Review the file at $ARGUMENTS for clarity and completeness.
```

**Trade-offs**:
- Pros: Reusable across different inputs. Still straightforward logic.
- Cons: Limited to simple substitution. No branching behavior.

---

### Level 3: Conditional

**What defines it**: Contains if/else logic or branching based on input characteristics.

**When to use it**: When the same command needs to behave differently depending on what it receives.

**Example**: A command that processes markdown differently than code files, or handles different file types.

```markdown
# analyze.md
If $ARGUMENTS contains .ts or .js:
  - Check for type safety issues
Else if $ARGUMENTS contains .md:
  - Check for broken links
Else:
  - Provide general analysis
```

**Trade-offs**:
- Pros: One command handles multiple scenarios. More intelligent.
- Cons: Logic can become complex. Harder to predict behavior.

---

### Level 4: Contextual

**What defines it**: Reads external files or project state before acting. Uses context to inform decisions.

**When to use it**: When the prompt needs to understand the broader environment. Common with project-aware commands.

**Example**: The `/tools:prime` command that gathers project context, or commands that read CLAUDE.md before working.

```markdown
# contextualized-review.md
1. Read CLAUDE.md to understand project conventions
2. Read the file at $ARGUMENTS
3. Review against project standards
4. Suggest improvements that fit the codebase
```

**Trade-offs**:
- Pros: Decisions informed by actual project state. More intelligent.
- Cons: Slower due to file reads. Can fail if context is missing.

---

### Level 5: Higher Order

**What defines it**: Invokes other commands as subroutines. Orchestrates multiple operations.

**When to use it**: For workflows that combine several distinct steps, or meta-commands that coordinate other commands.

**Example**: This book's `/do` command orchestration (`.claude/commands/do.md`) demonstrates higher-order coordination by classifying user requirements and routing to appropriate expert domains. Workflow commands that call research, implement, and review commands in sequence also fit this level.

```markdown
# feature-workflow.md
1. Call /research:gather-requirements $ARGUMENTS
2. Call /plan:design-architecture
3. Call /implement:build
4. Call /review:validate
```

**Trade-offs**:
- Pros: Compose complex behaviors from simpler parts. DRY.
- Cons: Dependencies between commands. Harder to debug failures.

---

### Level 6: Self-Modifying

**What defines it**: Updates its own template based on outcomes or feedback.

**When to use it**: When a command should learn from its usage patterns and improve itself over time.

**Example**: The `*_improve.md` pattern seen in some agentic systems where commands track their failures and update their instructions.

```markdown
# adaptive-analyzer.md
[CURRENT TEMPLATE]
Analyze code for: ${FOCUS_AREAS}

[IMPROVEMENT MECHANISM]
After each run:
- If analysis missed issues: add to FOCUS_AREAS
- If too verbose: add to SUPPRESS_PATTERNS
- Update this template
```

**Trade-offs**:
- Pros: Commands get better with use. Adapt to project needs.
- Cons: Non-deterministic. Can drift from original intent. Needs safeguards.

---

### Level 7: Meta-Cognitive

**What defines it**: Improves other commands, not just itself. Operates on the command system as a whole.

**When to use it**: For maintenance and evolution of the command ecosystem. Quality assurance for prompts.

**Example**: A bulk-update orchestrator that analyzes all commands and suggests improvements, or a command that identifies redundant commands and proposes consolidation.

```markdown
# command-optimizer.md
1. Scan all .md commands in .claude/
2. Identify patterns:
   - Duplicated logic
   - Commands that could be parameterized
   - Missing error handling
3. Generate improvement proposals
4. Execute approved updates
```

**Trade-offs**:
- Pros: System-wide optimization. Maintains command quality.
- Cons: Highest complexity. Requires understanding of entire system. Risk of breaking changes.

---

## Choosing the Right Level

**Start at the lowest level that solves the problem.**

- **Level 1-2**: Most commands should live here. Simple is better.
- **Level 3**: Use sparingly. Often a sign you need multiple commands instead.
- **Level 4**: Standard for project-aware tools. Worth the context-gathering cost.
- **Level 5**: Good for defined workflows. Keep orchestration logic simple.
- **Level 6**: Experimental. Needs monitoring and rollback capability.
- **Level 7**: Rare. Usually for tooling teams or advanced automation.

**Signals you need a higher level:**
- Creating many similar commands → Move from 1 to 2
- Copy-pasting logic between commands → Move to 5
- Manually tweaking commands after each use → Consider 6
- Spending more time maintaining commands than using them → Consider 7

**Signals you're at too high a level:**
- Can't predict what the command will do
- Debugging takes longer than the command saves
- Other developers avoid using it
- You're the only one who understands it

**The maturity sweet spot**: Most systems should have a pyramid distribution:
- Many Level 1-2 commands (foundation)
- Some Level 3-4 commands (core workflows)
- Few Level 5 commands (orchestration)
- Rare Level 6-7 commands (if any)

---

## Connections

- **To [Prompt Structuring](../2-prompt/2-structuring.md):** Structural choices (output templates, failure sentinels, state machines) enable prompts to move up maturity levels—the techniques that make higher levels possible
- **To [Self-Improving Experts](../6-patterns/2-self-improving-experts.md):** Self-modifying (Level 6) and meta-cognitive (Level 7) prompts parallel expert system evolution. The three-command expert pattern (Plan-Build-Improve) implements Level 6 maturity.
- **To [Knowledge Evolution](../7-practices/6-knowledge-evolution.md):** Tracking prompt maturity progression mirrors knowledge base maturity—both evolve from simple to sophisticated through observation and refinement
