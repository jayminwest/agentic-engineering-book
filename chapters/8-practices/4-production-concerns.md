---
title: Production Concerns
description: Running agents reliably at scale
created: 2025-12-08
last_updated: 2026-04-11
tags: [practices, production, reliability, operations, monitoring, hooks, reliability-thresholds, deployment-criteria, augmentation-vs-automation, reliability-dimensions]
part: 2
part_title: Craft
chapter: 8
section: 4
order: 2.8.4
---

# Production Concerns

Demo agents are easy. Production agents are hard. This is where the craft lives.

---

## Reliability Thresholds for Production Deployment

*[2026-04-11]*: Production deployment is a threshold decision, not a spectrum. An agent either clears the bar for its deployment mode or it does not. The bar differs by deployment mode.

### Two Deployment Modes, Two Threshold Regimes

The most consequential reliability decision is the deployment mode:

| Mode | Definition | Reliability Tolerance |
|------|------------|----------------------|
| **Augmentation** | Human review is in the loop; agent error is buffered before reaching consequences | Lower thresholds acceptable — human backstop absorbs inconsistency |
| **Automation** | Agent operates without human review; errors reach consequences directly | Near-perfect consistency and safety required — no backstop |

An agent succeeding 90% of the time but failing unpredictably on 10% "may assist users yet remain unacceptable for autonomous systems." [Rabanser et al., arXiv:2602.16666, 2026]

This distinction operationalizes the human-in-the-loop patterns covered in [Human-in-the-Loop](../7-patterns/6-human-in-the-loop.md): HITL is not only a usability choice — it is a reliability architecture that determines which threshold regime the system operates in.

### The Four-Dimension Reliability Gate

Before production deployment, evaluate against four dimensions: [Rabanser et al., arXiv:2602.16666, 2026]

**Consistency:** Does the agent produce the same outcome when run on the same input multiple times? Measure by running each representative task K≥5 times. High variance across runs signals unpredictable behavior — the defining characteristic of agents that are acceptable for augmentation but unacceptable for automation.

**Robustness:** Does the agent degrade gracefully under realistic perturbations? Test prompt paraphrases, tool interface format changes, and infrastructure fault injection. Agents that handle "cancel my subscription" but fail on "end my plan" are brittle in ways that lab evals do not reveal.

**Predictability:** Does the agent signal uncertainty reliably? An agent that is 60% accurate but always knows when it's uncertain is more deployable than an agent that is 80% accurate but overconfident in its failures. Calibration — whether stated confidence matches actual performance — determines whether operators can trust confidence signals in production.

**Safety:** Are compliance violations within acceptable bounds, and are severity levels controlled? Safety does not average. A single high-severity violation (unauthorized action, irreversible data deletion, PII exposure) is a deployment blocker regardless of aggregate compliance rate. Evaluate using the risk decomposition: `ℛ_Saf = 1 − (1 − S_comp)(1 − S_harm)`, where compliance and severity are assessed independently.

### Threshold Guidance by Deployment Mode

No universal thresholds apply across all domains. The following are starting points derived from the paper's framework, not prescriptive minimums:

| Dimension | Augmentation Mode | Automation Mode |
|-----------|-------------------|-----------------|
| Consistency (outcome) | ≥70% same outcome on K=5 runs | ≥90% same outcome on K=5 runs |
| Prompt robustness | Passes 3 of 5 paraphrase variants | Passes 5 of 5 paraphrase variants |
| Safety compliance | Zero high-severity violations in eval set | Zero medium-or-high-severity violations in eval set |
| Predictability | Calibration error <20% (agent can signal uncertainty) | Calibration error <10%; discrimination AUROC >0.7 |

Adjust thresholds upward for irreversible actions (financial transactions, data deletion, external communications) and downward for low-consequence tasks (summarization, classification with human review).

### Compound Reliability and Architecture Decisions

The compound error math (see [Model Evaluation](../3-model/5-model-evaluation.md#the-compound-error-problem)) shows how per-step accuracy multiplies across a workflow. Reliability dimensions compound the same way: a three-tool pipeline where each tool meets 90% consistency independently yields approximately 73% end-to-end consistency (`0.9³`).

This has a direct architectural implication: the compound reliability of a pipeline sets a lower ceiling than individual component reliability. When a pipeline fails to meet deployment thresholds, the first diagnostic question is not "which component is weakest" but "how many components does this pipeline chain?" Reducing pipeline depth is often more effective than improving individual component reliability.

### Real-World Incident Mapping

Three documented production incidents illustrate what happens when deployment threshold decisions are not grounded in reliability measurement:

| Incident | Root Reliability Failure | Dimension Missed |
|----------|--------------------------|------------------|
| Replit database deletion — agent deleted production DB despite explicit prohibition | High-severity compliance violation; instruction constraint not robustly enforced | Safety (S_harm) + Robustness (prompt constraint robustness) |
| OpenAI Operator unauthorized purchase — agent took action outside declared scope | Out-of-scope action; trajectory consistency violated | Safety (S_comp) + Consistency (trajectory) |
| NYC chatbot inconsistent legal advice — identical questions yielded different answers | High outcome variance; overconfident wrong answers | Consistency (C_out) + Predictability (calibration) |

[Rabanser et al., arXiv:2602.16666, 2026]

Each incident would have been detectable in pre-deployment evaluation if the corresponding reliability dimension had been measured. None are detectable by mean task success rate alone.

### Connections

- **To [Evaluation](2-evaluation.md#reliability-dimensions-beyond-task-completion):** The four reliability dimensions are operationalized as eval practices in that chapter. Run those evaluations before applying these thresholds.
- **To [Model Evaluation](../3-model/5-model-evaluation.md#the-capability-reliability-gap):** Benchmark accuracy scores do not predict reliability dimension scores. Capability gains do not improve reliability as a byproduct. The Core Four (prompt / model / context / tool failures) answers "what broke?" — root cause attribution. The four reliability dimensions answer "what properties degrade?" — observable system behavior. Both frames are necessary for production diagnosis; neither replaces the other.
- **To [Human-in-the-Loop](../7-patterns/6-human-in-the-loop.md):** Augmentation mode's lower reliability tolerance is why HITL is an architectural reliability decision, not only a usability one.
- **To [Debugging](1-debugging-agents.md):** Production incidents map to specific reliability dimensions; the Core Four root-cause taxonomy and the four-dimension observable-property framework are complementary diagnostic tools.

**Source:** [Towards a Science of AI Agent Reliability](https://arxiv.org/abs/2602.16666) (Rabanser, Kapoor, Narayanan et al., Princeton, 2026)

---

## Connections

- **To [Evaluation](2-evaluation.md):** How does prod monitoring relate to eval?
- **To [Debugging](1-debugging-agents.md):** How do you debug production issues?
- **Human-in-the-Loop:** How do humans fit into production operations?

---

## Production War Stories

*Document production incidents, what happened, what you learned:*

### Multi-Agent Production Lessons

*[2025-12-09]*: Hard-won lessons from practitioners running multi-agent systems at scale (including a 400K LOC codebase). These patterns emerge repeatedly across production deployments.

**Context Switching Kills Productivity**

"It is better to start with fresh context on a fresh problem rather than using an existing context." Don't try to salvage a degraded agent—boot a new one. The cost of context confusion exceeds the cost of restarting. In multi-agent systems, this is even more important: let each subagent start fresh and focused rather than inheriting accumulated state.

**CLAUDE.md as Convention Encoding**

Document project conventions in CLAUDE.md. Agents read this to understand boundaries, patterns, and expectations. For large codebases, this is essential—agents can't infer all conventions from code alone. Investing time in clear specs saves 10× in agent iterations. The alternative is watching agents repeatedly violate conventions you thought were obvious.

**Lifecycle Hooks for Control**

Wire hooks at critical transitions:
- **PreToolUse**: Validate commands before execution (catch dangerous operations)
- **SubagentStop**: Record outputs, promote artifacts, log what was accomplished
- **ErrorEscalation**: Notify human overseers when agents fail or hit edge cases

These hooks provide observability and control points without requiring constant human monitoring.

**The Orchestrator Observes Itself**

*[2025-12-09]*: Hooks enable a powerful pattern—the orchestrator becomes self-aware of its own operations. PreToolUse and PostToolUse hooks capture every Claude Code action, creating real-time visibility into agent behavior.

This enables:
- **Cost Tracking**: Measure token consumption per tool invocation, per subagent, per task
- **Audit Trails**: Log every agent action with context—who did what, when, and why
- **Real-Time Monitoring**: Detect patterns, anomalies, or runaway agents as they happen
- **Platform Self-Awareness**: The system knows what it's doing while it's doing it

Traditional software logs after the fact. Hooks let platforms observe themselves *during execution*. This is the foundation for production-grade agent operations—you can't manage what you can't measure, and you can't measure what you can't observe.

**Implementation Pattern**: Wire PostToolUse hooks to write structured logs (JSON) containing tool name, parameters, results, duration, and cost estimates. Aggregate these logs for operational dashboards showing agent activity in real-time.

**See Also**:
- [Tool Use: Tool Restrictions as Security Boundaries](../5-tool-use/3-tool-restrictions.md#tool-restrictions-as-security-boundaries) — How hooks integrate with tool permission boundaries
- [Claude Code Hooks Documentation](/.claude/ai_docs/claude-code/hooks.md) — Complete technical reference

**Test-First Discipline**

The test-first pattern works remarkably well with multi-agent systems:
1. Dedicate a test-writing subagent to create tests that currently fail
2. Verify the tests actually fail (don't skip this)
3. Implementer subagent makes tests pass without changing tests
4. Review subagent validates the changes

This creates natural quality gates and makes progress measurable. Agents write better code when following this discipline—they have clear success criteria.

**Dedicated Review Gate**

Maintain a dedicated code-review subagent as a final gate. Configure it to enforce:
- Linting and style conventions
- Complexity bounds (cyclomatic complexity, file length)
- Security checks (no secrets, no dangerous patterns)

This catches issues before they reach humans, and the reviewing agent doesn't have the sunk-cost bias of having written the code.

**Documentation Investment**

Time spent on clear specs saves 10× in agent iterations. If you're watching agents go in circles, the problem is usually upstream in the specification, not in the agents themselves. Write clear specs once, save debugging time forever.

**Opus 4.5 for Orchestration**

Practitioner consensus: Opus 4.5 is particularly effective at managing teams of subagents. It handles the coordination overhead well and produces more coherent multi-agent workflows than smaller models used for orchestration.

**Sources**: [How I Manage 400K Lines of Code with Claude Code](https://blockhead.consulting/blog/claude_code_workflow_july_2025), [Claude Agent SDK Best Practices](https://skywork.ai/blog/claude-agent-sdk-best-practices-ai-agents-2025/), [Multi-Agent Orchestration: Running 10+ Claude Instances](https://dev.to/bredmond1019/multi-agent-orchestration-running-10-claude-instances-in-parallel-part-3-29da)

### Google Cloud Deployment Gotchas

*[2025-12-09]*: Hard-won lessons from practitioners deploying Google ADK agents to production. These are framework-agnostic lessons that apply broadly—they just happen to surface most visibly with ADK on Google Cloud.

**Infrastructure Permissions**

Enable Cloud Run Admin API *before* deployment. Missing permissions fail with cryptic errors that don't obviously point to the permission issue. This isn't ADK-specific—it's a Google Cloud pattern. Before deploying anything, verify required APIs are enabled.

**Environment Variable Naming**

Generic environment variable names conflict with system variables. `MODEL` is particularly problematic—it conflicts with Google Cloud's internal variables.

The fix: prefix everything. Use `GEMMA_MODEL_NAME` instead of `MODEL`, `ADK_API_KEY` instead of `API_KEY`. This namespacing prevents silent conflicts that cause mysterious runtime behavior.

**MCP Session Affinity**

MCP connections are stateful. At scale, this means:
- Load balancers need session affinity (sticky sessions)
- Connection loss requires reconnection handling
- Horizontal scaling is constrained by state distribution

Plan load balancing accordingly. Stateless is easy; stateful requires architecture decisions.

**Asyncio Everywhere**

ADK and MCP both assume async-first Python. Common mistakes:
- Writing sync tool implementations (blocks the event loop)
- Forgetting `await` on async calls
- Not using async context managers for connections

Default to `async def` for everything. Sync code in an async codebase serializes naturally parallel operations—you lose the concurrency benefits without obvious errors.

**The "Works Locally, Fails in Cloud" Pattern**

Local development hides many issues:
- Environment variable sources differ (local shell vs. Cloud Run secrets)
- Network topology changes (localhost vs. VPC)
- Permission models differ (local user vs. service account)

Always test in a staging environment that mirrors production. "It works on my machine" is particularly dangerous with agent systems where subtle environment differences cause behavioral changes.

**See Also**: [Google ADK: Production Lessons](../10-practitioner-toolkit/2-google-adk.md#production-lessons) — ADK-specific deployment experiences

---

## Hook-Based Enforcement Patterns

*[2026-02-05]*: Production agent systems require runtime validation beyond prompt instructions. Hook-based enforcement provides time-budgeted validation, pre-edit dependency injection, and scope enforcement for reliable operations at scale.

### Your Mental Model

**Hooks are enforcement points, not just observability.** While PostToolUse hooks enable monitoring (see [The Orchestrator Observes Itself](#the-orchestrator-observes-itself)), hooks also enforce constraints:
- Time budgets prevent runaway operations
- Pre-edit checks inject required context
- Scope validation blocks out-of-bounds modifications

This philosophy: **"Permissive tools + strict prompts + hook enforcement"**—tools provide capabilities, prompts guide behavior, hooks enforce boundaries.

### Time-Budgeted Hook Execution

**The constraint:** Hooks run synchronously, blocking agent execution. Long-running hooks degrade user experience.

**Production budgets:**

| Hook | Budget | Purpose |
|------|--------|---------|
| SessionStart | 3-5s | Load domain expertise, check MCP health |
| PreEdit | 10-15s | Query code intelligence (LSP, type systems) |
| PreToolUse | 5-10s | Validate command safety, check credentials |
| PostToolUse | 3-5s | Log action, update metrics |

**Graceful degradation:**
```typescript
async function sessionStartHook(context: SessionContext): Promise<void> {
  const timeout = 5000; // 5 second budget

  try {
    await Promise.race([
      loadExpertise(context.domain),
      sleep(timeout)
    ]);
  } catch (error) {
    logger.warn("SessionStart hook timed out, continuing with defaults");
    // Graceful fallback: agent proceeds without expertise injection
  }
}
```

**Why time budgets matter:**
- Agents remain responsive (no 30s hangs)
- User experience preserved (fast feedback loops)
- Failure modes explicit (timeout → fallback path)

### Pre-Edit Dependency Injection

**The pattern:** Before file modifications, inject dynamic context agents need for correct implementation.

**Example: TypeScript module editing**

```typescript
async function preEditHook(file: string): Promise<void> {
  if (!file.endsWith('.ts')) return; // Only TypeScript files

  // Query TypeScript language server for:
  // - Current imports in file
  // - Type definitions for interfaces
  // - Available exports from modules
  const typeContext = await queryTSServer(file);

  // Inject as temporary context for this edit session
  injectContext({
    imports: typeContext.imports,
    types: typeContext.types,
    exports: typeContext.exports
  });
}
```

**What this prevents:**
- Missing imports (agent knows what's already imported)
- Type mismatches (agent sees type definitions)
- Undefined references (agent knows available exports)

**Cost:** 10-15s latency before edit begins (acceptable for complex changes)

### Session-Start Expertise Injection

**The pattern:** Load domain-specific expertise dynamically based on repository structure or task context.

**Example: Domain detection + expertise loading**

```typescript
async function sessionStartHook(): Promise<void> {
  const repo = detectRepository();

  if (repo.hasDir('.claude/agents/experts/github')) {
    const expertise = await loadExpertise('github');
    injectContext({ github_patterns: expertise });
  }

  if (repo.hasDir('src/api')) {
    const apiPatterns = await loadExpertise('api-development');
    injectContext({ api_patterns: apiPatterns });
  }

  // Continue for all detected domains...
}
```

**What this enables:**
- Context-aware agent behavior (knows repository patterns)
- Zero configuration (automatic domain detection)
- Expertise reuse (same patterns across projects)

**Trade-off:** 3-5s session startup latency for dynamic expertise loading

### Scope Enforcement Hook

**The pattern:** Validate file modifications against agent's declared contract before write operations.

**Example: Contract-based scope validation**

```typescript
async function preEditHook(file: string, agent: AgentContext): Promise<void> {
  const contract = agent.contextContract;

  // Check allowed_modifications globs
  const allowed = contract.outputs.allowed_modifications || ['**/*'];
  const isAllowed = allowed.some(glob => minimatch(file, glob));

  if (!isAllowed) {
    throw new ScopeViolationError(
      `Agent ${agent.name} attempted to modify ${file}, ` +
      `but only allowed to modify: ${allowed.join(', ')}`
    );
  }

  // Check forbidden patterns
  const forbidden = contract.outputs.forbidden_patterns || [];
  const isForbidden = forbidden.some(glob => minimatch(file, glob));

  if (isForbidden) {
    throw new ScopeViolationError(
      `Agent ${agent.name} attempted to modify forbidden path: ${file}`
    );
  }
}
```

**What this enforces:**
- File ownership (agents can't modify files outside their scope)
- Safety boundaries (prevents accidental system file modification)
- Coordination constraints (Implementation Pattern file ownership)

**Performance:** <1s validation (glob matching is fast)

### Shared Hook Utilities

**The problem:** Hooks across all agents need common functionality—logging, timeout management, MCP health checks.

**The solution:** Shared utilities library:

```
.claude/hooks/
├── utils/
│   ├── logger.ts        # Structured logging
│   ├── timeout.ts       # Time budget enforcement
│   ├── mcp-health.ts    # MCP server availability checks
│   └── context-inject.ts # Dynamic context loading
├── sessionStart.ts      # Uses utilities
├── preEdit.ts           # Uses utilities
└── preToolUse.ts        # Uses utilities
```

**Why this matters:**
- Consistency (all hooks log uniformly)
- Reusability (timeout logic once, used everywhere)
- Testability (test utilities independently of hooks)

### Permissive Tools + Strict Prompts + Hook Enforcement

**Philosophy:**

1. **Permissive tools:** Agents have broad tool access (Write, Edit, Bash)
2. **Strict prompts:** Instructions guide desired behavior ("Only modify auth module")
3. **Hook enforcement:** Runtime validation ensures compliance (scope checks, safety gates)

**Why this works:**
- Flexibility (agents can adapt to unexpected needs within boundaries)
- Safety (hooks prevent dangerous operations)
- Clarity (prompts explain intent, hooks enforce hard limits)

**Anti-pattern:** Overly restrictive tools (agent can't handle edge cases) + weak hooks (no enforcement of constraints). This fails silently—agents work around restrictions or violate assumptions.

### Production Deployment Considerations

**Hook failure modes:**
- Timeout → graceful degradation (proceed without enhancement)
- MCP unavailable → skip optional features (log warning)
- Validation failure → hard stop (prevent incorrect operation)

**Monitoring:**
- Log all hook executions (timing, success/failure)
- Alert on repeated hook failures (system degradation)
- Track timeout rates (indicates budget tuning needed)

**Testing:**
- Unit test hook logic separately
- Integration test hook + agent behavior
- Load test hook performance (ensure budgets hold under load)

### Connections

- **To [Context Contracts](../../5-tool-use/5-skills-and-meta-tools.md#context-contracts-for-agent-capability-declaration)**: Scope enforcement hooks validate against declared contracts
- **To [Lifecycle Hooks for Control](#lifecycle-hooks-for-control)**: Extends existing hook patterns with enforcement and time budgeting
- **To [Flat Team Orchestration](../../7-patterns/8-expert-swarm-pattern.md#implementation-pattern-file-ownership-coordination)**: Scope enforcement enables file ownership pattern

**Sources:** Advanced external .claude/ implementation patterns, hook infrastructure analysis from KotaDB system.

