---
title: Progressive Disclosure Pattern
description: Loading context in tiers based on relevance to enable unlimited expertise within fixed context budgets
created: 2026-01-30
last_updated: 2026-01-30
tags: [patterns, context, progressive-disclosure, context-loading, expertise]
part: 2
part_title: Craft
chapter: 7
section: 7
order: 2.7.7
---

# Progressive Disclosure Pattern

A context management pattern that loads information in tiers based on relevance, enabling effectively unlimited expertise within fixed context budgets.

---

## Core Insight

*[2025-12-09]*: Context windows are finite, but knowledge bases are not. Progressive disclosure addresses this asymmetry by loading information in tiers—maintaining a semantic index of everything available while loading full content only when needed.

The pattern mirrors human cognition: encyclopedias are not memorized, but their tables of contents are. The index lives in working memory; the content fetches on-demand.

---

## How It Works

### The Three-Tier Model

Information loads in three tiers:

1. **Metadata first** — Names, descriptions, summaries (~50-200 characters per item)
2. **Full content on selection** — Complete documentation when explicitly chosen (~500-5,000 words)
3. **Detailed resources on-demand** — Supporting files, source code, references (unbounded)

```
┌─────────────────────────────────────────────────────┐
│ Context Window                                       │
│                                                     │
│ ┌─────────────────────────────────────────────────┐ │
│ │ Tier 1: Metadata Index (~1-5% of budget)        │ │
│ │ - Skill A: "Handles authentication flows"       │ │
│ │ - Skill B: "Manages database migrations"        │ │
│ │ - Skill C: "Coordinates multi-agent tasks"      │ │
│ └─────────────────────────────────────────────────┘ │
│                                                     │
│ ┌─────────────────────────────────────────────────┐ │
│ │ Tier 2: Activated Content (~10-30% of budget)   │ │
│ │ [Full Skill A documentation loaded]             │ │
│ └─────────────────────────────────────────────────┘ │
│                                                     │
│ ┌─────────────────────────────────────────────────┐ │
│ │ Tier 3: On-Demand Resources (fetched as needed) │ │
│ │ → Read tool for supporting files                │ │
│ │ → Grep tool for code examples                   │ │
│ └─────────────────────────────────────────────────┘ │
│                                                     │
│ ┌─────────────────────────────────────────────────┐ │
│ │ Remaining: Working space for task execution     │ │
│ └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

### The Semantic Index

The metadata layer creates a semantic index—the agent knows what expertise exists without loading it. When a task requires specific knowledge, the agent:

1. Scans the index for relevant capabilities
2. Activates the matching item (loads full content)
3. Fetches supporting resources as needed during execution

This creates "effectively unlimited" expertise: the index can reference thousands of items, but only active items consume significant context.

---

## Concrete Example: Claude Skills

Claude Skills demonstrate this pattern in production:

**Initial Load (Tier 1):**
```yaml
skills:
  - name: authentication-expert
    description: "Handles OAuth flows, JWT validation, session management"
    triggers: ["auth", "login", "token", "session"]

  - name: database-migrations
    description: "Schema changes, data migrations, rollback strategies"
    triggers: ["migration", "schema", "alter table"]

  - name: api-design
    description: "REST conventions, versioning, error responses"
    triggers: ["endpoint", "REST", "API design"]
```

~100 characters per skill. 10 skills = ~1,000 characters for complete awareness.

**Activation (Tier 2):**
When a task involves authentication, `authentication-expert` loads:

```markdown
# Authentication Expert

## OAuth 2.0 Flows
- Authorization Code: For server-side apps with secure storage
- PKCE: For SPAs and mobile apps without secure storage
- Client Credentials: For machine-to-machine communication

## JWT Validation Checklist
1. Verify signature using public key
2. Check `exp` claim for expiration
3. Validate `iss` and `aud` claims
4. Confirm `iat` is not in the future

## Session Management Patterns
[... 500-5,000 words of expertise ...]
```

**Resources (Tier 3):**
During execution, the skill references supporting files:

```
Read: /examples/oauth-implementation.py
Read: /configs/jwt-validation.yaml
Grep: "refresh_token" in /src/auth/
```

### Token Economics

| Approach | 10 Skills | 100 Skills | 1000 Skills |
|----------|-----------|------------|-------------|
| **Eager Loading** | 50k tokens | 500k tokens | 5M tokens (impossible) |
| **Progressive Disclosure** | ~6k tokens | ~7k tokens | ~8k tokens |

Progressive disclosure scales logarithmically; eager loading scales linearly.

---

## Implementation Patterns

### Pattern 1: Tool Descriptions as Metadata

Tool definitions naturally support progressive disclosure. The description serves as Tier 1; the tool's functionality provides Tier 2/3.

```yaml
tools:
  - name: search_documentation
    description: "Search internal documentation. Use for API references,
                  architecture decisions, and coding standards."
    # Full search capability available on invocation
```

### Pattern 2: Structured Indices with Descriptions

Build explicit index structures for large knowledge bases:

```markdown
## Available Expertise

| Domain | Trigger Keywords | Summary |
|--------|-----------------|---------|
| Security | auth, encrypt, OWASP | Authentication, encryption, vulnerability patterns |
| Performance | optimize, cache, N+1 | Profiling, caching strategies, query optimization |
| Testing | test, mock, coverage | Unit testing, integration testing, test design |

To activate: "Load [Domain] expertise"
```

### Pattern 3: Hierarchical Disclosure

For deep knowledge structures, use nested tiers:

```
Level 0: Category summaries
  └── Level 1: Section overviews
        └── Level 2: Full documentation
              └── Level 3: Source code references
```

Example traversal:
```
"What testing patterns exist?" → Level 0 (category list)
"Tell me about integration testing" → Level 1 (section overview)
"How do I mock external services?" → Level 2 (full documentation)
"Show me the mock implementation" → Level 3 (source code via Read tool)
```

### Pattern 4: Lazy Loading with Prefetch Hints

Optimize latency by prefetching likely next-tier content:

```yaml
skill:
  name: api-design
  description: "REST conventions, versioning, error responses"
  prefetch_hints:
    - high_probability: error-handling-patterns
    - medium_probability: rate-limiting-strategies
```

When `api-design` activates, the system can speculatively load `error-handling-patterns` since it correlates strongly.

---

## Trade-offs

### The Core Trade-off

**Slight latency on selection** (additional tool call to fetch full content) **for dramatic capacity gains** (100x more knowledge accessible).

### Detailed Analysis

| Dimension | Progressive Disclosure | Eager Loading |
|-----------|----------------------|---------------|
| **Initial latency** | Low (metadata only) | High (everything loads) |
| **Access latency** | Medium (fetch on select) | Zero (already loaded) |
| **Context utilization** | Efficient (~5-30%) | Full (often 80%+) |
| **Scalability** | Excellent | Poor |
| **Discoverability** | Good (via index) | Perfect |
| **Complexity** | Medium | Low |

### Hidden Costs of Eager Loading

Anthropic's GitHub MCP integration illustrates the eager loading trap:

> "tens of thousands of tokens" consumed just to make repositories and issues accessible

This pre-loads capability descriptions that may never be used, leaving less space for actual task work. Progressive disclosure would load repo metadata first (~500 tokens), then fetch specific repo details on-demand.

---

## When to Use Progressive Disclosure

### Good Fit

- **Large knowledge bases** where most content will not be needed for any single task
- **Multi-domain expertise** where the agent needs awareness but not full activation
- **Tight context budgets** where capability breadth is essential but space is limited
- **Dynamic capability selection** where the agent should choose expertise based on task requirements
- **Scalable systems** that may grow to hundreds or thousands of knowledge items

### Poor Fit

- **Small, static knowledge sets** where eager loading costs less than infrastructure complexity
- **Guaranteed access patterns** where exactly which content will be needed is known in advance
- **Latency-critical paths** where additional tool calls are unacceptable
- **Simple retrieval** where a single Read or Grep suffices
- **Complete context requirements** where partial knowledge causes more harm than full loading

---

## Anti-Patterns

### Excessive Metadata

**Problem:** Metadata descriptions become so detailed they approach full content size.

```yaml
# Anti-pattern: metadata too heavy
skill:
  name: auth
  description: "OAuth 2.0 implementation including authorization code flow
                with PKCE extension for public clients, JWT token validation
                with RS256 signature verification, refresh token rotation
                with sliding window expiration, session management using
                httpOnly secure cookies with SameSite=Strict..."
```

**Solution:** Keep metadata to 50-200 characters. Full details belong in Tier 2.

### Missing Index Updates

**Problem:** New knowledge items are added without updating the semantic index.

**Solution:** Index updates must be part of the content addition workflow. Automate where possible.

### Over-Eager Activation

**Problem:** Activating multiple knowledge items "just in case" defeats the purpose.

```
# Anti-pattern: loading everything anyway
Task: Fix authentication bug
Activated: auth-expert, database-expert, api-expert, testing-expert, security-expert
```

**Solution:** Activate only what the current task step requires. Re-evaluate activation needs at each phase.

---

## Connections

- **To [Context Fundamentals](../4-context/1-context-fundamentals.md):** Progressive disclosure operationalizes the "quality over quantity" principle—the index provides quality awareness, on-demand loading provides focused depth.
- **To [Context Loading vs. Accumulation](../4-context/3-context-patterns.md):** Progressive disclosure is a form of context loading—deliberately constructing context rather than passively accumulating it.
- **To [Orchestrator Pattern](3-orchestrator-pattern.md):** Orchestrators can use progressive disclosure to select which specialized agents to invoke, loading agent capabilities on-demand.
- **To [Tool Design](../5-tool-use/1-tool-design.md):** Tool descriptions serve as natural Tier 1 metadata; tool execution provides Tier 2/3 content.
- **To [Claude Code Skills](../10-practitioner-toolkit/1-claude-code.md):** Production implementation of progressive disclosure in a practitioner toolkit.

---

## Sources

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic on progressive disclosure as context strategy
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills) — Production implementation demonstrating three-tier loading
- [Simon Willison: Claude Skills](https://simonwillison.net/2025/Oct/16/claude-skills/) — Practitioner analysis of progressive disclosure benefits
