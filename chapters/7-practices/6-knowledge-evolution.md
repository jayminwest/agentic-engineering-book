---
title: Knowledge Evolution
description: Guidelines for how knowledge base entries should be updated over time
created: 2025-12-08
last_updated: 2026-02-05
tags: [practices, maintenance, evolution]
part: 2
part_title: Craft
chapter: 7
section: 6
order: 2.7.6
---

# Knowledge Evolution

A knowledge base isn't static. It grows, refines, and occasionally prunes. But not all updates are equal—some preserve hard-won insights, others accidentally erase them.

---

## Your Mental Model

**Knowledge bases are gardens, not databases.** You don't just add rows—you prune, transplant, and sometimes let things go fallow. The goal isn't comprehensiveness; it's usefulness.

**Conservative updates protect against the "second system effect."** Early patterns came from real pain. Later refinements can feel more sophisticated but lose the original insight. Default to preservation.

**Timestamps are accountability, not decoration.** Every dated addition answers "when did we learn this?" and "from what context?" This lets future you judge relevance.

**Status reflects confidence, not completeness.** An evergreen entry might be short. A seedling might be long. Status is "how battle-tested is this?"

---

## The Grow-and-Refine Principle

*[2025-12-10]*

**Balance expansion with consolidation.** A knowledge base that only grows becomes unwieldy. One that constantly condenses loses nuance. Sustainable evolution requires both phases.

**Growth Phase:**
- Accumulate new insights freely during active learning
- Capture patterns as you discover them
- Don't worry about redundancy or overlap initially
- Let the knowledge base expand naturally as you build

**Refinement Phase:**
- Periodically deduplicate based on semantic similarity
- Prune entries with low utility metrics
- Consolidate overlapping patterns
- Respect size constraints (context window limits)

**Why this works:**
- Prevents unbounded growth that exceeds context windows
- Avoids premature compression that loses insights
- Creates breathing room for exploration without chaos
- Matches how humans learn: gather broadly, then synthesize

**The parallel to PRESERVE/APPEND/DATE/REMOVE:**
- PRESERVE and APPEND handle the growth phase
- REMOVE handles refinement
- DATE enables evidence-based decisions about what to refine

---

## The PRESERVE/APPEND/DATE/REMOVE Framework

### PRESERVE: Existing Patterns Stay

**Default to keeping what's there.** If an existing pattern works in practice, new information extends it rather than replaces it.

**What to preserve:**
- Patterns that solved real problems in real projects
- Concrete examples from actual implementations
- Hard-won lessons that contradicted initial assumptions
- Any content sourced from examples/ directories

**Preservation signals:**
```markdown
## Structural Patterns
### The Canonical 7-Section Structure
[Original pattern stays intact]

*[2025-12-08]*: Extended with XML tag alternative for nested sections.
```

### APPEND: New Learnings Added

**Add, don't rewrite.** New insights get their own sections, subsections, or inline dated annotations.

**Append patterns:**
- New examples in existing categories
- Additional questions in "Core Questions"
- Dated inline notes: `*[YYYY-MM-DD]*: New insight here`
- New subsections under existing headers
- Additional rows in tables
- New artifacts in "Artifacts & Examples"

**Example of good appending:**
```markdown
### Output Template Categories
[Existing table preserved]

*[2025-12-08]*: Added "Streaming" category for real-time output scenarios where results must be shown incrementally rather than batched.
```

### DATE: All New Entries Marked

**Timestamp attribution enables future judgment.** Every new section, subsection, or inline addition should be dated.

**Dating conventions:**
```markdown
# New top-level section
## New Section Title
*[2025-12-08]*

Content here...

---

# Existing section
*[2025-12-08]*: Inline addition to existing paragraph.

### Existing subsection
*[2025-12-08]*: New paragraph added to existing subsection.
```

**What gets dated:**
- Inline additions to existing content
- New sections or subsections
- New table rows (via inline note)
- Updated "Your Mental Model" additions
- New questions in "Core Questions"

**What doesn't get dated:**
- Typo fixes
- Formatting improvements
- Clarification of existing points without adding new information
- Reordering for better flow

### REMOVE: Only When Obsolete

**Deletion requires high confidence.** Remove only when:

1. **Multiple implementations contradict it** - Three or more projects proved the pattern wrong
2. **Technology shifted fundamentally** - API deprecated, tool replaced, paradigm obsolete
3. **Created provable harm** - Pattern led to bugs or degraded systems
4. **Superceded by better pattern** - New approach strictly dominates old one

### Learning Separation in Multi-Agent Systems

*[2026-02-05]*: Production multi-agent systems extend the learning separation principle to coordination boundaries. Teammates execute tasks only; learning happens post-hoc via dedicated improve-agents analyzing collective execution history.

**The coordination constraint:**

In flat team orchestration (see [Expert Swarm Pattern](../../6-patterns/8-expert-swarm-pattern.md)), multiple agents execute tasks in parallel. If agents update shared expertise during execution, race conditions emerge:

```
Agent 1 reads expertise.yaml (version A)
Agent 2 reads expertise.yaml (version A)
Agent 1 updates expertise.yaml → version B
Agent 2 updates expertise.yaml → version C (overwrites B)
Result: Agent 1's learnings lost
```

**The clean separation:**

**Execution Phase (teammates):**
- Read expertise.yaml for guidance
- Execute assigned tasks
- Do NOT update expertise.yaml
- Report completion to orchestrator

**Learning Phase (improve-agent):**
- Runs AFTER all teammates complete
- Analyzes git history from collective execution
- Updates expertise.yaml once with all learnings
- Next team benefits from accumulated knowledge

**Example workflow:**
```
1. Orchestrator spawns 10 build agents (read expertise.yaml)
2. Build agents execute tasks in parallel
3. All agents complete, orchestrator commits changes
4. Orchestrator spawns improve-agent
5. Improve-agent analyzes git diff, updates expertise.yaml
6. Next orchestration uses improved expertise
```

**Why this works:**
- No coordination overhead during execution (agents focus on tasks)
- No race conditions (single writer in learning phase)
- Expertise accumulates over time (improve-agent sees full context)
- Fast execution (agents don't pause to update knowledge)

**Connection to PRESERVE/APPEND/DATE/REMOVE:**

Learning separation determines WHEN updates happen (post-hoc), while PRESERVE/APPEND/DATE/REMOVE governs HOW updates happen (conservative, timestamped). Both patterns combine to create sustainable knowledge accumulation without coordination chaos.

**Sources:** Expert Swarm Pattern production evidence (commit 20500f1), advanced .claude/ implementation analysis.

*[2025-12-10]*: **Utility tracking makes removal evidence-based.** Rather than guessing which entries to prune, track metrics:

```markdown
# Utility Tracking Pattern
- Track helpful/harmful counters for each entry
- Increment helpful when entry solves a problem
- Increment harmful when entry leads to error or confusion
- Prune entries where harmful > helpful
- Review zero-interaction entries quarterly
```

This shifts removal from subjective judgment to measured outcomes.

**Removal process:**
```markdown
# Before removal
1. Check git history: When was this added? From which project?
2. Search examples/: Does this still appear in any project?
3. Search other entries: Is this referenced elsewhere?
4. Consider archival: Should this move to a "Deprecated Patterns" section?

# During removal
- Add inline note explaining removal: *[2025-12-08]: Removed [pattern] - contradicted by X, Y, Z projects*
- Update any cross-references in other files
- Update CLAUDE.md if section structure changed
```

**Example of justified removal:**
```markdown
*[2025-12-08]*: Removed "Phase Polling with 5-second intervals" pattern. Contradicted by multiple recent projects (e.g., KotaDB) which standardized on 10-15 second intervals to reduce API load. Original pattern caused rate limiting issues.
```

---

## Delta-Based Updates vs Full Rewrites

*[2025-12-10]*

**Prefer incremental operations over complete regeneration.** When updating knowledge bases, the temptation is to rewrite everything "cleanly." This causes context collapse and brevity bias.

### The Three Delta Operations

**ADD** - Append new entries to existing content
- Parallelizable: Multiple agents can add simultaneously
- Auditable: Clear diff shows what changed
- Reversible: Easy to undo specific additions
- Knowledge-preserving: Existing insights stay intact

**UPDATE** - Modify specific entries in place
- Surgical: Change only what needs changing
- Traceable: Timestamps show evolution
- Conservative: Defaults to preservation (see PRESERVE above)

**REMOVE** - Prune low-value entries (see utility tracking)
- Evidence-based: Remove only with metrics
- Documented: Explain why in commit/timestamp
- Rare: Deletion is the exception, not the norm

### The Anti-Pattern: Emergency Context Rewriting

**What it looks like:**
```markdown
# Agent receives: "Context is too large, rewrite more concisely"
# Agent produces: Condensed version missing 40% of original insights
```

**Why it fails:**
- **Brevity bias**: AI models over-compress to satisfy length constraints
- **Context collapse**: Nuance and examples get stripped out
- **Knowledge loss**: Hard-won lessons disappear silently
- **Non-reversible**: Can't easily restore what was removed

**Measured impact:**
- 75.1% fewer rollouts with delta-based updates vs full rewrites
- 82.3% latency reduction (fewer tokens to process)
- Preservation of edge case handling and anti-patterns
- Better parallelization (multiple agents can add independently)

### When Full Rewrites Are Justified

Rare cases where complete regeneration makes sense:
- **Format migration**: Switching frontmatter schema or structural conventions
- **Fundamental reorganization**: Moving from flat to hierarchical structure
- **Consolidation of duplicates**: After identifying semantic overlap
- **Schema changes**: When metadata structure evolves

Even then, prefer phased migration: rewrite one section at a time, verify quality, then move to next section.

### Practical Implementation

```markdown
# GOOD: Delta-based update
*[2025-12-10]*: Added streaming output category for real-time scenarios.

[New content appended to existing section]

# BAD: Full rewrite
## Output Categories
[Completely regenerated list missing previous examples]
```

**Guideline:** If you're tempted to "clean up" more than 30% of a file, you're probably doing a full rewrite. Step back and identify the specific delta instead.

---

## When to Update vs Create New

### Extend Existing Entry When:

- **Same domain, new example**: "Here's another way to structure validation"
- **Refinement of existing pattern**: "We discovered edge case handling for this"
- **Additional questions**: "What about [new scenario]?"
- **Implementation details**: "Here's how this works in practice"
- **Complementary insight**: "This also connects to..."

### Create New Entry When:

- **Different domain**: Prompt structuring vs cost optimization are separate concerns
- **Contradictory approach**: If you can't reconcile it with existing content, it might be a different pattern
- **Separate lifecycle**: Content that evolves independently from the parent topic
- **Cross-cutting concern**: Something that touches multiple foundations/patterns
- **Distinct mental model**: If you need a fundamentally different framing

### Handling Contradictions

**When new information contradicts existing patterns:**

1. **Verify the contradiction is real** - Are we comparing apples to apples? Different contexts might explain the difference.

2. **Document both approaches** - Add a section comparing the patterns:
   ```markdown
   ### Approach A: [Original]
   - Context: Works well for...
   - Trade-offs: ...

   ### Approach B: [New]
   *[2025-12-08]*
   - Context: Works well for...
   - Trade-offs: ...

   ### When to Use Which
   - Use A when: ...
   - Use B when: ...
   ```

3. **Seek synthesis** - Is there a higher-level pattern that encompasses both?

4. **Mark for investigation** - If uncertain, add to "Core Questions":
   ```markdown
   - Why do project X use approach A while project Y uses approach B?
   - What context makes approach B superior to approach A?
   ```

---

## Status Progression

Status reflects how battle-tested the content is, not how complete it feels.

### Seedling

**What it means:** Fresh capture, minimal structure, lots of questions

**Characteristics:**
- Mostly questions in "Core Questions"
- Minimal or empty "Your Mental Model"
- Few concrete examples
- May be incomplete sections

**Stays seedling if:**
- Not yet applied in a real project
- Still exploring the problem space
- Collecting initial thoughts

**Promote to growing when:**
- You've used the insight in at least one real project
- "Your Mental Model" has concrete statements
- At least one artifact or example exists

### Growing

**What it means:** Active refinement, patterns emerging, multiple examples

**Characteristics:**
- Clear "Your Mental Model" section
- Multiple concrete examples or artifacts
- Some questions answered, new questions emerging
- Applied in 1-3 projects

**Stays growing if:**
- Still discovering edge cases
- Patterns not fully generalized
- Actively being refined with each project

**Promote to mature when:**
- Patterns proven across 3+ diverse projects
- Most common failure modes documented
- Clear guidance on when/how to apply
- Anti-patterns identified

**Demote to seedling if:**
- Major contradictions discovered
- Fundamental rethinking required

### Mature

**What it means:** Stable patterns, well-understood trade-offs, comprehensive

**Characteristics:**
- Comprehensive examples
- Anti-patterns documented
- Clear decision frameworks
- Applied successfully in 3+ projects
- Few updates needed

**Stays mature if:**
- Occasional refinement but no major changes
- New projects confirm existing patterns
- Edge cases being filled in, not core patterns changing

**Promote to evergreen when:**
- No significant changes in 6+ months
- Proven across wide variety of contexts
- Referenced frequently by other entries
- Considered "settled wisdom"

**Demote to growing if:**
- Significant new patterns emerging
- Multiple recent updates changing recommendations

### Evergreen

**What it means:** Foundational, stable, rarely changes

**Characteristics:**
- Core principles that transcend specific implementations
- Minimal updates (typos, clarification, new examples)
- High confidence in recommendations
- Proven across many projects and contexts

**Stays evergreen if:**
- Only minor refinements
- Principles remain sound as ecosystem evolves

**Demote to mature if:**
- Major paradigm shift requires reconsidering fundamentals
- New information changes core recommendations

---

## Update Checklist

Before updating any entry:

```markdown
- [ ] Read the entire entry first
- [ ] Check git history: When was this last updated? By what project?
- [ ] Verify new information doesn't contradict existing patterns (or handle contradiction explicitly)
- [ ] Add timestamps to new content
- [ ] Update `last_updated` in frontmatter
- [ ] If structure changed significantly, update CLAUDE.md
- [ ] If status should change, update status in frontmatter
```

---

## Connections

- **[Debugging Agents](1-debugging-agents.md):** How evolution practices help debug knowledge base drift
- **[Workflow Coordination](5-workflow-coordination.md):** How to communicate knowledge base updates to collaborators
- **[Specs as Source Code](../8-mental-models/3-specs-as-source-code.md):** Knowledge bases are specifications that agents execute, not just reference documentation
- **[Context Management](../4-context/_index.md):** Delta-based updates prevent context collapse and respect window limits. Grow-and-refine principle directly addresses context size constraints.

---

## Sources

- Observed patterns from maintaining this knowledge base
- Conservative update principles from Evergreen note-taking methodology
- Status progression inspired by digital gardening practices
- Grow-and-refine principle from adaptive RAG systems research
- Delta-based update metrics (75.1% reduction, 82.3% latency improvement) from comparative studies on knowledge base maintenance strategies
- Utility tracking patterns from ML feedback systems and A/B testing methodologies
