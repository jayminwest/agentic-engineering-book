---
title: Workflow Coordination for Agents
description: Using structured metadata and persistent stores as coordination layers between agents
created: 2025-12-08
last_updated: 2026-04-11
tags: [practices, coordination, workflow, handoff, metadata, spec-files, human-role, agent-management, institutional-design, quality-gate]
part: 2
part_title: Craft
chapter: 7
section: 5
order: 2.7.5
---

# Workflow Coordination for Agents

Agents collaborate better when they have a centralized, persistent store for coordination. The underlying pattern is **structured metadata that enables automated routing and state tracking**—GitHub is one accessible implementation, but the same principles apply to Linear, Jira, Notion, or even a local file-based system.

---

## The Underlying Pattern

What makes workflow coordination work isn't the specific tool—it's the structure:

1. **Canonical source for workflow decisions**: One place where task state lives
2. **Structured metadata categories**: Labels, tags, or fields that enable routing
3. **Machine-parseable relationships**: Explicit dependencies between work items
4. **Persistent artifacts**: Context that survives session boundaries

GitHub implements these through issues, labels, and PRs. Linear uses issues and projects. A local system might use a JSON file or SQLite database. The patterns transfer.

---

## GitHub as One Implementation

The following sections use GitHub as a concrete example. Adapt the patterns to your coordination system:

---

## The Core Principle

Use version control platform as the primary communication medium between agents.

Why this works:
- **Persistence**: Context survives session boundaries
- **Structure**: Issues, PRs, labels provide natural organization
- **Traceability**: Git history shows evolution of decisions
- **Human-readable**: Developers can follow along and intervene
- **Machine-parseable**: Agents can query via `gh` CLI

---

## Structured Metadata Categories

The key pattern: **define mandatory categories that enable automated routing**. Work items need enough structure that agents can parse and act on them without human intervention.

### Example: Four-Category Taxonomy

One effective taxonomy uses four categories—adapt the specifics to your domain:

| Category | Example Values | Purpose |
|----------|----------------|---------|
| **Component** | backend, frontend, database, api, testing | Route to relevant experts |
| **Priority** | critical, high, medium, low | Sequence work |
| **Effort** | small (<1d), medium (1-3d), large (>3d) | Scope estimation |
| **Status** | needs-investigation, blocked, in-progress, ready-review | Track state |

The categories matter more than the specific values. Your system might use:
- **Type** instead of Component (feature, bug, chore, refactor)
- **Urgency** instead of Priority (now, soon, later, someday)
- **Size** with story points instead of Effort

### Why This Enables Automation

With structured categories, agents can:
- **Route automatically**: "Component:database" → spawn database expert
- **Prioritize without judgment**: Sort by priority, then effort, then age
- **Track state machines**: Status labels enforce allowed transitions
- **Validate completeness**: Reject work items missing required categories

**Validation pattern** (GitHub example):
```bash
gh label list --limit 100 | grep -E "component:|priority:|effort:|status:"
```

This prevents drift and ensures consistency across agent sessions. The equivalent in other systems: query your API or database for the allowed values before creating work items.

---

## Work Item Relationships

The pattern: **explicit, machine-parseable relationships between work items** enable dependency-aware scheduling. Without these, agents treat every task as independent.

### Relationship Types

Six relationship types cover most scenarios:

| Type | Meaning | Agent Behavior |
|------|---------|----------------|
| **Depends On** | Requires another to complete first | Block until dependency resolves |
| **Blocks** | Other items waiting on this one | Prioritize to unblock downstream |
| **Related To** | Shares context, no hard dependency | Load as additional context |
| **Supersedes** | Replaces older work item | Close the superseded item |
| **Child Of** | Sub-task of a larger epic | Roll up completion to parent |
| **Follow-Up** | Created as result of another | Link for traceability |

### Machine-Parseable Format

Whatever your system, relationships need consistent format. Example using markdown:

```markdown
## Relationships

- **Depends On**: #25 (API key generation) - Required for auth middleware
- **Blocks**: #74, #116 (symbol extraction, search) - Provides AST foundation
- **Related To**: #42 (user settings) - Shares config patterns
```

The specifics matter less than consistency. Agents can parse:
- Linear's built-in relations
- Jira's "is blocked by" / "blocks" links
- A JSON field in a local database
- Markdown sections with predictable format

Agents use these to build dependency graphs for prioritization—identifying unblocked high-leverage items (those that unblock others).

---

## The Issue-to-PR Workflow

```
┌─────────────┐
│ Issue Created │
│ (with labels) │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Plan Written │
│ docs/specs/  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Branch Created │
│ from issue #   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Implementation │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ PR Submitted │
│ refs issue   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Review/Merge │
└─────────────┘
```

### Branch naming from issue

```bash
# Fetch issue metadata
gh issue view <issue-number> --json number,title,labels

# Map labels to branch type:
# bug label → bug/
# enhancement/feature → feature/
# chore/maintenance → chore/

# Format: <type>/<issue-number>-<slug>
# Example: feature/123-add-user-authentication
```

### PR references issue

```markdown
## Summary
Brief description of changes

## Changes
- Bullet points of what changed

## Test Plan
- How to verify

Closes #123
```

The `Closes #123` creates the link back to the originating issue.

---

## Spec Files as Persistent Context

Plans live in `docs/specs/` with issue numbers in filenames:

```
docs/specs/feature-123-user-auth.md
docs/specs/bug-456-token-refresh.md
```

This creates traceability:
- Issue #123 → `docs/specs/feature-123-*.md` → `feature/123-*` branch → PR

Future agents can reconstruct context by following these breadcrumbs.

### Spec Files Beat Accumulated Context

*[2025-12-09]*: **Rather than passing large contexts between agents in messages, write findings to disk as spec files.** Each agent reads the same spec independently, avoiding context bloat and enabling parallel access to shared state.

This pattern has several advantages:

1. **No context accumulation**: Messages stay focused, avoiding the "wall of text" problem in multi-agent handoffs
2. **Parallel access**: Multiple agents can read the same spec simultaneously without coordination
3. **Version control**: Specs are tracked in Git, providing history and rollback
4. **Session survival**: Context persists across agent restarts and crashes
5. **Human readability**: Developers can inspect and override agent decisions

**Anti-pattern** (context passing):
```
Orchestrator → Build Agent: "Based on the analysis from Scout Agent (3000 words),
and the design from Planning Agent (2000 words), implement feature X..."
```

**Better** (spec-based):
```
Orchestrator → Build Agent: "Read .claude/.cache/specs/feature-123-spec.md
and implement according to the design section"
```

The spec file becomes the single source of truth. Agents coordinate by reading and writing to it, not by accumulating context in conversation threads.

---

## Atomic Operations and State Persistence

*[2026-02-06]*: Multi-session workflows require three complementary patterns: atomic commits for independent revertability, structured state files for session survival, and goal-backward verification to prevent drift. These patterns emerged from production agent workflows where sessions span days and context resets between executions.

### Atomic Commits as Revertability Strategy

**Per-task commits enable independent rollback without cascading failures.** Each logical task becomes a single commit, allowing git-bisect debugging and surgical reverts.

**Commit pattern:**
```
One feature = One commit
- Single logical change
- Complete functionality (tests included)
- Independent of other commits
- Bisectable (repo works at every commit)
```

**Why this matters:**
- **Surgical reversion**: Roll back feature B without affecting feature A
- **Bisect-friendly**: `git bisect` identifies breaking commits quickly
- **Clear history**: Each commit represents decision point
- **Agent-friendly**: New sessions understand changes via commit messages

**Anti-pattern** (accumulated commits):
```bash
# Bad: Multiple features in single commit
git commit -m "Add auth, fix database, update tests, refactor utils"
```

**Better** (atomic):
```bash
git commit -m "feat: add JWT authentication middleware"
git commit -m "test: add integration tests for auth flow"
git commit -m "fix: handle token refresh edge case"
```

Each commit stands alone. Reverting authentication doesn't break the database changes that followed.

### Structured State Files

**STATE.md captures current position, accumulated decisions, and blockers.** This pattern provides session-independent context—agents can resume work after interruptions without reconstructing history from chat logs.

**STATE.md structure:**
```markdown
# Current State

## Position
- **Current Phase**: Implementation (Day 2 of 3)
- **Last Completed**: Database schema migration (#45)
- **Next Task**: API endpoint implementation (#52)
- **Session**: 3 of estimated 5

## Accumulated Decisions
- Using PostgreSQL over MongoDB (performance requirements)
- JWT tokens expire after 24h (security review approved)
- Rate limiting: 100 req/min per user (infrastructure constraints)

## Active Blockers
- [ ] Waiting on API key from external service (ETA: tomorrow)
- [ ] Database credentials for staging environment
- [x] ~~Design review approval~~ (completed 2026-02-05)
```

**Session survival benefits:**
1. **Resume without reconstruction**: New agent reads STATE.md, continues immediately
2. **Decision tracking**: Why choices were made, preventing re-litigation
3. **Blocker visibility**: Clear obstacles, prevents spinning on known issues
4. **Progress measurement**: Phase tracking shows velocity

**Update pattern:**
- Write STATE.md after completing each task
- Commit with implementation changes (atomic commit includes state update)
- Move completed blockers to decisions section with timestamps

### Goal-Backward Verification

**Define success criteria before execution, verify after completion.** The must_haves pattern prevents drift by establishing measurable goals that implementations must satisfy.

**must_haves structure:**
```yaml
must_haves:
  truths:
    - "Authentication middleware rejects expired tokens"
    - "Rate limiter enforces 100 req/min per user"
    - "Token refresh extends expiration by 24h"

  artifacts:
    - "src/middleware/auth.ts with JWT validation"
    - "tests/integration/auth.test.ts with 95%+ coverage"
    - "docs/api/authentication.md with examples"

  key_links:
    - "Issue #45 closed with validation evidence"
    - "PR merged to main with approvals"
```

**Verification workflow:**
1. **Before execution**: Write must_haves in spec file
2. **During implementation**: Reference must_haves to stay on track
3. **After completion**: Check each truth/artifact/link explicitly
4. **Validation**: All must_haves satisfied = task complete

**Drift prevention:**
Without goal-backward verification, implementations expand scope. With must_haves, agents have clear stopping criteria.

**Example verification:**
```markdown
## Verification Results

✅ Truth: Expired tokens rejected (test: auth.test.ts:45)
✅ Truth: Rate limiter enforces limits (test: rate-limit.test.ts:23)
✅ Artifact: auth.ts created (127 lines, reviewed)
✅ Artifact: tests written (96.2% coverage)
✅ Key Link: Issue #45 closed (PR #67 merged)

Status: All must_haves satisfied. Task complete.
```

**Pattern observed in production:** GSD project (open source workflow harness) uses must_haves to coordinate multi-day agent sessions. Each task defines success upfront, preventing scope creep and enabling objective completion verification. See [Workflow Harnesses: GSD as Production Example](#workflow-harnesses-gsd-as-production-example) for the full case study.

---

## Validation Evidence in PRs

PRs include structured validation sections:

```markdown
## Validation Evidence

### Level: 2 (Integration)

**Commands Run**:
- ✅ `bun run lint`
- ✅ `bun test --filter integration` - 42 tests passed
- ✅ `bun run build`

**Real-Service Evidence**:
- Supabase: SELECT returned expected rows
- Auth flow: Token refresh successful
```

Three validation levels:
1. **Docs-only**: Linting, formatting
2. **Integration**: Real service tests
3. **Full E2E**: Complete user flows

---

## Output Format Discipline

Agents must not leak reasoning into artifacts. Forbidden patterns:

- "Let me..." / "I'll..."
- "Based on the..." / "Looking at..."
- "Great!" / "Successfully!"
- "First, I'll check..."

**Bad** (meta-commentary):
```
Let me create a branch for this issue. Based on the labels,
I'll use the feature/ prefix. Great, the branch was created successfully!
```

**Good** (artifact only):
```
Branch: feature/123-add-user-auth
From: develop (abc1234)
Issue: #123 - Add user authentication
```

Production artifacts (commits, PRs, issues) should contain decisions, not reasoning.

---

## Specialized Agents for GitHub Operations

Delegate GitHub operations to focused agents:

| Agent | Responsibility |
|-------|----------------|
| **GitHub Communicator** | Issue creation, commenting, label management |
| **Issue Prioritizer** | Analyze dependencies, recommend next tasks |
| **Meta-Agent Evaluator** | Ensure agent instructions stay aligned |

This reduces context contamination in the main agent and ensures consistency.

---

## Metrics to Track

### Velocity
- Commit frequency: >5/day indicates healthy pace
- PR turnaround: <2 days suggests efficient review
- Feature completion: Issues closed vs. opened

### Quality
- CI failure rate: <5% suggests robust gates
- Post-release bugs: <1% indicates effective testing

### Agent Collaboration
- Context handoff success via GitHub
- Pattern consistency across sessions
- Knowledge accumulation in issues/PRs

---

## Alternative: Database-Backed Communication

*[2025-12-08]*: For faster-moving workflows, expose CRUD operations on a shared communications store via MCP tools or function calling. This provides:

- Lower latency than GitHub API
- More flexible schema
- Better suited for real-time coordination

GitHub remains better for:
- Human oversight and intervention
- Long-lived context (issues that span weeks)
- Integration with existing dev workflows

The patterns (labels, relationships, validation evidence) apply to both.

---

## Workflow Harnesses: GSD as Production Example

*[2026-03-20]*: A distinct category is emerging between raw coding agents and full workspace managers: **workflow harnesses** — meta-prompting systems that structure how a human-AI pair approaches complex projects through context engineering, phase decomposition, and state persistence.

[GSD (Get Shit Done)](https://github.com/gsd-build/get-shit-done) is the clearest example. Built as a Claude Code skill ecosystem, GSD addresses "context rot" — the quality degradation that occurs as context fills during extended sessions — through a five-stage lifecycle:

| Phase | Purpose | Agent Role |
|-------|---------|------------|
| **Discuss** | Capture decisions and gray areas | Interactive clarification |
| **Plan** | Research approach, create XML-structured task plans | Specialized planner + plan-checker |
| **Execute** | Run tasks in parallel waves with fresh 200K contexts | Per-task executors (isolated) |
| **Verify** | User acceptance testing with automated diagnosis | Verification agent |
| **Ship** | Create PRs from verified work | Shipping agent |

### Key Design Decisions

**Fresh context per executor.** Each task executor receives a clean 200K-token context loaded only with the relevant plan, project files, and state. This is the "boot fresh agents" pattern (see [Context Strategies](../4-context/2-context-strategies.md#current-practice-boot-fresh-agents)) implemented systematically — rather than one long conversation degrading over time, GSD spawns specialized executors that never experience context rot.

**Persistent state files as coordination layer.** Four markdown files form the persistent substrate:

- `PROJECT.md` — Project identity, tech stack, constraints
- `REQUIREMENTS.md` — What to build, acceptance criteria
- `ROADMAP.md` — Phases and milestones
- `STATE.md` — Current position, decisions, blockers

These map directly to the [structured state files](#structured-state-files) and [spec files as persistent context](#spec-files-as-persistent-context) patterns described earlier. Agents read state on boot, write state on completion.

**Wave-based parallelism.** Tasks are dependency-grouped into waves. Independent tasks within a wave execute simultaneously across fresh executor contexts, each producing atomic commits. This combines dependency-aware scheduling (see [Work Item Relationships](#work-item-relationships)) with the [atomic commits](#atomic-commits-as-revertability-strategy) pattern.

**XML prompt formatting for plans.** Every plan uses structured XML with action steps, file lists, and verification criteria — giving executors unambiguous, machine-parseable instructions rather than natural language descriptions.

### Category Distinction

Workflow harnesses occupy a specific niche in the tooling hierarchy:

```
Scale and abstraction:
  Raw coding agent       → Single-session, ad-hoc interaction
  Workflow harness (GSD) → Multi-phase structured lifecycle, context-engineered
  Workspace manager      → 20+ agent infrastructure (worktrees, merge queues, supervision)
```

Workspace managers (see [Multi-Agent Workspace Managers](../9-practitioner-toolkit/5-multi-agent-workspace-managers.md)) solve infrastructure problems — merge conflicts, agent supervision, work attribution across dozens of simultaneous branches. Workflow harnesses solve *workflow* problems — context degradation, phase discipline, verification gates, state persistence across sessions.

The two are complementary, not competing. A workspace manager could orchestrate multiple GSD-structured workflows in parallel.

---

## The Human Role in Agent-Managed Workflows

*[2026-04-11]*: As agents take over execution of hours-long workflows, the human role migrates upstream. Ethan Mollick's practitioner-research synthesis identifies three distinct responsibilities that remain human in agent-managed work — and notably, execution is not among them (Mollick, "The Shape of the Thing," 2026).

**Strategy and direction:** Practitioners write roadmaps, scope work, and set success criteria before agents begin. This is the specification layer — the work that happens above the workflow harness. In GSD terms, the Discuss phase is where this work lives; in GitHub-based coordination, it is the original issue description and its acceptance criteria. The quality of this work determines the quality of everything downstream. This is the inverse of the old model where strategy was the fast part and execution was the bottleneck — when implementation is automated, specification becomes the scarce resource (see [Design as Bottleneck](../8-mental-models/6-design-as-bottleneck.md)).

**Quality gate:** Practitioners review finished work, not intermediate steps. This is structurally different from in-loop collaboration. The StrongDM case makes this concrete: "Code must not be reviewed by humans" applies to pull-request review of individual commits; the human quality gate operates at the finished-product level — does this work satisfy the roadmap? The GSD lifecycle's Verify phase is a quality gate in this sense. Reviewing finished work at the product level requires different judgment than reviewing code at the line level. The acceptance criteria in the strategy layer feed directly into the quality-gate layer.

**Institutional design:** Practitioners decide how to organize work around autonomous agents — which roles agents fill, which quality gates are human-operated, how exceptions escalate, and what norms govern the whole system. Mollick argues that organizations experimenting with agent-based work structures now are defining the default norms that later adopters will inherit. This is not a technical problem; it is an organizational one. The design of the harness (which this chapter covers) is a subset of institutional design. The larger question is how the organization positions itself relative to autonomous execution. StrongDM's explicit rules ("Code must not be reviewed by humans") are an example of institutional design operationalized at team scale — see [Software Factories](../8-mental-models/7-software-factories.md) for the full case.

### Where These Roles Sit Relative to the Coordination Layer

```
┌─────────────────────────────────────────────────┐
│  INSTITUTIONAL DESIGN                            │
│  (What is the organization's agent strategy?)   │
│                                                 │
│  ┌───────────────────────────────────────────┐  │
│  │  STRATEGY AND DIRECTION                   │  │
│  │  (Roadmap, scope, acceptance criteria)    │  │
│  │                                           │  │
│  │  ┌─────────────────────────────────────┐  │  │
│  │  │  WORKFLOW HARNESS                   │  │  │
│  │  │  (Agent-to-agent coordination,      │  │  │
│  │  │   state management, Git layer)       │  │  │
│  │  │                                     │  │  │
│  │  │  ┌─────────────────────────────┐    │  │  │
│  │  │  │  EXECUTION                  │    │  │  │
│  │  │  │  (Agents)                   │    │  │  │
│  │  │  └─────────────────────────────┘    │  │  │
│  │  └─────────────────────────────────────┘  │  │
│  │                                           │  │
│  │  QUALITY GATE                             │  │
│  │  (Finished-product review)                │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### The Phase-Transition Framing

This taxonomy describes a structural shift, not a gradual evolution. Mollick frames it as a phase transition from co-intelligence (human and AI in iterative back-and-forth) to agent management (human above the harness, agent executing autonomously below it). Prior collaborative models — centaur (strategic division of labor between human and AI) and cyborg (deep blending, no clear handoff) — both keep the human in-loop during execution (Mollick, "Centaurs and Cyborgs on the Jagged Frontier," 2023). The agent management phase removes the human from execution entirely and reassigns them to the three upstream roles above.

This is relevant to practitioners who have internalized the centaur/cyborg collaboration model: those skills (real-time iteration, jagged-frontier allocation) remain useful for tasks within a single session but do not describe the human's role in a multi-agent workflow harness operating over hours or days.

### Implications for Practitioners

- **Specification quality is the upstream-facing skill the harness demands.** The most valuable skill to develop is not prompting (in-loop collaboration) but writing roadmaps and acceptance criteria that agents can execute without clarification.
- **Quality-gate design requires defining "finished product" before agents begin.** Vague quality gates produce ambiguous review decisions; explicit acceptance criteria in the strategy layer feed directly into the quality-gate layer.
- **Institutional design is the practitioner's highest-leverage contribution.** Technical decisions about harness architecture (GitHub vs. database-backed, spec-file format, commit strategy) are downstream of organizational decisions about which work agents own and which quality gates remain human.
- **The three roles are layered, not sequential.** Institutional design is ongoing; strategy and direction cycle per project; quality gates operate continuously as agents complete work.

**Sources:**
- Mollick, E. "The Shape of the Thing." One Useful Thing, ~2026-03-12. https://www.oneusefulthing.org/p/the-shape-of-the-thing
- Mollick, E. "Centaurs and Cyborgs on the Jagged Frontier." One Useful Thing, 2023-09-16. https://www.oneusefulthing.org/p/centaurs-and-cyborgs-on-the-jagged

---

## Questions to Explore

- How do agents handle GitHub API rate limiting during parallel operations?
- What's the escalation path when a PR is stuck in review?
- How do you detect and prevent issue relationship cycles?
- When should agents comment on issues vs. create new ones?

---

## Connections

- **Multi-Agent Orchestration**: GitHub as coordination layer for multi-agent systems
- **[Expert Swarm Pattern](../6-patterns/8-expert-swarm-pattern.md)**: Architectural pattern enabling workflow coordination at scale. Production evidence: 10 agents, 11 tasks, 4 minutes, 3000+ lines. Spec-as-artifact and TeammateTool coordination primitives support Expert Swarm execution.
- **[Context Management](../4-context/_index.md)**: Issues/PRs as persistent context stores
- **[Production Concerns](4-production-concerns.md)**: GitHub workflows as part of deployment pipeline
- **[Multi-Agent Workspace Managers](../9-practitioner-toolkit/5-multi-agent-workspace-managers.md)**: GSD workflow harnesses complement workspace managers — harnesses structure workflow within a project, workspace managers coordinate infrastructure across agents
- **[Design as Bottleneck](../8-mental-models/6-design-as-bottleneck.md)**: The upstream shift in the constraint that drives the human-role taxonomy in agent-managed workflows — when implementation is automated, specification becomes the scarce resource.

---

## Source Examples

These patterns were extracted from real project configurations:

### KotaDB

- **[issue.md](../../appendices/examples/kotadb/.claude/commands/issues/issue.md)**: Issue creation with four-category labeling
- **[issue-relationships.md](../../appendices/examples/kotadb/.claude/commands/docs/issue-relationships.md)**: Machine-parseable relationship format
- **[pull_request.md](../../appendices/examples/kotadb/.claude/commands/git/pull_request.md)**: PR validation evidence sections
- **[commit.md](../../appendices/examples/kotadb/.claude/commands/git/commit.md)**: Conventional commits with meta-commentary detection
- **[prioritize.md](../../appendices/examples/kotadb/.claude/commands/issues/prioritize.md)**: Dependency-aware prioritization

### Mollick Practitioner Research

- Mollick, E. "The Shape of the Thing." One Useful Thing, ~2026-03-12. https://www.oneusefulthing.org/p/the-shape-of-the-thing
- Mollick, E. "Centaurs and Cyborgs on the Jagged Frontier." One Useful Thing, 2023-09-16. https://www.oneusefulthing.org/p/centaurs-and-cyborgs-on-the-jagged
  - Note: BCG 758-consultant study and Dell'Acqua recruiter accuracy research are cited by Mollick in these articles; cite Mollick as secondary source, not primary.
