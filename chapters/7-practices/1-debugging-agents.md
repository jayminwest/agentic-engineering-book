---
title: Debugging Agents
description: Finding and fixing what went wrong in agentic systems
created: 2025-12-08
last_updated: 2026-02-06
tags: [practices, debugging, troubleshooting, failure-modes, diagnostics]
part: 2
part_title: Craft
chapter: 7
section: 1
order: 2.7.1
---

# Debugging Agents

Agents fail in ways that traditional software doesn't. The debugging skills are different too.

Traditional debugging asks "why did this code path execute?" Agent debugging asks "why did the model choose this action?" The former traces deterministic execution; the latter reverse-engineers probabilistic decisions from limited observability. This fundamental difference shapes every debugging technique in this chapter.

---

## The Debugging Mindset

### Agent Debugging vs Traditional Debugging

| Traditional Code | Agent Systems |
|-----------------|---------------|
| Deterministic execution | Probabilistic decisions |
| Stack traces show cause | Only inputs/outputs visible |
| Same input → same output | Same input → varying outputs |
| Bugs reproduce reliably | Failures may be intermittent |
| Fix the code | Fix the prompt, context, tools, OR code |

Agent debugging requires working backward from observed behavior to infer what the model "thought." The model's internal reasoning remains opaque, so debugging relies on:
- Structured logging at decision points
- Systematic hypothesis testing
- Understanding model behavior patterns

### The Core Four Framework

Every agent failure traces to one or more of the "core four" components. Before debugging, identify which component is suspect:

| Component | Symptoms | First Check |
|-----------|----------|-------------|
| **Prompt** | Wrong interpretation, missed instructions | Does the prompt clearly specify what NOT to do? |
| **Model** | Capability limits, reasoning errors | Can a more capable model do this task? |
| **Context** | Hallucination, outdated info, missed details | What context was actually available? |
| **Tools** | Wrong tool choice, tool errors, bad outputs | Did tools return what was expected? |

**The diagnostic sequence**: Check tools first (easiest to verify), then context (what did the agent know?), then prompt (ambiguity?), then model (capability?) last.

---

## Diagnostic Decision Tree

When an agent fails, work through this decision tree to identify root cause:

### Step 1: Characterize the Failure

**Q: What type of failure occurred?**

| If the agent... | Jump to |
|----------------|---------|
| Produced wrong output | → Step 2A |
| Got stuck / looped | → Step 2B |
| Stopped too early | → Step 2C |
| Used wrong tool | → Step 2D |
| Made something up | → Step 2E |
| Crashed / errored | → Step 2F |

### Step 2A: Wrong Output

The agent completed but produced incorrect results.

1. **Check prompt clarity**: Does the prompt specify the expected format? Are there ambiguous instructions?
2. **Check context completeness**: Did the agent have access to all necessary information?
3. **Compare to examples**: If using few-shot examples, do they match the desired behavior?
4. **Verify tool outputs**: Did tools return correct data that the agent then misinterpreted?

**Common root causes**:
- Ambiguous success criteria in prompt
- Context window exceeded (lost early instructions)
- Examples contradicted by instructions
- Model capability insufficient for task complexity

### Step 2B: Stuck / Looping

The agent repeats actions without progress or enters infinite loops.

1. **Check exit conditions**: Is there a clear completion criterion?
2. **Review tool feedback**: Are tool errors being ignored?
3. **Examine retry logic**: Is the agent retrying the same failed action?
4. **Look for oscillation**: Is the agent alternating between two states?

**Common root causes**:
- No explicit stopping condition
- Tool returning same error without useful feedback
- Agent misinterpreting partial success as failure
- Context pollution from repeated attempts

### Step 2C: Stopped Too Early

The agent declared completion prematurely.

1. **Check success criteria**: Are completion conditions too easily satisfied?
2. **Review final state**: Did the agent miss steps in a multi-step process?
3. **Examine context usage**: Did the task description get truncated?

**Common root causes**:
- Vague completion criteria ("do your best")
- Long task list in prompt, model stopped after early items
- Context window filled, later instructions dropped
- Model chose easier interpretation of ambiguous requirements

### Step 2D: Wrong Tool Selection

The agent chose an inappropriate tool for the task.

1. **Check tool descriptions**: Are tools clearly distinguished?
2. **Review similar tools**: Are there tools with overlapping purposes?
3. **Examine tool availability**: Was the correct tool available?
4. **Look for capability assumptions**: Did the agent assume a tool could do something it couldn't?

**Common root causes**:
- Tool descriptions don't differentiate use cases
- Too many similar tools creating decision fatigue
- Correct tool not in allowed tools list
- Tool name or description misleading

### Step 2E: Hallucination

The agent fabricated information not present in context.

1. **Verify context provided**: Was the needed information actually in context?
2. **Check retrieval**: If using RAG, did retrieval return relevant results?
3. **Review confidence signals**: Did the agent express certainty about made-up facts?
4. **Examine pressure**: Was the agent pressured to answer despite missing info?

**Common root causes**:
- Information genuinely missing from context
- Context too long, relevant info buried
- Prompt doesn't allow "I don't know" responses
- Similar but different information confused the model

### Step 2F: Crash / Error

The agent or tooling produced an error.

1. **Read the error message**: What does it actually say?
2. **Check tool configuration**: Are tools properly configured?
3. **Review resource limits**: Token limits? Rate limits? Timeouts?
4. **Examine input validation**: Did invalid input cause the error?

**Common root causes**:
- API rate limits or authentication issues
- Context window overflow
- Tool returned unexpected format
- Environment configuration missing

---

## Common Failure Modes

### Context Overflow

**Symptoms**: Agent forgets early instructions, misses requirements mentioned at the start of a long conversation, produces outputs that ignore constraints specified initially.

**Diagnosis**:
- Check total token count vs model context limit
- Look for "primacy bias" reversal (late tokens dominate)
- Note if failures correlate with conversation length

**Root causes**:
- Context budget exhausted by tool outputs
- Long system prompts combined with long conversations
- Accumulated context from multi-turn interactions

**Fixes**:
- Summarize intermediate results instead of preserving raw outputs
- Move critical instructions to end of prompt (recency bias)
- Implement context pruning strategies
- Split into multiple smaller agent calls

### Tool Errors

**Symptoms**: Agent reports tool failures, produces partial results, or works around tool limitations in unexpected ways.

**Diagnosis**:
- Check tool return values in logs
- Verify tool authentication and configuration
- Test tools in isolation outside agent context

**Root causes**:
- Tool misconfiguration (API keys, endpoints)
- Tool returning error messages agent can't interpret
- Tool success format different than agent expects
- Race conditions in async tool calls

**Fixes**:
- Add structured error handling in tool implementations
- Provide clear error messages with remediation hints
- Validate tool outputs before returning to agent
- Add retry logic with exponential backoff

### Hallucination

**Symptoms**: Agent states facts not in context, makes up file paths or function names, fabricates API responses, invents prior conversation history.

**Diagnosis**:
- Compare agent claims to actual context provided
- Check retrieval results if using RAG
- Look for "confident but wrong" patterns

**Root causes**:
- Information genuinely not in context (expected knowledge)
- RAG retrieval returning irrelevant results
- Similar information creating confusion
- Pressure to answer without "I don't know" escape hatch

**Fixes**:
- Explicitly state what information is NOT available
- Add "if information is not in context, say so" instruction
- Improve retrieval quality and relevance filtering
- Use verification tools (agent can check its own claims)

### Instruction Drift

**Symptoms**: Agent follows instructions accurately at first, gradually deviates over extended conversations, ignores constraints that were respected earlier.

**Diagnosis**:
- Compare early vs late behavior in same conversation
- Check if critical instructions are near prompt start
- Look for context approaching limits

**Root causes**:
- Context filling with conversation history
- Critical instructions too far from active context
- Competing instructions accumulated over time

**Fixes**:
- Repeat key constraints at intervals
- Use structured conversation resets
- Summarize and restart for long workflows
- Place critical rules in multiple prompt positions

### Tool Selection Confusion

**Symptoms**: Agent chooses suboptimal tools, oscillates between similar tools, uses tool A when tool B would be more appropriate.

**Diagnosis**:
- Review available tools and their descriptions
- Check for overlapping tool capabilities
- Examine if tool descriptions distinguish use cases

**Root causes**:
- Too many tools with unclear differentiation
- Tool descriptions emphasize capabilities over use cases
- No guidance on tool selection strategy

**Fixes**:
- Reduce number of available tools
- Rewrite tool descriptions with clear "when to use" guidance
- Add tool selection hints in system prompt
- Use tool groups/categories to organize options

### Premature Termination

**Symptoms**: Agent stops before completing all steps, declares success when only partially done, skips items in a list.

**Diagnosis**:
- Check completion criteria in prompt
- Review what the agent reported as "done"
- Look for truncation in task lists

**Root causes**:
- Vague or easily-satisfied completion criteria
- Long task lists where model attention fades
- Implicit rather than explicit requirements

**Fixes**:
- Add explicit completion checklist
- Use numbered steps with "confirm each step complete"
- Require verification of completion conditions
- Break long task lists into phases

### Quality Gate Failures

*[2026-02-06]*: **Symptoms**: Agent progresses through multi-phase workflow despite incomplete artifacts, ambiguous specifications, or missing dependencies. Downstream phases fail because upstream artifacts weren't validated.

**Diagnosis:**
- Review phase transition points—was artifact complete before progression?
- Check artifact quality—are completion criteria met?
- Examine dependencies—does this phase reference things that don't exist?

**Root causes:**
- No validation between workflow phases
- Completion criteria too vague or easily satisfied
- Pressure to ship fast bypasses quality checks
- Agent unaware of artifact requirements for next phase

**Fixes:**

**Implement adversarial review gates:**
```markdown
# Between phases, add critical examination step

Phase 1: Research → Generate research summary
    ↓
Quality Gate: Is research summary complete?
- [ ] All dependencies identified
- [ ] Root cause documented with evidence
- [ ] Constraints explicitly stated
- [ ] Recommended approach justified
    ↓
If incomplete → return to Research
If complete → proceed to Plan
```

**Add artifact validation prompts:**
```markdown
Before accepting Research output:
- Does this answer "what problem are we solving?"
- Are all file paths/dependencies concrete (not "probably in auth.py")
- Can the next phase execute from this artifact alone?
```

**Example validation logic:**
```python
def validate_research_artifact(artifact: str) -> tuple[bool, str]:
    """Validate research artifact before allowing Plan phase."""
    checks = {
        "root_cause": "Root cause:" in artifact,
        "dependencies": "Dependencies:" in artifact,
        "evidence": any(marker in artifact for marker in ["/path/", "line ", ".py"]),
        "recommendation": "Recommend" in artifact or "Approach" in artifact
    }

    if not all(checks.values()):
        missing = [k for k, v in checks.items() if not v]
        return False, f"Incomplete research: missing {', '.join(missing)}"

    return True, "Research artifact complete"
```

**BMAD-METHOD implementation:**
BMAD uses adversarial review as XML task (one of 4 core framework tasks). Orchestrator critically examines artifacts before phase progression, catching incomplete PRDs, ambiguous user stories, and missing architectural decisions before they cascade into thousands of lines of wrong code.

**Why this matters:**
Bad research compounds exponentially. A 10-minute research mistake can create a 10-week refactoring disaster. Quality gates prevent cascade failures by validating artifacts when mistakes are cheap to fix (specification phase) rather than expensive (post-implementation).

**Trade-offs:**
- **Slower progression** through phases (validation takes time)
- **Higher quality output** (prevents compounding errors)
- **Better debugging** (clear failure point when gates reject artifacts)

**When to use quality gates:**
- Multi-phase workflows (Research → Plan → Build)
- High-cost-of-failure scenarios (production systems, regulated industries)
- Complex domains where upfront validation saves downstream rework
- Team environments where quality standards must be enforced

**When to skip:**
- Simple single-phase tasks (no handoff, no cascade risk)
- Prototyping (speed over quality initially)
- Well-understood domains (team expertise reduces validation need)

### State Confusion in Multi-Agent Systems

**Symptoms**: Agent acts on outdated information, conflicts between parallel agents, inconsistent state after handoffs.

**Diagnosis**:
- Trace state passing between agents
- Check timestamp of context provided to each agent
- Verify state serialization/deserialization

**Root causes**:
- State not passed completely between agents
- Parallel agents modifying shared state
- Stale context provided to downstream agents

**Fixes**:
- Use explicit state objects passed between agents
- Implement read/write locks for shared state
- Refresh context before critical decisions
- Add state verification steps at handoffs

---

## Debugging Tools and Techniques

### Structured Logging

Effective agent debugging requires structured logs at decision points, not just outcomes.

**What to log**:

```
[DECISION] Tool selection
  - Available tools: [list]
  - Chosen tool: {tool}
  - Selection reason: {from model output if available}

[TOOL_CALL] {tool_name}
  - Input: {parameters}
  - Output: {result}
  - Duration: {ms}
  - Status: {success/error}

[STATE] Context checkpoint
  - Token count: {current}/{max}
  - Key facts in context: [summary]
  - Turn number: {n}

[VALIDATION] Output check
  - Expected format: {schema}
  - Actual format: {observed}
  - Valid: {yes/no}
```

**Log analysis patterns**:
- Search for tool calls that precede failures
- Track token count growth through conversation
- Compare successful vs failed runs for divergence points

### Minimal Reproduction

When debugging, create the smallest case that reproduces the failure.

**Reduction process**:
1. Start with the full failing case
2. Remove context elements one at a time
3. Simplify the prompt while preserving failure
4. Reduce tool set to minimum needed
5. Document the minimal reproduction case

**Benefits**:
- Faster iteration on fixes
- Clearer root cause identification
- Portable test case for regression prevention

### A/B Testing Prompts

When the cause is unclear, test variations systematically.

**Structure**:
```
Base case: [original prompt]
Variation A: [change one element]
Variation B: [change different element]

Run each variation N times (N >= 5)
Compare success rates
```

**What to vary**:
- Instruction ordering
- Specificity of requirements
- Format of examples
- Position of constraints (beginning vs end)

### Context Inspection

Examine exactly what the agent saw.

**Techniques**:
- Dump full context at failure point
- Highlight what was present vs missing
- Check token counts for each context section
- Verify encoding/special characters

**Questions to answer**:
- Was the needed information in context?
- Where in the context was it? (position matters)
- What other information competed for attention?
- Did formatting obscure important content?

### Trace Replay

For complex multi-step failures, replay the trace step by step.

**Process**:
1. Capture full execution trace
2. Identify the step where behavior first diverged
3. Extract context as of that step
4. Re-run from that point with variations
5. Isolate the specific decision that went wrong

**Useful for**:
- Multi-agent coordination failures
- Long workflow debugging
- Intermittent failures that are hard to reproduce

### Model Comparison

When suspecting model capability, test across models.

| If behavior... | Suggests... |
|---------------|-------------|
| Fails on all models | Prompt/context issue, not model |
| Fails only on smaller models | Task complexity exceeds model capability |
| Fails inconsistently on same model | Edge case or prompt ambiguity |
| Succeeds on API but fails embedded | Environment/configuration issue |

**Caution**: Model comparison is expensive. Use only after ruling out prompt, context, and tool issues.

---

## Anti-Patterns

### Debugging Without Logs
**What it looks like**: Agent fails. The engineer stares at the prompt and tries to guess what went wrong. Changes are made based on intuition rather than data.

**Why it fails**:
- Debugging the theory of what happened, not what actually happened
- Intermittent failures are impossible to diagnose
- Can't distinguish "agent didn't try" from "agent tried and failed"
- Waste time fixing problems that don't exist

**Better alternative**: Instrument before debugging. Add logging at decision points, tool calls, and validation gates. Use event logs to trace execution. Read agent outputs and intermediate states. Debug the data, not the mental model.

### Changing Multiple Things at Once
**What it looks like**: Agent isn't working right. The engineer simultaneously changes the prompt structure, swaps the model, adjusts the context window, and modifies the tool permissions. One of those fixes it—but which one?

**Why it fails**:
- Can't identify root cause
- Might have fixed one thing but broken another
- No reproducible solution to apply elsewhere
- Regression hunting becomes impossible

**Better alternative**: Change one variable at a time. Make a hypothesis, change only what tests that hypothesis, verify the impact. If that didn't work, revert and try the next hypothesis. Keep notes on what was tried.

### Not Reproducing Before Fixing
**What it looks like**: Agent failed once. The engineer makes a change without verifying the original failure can be reproduced or that the fix prevents it.

**Why it fails**:
- "Fix" might do nothing, failure was intermittent
- Can't tell if future failures are regressions or new issues
- No confidence the problem is actually solved
- Accumulate superstitious fixes that don't address real problems

**Better alternative**: First, reproduce the failure reliably (even if just 2-3 times in a row). Document the repro steps. Then make the fix. Then verify the fix prevents the failure in 5+ consecutive runs. Without reproduction, there's no verification.

### Assuming It's the Model
**What it looks like**: Agent produces wrong output. Model capability gets blamed immediately: "Haiku isn't smart enough for this" or "Opus hallucinates here." Switch to bigger/newer model without investigating further.

**Why it fails**:
- Most agent failures are prompt, context, or tool issues, not model capability
- Bigger models are slower and more expensive
- Doesn't fix the actual problem—new model often fails the same way
- Misses learning opportunity about prompt design

**Better alternative**: Check the other "core four" first. Is the instruction clear? Does the agent have the right context? Are tools working correctly? Is the output format well-specified? Model capability is usually the last resort, not the first suspect.

### Debugging in Production
**What it looks like**: Agent fails in production. The engineer modifies the production prompt to add debugging output or try different approaches. Iteration happens on the live system.

**Why it fails**:
- Every iteration affects real users or systems
- Can't experiment freely without consequences
- Pressure leads to quick hacks instead of root cause fixes
- Lost changes if rollback is needed

**Better alternative**: Reproduce the failure locally or in a dev environment. Use production logs to understand the failure mode, but fix and test in isolation. Only deploy to production after verifying the fix in a safe environment.

### Ignoring Partial Successes
**What it looks like**: Agent works 70% of the time, fails 30%. Focus goes only to the failures, trying to fix what went wrong. Successes are never analyzed for what makes them different.

**Why it fails**:
- Successes often show what conditions enable correct behavior
- Failure analysis alone doesn't reveal missing prerequisites
- Can't identify environmental or input patterns that matter
- Miss the insight that "it works when X is true"

**Better alternative**: Compare successes to failures. What's different about the inputs, context, or environment when it works? Often the pattern is "succeeds when context includes Y" or "fails when input is ambiguous." This points directly to the fix.

### The Infinite Retry Loop
**What it looks like**: Add retry logic to handle intermittent failures. Agent fails, retries, fails, retries... infinitely, never surfacing the underlying issue.

**Why it fails**:
- Masks systemic problems with tactical patches
- Wastes tokens and time on doomed attempts
- No signal to humans that something is fundamentally broken
- Logs fill with noise, hiding real issues

**Better alternative**: Limit retries (2-3 max). On persistent failure, escalate to human or halt with detailed error. Log each retry with context. If retries are frequently triggered, that's a signal to fix the root cause, not increase the retry limit.

---

## Debugging War Stories

*Document specific debugging sessions—what happened, how you figured it out:*

### Multi-Agent Debugging Strategies

*[2025-12-09]*: Multi-agent systems have distinct failure modes that require different debugging approaches than single-agent systems.

**Check Single-Message Parallelism First**

When parallel agents run slower than expected or produce suspiciously sequential results, check whether they were invoked in a single message. This is the most common multi-agent debugging issue—developers expect parallelism but the framework serializes execution because Task calls were spread across multiple messages. Look at your orchestrator's outputs: are all parallel Task calls in one response?

**Graceful Degradation Patterns**

Multi-agent systems should degrade gracefully when individual agents fail:
- If one expert fails, note the failure and continue with available analyses
- Recommend manual review for failed expert domains
- Include recovery instructions in output
- Don't let one failing subagent tank the entire workflow

Implement this by checking subagent return status before synthesis. The orchestrator should be resilient to partial failures.

**Partial Success Handling**

Unlike single-agent (which either works or doesn't), multi-agent systems have complex success states:
- Commit successful changes before reporting failures
- Allow selective retry via phase parameters
- Never leave the workflow in an inconsistent state
- Track which subagents succeeded vs. failed

**Use Hooks to Trace Agent Transitions**

SubagentStop hooks record what each subagent produced. ErrorEscalation surfaces failures to the orchestrator. These create an audit trail for debugging:
- Which subagent ran when?
- What did it return?
- Where did it fail?

Without these hooks, multi-agent failures are nearly impossible to diagnose.

**Domain Isolation Debugging**

Each agent stays focused on its domain and doesn't need to know about other agents. If you see cross-domain confusion, something is leaking context incorrectly. The orchestrator is responsible for synthesis—individual experts should be testable in isolation.

**Coordination vs. Agent Failures**

Two distinct failure modes:
1. **Agent failure**: Individual subagent does wrong thing (debug like single-agent)
2. **Coordination failure**: Orchestrator routes incorrectly, synthesizes badly, or mismanages state

For coordination failures, the problem is usually in the orchestrator prompt or the spec file, not in the subagents themselves. Test subagents in isolation to rule out agent-level issues before debugging coordination.

**Sources**: [Orchestrator Pattern](../6-patterns/3-orchestrator-pattern.md), [Building agents with the Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk)

---

## Testing and Debugging Hooks

Hooks are automation that runs at specific events in your workflow. When they break, they're invisible—the system just doesn't behave as expected. Here's how to diagnose and fix hook issues.

### Manual Hook Testing

Test hooks outside of Claude Code before deploying them. This isolates hook logic from the rest of the system.

**Basic Testing Pattern:**

```bash
# Pipe JSON to stdin
echo '{"prompt": "/orchestrators:test"}' | python .claude/hooks/orchestrator_context.py

# Check exit code
echo $?

# Test with project environment variables
cd /path/to/project
CLAUDE_PROJECT_DIR=$(pwd) echo '{"tool_name": "Write"}' | python .claude/hooks/orchestrator_guard.py
```

**Testing with Realistic Input:**

Create test JSON files that mirror what Claude Code actually sends:

```bash
# test-pretool-input.json
{
  "sessionId": "test-123",
  "projectDir": "/Users/jayminwest/Projects/agentic-engineering-knowledge-base",
  "tool_name": "Write",
  "parameters": {
    "file_path": "/path/to/file.md",
    "content": "test content"
  }
}

# Test the hook
cat test-pretool-input.json | python .claude/hooks/orchestrator_guard.py
```

**Verifying Exit Codes:**

Exit codes control hook behavior:
- **Exit 0**: Success, allow operation
- **Exit 2**: Block operation, show stderr to Claude
- **Other**: Non-blocking error, visible in debug mode only

```bash
# Test blocking behavior
cat test-input.json | python hook.py
if [ $? -eq 2 ]; then
  echo "Hook correctly blocked the operation"
fi
```

**Testing Environment File Interaction:**

For hooks that read/write `CLAUDE_ENV_FILE`, test with a temporary file:

```bash
# Create temp environment file
export CLAUDE_ENV_FILE=$(mktemp)

# Run hook that writes to env file
echo '{"prompt": "/orchestrators:knowledge test"}' | python .claude/hooks/orchestrator_context.py

# Verify what was written
cat "$CLAUDE_ENV_FILE"

# Clean up
rm "$CLAUDE_ENV_FILE"
```

### Debugging Hook Issues

**Use Debug Mode:**

Run Claude Code with the `--debug` flag to see detailed hook execution:

```bash
claude --debug
```

This shows:
- Which hooks are triggered
- stdin/stdout/stderr content
- Exit codes
- Execution timing
- Decision results

**Use `/hooks` Command:**

Inside Claude Code, run `/hooks` to see all registered hooks and their configuration. This confirms:
- Hook is properly registered
- Matcher patterns are correct
- Hook file paths are valid
- Multiple hooks aren't conflicting

**Common Hook Issues:**

**1. Hook Not Running:**

Check matcher patterns. If you have:

```json
{
  "matcher": "Write",
  "type": "command",
  "command": "python hook.py"
}
```

This only matches the exact tool name "Write". It won't match "Edit" or "MultiEdit". Use wildcards for broader matching:

```json
{
  "matchers": ["Write", "Edit", "MultiEdit"]
}
```

**2. Timeout Errors:**

Default timeouts may be too short for complex hooks. Increase the timeout:

```json
{
  "type": "command",
  "command": "python slow-hook.py",
  "timeout": 30000
}
```

If timeouts persist, optimize the hook script. Hooks should complete in seconds, not minutes.

**3. JSON Parse Errors:**

Hooks must handle malformed JSON gracefully:

```python
try:
    input_data = json.load(sys.stdin)
except json.JSONDecodeError:
    # Don't crash - allow operation to continue
    sys.exit(0)
```

Crashing on bad JSON breaks Claude Code's workflow. Exit 0 to allow operations when you can't parse input.

**4. Permission Denied:**

Hook scripts must be executable:

```bash
chmod +x .claude/hooks/orchestrator_context.py
```

If using `#!/usr/bin/env -S uv run --script`, ensure `uv` is installed and in PATH.

### Verifying Orchestrator Enforcement

The orchestrator pattern uses two hooks working together:
1. `orchestrator_context.py` (UserPromptSubmit) - Sets context when orchestrator commands run
2. `orchestrator_guard.py` (PreToolUse) - Blocks Write/Edit tools in orchestrator context

**Verify Context is Set:**

When you run `/orchestrators:knowledge` or `/questions:orchestrator`, check that the context is set:

```bash
# During a Claude Code session, in another terminal:
cat "$HOME/.claude/env/session-<id>.env"

# Should contain:
# export CLAUDE_ORCHESTRATOR_CONTEXT="knowledge"
```

The session ID can be found via `/hooks` output or by listing `~/.claude/env/` files sorted by modification time.

**Verify Tools are Blocked:**

When orchestrator context is active, attempting to use Write or Edit should fail with a clear error message:

```
BLOCKED: Orchestrator 'knowledge' cannot use Write directly.
Delegate to a build agent using the Task tool instead.
Example: Task(subagent_type='knowledge-build-agent', prompt='...')
```

If you're not seeing this error when you should, check:
1. Is `CLAUDE_ORCHESTRATOR_CONTEXT` actually set in the env file?
2. Is the PreToolUse hook registered? (use `/hooks` to verify)
3. Is the hook script executable and working? (test manually)

**What to Look For in Error Messages:**

Good hook errors should:
- Identify the problem clearly: "BLOCKED: Orchestrator 'X' cannot use Y"
- Explain why: "Delegate to a build agent"
- Show how to fix it: "Example: Task(...)"

If errors are vague or missing, add better error messages to your hooks. Write to stderr for blocking errors:

```python
print(
    f"BLOCKED: Orchestrator '{orchestrator}' cannot use {tool_name} directly.\n"
    f"Delegate to a build agent using the Task tool instead.\n"
    f"Example: Task(subagent_type='knowledge-build-agent', prompt='...')",
    file=sys.stderr
)
sys.exit(2)
```

### Hook Logging Pattern

Hooks are hard to debug because they're invisible during normal operation. Add logging to understand what's happening.

**Logging to File:**

Don't use stdout (it becomes context for Claude). Write to a log file instead:

```python
import json
import sys
from pathlib import Path
from datetime import datetime

LOG_FILE = Path.home() / ".claude" / "logs" / "orchestrator_guard.log"

def log_hook_execution(message: str, data: dict = None):
    """Log hook execution for debugging."""
    LOG_FILE.parent.mkdir(parents=True, exist_ok=True)

    timestamp = datetime.now().isoformat()
    log_entry = f"[{timestamp}] {message}"

    if data:
        log_entry += f"\n{json.dumps(data, indent=2)}"

    with LOG_FILE.open("a") as f:
        f.write(log_entry + "\n\n")

def main():
    try:
        input_data = json.load(sys.stdin)
        tool_name = input_data.get("tool_name", "")

        log_hook_execution("PreToolUse triggered", {
            "tool": tool_name,
            "orchestrator": get_orchestrator_context()
        })

        # ... rest of hook logic

    except Exception as e:
        log_hook_execution(f"Hook error: {e}")
        sys.exit(0)
```

**What to Log:**

Log decision points, not every line:
- Hook triggered (with input summary)
- Context detected (orchestrator name, environment state)
- Decisions made (allow/block, why)
- Errors encountered

**Reviewing Logs:**

Tail the log during a Claude Code session:

```bash
tail -f ~/.claude/logs/orchestrator_guard.log
```

This shows hook execution in real-time, helping you understand:
- Which hooks are actually running
- What input they're receiving
- What decisions they're making
- Where errors occur

**Log Rotation:**

Hook logs can grow large. Implement basic rotation:

```python
def rotate_log_if_needed():
    """Keep log file under 1MB."""
    if LOG_FILE.exists() and LOG_FILE.stat().st_size > 1_000_000:
        # Move old log to backup
        backup = LOG_FILE.with_suffix(".log.old")
        LOG_FILE.rename(backup)
```

**Conditional Logging:**

Enable detailed logging only when debugging:

```python
DEBUG = os.environ.get("CLAUDE_HOOK_DEBUG", "false") == "true"

def log_hook_execution(message: str, data: dict = None):
    if not DEBUG:
        return
    # ... logging code
```

Then enable when needed:

```bash
export CLAUDE_HOOK_DEBUG=true
claude --debug
```

This keeps logs clean during normal operation while providing detail when troubleshooting.

---

## Case Studies

### Case Study 1: The Disappearing Instructions

**Symptom**: Code generation agent produced output that violated explicit constraints. The constraint "never use external dependencies" was clearly stated, but the agent added `import requests` to the code.

**Investigation**:
1. Checked the prompt—constraint was present at line 5 of a 200-line system prompt
2. Estimated token count: system prompt (4k) + conversation history (12k) + code context (8k) = 24k tokens
3. Constraint was in the first 3% of context
4. Model exhibited recency bias—late tokens dominated attention

**Root cause**: Context overflow creating effective instruction loss. The constraint existed but was too far from active generation for the model to weight it appropriately.

**Fix**:
- Moved critical constraints to the end of the system prompt
- Added constraint reminders in user messages
- Implemented a "constraints" section that gets repeated before code generation

**Lesson**: Position matters. Critical instructions near the end of context get more attention than those at the beginning in long contexts.

### Case Study 2: The Tool Selection Oscillation

**Symptom**: Agent alternated between `file_search` and `web_search` for the same query, never settling on one approach. Task timed out after 15 iterations.

**Investigation**:
1. Logged tool selection reasoning
2. Both tools returned partial results
3. Agent interpreted partial results as "wrong tool chosen"
4. Each tool's partial success triggered switching to the other

**Root cause**: No guidance on tool selection criteria. Tools had similar descriptions. Agent had no way to determine which was "correct" when both returned something.

**Fix**:
- Added explicit tool selection criteria to prompt: "Use file_search for content already in context. Use web_search only for information not in any provided files."
- Added "stick with your choice unless result is empty" instruction
- Implemented a max-switches-per-query limit

**Lesson**: Tool descriptions must include differentiation criteria, not just capability descriptions.

### Case Study 3: The Phantom Hallucination

**Symptom**: Agent confidently cited a function `validate_user_input()` that didn't exist in the codebase. Claimed it was in `auth.py` at line 45.

**Investigation**:
1. Searched codebase—no such function
2. Checked RAG retrieval—returned `validate_input()` from `forms.py`
3. Context also included `user_auth_check()` from `auth.py`
4. Agent combined parts of both into fabricated function name and location

**Root cause**: Conflation of similar-but-different retrieved results. RAG returned relevant but not exact matches; agent synthesized a plausible-sounding but non-existent reference.

**Fix**:
- Added verification step: "Before citing any function, use grep to confirm it exists"
- Improved RAG retrieval to return exact matches when available
- Added instruction: "If you cannot find the exact function, say so rather than approximating"

**Lesson**: Hallucinations often emerge from synthesis of partially-correct information, not pure fabrication. Verification tools prevent confident-but-wrong citations.

### Case Study 4: The Multi-Agent State Drift

**Symptom**: In a three-agent pipeline (scout → planner → builder), the builder produced code that contradicted the planner's specifications. Scout found the right files, planner made a correct plan, builder ignored parts of the plan.

**Investigation**:
1. Traced state passing between agents
2. Scout returned: "Found 3 relevant files: auth.py, middleware.py, routes.py"
3. Planner created spec referencing all three files
4. Builder received spec but only "auth.py" was in its context—the spec mentioned the files but didn't include their contents

**Root cause**: Spec file referenced files by name, but builder didn't have file contents. Builder proceeded with incomplete context rather than requesting missing information.

**Fix**:
- Changed spec format to include file contents, not just names
- Added builder validation: "Verify all referenced files are accessible before proceeding"
- Implemented explicit context handoff verification at phase transitions

**Lesson**: "Mentioned in spec" is not "available in context." State handoffs must include actual content, not references to content.

### Case Study 5: The Intermittent Success

**Symptom**: Identical prompt, identical context, but 60% success rate. Some runs produced correct output, others failed with apparent reasoning errors.

**Investigation**:
1. Compared successful vs failed runs
2. No obvious pattern in inputs
3. Examined model temperature—set to 0.7
4. Tracked reasoning chains: successful runs took different reasoning paths

**Root cause**: Prompt had multiple valid interpretation paths. At temperature 0.7, model sometimes followed the successful path, sometimes followed a plausible but wrong path.

**Fix**:
- Reduced temperature to 0.1 for this task
- Added explicit step-by-step reasoning structure to reduce path variance
- Made success criteria unambiguous

**Lesson**: Intermittent failures with identical inputs often indicate prompt ambiguity amplified by temperature. Low temperature + explicit structure reduces variance.

---

## Debugging Checklist

Quick reference for systematic debugging:

### Before Starting
- [ ] Can the failure be reproduced? (Try 3 times)
- [ ] Is the failure documented? (Inputs, outputs, error messages)
- [ ] Is there logging/tracing available?

### Initial Diagnosis
- [ ] Categorize failure type (wrong output, stuck, early stop, tool error, hallucination, crash)
- [ ] Check which "core four" component is suspect (tools → context → prompt → model)
- [ ] Review recent changes (what was different when it worked?)

### Investigation
- [ ] Examine exact context provided to agent
- [ ] Check token counts vs limits
- [ ] Review tool call inputs and outputs
- [ ] Compare successful vs failed runs

### Fix Verification
- [ ] Change only one variable at a time
- [ ] Verify fix in 5+ consecutive runs
- [ ] Document what was tried and what worked
- [ ] Add logging to detect regression

---

## Expanded Connections

- **To [Evaluation](2-evaluation.md):** Systematic evaluation catches bugs before they become debugging sessions. Eval failures provide reproducible test cases for debugging. Debug fixes should be validated against eval sets.
- **To [Prompt](../2-prompt/_index.md):** Many "agent bugs" are actually prompt bugs. Ambiguous instructions, missing constraints, and unclear formats cause predictable failures. Debug the prompt before debugging the agent.
- **To [Context](../4-context/_index.md):** Context issues cause hallucination, instruction drift, and state confusion. Debugging often reveals context wasn't what you thought. Context inspection should be routine.
- **To [Tool Use](../5-tool-use/_index.md):** Tool errors, wrong tool selection, and tool output misinterpretation cause observable failures. Tools are the easiest component to verify in isolation.
- **To [Model Behavior](../3-model/2-model-behavior.md):** Understanding model attention patterns, recency bias, and capability limits informs debugging hypotheses. Model behavior explains why position and format matter.
- **To [Production Concerns](4-production-concerns.md):** Production debugging requires logging, monitoring, and observability infrastructure. Invest in debugging capability before production deployment.
- **To [Orchestrator Pattern](../6-patterns/3-orchestrator-pattern.md):** Multi-agent debugging requires understanding coordination failures vs agent failures. Orchestrator patterns have distinct failure modes documented in this chapter.

---

## Open Questions

- How do you debug emergent behavior in multi-agent systems where no single agent is wrong?
- What's the minimum viable observability for agent systems in production?
- How do you distinguish model capability limits from prompt/context issues without expensive A/B testing?
- Can agents be designed to self-diagnose and report their own failure modes?

---


