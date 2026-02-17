---
title: Self-Improving Expert Commands
description: The Plan-Build-Improve pattern for creating commands that learn from experience
created: 2025-12-08
last_updated: 2026-02-05
tags: [patterns, meta-prompting, self-improvement]
part: 2
part_title: Craft
chapter: 6
section: 2
order: 2.6.2
---

# Self-Improving Expert Commands

A three-command expert pattern that creates a learning triangle where production experience feeds back into prompts, enabling commands to improve themselves over time.

---

## Overview

Self-Improving Expert Commands implement a specialized version of the Plan-Build-Review pattern where the "review" phase actively updates the expertise embedded in the planning and building commands. This creates a system where each cycle of usage makes the next iteration smarter.

The pattern separates two types of prompt content:
- **Expertise sections**: Mutable domain knowledge that evolves with experience
- **Workflow sections**: Stable process descriptions that define how the expert operates

Only Expertise sections get updated, ensuring the process remains consistent while knowledge accumulates.

---

## The Pattern

The pattern consists of three commands that form a learning triangle:

### Plan Command (`*_plan.md`)
Contains domain-specific knowledge and creates detailed specifications.

Structure:
- `## Expertise` - Mutable knowledge base (patterns, examples, anti-patterns)
- `## Workflow` - Stable planning process
- `## Report` - Output format template

Purpose: Analyzes requirements and produces a specification for implementation.

### Build Command (`*_build.md`)
Implements based on specifications, applying accumulated expertise.

Structure:
- `## Expertise` - Mutable implementation patterns
- `## Workflow` - Stable build process
- `## Report` - Output format template

Purpose: Executes the plan using learned best practices.

### Improve Command (`*_improve.md`)
Analyzes git history and updates expertise in Plan/Build commands.

Structure:
- `## Workflow` - Process for extracting learnings
- No Expertise section (this command updates others)

Purpose: Mines production experience and updates Plan/Build expertise.

---

## Key Design Principles

### 1. Separate Expertise from Workflow
Only Expertise sections are designed to be modified. Workflow sections remain stable, providing consistent process execution while knowledge evolves.

### 2. Conservative Update Rules
The Improve command follows strict guidelines for updating expertise:

- **PRESERVE**: Keep patterns that are still relevant and working
- **APPEND**: Add new learnings with timestamps for provenance
- **DATE**: Prefix all new entries with `*[YYYY-MM-DD]*:` for tracking
- **REMOVE**: Delete only patterns proven ineffective with clear evidence

### 3. Git History as Learning Signal
Production experience is captured in git commits. The Improve command analyzes:
- Recent commits in the domain area
- Successful patterns that emerged
- Issues that were fixed
- Anti-patterns that caused problems

### 4. Timestamped Entries for Provenance
All expertise updates include timestamps showing when the learning was captured:

```markdown
### API Contract Patterns

**Zod-Based Validation (from #485):**
*[2025-11-15]*: All API routes should validate requests with Zod schemas...
```

This enables tracking the evolution of knowledge and pruning outdated patterns.

---

## When to Use

### Good Fit
- **Domains with recurring decisions**: Areas where similar choices come up repeatedly
- **Emerging patterns**: Situations where best practices crystallize over time
- **Codebase-specific knowledge**: Learnings that are unique to this project
- **Complex integration points**: External services with many edge cases to learn

### Poor Fit
- **One-off tasks**: No opportunity for accumulated learning
- **Fixed requirements**: Domain knowledge doesn't evolve
- **Trivial operations**: Overhead exceeds benefit
- **Highly generic domains**: Better served by general documentation

---

## Implementation

### Directory Structure

```
.claude/commands/experts/
└── [domain]-expert/
    ├── [domain]_expert_plan.md
    ├── [domain]_expert_build.md
    └── [domain]_expert_improve.md
```

### Plan Command Template

```markdown
---
description: Analyze [domain] requirements and create specification
argument-hint: <requirement-context>
---

# [Domain] Expert - Plan

You are a [Domain] Expert specializing in analysis and planning.

## Variables

USER_PROMPT: $ARGUMENTS

## Instructions

- Analyze requirements from [domain] perspective
- Apply accumulated expertise patterns
- Produce detailed specification
- Consider past learnings documented below

## Expertise

### [Category 1]

**Pattern Name (from #issue-number):**
*[2025-12-08]*: Description of pattern with examples...

### [Category 2]

**Anti-Pattern to Avoid:**
*[2025-11-20]*: Description of what not to do and why...

## Workflow

1. **Understand Context**
   - Parse USER_PROMPT
   - Identify scope

2. **Apply Expertise**
   - Check relevant patterns
   - Consider anti-patterns

3. **Formulate Recommendations**
   - List requirements
   - Provide guidance

## Report

[Template for plan output]
```

### Build Command Template

```markdown
---
description: Implement [domain] solution from specification
argument-hint: <path-to-spec>
---

# [Domain] Expert - Build

You are a [Domain] Expert specializing in implementation.

## Variables

PATH_TO_SPEC: $ARGUMENTS

## Instructions

- Load specification from PATH_TO_SPEC
- Apply implementation expertise
- Follow codebase standards
- Test thoroughly

## Expertise

### Implementation Patterns

**Standard Approach:**
*[2025-12-01]*: Code examples and guidance...

### Code Standards

**Specific Requirements:**
- Pattern to follow
- Tools to use

## Workflow

1. **Load Specification**
   - Read spec file
   - Extract requirements

2. **Implement Solution**
   - Apply patterns
   - Follow standards

3. **Verify**
   - Run tests
   - Check standards

## Report

[Template for build output]
```

### Improve Command Template

```markdown
---
description: Analyze recent changes and update expert knowledge
---

# [Domain] Expert - Improve

You analyze recent [domain] changes and update expert knowledge.

## Workflow

1. **Analyze Recent Changes**
   ```bash
   git log --oneline -20 -- "[relevant-paths]"
   git diff main -- "[relevant-files]"
   ```

2. **Extract Learnings**
   - Identify successful patterns
   - Note approaches that worked
   - Document fixes and improvements

3. **Identify Anti-Patterns**
   - Review issues fixed
   - Note failure modes
   - Capture prevention strategies

4. **Update Expertise**
   - Edit `[domain]_expert_plan.md` Expertise sections
   - Edit `[domain]_expert_build.md` Expertise sections
   - Add timestamps to all new entries
   - Remove only with clear evidence

## Report

**Improvement Report**

**Changes Analyzed:** [summary]

**Learnings Extracted:**
- Pattern: why it worked
- Pattern: benefit observed

**Anti-Patterns Identified:**
- What to avoid: why

**Expertise Updates Made:**
- File: changes made
- Sections updated
```

---

## Examples

The pattern is implemented in several projects in this knowledge base:

### KotaDB Experts
`appendices/examples/kotadb/.claude/commands/experts/`

Domain-specific experts for the KotaDB project, each accumulating knowledge about their specialized area. These demonstrate production-ready patterns including:
- Webhook idempotency and external service integration
- Validation patterns for APIs
- Environment-specific configuration resolution
- Anti-patterns discovered through debugging

### Questions Workflow Expert
`.claude/agents/experts/questions/`

Specializes in iterative content development through question-driven exploration. The expertise in the four agents (ask, build, deepen, format) has evolved to include:
- Question selection patterns that elicit substantive responses
- Voice preservation techniques during content synthesis
- Follow-up question templates that reveal genuine depth
- Format cleanup strategies for real-world edge cases
- Anti-patterns discovered through workflow execution

Note: This expert has a non-standard structure (4 agents instead of 3) but follows the self-improvement pattern through `questions-improve-agent.md`.

---

## Anti-Patterns

### Updating Workflow Sections
**Problem**: Modifying the stable process sections makes the expert unpredictable.

**Why it's bad**: The workflow is the expert's consistent methodology. Changing it means each execution could behave differently, making the system unreliable.

**Solution**: Only update Expertise sections. If the workflow itself needs to change, that's a major revision requiring deliberate design.

### Removing Patterns Without Evidence
**Problem**: Deleting expertise based on hunches rather than actual failures.

**Why it's bad**: You might be removing knowledge that's still valuable in edge cases or specific contexts.

**Solution**: Only remove patterns when you have clear evidence they cause problems. Document why in the improve commit.

### Not Dating New Expertise Entries
**Problem**: Adding new patterns without timestamp prefixes.

**Why it's bad**: Can't track when knowledge was added, making it hard to prune outdated patterns or understand evolution.

**Solution**: Always prefix new expertise entries with `*[YYYY-MM-DD]*:` to establish provenance.

### Letting Expertise Sections Grow Unbounded
**Problem**: Continuously adding patterns without consolidation or organization.

**Why it's bad**: Expertise becomes harder to navigate and apply. Context windows fill up with outdated or redundant information.

**Solution**: Periodically consolidate related patterns, remove duplicates, and organize into clear categories. Consider splitting overgrown experts into specialized sub-experts.

### Improving Without Production Evidence
**Problem**: Running the Improve command without actual production usage to analyze.

**Why it's bad**: You end up adding theoretical patterns that haven't been validated in real usage.

**Solution**: Only run Improve after meaningful production usage. Look for concrete evidence in git history, issues, and actual code changes.

### Mixing Multiple Domains in One Expert
**Problem**: Creating a single expert that tries to cover too many different concerns.

**Why it's bad**: Expertise becomes unfocused, and different domains may evolve at different rates or in conflicting directions.

**Solution**: Keep experts focused on clear domain boundaries. Split into multiple experts if different areas of knowledge emerge.

---

## Questions to Explore

### How do you decide what's worth capturing vs. discarding?
When analyzing production experience, which patterns represent genuine learnings versus one-off solutions? What criteria distinguish reusable knowledge from context-specific fixes?

### What's the right consolidation strategy?
As expertise accumulates, how do you balance comprehensiveness with clarity? When should you consolidate multiple specific patterns into a general principle, and when should you keep them separate?

### How does this interact with external documentation?
When should expertise reference external docs versus inline them? How do you keep expertise fresh when the external tools/APIs evolve?

### Can Improve be automated?
Could you run the Improve command automatically after certain triggers (e.g., merged PRs, fixed issues)? What would be the tradeoffs of automation versus deliberate human-guided improvement?

---

## Operational Insights

*[2025-12-09]*: After extended use of self-improving expert systems, several insights emerged about why this pattern works reliably in practice:

### Thin Orchestrator, Fat Specialists

The orchestrator's job is **coordination, not execution**. It parses arguments, spawns the right agent with the right context, handles user review checkpoints, and reports results. The heavy lifting—understanding patterns, writing specs, implementing solutions—happens in the specialized agents. This keeps the orchestrator reliable and predictable.

### Spec File as Multi-Purpose Artifact

The intermediate artifact (spec file) serves multiple roles:
- **Contract** between plan and build phases
- **Checkpoint** for user review before potentially destructive changes
- **Context carrier** without bloating agent prompts
- **Resumable state** for interrupted workflows

### Maintenance Burden Reduction

The self-improvement value isn't dramatic compound effects—it's **maintenance burden reduction**. You don't have to:
- Remember what patterns you've used
- Manually update agent knowledge
- Re-explain context from prior implementations

The improve agent handles institutional memory. Even if individual updates are small, the cumulative effect is that expertise doesn't rot.

### Sequential When Dependencies Exist

The strict plan → build → improve sequence works because there's a genuine dependency chain:
- Build **needs** the spec file from plan
- Improve **needs** the completed implementation to analyze

Don't force parallelism where it doesn't fit. Iterative workflows (like question-driven content development) may loop, but expert workflows are pipelines.

### Reliability Through Delegation

Reliability comes from correct delegation. Each agent has a focused job with restricted tools. When the orchestrator tries to do too much itself, failures creep in.

---

## Three-Role Architecture for Self-Improvement

*[2025-12-10]*: The conceptual "self-improving expert" pattern has a formalized execution-level architecture that separates learning into three distinct roles. This addresses a critical failure mode: when evaluation and rewriting are combined in a single agent, you get **premature optimization** and **compression bias**—the model prematurely discards exploration paths or compresses nuanced lessons into oversimplified patterns.

### The Three Roles

The architecture separates concerns across three specialized components:

#### 1. Generator: Exploration Without Judgment

**Purpose**: Produce reasoning trajectories and explore solution space.

**Key Constraint**: **No evaluation or judgment**. The Generator simply executes tasks and produces outputs, including intermediate reasoning, successful attempts, and failures.

**Why separation matters**: When generation and evaluation happen together, the model self-censors. Potentially valuable failed approaches never get recorded because they're discarded before reaching the learning pipeline.

**Output**: Raw execution traces—what was tried, what happened, reasoning steps taken.

#### 2. Reflector: Iterative Insight Extraction

**Purpose**: Extract concrete, actionable insights from successes and failures through iterative refinement.

**Key Constraint**: **Multiple refinement rounds** (typically up to 5 iterations). Each iteration deepens the analysis, moving from surface observations to underlying principles.

**Why iteration matters**: First-pass analysis is typically shallow. Ablation studies show measurable improvement (+1.7% in ACE framework benchmarks) from multi-iteration refinement versus single-pass reflection.

**Output**: Structured lessons learned—what patterns emerged, why they worked/failed, under what conditions.

**Implementation Pattern**:
```python
def reflect(execution_trace, max_iterations=5):
    """Extract insights through iterative refinement."""
    insights = initial_analysis(execution_trace)

    for iteration in range(max_iterations):
        # Each iteration deepens understanding
        insights = refine_insights(
            previous_insights=insights,
            execution_trace=execution_trace,
            iteration=iteration
        )

        # Stop if insights have converged
        if has_converged(insights):
            break

    return insights
```

#### 3. Curator: Deterministic Knowledge Integration

**Purpose**: Convert lessons into structured delta entries and merge them into the knowledge base.

**Key Innovation**: **Uses deterministic logic, not LLM inference**. This prevents compression bias—the tendency of language models to oversimplify or merge distinct patterns into vague generalizations.

**Why deterministic curation matters**: LLMs naturally compress information. When you ask an LLM to merge new insights with existing expertise, it often:
- Discards specificity for brevity
- Merges related-but-distinct patterns
- Loses edge case details
- Smooths over conflicts rather than preserving them as decision points

Deterministic merging preserves granularity through explicit rules:
- Append new entries with timestamps
- Preserve existing entries unless explicitly marked for removal
- Use structural markers (headings, lists) to maintain organization
- Apply conflict resolution rules without interpretation

**Implementation Pattern**:
```python
def curate_knowledge(insights, existing_expertise):
    """Deterministic knowledge base update."""
    delta = create_delta_entry(
        timestamp=datetime.now(),
        insights=insights,
        source_ref=execution_id
    )

    # Deterministic merging: no LLM interpretation
    updated_expertise = merge_deterministic(
        existing=existing_expertise,
        delta=delta,
        rules={
            'append_new': True,
            'preserve_timestamps': True,
            'deduplicate_by': 'semantic_hash',
            'conflict_resolution': 'keep_both_with_marker'
        }
    )

    return updated_expertise
```

### Why This Architecture Works

**Separation prevents premature compression**: By splitting generation, reflection, and curation, each phase can optimize for its specific goal without conflicting constraints.

**Multi-iteration refinement adds measurable value**: The reflection phase's iterative deepening produces demonstrably better insights than single-pass analysis. The +1.7% improvement may seem modest, but in compound learning systems, consistent small gains accumulate.

**Deterministic curation preserves detail**: Using programmatic logic instead of LLM-based merging prevents the gradual erosion of specificity that happens when models "smooth over" conflicts or "tidy up" edge cases.

### Mapping to Plan-Build-Improve

The three-role architecture operationalizes the conceptual pattern described earlier in this entry:

| Conceptual Role | Execution Role | Primary Activity |
|-----------------|----------------|------------------|
| **Build** | Generator | Execute tasks, produce traces |
| **Improve (Phase 1)** | Reflector | Extract insights from traces |
| **Improve (Phase 2)** | Curator | Merge insights into expertise |

The key insight: what appears as a single "Improve" command at the user interface level actually decomposes into two distinct execution roles (Reflector + Curator) with different mechanisms—one LLM-based, one deterministic.

### Implementation Considerations

**When to apply this architecture**:
- Long-lived expert systems accumulating knowledge over months/years
- Domains where subtle distinctions matter (compression would lose critical nuance)
- Systems where you can instrument execution to produce detailed traces
- Contexts where you can afford the computational cost of multi-iteration reflection

**When simpler approaches suffice**:
- Short-term projects where knowledge doesn't need to accumulate
- Domains with well-established patterns (no new insights emerging)
- Resource-constrained environments where iteration cost exceeds benefit
- Situations where human curation can happen frequently enough to prevent drift

**Sources**:
- ACE (Agentic Cognitive Engine) framework ablation studies
- Production experience with multi-expert self-improving systems

---

## Expertise-as-Mental-Model Variant

*[2025-12-25]*: Experience with large-scale agent systems revealed a variant of this pattern that treats expertise as a **queryable mental model** rather than embedded prompt sections. This book's own `.claude/agents/experts/` implementation demonstrates this approach. Key differences:

### Structure

Instead of expertise embedded in command files, a separate `expertise.yaml` (500-700 lines) contains all domain knowledge. This book uses this structure for all 11 expert domains:

```
.claude/agents/experts/<domain>/
├── expertise.yaml              # Complete knowledge base in YAML
├── <domain>-question-agent.md  # Read-only Q&A interface
├── <domain>-plan-agent.md      # Specification creation
├── <domain>-build-agent.md     # Implementation from specs
└── <domain>-improve-agent.md   # Expertise validation and updates
```

### Expertise YAML Format

Structured YAML with consistent sections:
- `overview` - Domain scope and rationale
- `core_implementation` - Key files and conventions
- `key_operations` - Domain-specific operations documented
- `decision_trees` - Choice frameworks with observed usage
- `patterns` - High-level workflows with trade-offs
- `safety_protocols` - Immutable safety rules
- `best_practices` - Evolving guidance (mutable)
- `known_issues` - Current limitations (mutable)
- `potential_enhancements` - Roadmap items (mutable)

### Question Agent Pattern

A read-only interface for querying expertise without code changes:
1. Read expertise.yaml
2. Locate relevant sections based on question
3. Answer with evidence from expertise + code references
4. Provide context on pattern rationale and pitfalls

Tool restrictions: `Read, Glob, Grep` only (no write access)

Model: `haiku` for fast, cost-effective queries

### Self-Improvement with Constraints

The improve-agent enforces constraints that prevent expertise bloat:
- **PRESERVE/APPEND/DATE/REMOVE rules** for controlled updates
- **Git history analysis** to focus on recent changes
- **Timestamp tracking** for all new entries
- **Mutable vs stable section** distinction

### Mental Model Philosophy

The key insight: treat expertise files as **updatable mental models**, not static documentation. A common pattern:

> "Think of the expertise file as your mental model and memory reference for all [domain]-related functionality"

This reframes the improve workflow from "updating documentation" to "validating and correcting understanding."

### When to Use This Variant

**Prefer expertise.yaml when:**
- Expertise exceeds 100 lines (too large to embed in prompts)
- Multiple agents need the same expertise (question, plan, build)
- Queryable interfaces are valuable (question agent)
- Domain has many operations worth documenting (15+ operations)

**Stick with embedded expertise when:**
- Expertise is compact (<100 lines)
- Single command uses the expertise
- Overhead of separate file not justified

**Example**: This book's github expert domain (`.claude/agents/experts/github/`) demonstrates the full pattern with 550-line expertise.yaml covering commit conventions, branch workflows, PR structure, and safety protocols.

---

## Expert Domains: From Pattern to System

*[2025-12-26]*: The self-improving expert pattern evolved from individual command sets into a system-wide architecture spanning 11 domains with standardized four-agent teams. This transformation emerged through five key commits:

**Evolution Timeline:**

| Commit | Date | Change | Impact |
|--------|------|--------|--------|
| c67ed43 | Dec 26 | plan_build_improve lifecycle introduced | Established 3-command pattern |
| 39e7904 | Dec 26 | Standardization to 4-agent pattern | Added question agent (haiku, read-only) |
| 353d576 | Dec 26 | **CRITICAL**: Coordinators→skills, flat /do routing | Eliminated orchestration layer bloat |
| 35a871a | Dec 26 | GitHub agent absorption | Demonstrated domain absorption pattern |
| 5002a1c | Dec 26 | Bulk expertise update across 11 domains | System-wide knowledge standardization |
| 89fbbf9 | Dec 26 | Color coding standardization | Visual agent role identification |

The shift from individual self-improving experts to a coordinated expert domain system creates emergent benefits that exceed the sum of individual agents:

**System-Level Benefits:**

| Benefit | Individual Experts | Expert Domain System |
|---------|-------------------|----------------------|
| **Knowledge Consistency** | Each expert maintains isolated knowledge | Shared expertise.yaml format enables cross-domain patterns |
| **Tool Boundaries** | Ad-hoc tool selection per agent | Strict role-based tools (plan: no Write, question: read-only) |
| **Routing Clarity** | Manual dispatcher logic | Pattern-based /do classification (A-E patterns) |
| **Learning Surface** | Expertise isolated to command prompts | expertise.yaml provides queryable knowledge base |
| **Absorption Path** | No clear upgrade path for standalone agents | 8-step absorption pattern for integration |
| **Question Handling** | Mixed with implementation concerns | Dedicated question-agent (haiku model, safe exploration) |

Commit 353d576 represents the critical architectural pivot: **eliminating coordinator agents in favor of direct skill invocation**. The previous architecture had coordinator layers between user commands and expert agents, creating context overhead and unclear responsibility boundaries. The flat /do routing sends user requirements directly to domain experts based on pattern classification (A-E), with skills providing cross-domain capabilities like research and TOC generation.

**Quantified Scale:**
- **11 domains** × 4 agents = 44 specialized agents
- **11 expertise.yaml files** (500-600 lines each) = ~6,000 lines of structured knowledge
- **Zero coordinator bloat** after 353d576 refactoring
- **Single entry point** (/do) routes to all domains

This architecture separates **cross-cutting concerns** (skills) from **domain expertise** (expert teams), creating a system where knowledge accumulates in structured, queryable formats while maintaining clean tool boundaries and execution flows.

---

## The 4-Agent Pattern Template

*[2025-12-26]*: Each expert domain implements a standardized four-agent team with distinct roles, tools, and color coding. The pattern emerged from standardization commit 39e7904 and now spans all 11 domains.

### Plan Agent (Yellow)

**Purpose**: Requirements analysis and specification creation.

**Tools**: `Read, Glob, Grep, Write`

**Model**: `sonnet`

**Constraints**:
- NO execution (builds specifications, doesn't implement)
- Write access limited to spec file creation
- No Bash (analysis only, not operational)

**Frontmatter Example** (from `.claude/agents/experts/github/github-plan-agent.md`):
```yaml
---
name: github-plan-agent
description: Plans GitHub operations (commits, branches, PRs, issues, releases). Expects: USER_PROMPT (operation requirement), HUMAN_IN_LOOP (optional, default: false)
tools: Read, Glob, Grep, Write
model: sonnet
color: yellow
---
```

**Workflow Structure**:
1. **Understand Requirement** - Parse USER_PROMPT for domain-specific context
2. **Check State Needs** - Identify what repository/codebase state info is needed
3. **Plan Command Sequence** - Break down operation into safe, ordered steps
4. **Identify Safety Checks** - Pre-operation validations before destructive actions
5. **Determine Approval Gates** - If HUMAN_IN_LOOP, mark decision points
6. **Save Specification** - Write detailed spec to `.claude/.cache/specs/<domain>/`

**Output**: Specification file with command sequences, safety checks, and approval gates.

**Why Yellow**: Planning phase—caution before execution.

### Build Agent (Green)

**Purpose**: Implementation from specification.

**Tools**: Varies by domain type—
- **Knowledge domains**: `Read, Write, Edit, Glob, Grep` (file operations)
- **Operational domains**: `Read, Write, Edit, Glob, Grep, Bash` (execution)

**Model**: `sonnet`

**Constraints**:
- NO planning (follows specification exactly)
- MUST read spec file (PATH_TO_SPEC or SPEC variable)
- Updates frontmatter timestamps (last_updated)
- Validates implementation against spec requirements

**Frontmatter Example** (from `.claude/agents/experts/knowledge/knowledge-build-agent.md`):
```yaml
---
name: knowledge-build-agent
description: Builds book content from specs. Expects: SPEC (path to spec file), USER_PROMPT (optional context)
tools: Read, Write, Edit, Glob, Grep, WebSearch
model: sonnet
color: green
---
```

**Workflow Structure**:
1. **Load Specification** - Read spec file from PATH_TO_SPEC
2. **Review Target Context** - Read existing files, check patterns, verify no conflicts
3. **Implement Changes** - Apply spec (create new, extend existing, update indexes)
4. **Implement Cross-References** - Bidirectional links with contextual descriptions
5. **Update Index Files** - Maintain _index.md tables, alphabetical order
6. **Apply Voice Standards** - Third-person, evidence-grounded, direct statements
7. **Verify Implementation** - Check frontmatter, formatting, links, voice consistency

**Output**: Implemented changes with verification report.

**Why Green**: Execution phase—proceed with implementation.

**Domain-Specific Tool Variation**:
- **github-build-agent**: Adds `Bash` for git/gh CLI operations
- **knowledge-build-agent**: Adds `WebSearch` for research during writing
- **orchestration-build-agent**: Bash for agent spawning tests

### Improve Agent (Purple)

**Purpose**: Git history analysis and expertise updates.

**Tools**: `Read, Write, Edit, Glob, Grep, Bash`

**Model**: `sonnet`

**Constraints**:
- ONLY updates expertise.yaml (preserves agent prompt stability)
- MUST add timestamps to new entries
- MUST preserve existing valid patterns
- Uses git commands for historical analysis

**Frontmatter Example** (from `.claude/agents/experts/github/github-improve-agent.md`):
```yaml
---
name: github-improve-agent
description: Improves GitHub expertise from repository patterns. Expects: FOCUS_AREA (optional)
tools: Read, Write, Edit, Glob, Grep, Bash
model: sonnet
color: purple
---
```

**Workflow Structure**:
1. **Analyze Recent Changes** - Git log, branch analysis, PR/issue review, release patterns
2. **Extract Workflow Learnings** - Six focus areas: commit quality, branch naming, PR structure, workflow effectiveness, collaboration, safety adherence
3. **Identify Effective Patterns** - New workflows, successful structures, resolution approaches
4. **Review Issues and Resolutions** - Error recovery patterns, preventable issues
5. **Assess Expert Effectiveness** - Compare planned vs actual outcomes, identify gaps
6. **Update Expertise** - Edit expertise.yaml with PRESERVE/APPEND/DATE/REMOVE rules
7. **Document Pattern Discoveries** - Name patterns, add context, link evidence

**Update Rules** (from github-improve-agent.md):
```yaml
# In expertise.yaml
patterns:
  feature_branch_workflow:
    name: Feature Branch Workflow
    context: Observed from commits abc123, def456, ghi789
    implementation: |
      1. Create branch: feature/<issue>-<slug>
      2. Multiple commits during development
      ...
    trade_offs:
      - advantage: Clean main history
        cost: Lost granular commit history on merge
    real_examples:
      - commit: abc123
        note: Feature PR #45 - exemplar structure
    timestamp: 2025-12-26
```

**Output**: Updated expertise.yaml with timestamped learnings and evidence.

**Why Purple**: Reflection phase—synthesizing experience into knowledge.

### Question Agent (Cyan)

**Purpose**: Advisory Q&A and safe exploration.

**Tools**: `Read, Glob, Grep` (READ-ONLY)

**Model**: `haiku` (faster, cheaper for queries)

**Constraints**:
- NO write access (pure advisory role)
- NO Task spawning (doesn't coordinate)
- Answers from expertise.yaml + codebase analysis
- Safe for speculative queries

**Frontmatter Example** (from `.claude/agents/experts/orchestration/orchestration-question-agent.md`):
```yaml
---
name: orchestration-question-agent
description: Answers orchestration questions. Expects: USER_PROMPT (required question)
tools: Read, Glob, Grep
model: haiku
color: cyan
---
```

**Workflow Structure**:
1. **Understand Question** - Parse for domain topic, identify expertise sections
2. **Load Expertise** - Read expertise.yaml, find relevant sections, gather examples
3. **Formulate Answer** - Direct answer from expertise with supporting evidence
4. **Provide Context** - Explain pattern rationale, note pitfalls, suggest related topics

**Common Question Patterns**:
```markdown
### Orchestration Domain
Q: When should I use parallel vs sequential execution?
A: Parallel when operations are independent. Sequential when B depends on A's output.

### GitHub Domain
Q: Why can't I force push?
A: Safety protocol. Force push rewrites history, breaks collaborators' repos.

### Knowledge Domain
Q: How do I structure a new chapter?
A: Use standard frontmatter with part/chapter/section, start with Core Questions...
```

**Output**: Concise answer with expertise source references.

**Why Cyan**: Query phase—information retrieval without modification.

**Why Haiku**: Question-answering requires fast response, lower context needs than implementation. Haiku model provides 4× cost reduction and 2× speed improvement for queries that don't require deep reasoning.

---

## expertise.yaml: Structured Knowledge Schema

*[2025-12-26]*: The expertise.yaml format provides a 500-600 line structured knowledge base that replaces embedded expertise sections in agent prompts. This separation enables queryable knowledge (via question-agent), self-improvement (via improve-agent), and cross-domain pattern sharing.

### Schema Sections

**1. overview** (30-50 lines)
```yaml
overview:
  description: |
    High-level summary of domain purpose and scope.
  scope: |
    What's covered and what's explicitly excluded.
  rationale: |
    Why this domain needs dedicated expertise.
```

**Purpose**: Establishes domain boundaries and justifies specialization.

**2. core_implementation** (40-80 lines)
```yaml
core_implementation:
  primary_files:
    - path: .git/
      purpose: Local repository state
    - path: .github/
      purpose: GitHub-specific configurations

  key_conventions:
    - name: Conventional Commits
      summary: Structured commit message format for clarity
    - name: Branch Naming
      summary: Type-prefixed branches with issue numbers
```

**Purpose**: Anchors expertise to specific codebase locations and established conventions.

**Example** (from github/expertise.yaml lines 18-36): Documents that git operations center on `.git/` and `.github/` directories, following Conventional Commits and branch naming conventions.

**3. key_operations** (200-300 lines, largest section)
```yaml
key_operations:
  create_conventional_commit:
    name: Create Conventional Commit
    description: Format and execute git commit following Conventional Commits
    when_to_use: Committing changes to repository
    approach: |
      Format: <type>(<scope>): <description>

      Types: feat, fix, chore, docs, test, refactor, perf, ci, build, style

      Always include:
      - Co-Authored-By: Claude <noreply@anthropic.com>
    examples:
      - type: feat
        message: "feat(agents): add GitHub expert domain\n\nImplements plan/build/improve/question pattern..."
    pitfalls:
      - what: Generic commit messages ("update files")
        why: Unclear what changed or why
        instead: Specific description with type and scope
```

**Purpose**: Exhaustive documentation of 8-15 core domain operations with examples and anti-patterns.

**Structure Pattern**:
- **name**: Human-readable operation name
- **description**: One-line summary
- **when_to_use**: Context for application
- **approach**: Step-by-step implementation (often multi-line YAML literal)
- **examples**: Real-world cases with code/commands
- **pitfalls**: Common mistakes with explanations

**Example** (from github/expertise.yaml lines 38-71): The `create_conventional_commit` operation documents commit types, format structure, Co-Authored-By requirement, with anti-pattern warning about generic messages.

**4. decision_trees** (60-100 lines)
```yaml
decision_trees:
  commit_type_selection:
    name: Conventional Commit Type Selection
    entry_point: What changed in this commit?
    branches:
      - condition: New functionality added
        action: feat
        observed_usage: 9/20 conventional commits (most common)
      - condition: Bug fixed
        action: fix
        observed_usage: 3/20 conventional commits
      - condition: Dependencies updated, config changed
        action: chore
        observed_usage: 1/20 conventional commits
    timestamp: 2025-12-26
```

**Purpose**: Codifies decision logic for common domain choices.

**Structure**: Entry point question with conditional branches leading to actions, often with observed usage data.

**Example** (from github/expertise.yaml lines 211-239): Commit type selection tree shows feat (45% usage) and refactor (20% usage) are most common in this repository.

**5. patterns** (100-150 lines)
```yaml
patterns:
  feature_branch_workflow:
    name: Feature Branch Workflow
    context: Standard feature development flow
    implementation: |
      1. Create issue describing feature
      2. Create branch: feature/<issue>-<slug>
      3. Make commits following Conventional Commits
      4. Push branch regularly
      5. Create PR when ready
      6. Address review feedback
      7. Squash merge to main
      8. Delete branch
    trade_offs:
      - advantage: Isolated development, clear PR scope
        cost: Overhead of branch management
    real_examples:
      - commit: 35a871a
        note: "feat(experts): add github expert domain - exemplar commit format"
```

**Purpose**: Higher-level workflow patterns with trade-off analysis and real evidence.

**Structure**: Named patterns with context, implementation steps, explicit trade-offs (advantage/cost pairs), and links to real commits/files.

**Example** (from github/expertise.yaml lines 267-307): Documents both feature_branch_workflow and direct_to_main_workflow with analysis showing this repository uses direct-to-main (0 PRs observed).

**6. safety_protocols** (40-60 lines)
```yaml
safety_protocols:
  - protocol: Never Force Push
    description: Do not use `git push --force` without explicit user request
    rationale: Rewrites history, breaks collaborators' local repos
    exception: User explicitly requests it for known reason
    timestamp: 2025-12-26
```

**Purpose**: Immutable safety rules that protect users and codebases.

**Structure**: Protocol name, description, rationale, exceptions, timestamp.

**Example** (from github/expertise.yaml lines 398-427): Five safety protocols including force push prevention, credential exclusion, Co-Authored-By requirement.

**7. best_practices** (60-100 lines, MUTABLE)
```yaml
best_practices:
  - category: Commit Messages
    practices:
      - practice: Use Conventional Commits format
        evidence: Industry standard for structured commit history
        timestamp: 2025-12-26
      - practice: Write descriptive commit bodies explaining "why"
        evidence: Helps future developers understand intent
        observed: 20/34 recent commits include detailed body explanations
        timestamp: 2025-12-26
```

**Purpose**: Evolving best practices with evidence and observation data. Updated by improve-agent.

**Structure**: Categories with practices, each citing evidence/observations and timestamps.

**Example** (from github/expertise.yaml lines 428-462): Commit message best practices cite observed behavior (20/34 commits have detailed bodies) and include emerging pattern of emoji in Claude Code attribution.

**8. known_issues** (30-50 lines, MUTABLE)
```yaml
known_issues:
  - issue: Force push protection relies on prompt constraints only
    workaround: Explicit checks in build agent before git push --force
    status: open
    evidence: git reflog shows 1 instance of reset/force operations in 2 weeks
    timestamp: 2025-12-26
```

**Purpose**: Current limitations and their workarounds. Updated by improve-agent.

**Structure**: Issue description, workaround, status (open/resolved), evidence, timestamp.

**Example** (from github/expertise.yaml lines 488-512): Documents that 28% of commits use generic "update" messages, impact on history comprehension, status as open issue.

**9. potential_enhancements** (40-60 lines, MUTABLE)
```yaml
potential_enhancements:
  - enhancement: Automated commit message linting
    rationale: Validate Conventional Commits format before commit
    effort: low
    impact: Would prevent 28% of commits from being generic "update" messages
    timestamp: 2025-12-26
```

**Purpose**: Future improvement roadmap with effort/impact analysis.

**Structure**: Enhancement name, rationale, effort estimate, quantified impact, timestamp.

**Example** (from github/expertise.yaml lines 513-550): Five enhancements ranging from commit linting (low effort, 28% impact) to release notes auto-generation (medium effort, requires 100% Conventional Commits adoption).

### Mutability Strategy

**Stable Sections** (preserved by improve-agent):
- `overview` - Domain definition rarely changes
- `core_implementation` - File structure is stable
- `key_operations` structure - Operation names/signatures stable
- `safety_protocols` - Safety rules are immutable

**Mutable Sections** (updated by improve-agent):
- `key_operations.*.examples` - Add real examples from repository
- `decision_trees.*.observed_usage` - Update statistics from git analysis
- `patterns.*.real_examples` - Link to new exemplar commits
- `best_practices` - Append new practices with evidence
- `known_issues` - Add/resolve issues with timestamps
- `potential_enhancements` - Evolving roadmap

**Target Size**: 500-600 lines per domain (github/expertise.yaml: 550 lines)

**Why This Works**: Separating stable domain knowledge (operations, protocols) from evolving observations (examples, usage stats, issues) allows improve-agent to add learnings without destabilizing core expertise. The timestamp-driven mutability creates an audit trail showing knowledge evolution.

---

## Multi-Domain Coordination

*[2025-12-26]*: The 11-domain expert system creates specialized knowledge territories with clear boundaries and coordinated routing through the /do command.

### The 11 Expert Domains

| Domain | Agents | Expertise Lines | Purpose |
|--------|--------|----------------|---------|
| **agent-authoring** | plan, build, improve, question | ~520 | Agent creation and configuration (.claude/agents/) |
| **audit** | plan, build, improve, question | ~480 | External codebase auditing and analysis |
| **book-structure** | plan, build, improve, question | ~560 | Frontmatter, chapters, TOC management |
| **claude-config** | plan, build, improve, question | ~490 | Claude Code configuration (.clauderc, prompts) |
| **do-management** | plan, build, improve, question | ~530 | /do command routing and classification |
| **external-teacher** | plan, build, improve, question | ~510 | Teaching external projects .claude/ setup |
| **github** | plan, build, improve, question | 550 | Git/GitHub operations (commits, PRs, branches) |
| **knowledge** | plan, build, improve, question | ~540 | Book content updates (chapters/, STYLE_GUIDE) |
| **orchestration** | plan, build, improve, question | ~505 | Coordination patterns (Task, parallelism, specs) |
| **questions** | ask, build, deepen, format, improve | ~590 | Question-driven content development |
| **research** | plan, build, improve, question | ~470 | External source research and synthesis |

**Total**: 44 agents, ~5,745 lines of structured expertise.

**Note**: Questions domain has non-standard structure (5 agents instead of 4) with ask/build/deepen/format workflow, but follows self-improvement pattern via questions-improve-agent.

### Flat /do Routing (Post-353d576)

The /do command routes user requirements directly to expert domains based on pattern classification:

**Pattern A - Direct Questions** → **question-agent** (domain-specific)
```
User: "/do How do I structure a PR?"
→ github-question-agent (haiku, read-only)
```

**Pattern B - Simple Implementation** → **build-agent** (skip planning)
```
User: "/do Fix typo in README.md line 42"
→ knowledge-build-agent (with inline spec)
```

**Pattern C - Standard Plan→Build** → **plan-agent** → [approval] → **build-agent**
```
User: "/do Add new section on context patterns"
→ knowledge-plan-agent → spec file → [user reviews] → knowledge-build-agent
```

**Pattern D - Full Lifecycle** → **plan** → [approval] → **build** → **improve**
```
User: "/do Implement GitHub release workflow"
→ github-plan-agent → spec → [approval] → github-build-agent → github-improve-agent
```

**Pattern E - Improve Only** → **improve-agent** (analysis mode)
```
User: "/do Update GitHub expertise from recent commits"
→ github-improve-agent (analyzes git log, updates expertise.yaml)
```

**Routing Logic**: Implemented in `.claude/agents/experts/do-management/` expert domain. The do-management-plan-agent classifies user requirements into A-E patterns and spawns appropriate agent sequence.

### Skills vs Experts Distinction

The 353d576 refactoring separated **cross-cutting capabilities** (skills) from **domain expertise** (experts):

**Skills** (in `.claude/skills/`):
- `orchestrating-knowledge-workflows` - Plan→build→improve orchestration
- `researching-external-sources` - Parallel web research
- `executing-comprehensive-reviews` - Multi-type review routing
- `managing-book-operations` - TOC generation, metadata validation

**Experts** (in `.claude/agents/experts/<domain>/`):
- Domain-specific knowledge (expertise.yaml)
- 4-agent teams (plan/build/improve/question)
- Focused on single knowledge territory

**Key Difference**: Skills are **invoked by description match** (Claude Code automatic loading based on user request). Experts are **explicitly routed** through /do pattern classification.

**Anti-Pattern**: Don't create skills that duplicate expert domains. Skills should provide workflow orchestration (like plan→build→improve cycle), not domain knowledge (that belongs in expertise.yaml).

### Avoiding Domain Redundancy

With 11 domains, clear boundaries prevent overlap:

**Boundary Examples**:
- **github** domain: Git commands, PR structure, commit format
- **orchestration** domain: Task tool usage, parallelism, coordinator patterns
- **knowledge** domain: Book content structure, STYLE_GUIDE, frontmatter
- **book-structure** domain: TOC generation, chapter organization, metadata validation

**Overlap Resolution**:
When operations span domains, use **primary domain routing** with cross-references:

```
User: "/do Create GitHub PR for new book chapter"

Primary: github-plan-agent (handles PR creation)
Cross-Reference: Checks book-structure expertise for chapter frontmatter validation
```

**Expertise Cross-References**:
expertise.yaml files link to related domains:
```yaml
# In github/expertise.yaml
patterns:
  agent_absorption_pattern:
    # ... implementation details
    see_also:
      - domain: agent-authoring
        topic: 4-agent pattern structure
      - domain: do-management
        topic: routing update after absorption
```

This creates a **web of knowledge** where domains remain focused but acknowledge intersections.

---

## Agent Absorption: Case Study

*[2025-12-26]*: Commit 35a871a demonstrates the eight-step pattern for absorbing standalone agents into the expert domain system. This case study documents the GitHub agent absorption that converted a 383-line standalone agent into a 4-agent expert domain with structured expertise.

### The Absorption Pattern (8 Steps)

**Step 1: Identify Standalone Agent for Absorption**

**Before State**:
- File: `.claude/agents/github-versioning-agent.md` (383 lines)
- Purpose: Git/GitHub CLI operations (commits, branches, PRs, issues, releases)
- Problem: Monolithic structure, no self-improvement, difficult to query

**Absorption Criteria**:
- Agent handles coherent domain (GitHub operations)
- Operations recur frequently (commits every day)
- Domain knowledge would benefit from accumulation (commit patterns, PR structures)
- Standalone structure limits evolution

**Step 2: Create Domain Directory**

```bash
mkdir -p .claude/agents/experts/github
```

**Naming Convention**: Domain name in singular form (github, orchestration, knowledge), not pluralized.

**Step 3: Extract Domain Knowledge to expertise.yaml**

**Source Material**: github-versioning-agent.md contained embedded knowledge:
- Conventional Commits format examples
- Branch naming conventions
- PR structure templates
- Safety protocols (no force push, no credential commits)

**Extraction Process**:
1. Identify **stable knowledge** → `core_implementation`, `safety_protocols`
2. Extract **operations** → `key_operations` (create_conventional_commit, create_feature_branch, etc.)
3. Document **workflows** → `patterns` (feature_branch_workflow, hotfix_workflow)
4. Add **decision trees** → commit_type_selection, branch_workflow_selection
5. Initialize **mutable sections** → best_practices, known_issues, potential_enhancements

**Result**: `.claude/agents/experts/github/expertise.yaml` (550 lines)

**Key Insight**: Monolithic agent contained ~150 lines of actual knowledge mixed with 233 lines of workflow instructions. Expertise extraction **separated knowledge from process**, enabling independent evolution.

**Step 4: Create plan-agent from Analysis Capabilities**

**Source**: github-versioning-agent.md had "planning" concerns mixed with execution.

**Extraction**:
```yaml
---
name: github-plan-agent
description: Plans GitHub operations (commits, branches, PRs, issues, releases)
tools: Read, Glob, Grep, Write  # Write for spec creation only
model: sonnet
color: yellow
---
```

**Workflow** (7 steps):
1. Understand operation requirement
2. Check repository state needs
3. Plan command sequence
4. Identify safety checks
5. Determine approval gates
6. Plan workflow-specific requirements (commits/branches/PRs/issues/releases)
7. Save specification to `.claude/.cache/specs/github/`

**Lines**: 229 lines (60% workflow, 40% quick reference from expertise.yaml)

**Step 5: Create build-agent from Execution Capabilities**

**Source**: github-versioning-agent.md had execution logic for git/gh commands.

**Extraction**:
```yaml
---
name: github-build-agent
description: Executes GitHub operations from spec
tools: Read, Edit, Bash  # Bash for git/gh CLI
model: sonnet
color: green
---
```

**Workflow** (6 steps):
1. Load specification from SPEC variable
2. Verify repository state
3. Execute command sequence (git/gh commands)
4. Apply safety checks at each step
5. Handle approval gates (if HUMAN_IN_LOOP)
6. Report execution results

**Lines**: 187 lines (execution-focused, references expertise.yaml for conventions)

**Step 6: Create improve-agent for Git History Learning**

**New Capability** (didn't exist in standalone agent):

```yaml
---
name: github-improve-agent
description: Improves GitHub expertise from repository patterns
tools: Read, Write, Edit, Glob, Grep, Bash  # Bash for git log analysis
model: sonnet
color: purple
---
```

**Workflow** (7 steps):
1. Analyze recent changes (git log, branch analysis, PR/issue review)
2. Extract workflow learnings (6 focus areas: commit quality, branch naming, PR structure, workflow effectiveness, collaboration, safety adherence)
3. Identify effective patterns
4. Review issues and resolutions
5. Assess expert effectiveness
6. Update expertise.yaml (PRESERVE/APPEND/DATE/REMOVE rules)
7. Document pattern discoveries with evidence

**Lines**: 270 lines

**Key Innovation**: The improve-agent analyzes actual git history to discover patterns:
```bash
# Commit pattern analysis
git log --format="%s" -30  # Extract commit messages
git log --graph --oneline --all --decorate -30  # Branch topology

# PR analysis
gh pr list --state all --limit 50
gh pr view <number> --json title,body,reviews,state
```

This enables **repository-specific learning**: observed that this repo uses direct-to-main workflow (0 PRs), favors feat (45%) and refactor (20%) commits, has 28% generic "update" messages.

**Step 7: Create question-agent for Read-Only Q&A**

**New Capability** (advisory interface):

```yaml
---
name: github-question-agent
description: Answers GitHub operation questions
tools: Read, Glob, Grep  # Read-only, no modifications
model: haiku  # Faster, cheaper for queries
color: cyan
---
```

**Workflow** (4 steps):
1. Understand question (parse for GitHub topic)
2. Load expertise (read expertise.yaml, find relevant sections)
3. Formulate answer (direct answer from expertise with examples)
4. Provide context (explain rationale, note pitfalls, suggest related topics)

**Lines**: 150 lines

**Common Questions Handled**:
- "Why can't I force push?" → Safety protocol explanation
- "How do I structure a PR?" → PR template guidance
- "What commit type should I use?" → Decision tree walkthrough

**Key Benefit**: Haiku model provides 4× cost reduction for simple Q&A vs sonnet-based build/plan agents.

**Step 8: Delete Original Agent, Update Routing**

**Deletions**:
```bash
rm .claude/agents/github-versioning-agent.md  # -383 lines
```

**Additions**:
```bash
.claude/agents/experts/github/
├── expertise.yaml           # +550 lines
├── github-plan-agent.md     # +229 lines
├── github-build-agent.md    # +187 lines
├── github-improve-agent.md  # +270 lines
└── github-question-agent.md # +150 lines

Total: +1,386 lines
```

**Net Change**: +1,003 lines (3.6× expansion)

**Routing Updates**:

**CLAUDE.md** - Added github domain to experts table:
```markdown
| github | plan, build, improve, question | GitHub operations: commits, branches, PRs, releases |
```

**do.md** (do-management expertise) - Added routing logic:
```yaml
# In do-management/expertise.yaml
domain_routing:
  github:
    triggers: ["commit", "branch", "PR", "pull request", "issue", "release", "git", "github"]
    agent_pattern: github-{plan|build|improve|question}-agent
```

**Commit Message** (35a871a):
```
feat(experts): add github expert domain with full 4-agent pattern

Migrates github-versioning-agent → experts/github/ with:
- expertise.yaml (550 lines) - structured GitHub knowledge
- 4-agent pattern: plan (yellow), build (green), improve (purple), question (cyan)
- Self-improvement capability via git log analysis
- Read-only question interface (haiku model)

Deletes github-versioning-agent.md (383 lines, now obsolete).

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Before vs After Comparison

| Aspect | Standalone Agent | Expert Domain |
|--------|-----------------|--------------|
| **Structure** | Single 383-line file | 5 files (1,386 lines) |
| **Knowledge Format** | Embedded in prompt | Structured expertise.yaml |
| **Self-Improvement** | None | improve-agent with git analysis |
| **Queryability** | Not queryable | question-agent (haiku, read-only) |
| **Tool Boundaries** | Mixed (all tools in one agent) | Strict (plan: no Bash, question: read-only) |
| **Planning/Execution** | Intermingled | Separated (plan → spec → build) |
| **Evidence Base** | Static examples | Real examples from git history |
| **Routing** | Direct invocation | Pattern-based /do routing |

### Quantified Benefits

**From improve-agent analysis** (ran after 2 weeks):
- **20/50 commits** follow Conventional Commits (40% adherence)
- **Most common types**: feat (9), refactor (4), fix (3), chore (1)
- **Direct-to-main workflow** detected (0 PRs in repository)
- **28% commits** use generic "update" messages (identified as known_issue)
- **1 force push** detected via git reflog (safety protocol validation)

**Knowledge accumulation**: expertise.yaml gained 6 patterns, 3 decision trees, 4 best practices, 4 known issues, 5 potential enhancements—all with timestamps and evidence from actual repository usage.

**Absorption ROI**: 3.6× code expansion, but enables continuous knowledge accumulation without agent prompt modifications. The improve-agent runs periodically, expertise evolves, plan/build agents automatically benefit from updated knowledge.

---

## Agent Registry Pattern

*[2026-02-05]*: As expert domains scale from 1 to 10+, manual agent discovery becomes bottleneck. Agent registries auto-generate machine-readable catalogs from agent frontmatter, enabling programmatic discovery and routing.

### The Discovery Problem at Scale

With 11 domains × 4 agents = 44 specialized agents, routing questions emerge:
- Which agents can modify TypeScript files?
- Which agents use Haiku model (for cost-sensitive queries)?
- Which agents have Write access (vs read-only)?

Manual maintenance doesn't scale. Registry generation solves this.

### Registry Generation from Frontmatter

**Source material:** Agent frontmatter contains structured metadata:

```yaml
---
name: knowledge-build-agent
tools: Read, Write, Edit, Glob, Grep, WebSearch
model: sonnet
color: green
output-style: practitioner-focused
---
```

**Generated indices:**

**Capability Index:**
```json
{
  "can_write": [
    "knowledge-build-agent",
    "github-build-agent",
    "book-structure-build-agent"
  ],
  "read_only": [
    "knowledge-question-agent",
    "github-question-agent",
    "orchestration-question-agent"
  ]
}
```

**Model Index:**
```json
{
  "haiku": ["knowledge-question-agent", "github-question-agent"],
  "sonnet": ["knowledge-build-agent", "github-plan-agent"],
  "opus": []
}
```

**Tool Matrix:**
```json
{
  "Read": ["*-agent"],
  "Write": ["*-build-agent", "*-plan-agent"],
  "Bash": ["github-build-agent", "orchestration-build-agent"],
  "WebSearch": ["knowledge-build-agent", "research-build-agent"]
}
```

### Routing Applications

**Cost-sensitive queries:** Route to Haiku agents when speed/cost matters
```
Query: "How do I structure a PR?"
→ github-question-agent (haiku, read-only, 4× cheaper)
```

**Capability-aware delegation:** Route based on required tool access
```
Task: "Implement new section"
→ knowledge-build-agent (has Write, Edit, WebSearch)
```

**Domain discovery:** "Which agents can help with GitHub operations?"
```
Query index: domain="github" AND tools CONTAINS "Bash"
→ [github-build-agent, github-improve-agent]
```

### Registry Update Strategy

**Option 1: Build-time generation**
- Script scans `.claude/agents/experts/` at build time
- Generates `agent-registry.json`
- Commit to repository (static lookup)

**Option 2: Runtime query**
- On-demand frontmatter parsing
- No pre-generated file required
- Slower but always fresh

**Option 3: Hybrid with cache invalidation**
- Generate on first query, cache result
- Invalidate when `.claude/agents/` directory changes
- Balance speed and freshness

### When to Use Registry Pattern

**Good fit:**
- 10+ agents across multiple domains
- Programmatic routing logic (like /do command)
- Cost-sensitive delegation (model tier selection)
- Multi-agent orchestration systems

**Poor fit:**
- Small agent count (<5 agents)
- Manual agent selection sufficient
- Single-domain systems

### Connections

- **To [Context Contracts](../../5-tool-use/5-skills-and-meta-tools.md#context-contracts-for-agent-capability-declaration)**: Registry provides discovery layer; contracts provide capability schema
- **To [Expert Swarm](8-expert-swarm-pattern.md)**: Registry enables orchestrator to discover appropriate workers for swarm coordination

**Implementation Note:** This book's 11-domain expert system would benefit from registry generation for the /do command routing logic.

**Sources:** Advanced external .claude/ implementation patterns, 4-agent pattern standardization commit analysis (39e7904).

---

## Connections

- **To [Plan-Build-Review Pattern](1-plan-build-review.md)**: The general pattern this specializes. The three-command structure (plan/build/improve) implements the conceptual separation of concerns. The expert domains paradigm extends plan-build-review to an 11-domain system with 44 agents, demonstrating how the pattern scales from individual workflows to system architecture.
- **To [Expert Swarm Pattern](8-expert-swarm-pattern.md)**: Expertise.yaml becomes shared knowledge source for parallel workers. Expert acts as swarm coordinator, passing expertise path to spawned agents. Enables scale (10+ parallel) with consistency (shared expertise). The improve agent runs post-swarm to analyze collective execution and update expertise for future swarms.
- **To [Prompts/Structuring](../2-prompt/2-structuring.md)**: How mutable sections enable learning. The Expertise vs. Workflow separation depends on prompt structure that supports targeted updates. The expertise.yaml schema represents the ultimate evolution of structured prompts—separating stable domain knowledge from evolving observations with explicit mutability boundaries.
- **To [Claude Code](../9-practitioner-toolkit/1-claude-code.md)**: Concrete implementation in slash commands. The orchestrator pattern shows how self-improving experts deploy in practice. The /do flat routing and skill-based loading demonstrate Claude Code's native support for multi-agent coordination without custom orchestration layers.
- **To [Evaluation](../7-practices/2-evaluation.md)**: How to measure if Improve is improving. The three-role architecture's +1.7% improvement from multi-iteration reflection demonstrates the value of measurement in validating architectural choices. The GitHub agent absorption case study shows measurable knowledge accumulation: 6 patterns, 3 decision trees, 4 best practices discovered from 2 weeks of git history analysis.
