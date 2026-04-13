---
title: Prompt Types
description: A maturity model for understanding prompt complexity and capability
created: 2025-12-08
last_updated: 2025-12-08
tags: [foundations, prompt-engineering, maturity-model]
part: 1
part_title: Foundations
chapter: 2
section: 1
order: 1.2.1
---

# Prompt Types

Prompts aren't monolithic. They exist on a spectrum of complexity, from simple static instructions to self-improving meta-cognitive systems. Understanding where your prompt falls on this spectrum helps you:

- Choose the right level of complexity for your use case
- Identify opportunities for evolution
- Avoid over-engineering simple tasks

This model describes seven levels of prompt maturity, each building on the capabilities of the previous level.

---

## The Seven Levels

| Level | Name | Key Characteristic |
|-------|------|-------------------|
| 1 | Static | Fixed instructions, no variation |
| 2 | Parameterized | Accepts input to customize behavior |
| 3 | Conditional | Branches based on input or state |
| 4 | Contextual | Incorporates external information |
| 5 | Composed | Invokes other prompts/commands |
| 6 | Self-Modifying | Updates itself based on execution |
| 7 | Meta-Cognitive | Improves other prompts in the system |

---

## Level 1: Static

**Definition:** Hardcoded instructions that produce consistent output regardless of context.

**Characteristics:**
- No variables or parameters
- No external dependencies
- Predictable, deterministic behavior
- Simple task automation

**When to Use:**
- Quick reference information
- Boilerplate generation
- Consistent formatting tasks
- Documentation displays

**Example:**
```markdown
# Code Review Checklist

Review the following aspects:
1. Are there any obvious bugs?
2. Is error handling present and appropriate?
3. Are there security vulnerabilities?
4. Is the code readable and well-named?
5. Are there adequate tests?

Provide a summary with pass/fail for each item.
```

**Limitations:**
- Cannot adapt to different contexts
- No personalization possible
- Must create new prompts for variations

---

## Level 2: Parameterized

**Definition:** Prompts that accept input via variables (e.g., `$ARGUMENTS`, `{{variable}}`) to customize behavior.

**Characteristics:**
- Uses placeholder variables
- Output varies based on input
- Input validation may be present
- Single execution path with variable substitution

**When to Use:**
- User-specific customization
- Template-based generation
- Repeated tasks with varying inputs

**Example:**
```markdown
# Generate Commit Message

Based on the following changes:
$ARGUMENTS

Generate a conventional commit message following this format:
- type(scope): description
- Body explaining the "why"
- Footer with any breaking changes or issue references
```

**Key Design Decisions:**
- What inputs are required vs. optional?
- How should missing inputs be handled?
- What validation, if any, should be applied?

---

## Level 3: Conditional

**Definition:** Prompts that branch execution based on input values, system state, or runtime conditions.

**Characteristics:**
- If/else or switch logic in instructions
- Multiple execution paths
- State-dependent behavior
- May check prerequisites before proceeding

**When to Use:**
- Multi-path workflows
- Input routing and classification
- Adaptive responses based on context type

**Example:**
```markdown
# Handle User Request

Analyze the request: $ARGUMENTS

If the request is a BUG REPORT:
  - Extract reproduction steps
  - Identify affected components
  - Suggest severity level
  - Output in bug template format

If the request is a FEATURE REQUEST:
  - Clarify the user problem being solved
  - Identify affected systems
  - Output in feature template format

If the request is UNCLEAR:
  - Ask clarifying questions
  - Do not proceed until classification is clear
```

**Key Design Decisions:**
- What are the branch conditions?
- Is there a default/fallback path?
- How deep should the branching go?

---

## Level 4: Contextual

**Definition:** Prompts that reference external files, documentation, or system state to inform execution.

**Characteristics:**
- Reads from specific files or APIs
- Uses file content to guide behavior
- May aggregate multiple sources
- Context-aware decision making

**When to Use:**
- Knowledge-intensive tasks
- Codebase-aware operations
- Documentation-driven workflows
- Tasks requiring current state

**Example:**
```markdown
# Architecture Review

Before proceeding, read the following files:
- `docs/ARCHITECTURE.md` for system overview
- `src/config/schema.ts` for data structures
- `.env.example` for required environment variables

Using this context, review the proposed changes in $ARGUMENTS:
1. Does this align with existing architectural patterns?
2. Are there existing utilities that should be reused?
3. What documentation needs updating?
```

**Key Design Decisions:**
- Which files are essential vs. optional?
- How to handle missing files?
- How much context is too much? (token limits)
- How to keep file references up-to-date?

---

## Level 5: Composed (Higher-Order)

**Definition:** Prompts that invoke other prompts/commands or accept complex structured context.

**Characteristics:**
- Invokes other commands as subroutines
- Accepts multi-line or structured input
- Coordinates multiple operations
- May spawn parallel work or subagents
- Often includes domain expertise sections

**When to Use:**
- Complex multi-step workflows
- Orchestration of specialized commands
- Expert system patterns
- Pipeline architectures

**Example:**
```markdown
# Implementation Workflow

Context: $ARGUMENTS (path to specification file)

## Phase 1: Planning
Invoke /workflows:plan with the specification

## Phase 2: Implementation
For each item in the plan:
  1. Implement the changes
  2. Invoke /testing:validate on changed files
  3. If validation fails, fix and re-validate

## Phase 3: Review
Invoke /review:code-review on all changes

## Expertise: Implementation Patterns
- Prefer composition over inheritance
- Use dependency injection for testability
- Keep functions under 50 lines
- Extract magic numbers to named constants
```

**Key Design Decisions:**
- What commands can be composed?
- How is state passed between invocations?
- What happens if a sub-command fails?
- How much expertise to embed vs. delegate?

---

## Level 6: Self-Modifying

**Definition:** Prompts that update their own content based on execution results or accumulated learning.

**Characteristics:**
- Modifies its own template file
- Accumulates knowledge over time
- Updates expertise sections
- Reflects on execution outcomes
- Learning persists across sessions

**When to Use:**
- Domain experts that improve with use
- Pattern libraries that grow
- Mistake prevention through learning
- Project-specific adaptation

**Example:**
```markdown
# Testing Expert

## Instructions
1. Analyze recent test failures in git history
2. Extract patterns from successful test fixes
3. Update the Expertise section below with new learnings
4. Apply learnings to current testing task

## Self-Improvement Protocol
Trigger: After completing 5+ testing tasks
Sources: git log of test files, PR comments on test coverage
Extraction: Identify repeated patterns and anti-patterns
Integration: Add to Expertise section with examples
Validation: Verify new rules don't conflict with existing ones

## Expertise (Auto-Updated)
### Async Testing
- Always use `waitFor` instead of arbitrary delays
- Mock external services at the boundary, not internally
- [Added 2024-01-15]: Use `vi.useFakeTimers()` for time-dependent tests

### Component Testing
- Test behavior, not implementation details
- Prefer user-event over fireEvent
- [Added 2024-01-20]: Always test error states, not just happy path
```

**Key Design Decisions:**
- What triggers self-improvement?
- How to validate learned patterns?
- How to prevent drift or conflicting rules?
- How to version the prompt's evolution?

**Git History as Learning Signal:**

*[2025-12-08]*: Git diffs are the primary learning signal for self-improving prompts. The pattern:

1. Run `git diff` (uncommitted), `git diff --cached` (staged), and `git log` (recent commits)
2. Analyze changes to domain-relevant files for new patterns, anti-patterns, or refined techniques
3. Extract learnings that indicate improved understanding (e.g., "new hook pattern discovered", "better error handling approach")
4. Update the Expertise section with dated references to the commit or change that revealed the insight
5. Keep Workflow sections stable—only update Expertise

This creates a feedback loop where prompts learn from the actual evolution of the codebase. The git history serves as ground truth: "This commit changed how we handle X, therefore our expertise should reflect this new understanding." See `appendices/examples/kotadb/.claude/commands/experts/*/` for implementation patterns.

**Conservative Update Rules:**

*[2025-12-08]*: To prevent knowledge loss during self-modification, enforce conservative update discipline using explicit operations:

- **PRESERVE**: Keep existing patterns unless confirmed obsolete through evidence
- **APPEND**: Add new learnings with references to commit history or issue numbers
- **DATE**: Timestamp new entries with context (e.g., "Added after #123")
- **REMOVE**: Only delete patterns contradicted by multiple recent implementations

This prevents premature deletion of valuable patterns. Require evidence from multiple implementations before removing any existing knowledge. Pattern removal should be rare—most evolution happens through APPEND operations that layer new understanding on top of existing patterns. See `examples/kotadb/.claude/commands/experts/*_improve.md` for implementations.

---

## Level 7: Meta-Cognitive

**Definition:** Prompts that analyze, evaluate, and improve other prompts in the system.

**Characteristics:**
- Operates on other prompt templates
- Evaluates prompt effectiveness
- Proposes or applies improvements to other files
- System-wide optimization capability
- May orchestrate multiple Level 6 improvements

**When to Use:**
- Prompt library maintenance
- Quality assurance for prompt systems
- Automated prompt optimization
- System-wide consistency enforcement

**Example:**
```markdown
# Prompt Quality Analyzer

## Instructions
1. Scan all prompts in `.claude/commands/`
2. For each prompt, evaluate against quality criteria:
   - Clarity: Are instructions unambiguous?
   - Structure: Is formatting consistent?
   - Completeness: Are edge cases handled?
   - Level Appropriateness: Is complexity justified?
3. Generate improvement recommendations
4. For approved improvements, apply changes and verify system integrity

## Quality Criteria
- Level 1-2 prompts should not exceed 50 lines
- Level 3+ prompts must have explicit error handling
- Level 5+ prompts must document sub-command dependencies
- All prompts must declare their level in frontmatter

## Improvement Protocol
1. Identify candidate prompts below quality threshold
2. Generate specific, actionable improvements
3. Present recommendations with before/after examples
4. Apply changes only after human approval
5. Run integration tests after changes
```

**Key Design Decisions:**
- What authority does the meta-prompt have?
- How to prevent runaway self-modification?
- What approval gates exist?
- How to ensure system stability?

**Implementation Pattern: Bulk-Update Coordination**

*[2025-12-08]*: Level 7 meta-cognitive prompts can coordinate multiple Level 6 self-improving prompts using parallel delegation. The bulk-update pattern spawns improve commands in parallel using lightweight models (e.g., Haiku), where each sub-agent invokes a single `*_improve.md` command in its own isolated context window. This achieves:

- **True Parallelism**: Each expert runs simultaneously in separate contexts
- **Failure Isolation**: One failed improvement doesn't affect others
- **Context Isolation**: No cross-pollution between expert analyses
- **Cost Efficiency**: Uses fast, inexpensive models for simple delegation
- **Scalability**: Supports system-wide optimization without context overflow

Example: A bulk-update orchestrator spawns 8 sub-agents in a single message using the Task tool, each running a domain expert's improvement command. Results are aggregated into a consolidated report showing status, updates made, and files modified across all experts.

---

## Level Requirements Matrix

| Level | Variables | External Context | Invokes Others | Self-Updates | Updates Others |
|-------|-----------|-----------------|----------------|--------------|----------------|
| 1 | No | No | No | No | No |
| 2 | Yes | No | No | No | No |
| 3 | Optional | No | No | No | No |
| 4 | Optional | Yes | No | No | No |
| 5 | Yes | Optional | Yes | No | No |
| 6 | Optional | Optional | Optional | Yes | No |
| 7 | Optional | Yes | Yes | Yes | Yes |

---

## Evolution Paths

Prompts naturally evolve as requirements grow. Common progressions:

```
Level 1 → Level 2
  "I need this to work with different inputs"
  Add: $ARGUMENTS or template variables

Level 2 → Level 3
  "I need different behavior for different input types"
  Add: Conditional logic, input classification

Level 3 → Level 4
  "I need this to understand my codebase"
  Add: File references, context injection

Level 4 → Level 5
  "I need this to coordinate with other commands"
  Add: Sub-command invocation, expertise sections

Level 5 → Level 6
  "I want this to get better over time"
  Add: Self-improvement protocol, learning triggers

Level 6 → Level 7
  "I want to improve the whole system"
  Add: Cross-prompt analysis, system-wide optimization
```

---

## Best Practices

### Start Low, Evolve Up
- Begin at the lowest level that meets requirements
- Over-engineering wastes effort and adds failure modes
- Let actual usage drive evolution

### Document the Level
- Declare the prompt level explicitly
- Makes maintenance and evolution intentional
- Helps others understand complexity expectations

### Separate Concerns at Higher Levels
- Level 5+: Keep orchestration logic separate from domain expertise
- Level 6+: Isolate self-modification rules from core functionality
- Level 7: Require approval gates for cross-prompt changes

### Test at Each Level
- Level 1-2: Spot-check output
- Level 3-4: Test each branch/context path
- Level 5+: Integration tests for composition
- Level 6-7: Regression tests for learned behavior

---

## Connections

- **[Prompt](_index.md):** The foundational concepts that apply across all levels
- **[Prompt Language](3-language.md):** Language choice affects effectiveness at every level—higher levels require more precise linguistic patterns
- **[Patterns](../7-patterns/_index.md):** Higher-level prompts often implement agentic patterns
- **Multi-Agent:** Level 5+ prompts enable multi-agent coordination

---

## Sources

This maturity model is adapted from the 7-Level Prompt Maturity Model developed at [agenticengineer.com](https://agenticengineer.com).

---

## Questions to Explore

- What's the highest level that's practically maintainable?
- How do you debug a self-modifying prompt that's learned bad patterns?
- At what point does prompt complexity suggest you need code instead?
- How do you version control prompts that modify themselves?
