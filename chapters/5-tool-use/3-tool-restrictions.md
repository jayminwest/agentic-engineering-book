---
title: Tool Restrictions and Security
description: Using tool access controls as security boundaries and capability constraints
created: 2025-12-10
last_updated: 2026-01-17
tags: [security, restrictions, permissions, tool-access, safety]
part: 1
part_title: Foundations
chapter: 5
section: 3
order: 1.5.3
---

# Tool Restrictions and Security

Tool restrictions aren't just about capability—they're security boundaries. Treat tool access like production IAM: deny-all by default, allowlist only what each agent needs.

---

## Tool Restrictions as Security Boundaries

*[2025-12-09]*: In multi-agent systems, tool restrictions aren't just about capability—they're security boundaries. Treat tool access like production IAM: deny-all by default, allowlist only what each subagent needs.

**Principle**: Each subagent should have the minimum tool set required for its role. This isn't just defense-in-depth—it also helps the agent stay focused on its domain by reducing distraction from irrelevant capabilities.

**Common Patterns**:

| Role | Tools | Rationale |
|------|-------|-----------|
| Reviewer/Analyzer | Read, Grep, Glob | Read-only; can't accidentally modify files |
| Test Runner | Bash, Read, Grep | Execute tests and read results; no file editing |
| Builder/Implementer | Read, Edit, Write, Grep, Glob | Full modification access for implementation |
| Orchestrator | Task, Read, Glob | Routes work, has minimal direct access |
| Scout/Explorer | Read, Grep, Glob, WebFetch | Discovery only, no modification |

**Implementation**: In Claude Agent SDK, configure via YAML frontmatter (`tools: [Read, Grep, Glob]`) or the programmatic `tools` array. Filesystem definitions in `.claude/agents/*.md` make permissions visible and auditable.

**Anti-Pattern**: Giving all agents full tool access "for flexibility." This is the fastest path to unsafe autonomy. Instead:
- Require explicit confirmation for sensitive actions (git push, infrastructure changes)
- Restrict agents to relevant directories when possible
- Log tool usage for auditability

**Production Lesson**: Permission sprawl compounds. Start restrictive and expand only when you hit actual blockers. It's much easier to grant additional permissions than to clean up after an agent with too much access does something unexpected.

**See Also**:
- [Orchestrator Pattern: Capability Minimization](../7-patterns/3-orchestrator-pattern.md#capability-minimization) — How tool restriction becomes an architectural forcing function for delegation

**Sources**: [Subagents in the SDK - Claude Docs](https://platform.claude.com/docs/en/agent-sdk/subagents), [Claude Agent SDK Best Practices](https://skywork.ai/blog/claude-agent-sdk-best-practices-ai-agents-2025/), [Best practices for Claude Code subagents](https://www.pubnub.com/blog/best-practices-for-claude-code-sub-agents/)

---

## MCP Tool Declarations in Frontmatter

*[2025-12-09]*: MCP (Model Context Protocol) tools extend agent capabilities beyond native tools. Declaring them in YAML frontmatter follows a consistent pattern across projects.

**Naming Convention**: `mcp__<server>__<tool>`
- Double underscores separate the three components
- Server names can include hyphens: `mcp__firecrawl-mcp__firecrawl_scrape`
- Examples:
  - `mcp__playwright__browser_navigate` (browser automation)
  - `mcp__supabase__execute_sql` (database operations)
  - `mcp__kotadb__search_code` (custom code search)

**Two Frontmatter Fields** (context-dependent):
- **`tools:`** — Used in agent definitions to declare capabilities
- **`allowed-tools:`** — Used in commands to restrict available tools

**Mixed Declarations**: MCP tools combine naturally with native tools:
```yaml
tools: mcp__playwright__browser_click, mcp__firecrawl-mcp__firecrawl_scrape, Write, Read, Edit
```

**Role-Based MCP Assignment**: Different agents get different MCP tool subsets:

| Role | MCP Tools | Purpose |
|------|-----------|---------|
| Scout | `search_code`, `list_recent_files` | Read-only exploration |
| Builder | `search_code`, `analyze_change_impact` | Implementation support |
| Validator | `browser_navigate`, `browser_snapshot`, `browser_click` | UI verification |
| Scraper | `firecrawl_scrape`, `firecrawl_search` | Documentation fetching |

**Permission Patterns at Settings Level**: Glob patterns control MCP access without per-tool whitelisting:
```json
{
  "permissions": { "allow": ["mcp__kotadb__*", "mcp__playwright__*"] },
  "enableAllProjectMcpServers": true
}
```

**Multi-Instance Pattern**: Some projects separate staging and production:
- `mcp__kotadb-staging__search_code`
- `mcp__kotadb-production__search_code`

**Gap**: Frontmatter only references tools by name—server configuration (endpoints, auth, schemas) happens elsewhere (`.mcp.json`, environment variables, or external config). The declaration pattern is separate from the instantiation pattern.

**See Also**:
- [Claude Code: Subagent System](../10-practitioner-toolkit/1-claude-code.md#subagent-system) — How MCP tools integrate with subagent definitions
- [Orchestrator Pattern: Tool Assignment](../7-patterns/3-orchestrator-pattern.md#tool-assignment) — Role-based MCP tool assignment in multi-agent workflows

---

## Wildcard Permission Patterns

*[2026-01-11]*: Claude Code 2.1.0 introduced wildcard pattern matching for Bash tool permissions. This enables more flexible permission policies that reduce prompt fatigue while maintaining security boundaries.

**Syntax:** `Bash(<pattern>)` where `*` matches any characters

**Examples:**

| Pattern | Matches | Use Case |
|---------|---------|----------|
| `Bash(npm *)` | `npm install`, `npm run build`, `npm test` | Package management |
| `Bash(git *)` | `git status`, `git commit`, `git push` | Version control |
| `Bash(docker compose *)` | `docker compose up`, `docker compose down` | Container orchestration |
| `Bash(pytest *)` | `pytest tests/`, `pytest -v` | Test execution |
| `Bash(make *)` | `make build`, `make clean`, `make deploy` | Build automation |

**Configuration in settings.json:**

```json
{
  "permissions": {
    "allow": [
      "Bash(npm *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(pytest *)"
    ],
    "deny": [
      "Bash(git push *)",
      "Bash(rm -rf *)"
    ]
  }
}
```

**Security Considerations:**

Wildcards expand the permission surface. Apply least-privilege principles:

- **Prefer specific patterns:** `Bash(npm run test)` over `Bash(npm *)`
- **Combine allow and deny:** Allow broad patterns, deny dangerous subsets
- **Test coverage:** Verify patterns match intended commands before production use

**Anti-Pattern: Over-Broad Wildcards**

```json
{
  "permissions": {
    "allow": ["Bash(*)"]
  }
}
```

This effectively disables Bash permission prompts entirely—equivalent to `--dangerously-skip-permissions` for shell commands. Defeats the purpose of permission controls.

**Sources:** [Claude Code Changelog 2.1.0](https://code.claude.com/docs/en/changelog)

---

## Permission Bypass Vulnerabilities

*[2026-01-17]*: Security fixes in Claude Code 2.1.6-2.1.7 revealed attack surfaces in permission patterns that warranted documentation.

### Line Continuation Injection

Shell allows command continuation with backslash. This can bypass single-line permission checks:

```bash
# Approved permission: Bash(git add)
# Attack vector:
git add \
&& rm -rf /  # Continuation executes without restriction
```

**Mitigation:** Claude Code 2.1.7 validates across line continuations, treating the entire multi-line command as a single unit for permission matching.

### Glob Expansion Escapes

Wildcard patterns can expand beyond intended scope:

```bash
# Approved: Bash(rm temp/*)
# Risk: rm temp/* expands to include unexpected files
#       if temp/ contains symlinks or unexpected entries
```

**Mitigation:** Prefer explicit paths over wildcards for destructive operations. Filesystem state at execution time determines actual expansion.

### Best Practices for Secure Permissions

1. **Prefer exact matches** over wildcards: `Bash(git status)` not `Bash(git *)`
2. **Test patterns in isolation** before production deployment
3. **Layer defenses**: combine allow-lists with explicit deny-lists
4. **Audit filesystem separately** from permission string validation
5. **Log all permission decisions** for post-incident analysis

### Testing Permission Enforcement

Before trusting a permission pattern, verify behavior in a sandboxed environment:

```python
# Test script: verify permission actually blocks
test_cases = [
    ("git add .\n&& rm -rf /", "should_block"),  # line continuation
    ("rm ../outside/*", "should_block"),          # path traversal
    ("git status", "should_allow"),               # exact match
]
```

Run permission tests in isolation before production use. Permission validation and filesystem state are separate concerns—test both.

---

## Leading Questions

- How do you test that tool restrictions are actually enforced?
- What happens when an agent needs temporary elevated permissions?
- How do you handle tool access in development vs. production?
- When should tools fail loudly vs. silently when permission is denied?
- How do you audit tool usage patterns to detect permission issues?

---

## Connections

- **To [Tool Selection](2-tool-selection.md):** Restrictions affect what's selectable
- **To [Scaling Tools](4-scaling-tools.md):** MCP deployment patterns intersect with security
- **To [Orchestrator Pattern](../7-patterns/3-orchestrator-pattern.md):** Delegation as security enforcement
- **To [Production Concerns](../8-practices/4-production-concerns.md):** Security in production environments
