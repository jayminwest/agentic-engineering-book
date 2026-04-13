---
title: Human-in-the-Loop Pattern
description: Strategic placement of human approval gates in agentic workflows for risk management and quality control
created: 2026-01-30
last_updated: 2026-01-30
tags: [patterns, human-approval, risk-management, workflow, production]
part: 2
part_title: Craft
chapter: 7
section: 6
order: 2.7.6
---

# Human-in-the-Loop Pattern

Strategic insertion of human approval checkpoints in agentic workflows to manage risk, ensure quality, and maintain human oversight over consequential decisions.

---

## Core Insight

*[2026-01-30]*: **Not all agent actions carry equal risk.** The human-in-the-loop pattern recognizes that some operations (reading files, running tests) are safe to automate fully, while others (deploying to production, deleting data, sending communications) warrant human review. The key is placing approval gates at the right points—too few gates and you risk costly mistakes; too many and you've just built an expensive chatbot.

The pattern answers three questions:
1. **When** should an agent pause for human approval?
2. **Where** in the workflow should gates be placed?
3. **How** should the approval interaction be structured?

---

## When to Require Human Approval

### Risk-Based Gate Criteria

Not every action needs a gate. Use these criteria to decide:

| Risk Factor | Low (Auto-proceed) | Medium (Notify) | High (Require Approval) |
|-------------|-------------------|-----------------|------------------------|
| **Reversibility** | Git commit (revertible) | Config change | Database migration |
| **Blast radius** | Single file edit | Module changes | Production deployment |
| **Cost** | API call < $0.10 | Batch operation < $10 | Operation > $100 |
| **Sensitivity** | Internal code | Customer data access | Credentials, payments |
| **Precedent** | Routine operation | First-time pattern | Novel approach |

### The Gate Decision Tree

```
Is this action reversible within 5 minutes?
 |-- Yes --> Is it modifying production systems?
 |            |-- Yes --> GATE: Require approval
 |            +-- No  --> Auto-proceed
 +-- No  --> GATE: Require approval

Does this action affect external parties (customers, APIs, services)?
 |-- Yes --> GATE: Require approval
 +-- No  --> Continue evaluation

Is this the first time performing this type of operation?
 |-- Yes --> GATE: Require approval (establish precedent)
 +-- No  --> Follow established pattern
```

### Common Gate Triggers

**Always gate:**
- Production deployments
- Database schema changes
- External API calls with side effects (sending emails, creating accounts)
- Credential or permission changes
- Cost-incurring operations above threshold
- Deleting data (especially without backups)

**Consider gating:**
- First execution of a new workflow
- Changes to security-sensitive code
- Operations affecting multiple systems
- Novel approaches the agent hasn't used before

**Safe to auto-proceed:**
- Reading files and exploring codebases
- Running tests (in isolated environments)
- Creating branches (not merging)
- Generating documentation drafts
- Local development operations

---

## Gate Placement Strategies

### Pre-Action Gates

The agent presents a plan and waits for approval before execution.

```
Agent: "I will deploy version 2.3.1 to production. This will:
        - Update 3 API endpoints
        - Run database migration #47
        - Restart 5 service instances

        Estimated downtime: 30 seconds
        Rollback plan: revert to v2.3.0 via deploy script

        Proceed? [approve/reject/modify]"
```

**Best for:**
- Irreversible operations
- High-cost actions
- Operations with external effects

**Trade-off:** Blocks workflow progress until human responds. Use sparingly for time-sensitive workflows.

### Post-Action Review Gates

The agent executes, then pauses for human review before proceeding to the next phase.

```
Agent: "I completed the code refactoring. Summary:
        - Changed 12 files
        - Updated 47 function signatures
        - All tests passing (143/143)

        Please review changes before I create the PR.
        [continue/revert/modify]"
```

**Best for:**
- Complex multi-step workflows where intermediate work is valuable
- Operations that are revertible
- When blocking is more expensive than potential rework

**Trade-off:** Human may need to undo completed work if they reject.

### Checkpoint Gates (Plan-Build-Review)

Natural gate placement at phase transitions in the [Plan-Build-Review](1-plan-build-review.md) pattern:

```
Research --> [GATE: approve research findings]
         --> Plan --> [GATE: approve spec before implementation]
                  --> Build --> [GATE: review before merge/deploy]
```

**This is the recommended default.** The spec file created during planning serves as the approval artifact—humans review the spec, not raw code changes.

### Escalation Gates

The agent proceeds autonomously until it encounters uncertainty or risk.

```
Agent: "I encountered an unexpected situation:
        - Found 3 circular dependencies I didn't anticipate
        - Options:
          A) Refactor module X to break cycles (2 hours)
          B) Add temporary workaround (30 min, tech debt)
          C) Abort and escalate to team lead

        Which approach should I take?"
```

**Best for:**
- Workflows with high autonomy but need fallback
- Experienced agents with good self-assessment
- Reducing gate overhead while preserving safety

---

## Synchronous vs. Asynchronous Approval

### Synchronous (Blocking)

The agent waits for human response before continuing.

**Implementation:**
```python
def execute_with_approval(action, context):
    spec = generate_spec(action, context)

    # Block until human responds
    approval = request_human_approval(spec)

    if approval.approved:
        return execute(action)
    elif approval.modified:
        return execute_with_approval(approval.modifications, context)
    else:
        return abort_with_reason(approval.reason)
```

**When to use:**
- Interactive sessions where human is actively engaged
- High-stakes operations where delay is acceptable
- Workflows that genuinely cannot proceed without human judgment

**Trade-off:** Blocks workflow. Human availability becomes a bottleneck.

### Asynchronous (Non-blocking)

The agent queues the approval request and continues with other work or terminates cleanly.

**Implementation:**
```python
def execute_with_async_approval(action, context):
    spec = generate_spec(action, context)

    # Queue for human review
    approval_id = queue_approval_request(spec)

    # Agent can terminate or do other work
    return {
        "status": "pending_approval",
        "approval_id": approval_id,
        "resume_command": f"approve {approval_id} && continue-workflow"
    }
```

**When to use:**
- Long-running workflows spanning hours/days
- Batch operations where multiple approvals accumulate
- Operations where human review takes time (code review, security audit)

**Trade-off:** Requires workflow state management and resume capability.

### Hybrid: Timeout with Escalation

The agent waits briefly, then escalates or proceeds with conservative default.

```python
def execute_with_timeout(action, context, timeout=300):
    approval = request_approval_with_timeout(action, timeout)

    if approval.received:
        return handle_approval(approval)
    elif action.has_safe_default:
        log_warning("Timeout reached, proceeding with safe default")
        return execute_safe_default(action)
    else:
        return escalate_to_backup_approver(action)
```

**When to use:**
- Production operations with SLAs
- Workflows where delay has cost
- When safe fallback behavior exists

---

## Gate Interaction Design

### Effective Approval Requests

A good approval request provides everything the human needs to decide:

```markdown
## Action Requested
Deploy authentication service v2.1.0 to production

## Context
- Previous version: v2.0.3 (deployed 2 weeks ago)
- Changes: OAuth2 refresh token handling, rate limiting
- Commits: 12 commits from 3 developers
- Tests: 247 passing, 0 failing, 94% coverage

## Risk Assessment
- **Rollback plan:** Blue-green deployment, instant rollback available
- **Blast radius:** All authenticated users (~50K active sessions)
- **Failure mode:** If refresh tokens fail, users must re-authenticate

## Recommendation
APPROVE - Changes are well-tested and rollback is instant.

## Options
- [APPROVE] Proceed with deployment
- [REJECT] Cancel and provide reason
- [DELAY] Schedule for off-peak hours (2am UTC)
- [MODIFY] Specify changes to deployment plan
```

**Key elements:**
1. **Clear action statement** - What exactly will happen
2. **Sufficient context** - What the human needs to know
3. **Risk assessment** - What could go wrong and mitigation
4. **Explicit options** - Not just approve/reject

### Approval Artifacts

For complex approvals, use artifacts rather than inline descriptions:

**Spec files** (from Plan-Build-Review):
```
Agent: "Review the spec at docs/specs/auth-v2.1-deploy.md
        and respond with 'approve spec' or provide modifications."
```

**Diff summaries**:
```
Agent: "Review the PR at github.com/org/repo/pull/234
        12 files changed, +340/-120 lines
        Key changes: [summarized]"
```

**Decision logs** (for audit trail):
```markdown
# Decision Log: Deploy auth-v2.1.0

## Request
Agent: deploy-bot-7
Time: 2026-01-30T14:23:00Z
Action: Production deployment

## Approval
Approver: jane@company.com
Time: 2026-01-30T14:25:30Z
Decision: APPROVED
Notes: "Verified test coverage, rollback plan looks solid"

## Execution
Started: 2026-01-30T14:26:00Z
Completed: 2026-01-30T14:28:45Z
Status: SUCCESS
```

---

## Claude Code Implementation

### Plan Mode Requirement

Claude Code's `plan_mode_required` flag implements human-in-the-loop for plan approval:

```yaml
# In teammate spawn
{
  "name": "risky-builder",
  "plan_mode_required": true,
  "prompt": "Implement database migration..."
}
```

When enabled:
1. Agent enters plan mode automatically
2. Agent must call `ExitPlanMode` to request approval
3. Team lead receives plan approval request
4. Agent can only proceed after explicit approval

**See:** [Claude Code: TeammateTool](../10-practitioner-toolkit/1-claude-code.md#teammatetool-native-multi-agent-coordination-hidden) for implementation details.

### Lifecycle Hooks for Gates

Use lifecycle hooks to implement custom gates:

```json
// settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "python3 gate-check.py --tool=Bash --args=\"$ARGUMENTS\""
      }
    ]
  }
}
```

The hook script can:
- Allow the operation (exit 0)
- Block with message (exit non-zero with stderr)
- Prompt for human input before returning

**See:** [Production Concerns: Lifecycle Hooks](../8-practices/4-production-concerns.md) for hook patterns.

### The HUMAN_IN_LOOP Variable Pattern

Expert agents can accept a flag to toggle approval requirements:

```markdown
---
name: deploy-agent
description: Deploy services. Expects USER_PROMPT, HUMAN_IN_LOOP (default true)
---

## Workflow

1. Generate deployment plan
2. If HUMAN_IN_LOOP:
   - Save plan to spec file
   - Present summary and wait for approval
   - Proceed only after explicit approval
3. If not HUMAN_IN_LOOP:
   - Log plan for audit
   - Proceed with automated checks only
4. Execute deployment
```

This allows the same agent to operate in both supervised and autonomous modes based on context.

---

## Relationship to Autonomous Loops

The [Autonomous Loops (Ralph Wiggum)](4-autonomous-loops.md) pattern deliberately minimizes human intervention—iteration discovers the path. Human-in-the-loop is the complementary pattern for when you need human judgment.

**Use autonomous loops when:**
- Task has machine-verifiable success criteria (tests pass)
- Operations are reversible (git commits)
- Failures are acceptable learning data
- Cost of iteration < cost of human time

**Use human-in-the-loop when:**
- Success requires human judgment
- Operations are irreversible or high-stakes
- External systems are affected
- Compliance or audit requires human approval

**Hybrid approach:** Autonomous loops with escalation gates:
```
while not complete:
    result = attempt_iteration()
    if result.needs_human:
        approval = request_human_input(result.question)
        incorporate_feedback(approval)
    elif result.failed and iterations > threshold:
        escalate_to_human()
        break
```

---

## Anti-Patterns

### Gate Fatigue

**Problem:** Too many approval requests desensitize humans to risk.

**Symptoms:**
- Humans approve without reading
- Approval latency increases over time
- "Rubber stamp" culture develops

**Solution:** Reserve gates for genuinely high-risk operations. Use tiered approval levels—some operations need acknowledgment (click to continue), others need review (read and confirm), few need approval (deliberate decision).

### Vague Approval Requests

**Problem:** Agent asks "Should I proceed?" without context.

**Symptoms:**
- Humans ask clarifying questions
- Approvals are guesses
- Post-hoc disputes about what was approved

**Solution:** Approval requests must include action, context, risk assessment, and explicit options. If you can't articulate what's being approved, the gate is premature.

### Synchronous Gates in Async Workflows

**Problem:** Blocking gate in a workflow designed for autonomy.

**Symptoms:**
- Workflow stalls for hours waiting for approval
- Agents timeout or lose context
- Humans feel pressured to respond immediately

**Solution:** Match gate synchronicity to workflow. Async workflows need async approval with state persistence and resume capability.

### Missing Rollback Plans

**Problem:** Gate approves an action with no recovery path.

**Symptoms:**
- Approval request says "Proceed?" with no mention of reversal
- Human approves hoping nothing goes wrong
- Failure requires manual emergency intervention

**Solution:** Every high-risk gate must include rollback plan. "If this fails, here's how we recover." No rollback plan = not ready for approval.

### Gates Without Audit Trail

**Problem:** Approvals happen but aren't recorded.

**Symptoms:**
- "Who approved this?" has no answer
- Compliance audits fail
- Post-incident review lacks data

**Solution:** Log every approval request and response with timestamp, approver, decision, and any modifications. Treat approval logs as production data.

---

## When to Use This Pattern

### Good Fit

**Workflows with consequential actions:**
- Production operations (deployments, migrations, data changes)
- External communications (emails to customers, API calls with side effects)
- Financial operations (charges, refunds, transfers)
- Access control changes (permissions, credentials)

**Compliance requirements:**
- SOC2, HIPAA, or other frameworks requiring human oversight
- Audit trails for regulatory review
- Separation of duties (agent proposes, human approves)

**Trust building:**
- New workflows where agent reliability is unproven
- High-value operations where mistakes are costly
- Situations where human judgment genuinely adds value

### Poor Fit

**Fully automatable workflows:**
- Test execution in isolated environments
- Code formatting and linting
- Documentation generation from code
- Development environment operations

**Time-critical operations where human latency is unacceptable:**
- Auto-scaling responses
- Incident remediation (initial triage)
- High-frequency trading decisions

**Operations where human adds no value:**
- Deterministic transformations
- Operations with machine-verifiable correctness
- Workflows where human would always approve

---

## Questions to Explore

### How do you calibrate gate placement over time?
As trust in an agent grows, should gates be removed? How do you measure "trustworthiness" of an agent workflow? What signals indicate gates can be relaxed?

### What's the right approval latency target?
Faster approvals enable agent productivity but may sacrifice review quality. How do you balance responsiveness with thoroughness? Does approval latency correlate with decision quality?

### How do you handle approval delegation?
When the primary approver is unavailable, who can approve? How do you prevent delegation from undermining the gate's purpose?

### Can approval patterns be learned?
If a human always approves a certain class of operation, should the agent learn to auto-proceed? What's the risk of encoding biases into automation?

---

## Connections

- **To [Plan-Build-Review](1-plan-build-review.md):** Natural gate placement at plan approval. The spec file serves as the approval artifact, enabling review without reading raw code.
- **To [Autonomous Loops](4-autonomous-loops.md):** Complementary pattern. Ralph Wiggum minimizes gates; human-in-the-loop maximizes oversight. Use both based on risk profile.
- **To [Production Concerns](../8-practices/4-production-concerns.md):** Lifecycle hooks enable gate implementation. PreToolUse hooks can enforce approval before dangerous operations.
- **To [Orchestrator Pattern](3-orchestrator-pattern.md):** Gates fit naturally at phase transitions. Orchestrators can embed approval logic in phase gating.
- **To [Self-Improving Experts](2-self-improving-experts.md):** The 4-agent pattern includes HUMAN_IN_LOOP flag for plan agents, enabling per-operation approval control.
