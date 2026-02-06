---
title: Workflow Coordination for Agents
description: Using structured metadata and persistent stores as coordination layers between agents
created: 2025-12-08
last_updated: 2026-02-06
tags: [practices, coordination, workflow, handoff, metadata, spec-files]
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

**Pattern observed in production:** GSD project (12K-star open source tool) uses must_haves to coordinate multi-day agent sessions. Each task defines success upfront, preventing scope creep and enabling objective completion verification.

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

---

## Source Examples

These patterns were extracted from real project configurations:

### KotaDB

- **[issue.md](../../appendices/examples/kotadb/.claude/commands/issues/issue.md)**: Issue creation with four-category labeling
- **[issue-relationships.md](../../appendices/examples/kotadb/.claude/commands/docs/issue-relationships.md)**: Machine-parseable relationship format
- **[pull_request.md](../../appendices/examples/kotadb/.claude/commands/git/pull_request.md)**: PR validation evidence sections
- **[commit.md](../../appendices/examples/kotadb/.claude/commands/git/commit.md)**: Conventional commits with meta-commentary detection
- **[prioritize.md](../../appendices/examples/kotadb/.claude/commands/issues/prioritize.md)**: Dependency-aware prioritization
