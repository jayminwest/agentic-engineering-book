---
title: Specs as Source Code
description: Specifications are the primary programming surface in agentic systems
created: 2025-12-10
last_updated: 2026-02-06
tags: [mental-models, specifications, documentation, programming]
part: 3
part_title: Perspectives
chapter: 8
section: 3
order: 3.8.3
---

# Specs as Source Code

**Throwing away prompts after generating code is like checking in compiled binaries while discarding source.**

This mental model, articulated by Sean Grove, reframes how we think about specifications, research documents, and plans in agentic systems.

---

## The Core Shift

In traditional programming:
- Source code is the truth
- Documentation is secondary
- Code is machine-readable and executable

In agentic programming:
- **Specs are the truth**
- Generated code is secondary (can be regenerated)
- **Specs are machine-readable and executable** (by agents)

```
Traditional:                    Agentic:

┌──────────────┐               ┌──────────────┐
│ Source Code  │               │ Specification│
└──────┬───────┘               └──────┬───────┘
       │ compile                      │ agent reads
       ▼                              ▼
┌──────────────┐               ┌──────────────┐
│   Binary     │               │ Generated    │
│ (throwaway)  │               │ Code         │
└──────────────┘               │ (throwaway)  │
                               └──────────────┘
```

When you discard the prompt that generated working code, you've lost the source. You're left maintaining compiled output.

---

## Your Mental Model

**Specs are machine-readable, testable, enforceable contracts.** Not wishful thinking in a Google Doc—they're the primary programming surface. Agents read specs, not vibes.

In agentic systems, 80-90% of programming is structured communication. The specification IS the program. The code is just one artifact the specification produces.

This changes what you version control, what you review, and what you test.

---

## What This Looks Like in Practice

### Specs Become First-Class Artifacts

```
# Traditional project structure
src/
  main.py          # This is what you maintain
  utils.py
docs/
  notes.md         # Throwaway reference
  plan.txt         # Deleted after implementation

# Agentic project structure
specs/
  architecture.md  # Source of truth - version controlled
  requirements.md  # Tested against implementation
  plan.md          # Executable by plan-build-review
src/
  main.py          # Can be regenerated from specs
  utils.py         # Generated code, not hand-maintained
```

### You Version Control Prompts Like Code

Research documents, planning artifacts, and agent instructions are checked into version control because they're the source code:

```bash
git diff specs/authentication-flow.md

- Agent should validate JWT tokens
+ Agent should validate JWT tokens and check revocation list
+ See security-requirements.md section 3.2 for revocation protocol
```

This diff is more important than the code diff it produces. The spec change is the actual change. The code change is a compilation artifact.

### You Review and Test Specs

Code review becomes spec review:

```markdown
# PR: Add user authentication

Changes to specs/:
  + authentication-flow.md
  + security-requirements.md
  + error-handling.md

Generated implementation in src/:
  + auth.py (generated from specs)
  + tests.py (generated from specs)
```

The reviewer focuses on whether the spec is correct, complete, and testable. If the spec is right, the code can always be regenerated.

You write tests that validate the spec was followed:

```python
def test_auth_follows_spec():
    """Verify implementation matches authentication-flow.md section 2."""
    spec = load_spec("authentication-flow.md")
    assert implementation.validates_jwt == spec.requires_jwt_validation
    assert implementation.checks_revocation == spec.requires_revocation_check
```

---

## The Implications

### Research Documents ARE Source Code

That document where you researched authentication approaches? That's not a throwaway—it's the source of truth for why the system works the way it does.

When you need to change authentication later, you don't dig through code trying to reverse-engineer the reasoning. You read the research document, update it with new findings, and regenerate.

### Plans ARE Source Code

The plan you wrote before implementing a feature isn't scaffolding to discard. It's executable source code for the plan-build-review pattern.

```markdown
# feature-plan.md

## Approach
Use Redis for session storage with 24-hour TTL.

## Dependencies
- Redis client library
- Session serialization logic

## Implementation Steps
1. Add Redis connection pool
2. Implement session CRUD operations
3. Add TTL configuration
```

An agent can execute this plan directly. It's not pseudo-code—it's the program.

*[2026-02-06]*: Production example from GSD (Get Shit Done) project—PLAN.md files are not transformed into prompts, they ARE the prompts. The executor reads them verbatim. This imposes strict requirements: plans must be unambiguous, action-oriented, and include verification criteria. GSD uses semantic XML within markdown (`<action>`, `<verify>`, `<done>`) for Claude comprehension. Real-world validation of treating plans as executable specifications.

### Documentation IS Executable

When documentation is machine-readable and structured correctly, agents can use it directly:

```markdown
# api-spec.md

## Endpoint: POST /users
**Auth required**: Yes
**Rate limit**: 100/hour
**Parameters**:
- email: string, required, must be valid email
- name: string, required, 1-100 chars
```

This isn't just human documentation. An agent building a client can read this spec and generate correct implementation. An agent testing the API can verify the implementation matches the spec.

The spec is the source. Everything else derives from it.

---

## When to Apply This Model

### Good Fit

**Multi-agent systems**: When multiple agents need to coordinate, specs become the shared interface. Agents read the same specs humans do.

**Long-lived projects**: When you'll maintain code for months/years, specs as source code means future agents can understand the system by reading specs, not archaeologically excavating code.

**Generated code**: When agents generate implementation, the spec is what you maintain. The generated code is disposable.

**Complex domains**: When the "why" is as important as the "what," specs capture reasoning that code can't express.

### Poor Fit

**One-off scripts**: For throwaway automation, the mental overhead of treating specs as source code isn't worth it. Just write the script.

**Exploratory prototypes**: When you're still figuring out what to build, heavyweight specs slow you down. Prototype first, spec later.

**Stable, finished systems**: If the code is done and won't change, maintaining parallel specs is overhead without benefit.

---

## Living Artifacts: Documentation as Primary, Code as Derivative

*[2026-02-06]*: BMAD-METHOD demonstrates an extreme application of specs-as-source-code philosophy: **code is merely a downstream derivative of specifications**. In traditional development, source code is the truth and documentation is secondary (often outdated). BMAD inverts this completely.

### The Inversion

```
Traditional:                      BMAD Living Artifacts:

Source Code                       Documentation (PRD, Architecture, Stories)
    ↓                                 ↓
Documentation                     Source Code
(often outdated)                  (can be regenerated)
```

### Four-Phase Artifact Methodology

BMAD structures development as document-producing phases:

**1. Analysis Phase (optional)**
- Outputs: Product Brief, Research Summary
- Purpose: Problem definition before solution

**2. Planning Phase**
- Outputs: PRD (Product Requirements Document), UX Design
- Purpose: Requirements + user flows

**3. Solutioning Phase**
- Outputs: Architecture Document, Epics/Stories, Readiness Assessment
- Purpose: Technical design + implementation plan

**4. Implementation Phase**
- Outputs: Working Code, Code Reviews, Test Automation
- Purpose: Execute designs from phases 1-3

**Key insight:** Phases 1-3 produce *documents*. Phase 4 produces code *from* those documents. Each document becomes context for the next phase and audit trail for future changes.

### Compliance Value

For organizations under regulatory constraints (SOC 2, HIPAA, financial services), living artifacts transform compliance from post-hoc documentation to inherent process:

- **Versioned decisions**: Each document captures not just WHAT was built, but WHY
- **Audit trails**: Automated extraction of decision history for auditors
- **Requirement traceability**: Code links back to stories, stories link to architecture, architecture links to PRD
- **Change justification**: When code changes later, documents explain original intent

### When Code Changes

Traditional approach: Archaeologically excavate code to understand intent, make changes, hope nothing breaks.

Living artifacts approach:
1. Read original PRD/Architecture documents (context for why it works this way)
2. Update documents with new requirements or constraints
3. Regenerate code from updated specifications
4. Documents and code stay synchronized

### Adversarial Review Gates

BMAD implements quality gates between phases—an orchestrator critically examines each artifact before allowing progression. This prevents cascade failures:

- Incomplete PRD → flawed architecture → thousands of lines of wrong code
- Ambiguous user story → unclear implementation → bugs in production

**Quality gate checks:**
- Are completion criteria met?
- Are ambiguities resolved?
- Are dependencies documented?
- Can downstream phase execute from this artifact?

### Scale-Adaptive Artifact Depth

Not every project needs full four-phase documentation. BMAD adjusts:

**Quick Flow (3-step rapid path):**
```
/quick-spec → /dev-story → /code-review
```
Skips Product Brief, PRD, Architecture for bug fixes and small features.

**Full Planning Path (6-phase comprehensive):**
```
/product-brief → /create-prd → /create-architecture →
/create-epics-and-stories → /sprint-planning → [Dev Story Cycle]
```
Required for new products, multi-team efforts, compliance needs.

**Framework intelligence:** System recommends path based on project type. Medical diagnostic system → Full Planning. Bug fix → Quick Flow.

### Production Evidence

BMAD-METHOD (MIT-licensed, 34.5k GitHub stars, 19 releases) demonstrates this approach at scale:
- 68 workflows spanning full SDLC
- 26 specialized agents executing from documents
- Modular artifact system enabling reuse across projects
- Community adoption validates practical viability

### Living Artifacts vs. Traditional Specs

| Traditional Specs | Living Artifacts |
|------------------|------------------|
| Written after code (documentation) | Written before code (specification) |
| Code is source of truth | Documents are source of truth |
| Specs drift from reality | Code regenerated from specs |
| "Comments lie, code doesn't" | "Code is derivative, specs are canonical" |
| Manual synchronization required | Specs → Code generation enforces sync |

### Connections to Other Mental Models

**To [Context as Code](4-context-as-code.md):** Living artifacts are structured knowledge that agents execute—they're source code for behavior.

**To [Plan-Build-Review](../6-patterns/1-plan-build-review.md):** Living artifacts provide the research and plan documents that build phase executes from.

**To [Agent-as-Code](#):** BMAD extends "specs as source code" to agents themselves—agent definitions are markdown + YAML artifacts version-controlled like specifications.

### When to Use Living Artifacts

**Good fit:**
- Regulated industries requiring audit trails
- Long-lived systems where "why" matters as much as "what"
- Multi-team projects needing shared understanding
- Complex domains where upfront design prevents costly rework

**Overkill for:**
- Throwaway prototypes
- One-person hobby projects
- Well-understood, stable problem spaces
- Projects without compliance requirements

### Implementation Considerations

**Tooling requirements:**
- Agents capable of reading structured documents (PRDs, architecture specs)
- Quality gates between phases (validation before progression)
- Version control for documents (Git for markdown/YAML)
- Traceability system (linking code → stories → architecture → PRD)

**Process requirements:**
- Team buy-in on documents-first philosophy
- Discipline to update documents before code
- Quality standards for artifact completeness
- Review processes for specifications

### Open Questions

- How to handle exploratory coding where specs can't be written upfront?
- What level of spec detail is optimal? (too vague → unusable, too detailed → brittle)
- Can living artifacts work for frontend development, or only backend?
- How to migrate existing code-first projects to living artifacts approach?

---

## Common Pitfalls

### Spec Drift

**Problem**: Specs and implementation diverge. The spec says one thing, the code does another.

**Solution**: Test that implementation matches specs. Make spec updates part of your change workflow. If code changes without spec update, the PR is incomplete.

### Over-Specification

**Problem**: Specs become so detailed they're harder to maintain than code.

**Solution**: Specs should capture intent and constraints, not line-by-line implementation. Leave room for agent judgment.

### Vague Specs

**Problem**: Specs are too high-level to be executable. Agents can't generate correct code from them.

**Solution**: Think "testable." If you can't test whether the spec was followed, it's too vague. Add concrete examples and constraints.

---

## Connections

- **To [Context as Code](4-context-as-code.md)**: Living artifacts are structured context that determines agent behavior
- **To [Plan-Build-Review](../6-patterns/1-plan-build-review.md)**: The plan IS the source code, not scaffolding—BMAD's four-phase methodology demonstrates this at scale
- **To [Self-Improving Experts](../6-patterns/2-self-improving-experts.md)**: Expertise files are specs for agent behavior—same "spec as source code" philosophy
- **To [Knowledge Evolution](../7-practices/6-knowledge-evolution.md)**: Knowledge bases are specs for domain understanding
- **To [Prompt Structuring](../2-prompt/2-structuring.md)**: Structured prompts are executable specifications

## Sources

- Sean Grove's original articulation of specs-as-source-code mental model
- [BMAD-METHOD GitHub Repository](https://github.com/bmad-code-org/BMAD-METHOD) - Production implementation of living artifacts methodology
- [BMAD Scout Report](/.claude/.cache/research/external/bmad-method-scout-report.md) - Comprehensive analysis
