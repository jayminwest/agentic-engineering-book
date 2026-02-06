---
title: Skills and Meta-Tools
description: How skills blur the boundary between tools and prompts through progressive context disclosure
created: 2025-12-10
last_updated: 2026-02-05
tags: [skills, meta-tools, progressive-disclosure, specialization]
part: 1
part_title: Foundations
chapter: 5
section: 5
order: 1.5.5
---

# Skills and Meta-Tools

Skills blur the boundary between tools and prompts. They're not actions an agent can take, but temporary behavioral modifications that change how the agent reasons about specialized domains.

---

## Skills as Meta-Tools

*[2025-12-09]*: Skills blur the boundary between tools and prompts. They're not actions an agent can take, but temporary behavioral modifications that change how the agent reasons about specialized domains.

**The Meta-Tool Pattern**: The "Skill" tool (capital S) acts as a dispatcher to individual skills. When invoked, a skill modifies two contexts simultaneously:
- **Reasoning context**: Injects temporary behavioral instructions
- **Execution context**: Applies tool permission overrides

This creates specialized agent modes without forking the entire agent configuration.

**Three Key Differences from Traditional Tools**:

| Aspect | Traditional Tools | Skills |
|--------|------------------|--------|
| **Mechanism** | Direct actions (read file, call API) | Injected instructions + permission changes |
| **Selection** | Algorithmic matching (name/parameters) | LLM reasoning about descriptions |
| **Token Cost** | ~100 tokens per invocation | ~1,500+ tokens per invocation |
| **Purpose** | Execute specific operations | Specialize agent behavior |

**The Trade**: Skills trade token overhead for contextual specialization. Instead of front-loading every possible skill instruction into the base prompt, skills use progressive disclosure—the agent discovers capabilities through metadata, then loads full instructions only when needed.

**Generalization Beyond Claude**: The pattern isn't Claude-specific. Any agentic system can implement progressive disclosure:
1. **Discovery layer**: Short metadata about available capabilities
2. **Activation layer**: Full instructions loaded on-demand
3. **Resource layer**: Detailed references and examples (optional)

This balances discoverability (agent knows what's possible) with context efficiency (doesn't pay token cost until invoked).

**Example Flow**:
```
1. Agent sees skill metadata: "Python debugging skill available"
2. Agent encounters Python bug, invokes Skill tool with "python-debugging"
3. System injects debugging instructions + enables relevant tools (Bash, Read)
4. Agent now reasons with debugging strategies in context
5. After task completion, injected instructions expire
```

**Why This Matters**: Skills represent a third category beyond "tools" and "prompts":
- **Tools**: What the agent can *do* (read files, make API calls)
- **Prompts**: How the agent *thinks* (general reasoning patterns)
- **Skills**: Domain-specific *thinking modes* (temporary reasoning specialization)

**In Practice**: Claude Code implements this with a skills library—data analysis, git operations, debugging, etc. Each skill packages domain expertise as injectable context rather than requiring separate specialized agents.

**See Also**:
- [Context: Progressive Disclosure](../4-context/3-context-patterns.md#progressive-disclosure-pattern) — Managing context window through layered information
- [Claude Code: Skills System](../9-practitioner-toolkit/1-claude-code.md#skills-system) — Concrete implementation in production
- [Prompt](../2-prompt/_index.md) — Model-invoked vs user-invoked patterns for skill activation

**Sources**: [Claude Code Skills Docs](https://code.claude.com/docs/en/skills), [Skills Deep Dive](https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/), [Skills Explained](https://www.claude.com/blog/skills-explained)

---

## Context Contracts for Agent Capability Declaration

*[2026-02-05]*: Advanced .claude/ implementations use declarative JSON schemas to declare agent input requirements and output scopes. This enables pre-spawn validation and scope enforcement, extending the progressive disclosure pattern from runtime to orchestration design.

### The Pattern

Agents declare context contracts in structured metadata:

```json
{
  "inputs": {
    "spec_file": { "type": "path", "required": true },
    "expertise": { "type": "path", "required": false },
    "memory": { "type": "context", "required": false }
  },
  "outputs": {
    "allowed_modifications": ["src/**/*.ts", "docs/**/*.md"],
    "forbidden_patterns": ["**/node_modules/**", "**/.git/**"]
  }
}
```

### Three Validation Gates

**Pre-Spawn Validation:**
- Verify required context (spec_file, expertise) exists before spawning agent
- Prevents agent failures from missing inputs
- Reduces wasted tokens on doomed agent invocations

**Scope Enforcement:**
- Hooks validate file modifications against `allowed_modifications` globs
- Block writes outside declared scope
- Creates machine-readable capability boundaries

**Registry Generation:**
- Auto-generate agent capability catalog from contracts
- Enables programmatic agent discovery ("which agents can modify TypeScript files?")
- Powers routing logic without manual maintenance

### When to Use Context Contracts

**Good fit:**
- Multi-agent systems with 10+ specialized agents
- Production environments requiring audit trails
- Systems where agents coordinate through orchestrator routing
- Domains with strict scope boundaries (microservices, security zones)

**Poor fit:**
- Simple single-agent workflows
- Exploratory development without clear boundaries
- Systems where flexibility > enforcement

### Connections

- **To [Skills](5-skills-and-meta-tools.md#skills-as-meta-tools)**: Context contracts extend progressive disclosure from runtime (Skills load on-demand) to orchestration design (agents declare requirements)
- **To [Tool Restrictions](3-tool-restrictions.md)**: Contracts provide schema-based validation where tool restrictions provide runtime enforcement
- **To [Self-Improving Experts](../6-patterns/2-self-improving-experts.md#agent-registry-pattern)**: Registry generation from contracts enables discovery layer for expert systems

**Sources:** Advanced external .claude/ implementation patterns documented in KotaDB scouting analysis.

---

## Leading Questions

- When should you use a skill vs. a traditional tool?
- How do you decide what to put in a skill's discovery metadata vs. activation payload?
- Can skills invoke other skills? Should they?
- How do you measure whether a skill is worth the token overhead?
- What domains benefit most from the skill pattern?
- How do you version skills in production?
- What happens when multiple skills are active simultaneously?

---

## Connections

- **To [Context](../4-context/_index.md):** Progressive disclosure pattern
- **To [Prompt](../2-prompt/_index.md):** Skills as prompt injection mechanism
- **To [Tool Selection](2-tool-selection.md):** Skills affect tool availability dynamically
- **To [Scaling Tools](4-scaling-tools.md):** Skills as a scaling pattern for specialized capabilities
- **To [Claude Code](../9-practitioner-toolkit/1-claude-code.md):** Production implementation
- **To [Cost and Latency](../7-practices/3-cost-and-latency.md):** Token cost comparison across feature types
