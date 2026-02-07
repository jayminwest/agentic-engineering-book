---
title: Multi-Agent Collaboration Pattern
description: Orchestrated real-time collaboration with role-authentic disagreement
created: 2026-02-06
last_updated: 2026-02-06
tags: [patterns, multi-agent, collaboration, orchestration, party-mode]
part: 2
part_title: Craft
chapter: 6
section: 9
order: 2.6.9
---

# Multi-Agent Collaboration Pattern

## Opening Statement

Multi-agent collaboration enables simultaneous input from multiple specialized perspectives in a single conversation, with agents authentically agreeing, disagreeing, and building on each other's ideas while a human guides toward resolution.

## Your Mental Model

**Think "team design meeting" not "sequential handoffs."**

Traditional orchestration runs agents in sequence or parallel but isolated. Multi-agent collaboration creates a shared conversation space where agents interact with each other's ideas. The orchestrator selects 2-3 relevant agents per message (not all agents every time—that creates chaos), agents respond in character, and the human steers the discussion.

## Core Pattern

### The Conversation Flow

```
User: "How should we handle authentication?"
    ↓
Orchestrator selects: Security Expert, Architect, Developer
    ↓
Security Expert: "Use OAuth2 with PKCE flow—industry standard"
Architect: "Agree on OAuth2, but store tokens in Redis not localStorage"
Developer: "That adds Redis dependency—could we use httpOnly cookies?"
    ↓
User: "What about mobile apps?"
    ↓
Orchestrator selects: Security Expert, Developer, UX Designer
    ↓
[Agents respond with mobile-specific perspectives]
```

### Key Characteristics

1. **Selective Participation**: Orchestrator picks 2-3 agents per message based on relevance
2. **Role Authenticity**: Agents maintain persona—security experts worry about threats, developers about maintainability
3. **Genuine Disagreement**: Agents disagree when perspectives conflict, not artificial consensus
4. **Building on Ideas**: Later responses reference earlier responses
5. **Human Steering**: User asks follow-ups, challenges assumptions, redirects focus

## When to Use

### Good Fit

**Complex decisions with multiple tradeoffs:**
- Architecture decisions affecting multiple concerns (security, performance, maintainability)
- Strategic planning requiring diverse perspectives
- Post-mortems analyzing failures from multiple angles
- Design reviews where different expertise matters

**Brainstorming and ideation:**
- Problem definition (what are we solving?)
- Solution exploration (how could we solve it?)
- Risk assessment (what could go wrong?)

**Indicators this pattern fits:**
- No single "right answer"—tradeoffs matter more than correctness
- Multiple stakeholder perspectives needed
- Decision requires synthesis of conflicting concerns
- Exploratory rather than execution-focused

### Poor Fit

**Execution tasks:**
- Code implementation (use Builder pattern)
- Data transformation (single agent sufficient)
- Mechanical refactoring (Autonomous Loops)
- Debugging specific issues (focused expertise beats broad discussion)

**Well-defined problems:**
- Requirements already clear
- Solution approach determined
- No design tradeoffs remaining
- Simple enough that multiple perspectives add noise

## Implementation Patterns

### Agent Selection Strategy

**Fixed selection (BMAD approach):**
```markdown
Available agents: PM, Architect, Developer, QA, UX, Scrum Master

Orchestrator analyzes user message and selects 2-3 relevant agents:
- Technical question → Architect, Developer
- Requirements discussion → PM, Architect, UX
- Process concern → Scrum Master, PM
- Quality discussion → QA, Developer, Architect
```

**Dynamic selection:**
```python
def select_agents(user_message: str, available_agents: list) -> list:
    """Select 2-3 most relevant agents for this message."""
    relevance_scores = score_agent_relevance(user_message, available_agents)
    return top_k(relevance_scores, k=3)
```

### Conversation State Management

**Context accumulation challenge:**
Each agent response adds context. With 5 agents × 3 rounds = 15 responses, context fills quickly.

**Mitigation strategies:**
1. **Selective history**: Pass only last 2-3 exchanges, not full conversation
2. **Summary checkpoints**: After 10 messages, summarize and start fresh
3. **Agent-specific context**: Each agent sees conversation through their lens (e.g., security expert only sees security-relevant exchanges)

### Disagreement Protocol

**Authentic disagreement example:**
```
Security Expert: "Never store tokens in localStorage—XSS vulnerability"
Developer: "I disagree—with proper CSP headers, localStorage is fine and simpler to implement"
Architect: "Both points valid. Security concern is real, but CSP mitigates.
           The question is: do we have CSP infrastructure already?"
```

Agents should:
- State disagreement clearly ("I disagree because...")
- Provide reasoning, not just opinion
- Acknowledge valid points from other agents
- Ask clarifying questions when perspectives conflict

## Concrete Example: BMAD Party Mode

BMAD-METHOD (MIT-licensed, 34.5k stars) demonstrates this pattern in production.

**Trigger:** `/party-mode` command

**Available personas:**
- Mary (Analyst) - Brainstorming, research
- John (PM) - Requirements, roadmaps
- Winston (Architect) - System design
- Amelia (Developer) - Implementation
- Quinn (QA) - Testing strategy
- Bob (Scrum Master) - Process, retrospectives
- Sally (UX) - User experience
- Paige (Technical Writer) - Documentation

**Example session:**
```
User: "We need to add real-time notifications to the app."

BMad Master selects: John (PM), Winston (Architect), Amelia (Developer)

John: "Key question: what events trigger notifications? User mentions?
       System alerts? We need to define the notification taxonomy."

Winston: "Architecturally, we're choosing between WebSockets (persistent
         connection) or Server-Sent Events (simpler, one-way). Scale matters
         here—how many concurrent users?"

Amelia: "WebSockets add complexity—reconnection logic, connection pools,
         horizontal scaling gets tricky. SSE works if we don't need
         bidirectional. What's the client requirement?"

User: "Mostly server-to-client notifications. Scale is ~10k concurrent users."

BMad Master selects: Winston (Architect), Amelia (Developer), Quinn (QA)

Winston: "10k concurrent WebSocket connections is manageable. But SSE
         might be simpler for one-way. Recommendation: prototype both."

Amelia: "Agree. I'd start with SSE—faster to prototype, easier to test."

Quinn: "From testing perspective, SSE is definitely easier—less state to
       manage in test harness. WebSockets require connection lifecycle
       testing."
```

**Key observations:**
- BMad Master acts as orchestrator, selecting relevant agents per message
- Agents maintain role consistency (architect thinks scale, developer thinks implementation complexity)
- Discussion builds toward resolution without forced consensus
- User can redirect by asking domain-specific questions

**Implementation details:**
- Agent selection via fuzzy matching on conversation content
- 2-3 agents per response (not all agents)
- v6.0.0 made party mode universally available across all agents
- Can combine with any workflow (party mode for planning, single agent for execution)

## Comparison with Related Patterns

### vs. Orchestrator Pattern

| Multi-Agent Collaboration | Orchestrator Pattern |
|---------------------------|---------------------|
| **Conversation space** - agents respond to each other | **Isolated execution** - agents work independently |
| **Sequential dialogue** - turn-based discussion | **Parallel execution** - simultaneous task completion |
| **Exploratory** - finding answers through discussion | **Execution-focused** - implementing known approach |
| **Human-guided** - user steers conversation | **Autonomous** - orchestrator decides workflow |

**When to choose:**
- Multi-Agent Collaboration: Design phase, unclear requirements, need multiple perspectives
- Orchestrator: Implementation phase, clear requirements, parallel execution needed

### vs. Expert Swarm Pattern

| Multi-Agent Collaboration | Expert Swarm |
|---------------------------|--------------|
| **Shared conversation** - agents aware of each other | **Independent analysis** - agents work in isolation |
| **Synthesis through dialogue** - resolution emerges | **Synthesis after completion** - coordinator merges results |
| **Real-time steering** - user redirects during conversation | **Batch processing** - specification defined upfront |

**When to choose:**
- Multi-Agent Collaboration: Uncertain problem space, need to explore tradeoffs
- Expert Swarm: Known domain, need comprehensive coverage, can define upfront

### vs. Human-in-the-Loop

| Multi-Agent Collaboration | Human-in-the-Loop |
|---------------------------|-------------------|
| **Continuous involvement** - human guides every step | **Approval gates** - human validates at checkpoints |
| **Steering mechanism** - human shapes direction | **Validation mechanism** - human verifies correctness |
| **Exploratory tasks** - figuring out what to do | **Execution tasks** - doing what's defined |

**When to choose:**
- Multi-Agent Collaboration: Complex decision-making, brainstorming, design discussions
- Human-in-the-Loop: High-stakes execution, compliance validation, sensitive operations

## Orchestrator Design Considerations

### Selection Heuristics

**Content analysis:**
- Technical keywords → Architect + Developer
- "user" or "UX" → UX + PM
- "test" or "quality" → QA + Developer
- "process" or "workflow" → Scrum Master + PM

**Conversation history:**
- If recent exchanges involved security → keep Security Expert active
- If previous agents disagreed → include both in next round to resolve
- If new topic introduced → select fresh relevant agents

**Participation limits:**
- Never select all agents (creates noise)
- Never select fewer than 2 (defeats collaboration purpose)
- Ideal: 2-3 agents per message
- Allow agents to "pass" if not relevant to current question

### Synthesis Strategy

After 5-10 rounds of dialogue, orchestrator should synthesize:

```markdown
## Synthesis

**Consensus points:**
- Use OAuth2 for authentication (all agents agree)
- Redis for session storage (Security + Architect recommendation)

**Open tradeoffs:**
- Developer concerned about Redis operational complexity
- Architect notes Redis enables horizontal scaling we'll need

**Recommendation:**
Use OAuth2 + Redis, with developer creating operational runbook
for Redis management to address complexity concern.

**Next steps:**
1. Developer: Prototype OAuth2 + Redis integration
2. Architect: Document scaling approach
3. PM: Define monitoring requirements
```

## Context Management

**Challenge:** Multi-agent conversations consume context rapidly.

**Token economics example:**
- 10 messages × 3 agents/message × 200 tokens/response = 6,000 tokens
- Add user messages (10 × 50 tokens) = 500 tokens
- System prompts for 6 agents (6 × 400 tokens) = 2,400 tokens
- **Total: 8,900 tokens for modest 10-message conversation**

**Strategies:**

1. **Compressed history**: After 5 messages, summarize earlier discussion
2. **Agent-specific views**: Security expert doesn't need full UX discussion
3. **Progressive dropout**: Less relevant agents stop participating as conversation focuses
4. **Periodic checkpoints**: Synthesize every N messages, start fresh from synthesis

## Evaluation Criteria

How to measure if multi-agent collaboration is working:

**Process metrics:**
- **Participation balance**: No agent dominates (ideal: even distribution ±20%)
- **Disagreement rate**: 20-40% of responses include disagreement (too low = groupthink, too high = unproductive conflict)
- **Resolution rate**: Conversations converge toward decisions (not endless debate)

**Outcome metrics:**
- **Decision quality**: Post-implementation review of decisions made
- **Blind spot detection**: Did collaboration catch issues single-agent approach missed?
- **Time to decision**: Compare with sequential expert consultation

**Qualitative assessment:**
- Are agents maintaining role authenticity?
- Do disagreements feel genuine and valuable?
- Does conversation lead to insights vs. noise?

## Anti-Patterns

### All Agents, Every Message

**What it looks like:** Every message triggers responses from all available agents.

**Why it fails:**
- Information overload (8 agents × 200 tokens = 1,600 tokens per round)
- Irrelevant perspectives create noise
- Context window exhausts rapidly
- Signal-to-noise ratio plummets

**Fix:** Orchestrator selects 2-3 most relevant agents per message.

### Forced Consensus

**What it looks like:** Orchestrator pushes agents to agree, or agents always agree to avoid conflict.

**Why it fails:**
- Real tradeoffs get hidden
- Best solution may require accepting one perspective over another
- Misses opportunity to explore tension between valid concerns

**Fix:** Encourage authentic disagreement. Orchestrator synthesizes tradeoffs, doesn't force resolution.

### Human Passenger

**What it looks like:** User asks initial question, then watches agents debate without steering.

**Why it fails:**
- Agents can't read user's priorities
- Conversation meanders without direction
- Misses human judgment on which concerns matter most

**Fix:** User actively steers—"focus on security for now," "what if we prioritize simplicity?"

### No Synthesis

**What it looks like:** Conversation generates valuable perspectives but never converges to actionable output.

**Why it fails:**
- Exploration without resolution wastes effort
- Team left without decision
- Can't transition to execution phase

**Fix:** Orchestrator synthesizes after reasonable discussion depth (5-10 exchanges), surfaces consensus + open tradeoffs + recommendation.

## Implementation Checklist

Planning to implement this pattern? Consider:

- [ ] Define agent personas with distinct perspectives
- [ ] Create agent selection heuristics based on conversation content
- [ ] Implement 2-3 agent per message limit
- [ ] Add context management (summarization or selective history)
- [ ] Enable authentic disagreement in agent prompts
- [ ] Design synthesis mechanism for convergence
- [ ] Add conversation length limits (auto-synthesize after N messages)
- [ ] Test with real design discussions to tune relevance matching
- [ ] Monitor token consumption and adjust as needed

## Open Questions

- What's the optimal conversation length before synthesis is required?
- Can agent selection be learned from conversation effectiveness data?
- How to handle situations where agents genuinely cannot resolve disagreement?
- Does conversation ordering matter (which agent responds first)?
- Can this pattern scale beyond 5-6 distinct personas before becoming unwieldy?

## Connections

- **To [Orchestrator Pattern](3-orchestrator-pattern.md):** Multi-agent collaboration uses orchestration infrastructure but applies it to collaborative dialogue instead of parallel execution
- **To [Human-in-the-Loop](6-human-in-the-loop.md):** Continuous human steering is central to this pattern—humans guide every conversational turn
- **To [Context Management](../4-context/_index.md):** Rapid context consumption requires aggressive management—summarization, selective history, and periodic resets
- **To [Prompt Engineering](../2-prompt/_index.md):** Agent personas must be prompted to maintain role authenticity and enable genuine disagreement

## Sources

- [BMAD-METHOD GitHub Repository](https://github.com/bmad-code-org/BMAD-METHOD) - Open source implementation
- [Party Mode Documentation](https://docs.bmad-method.org/explanation/party-mode/) - BMAD's approach
- BMAD-METHOD Scout Report (`.claude/.cache/research/external/bmad-method-scout-report.md`)
