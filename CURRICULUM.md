# Agentic Engineering Curriculum

This curriculum provides structured learning paths for developing competency in agentic engineering. Choose the path that matches your background and goals.

**How to use this curriculum:**
- Start with the self-assessment to identify your current level
- Follow the recommended path for your persona
- Complete self-assessment checkpoints at each stage
- Work through example projects to validate mastery
- Use the problem-driven index when debugging specific issues

**Related Resources:**
- [Table of Contents](TABLE_OF_CONTENTS.md) - Complete book structure
- [Preface](PREFACE.md) - Book introduction and philosophy
- [Style Guide](STYLE_GUIDE.md) - Writing conventions and voice standards

---

## Choose Your Learning Path

Select the path that best matches your current experience:

| Path | Best For | Time Commitment | Project Outcome |
|------|----------|----------------|-----------------|
| [Beginner](#beginner-path-new-to-ai-engineering) | New to AI engineering, limited LLM experience | 8-12 hours over 2-3 weeks | Single-agent system with tools |
| [Developer](#developer-path-ai-experience) | Experience with LLMs, limited agentic systems | 6-10 hours over 1-2 weeks | Multi-agent coordination system |
| [Practitioner](#practitioner-path-experienced) | Production agentic systems experience | 4-6 hours targeted deep-dives | Meta-framework or contribution |
| [Problem-Driven](#problem-driven-index) | Debugging specific issues right now | As needed | Specific problem resolution |

**Recommendation:** If unsure, start with the [self-assessment](#self-assessment-where-should-i-start) to validate your starting point.

---

## Self-Assessment: Where Should I Start?

### Beginner Path (Start Here If...)

Answer yes/no to these questions:

- [ ] I understand what LLMs are but haven't built applications with them
- [ ] I'm unfamiliar with concepts like "context window" or "tool calling"
- [ ] I haven't designed prompts beyond simple chat interactions
- [ ] I don't know the differences between models (GPT-4, Claude, Gemini)
- [ ] I haven't built systems where agents take autonomous actions

**3+ yes answers?** Start with the [Beginner Path](#beginner-path-new-to-ai-engineering).

### Developer Path (Start Here If...)

Answer yes/no to these questions:

- [ ] I've built LLM applications with basic prompting and API calls
- [ ] I understand context windows and token limits
- [ ] I've implemented tool calling or function calling
- [ ] I've debugged agent failures and understand common issues
- [ ] I haven't designed multi-agent coordination systems
- [ ] I haven't taken agentic systems to production

**3+ yes answers?** Start with the [Developer Path](#developer-path-ai-experience).

### Practitioner Path (Start Here If...)

Answer yes/no to these questions:

- [ ] I've deployed agentic systems to production
- [ ] I've designed multi-agent architectures with orchestration
- [ ] I understand cost/latency tradeoffs and optimization strategies
- [ ] I've built custom frameworks or patterns for my organization
- [ ] I want to deepen expertise in specific advanced topics

**3+ yes answers?** Start with the [Practitioner Path](#practitioner-path-experienced).

### Problem-Driven (Use This If...)

- [ ] I have a specific bug or issue I need to resolve now
- [ ] I'm looking for targeted guidance on a particular symptom
- [ ] I need quick reference material, not a learning sequence

**Use the [Problem-Driven Index](#problem-driven-index)** to jump directly to relevant sections.

---

## Beginner Path: New to AI Engineering

**Goal:** Build foundational understanding of the four pillars and create a working single-agent system.

**Total Time:** 8-12 hours over 2-3 weeks

**Prerequisites:** None. Programming experience helpful but not required.

### Stage 1: Foundations (2-3 hours)

**Core Reading:**
- [Chapter 1: Foundations](chapters/1-foundations/_index.md) (45 min)
- [Twelve Leverage Points](chapters/1-foundations/1-twelve-leverage-points.md) (30 min)
- [Chapter 2: Prompt](chapters/2-prompt/_index.md) (45 min)
- [Prompt Types](chapters/2-prompt/1-prompt-types.md) (30 min)

**Key Competencies:**
- Understand the four pillars (prompt, model, context, tools)
- Identify different prompt types and when to use each
- Explain how pillars interact and affect each other
- Recognize the twelve leverage points in agentic systems

**Self-Assessment Checkpoint:**

Can you answer these questions?

- [ ] What are the four pillars and how do they interact?
- [ ] What's the difference between a system prompt and a task prompt?
- [ ] Why does adding more tools potentially degrade performance?
- [ ] What happens when context windows fill up?
- [ ] How has the hierarchy of pillars changed over time?

**Recommended Exercise:**

Design three different prompts for the same task (building a file organizer):
1. Static instruction prompt
2. Template-based prompt with placeholders
3. Structured prompt with explicit output format

Compare effectiveness. What differences do you observe?

### Stage 2: Model & Context (3-4 hours)

**Core Reading:**
- [Chapter 3: Model](chapters/3-model/_index.md) (30 min)
- [Model Selection](chapters/3-model/1-model-selection.md) (45 min)
- [Chapter 4: Context](chapters/4-context/_index.md) (45 min)
- [Context Fundamentals](chapters/4-context/1-context-fundamentals.md) (45 min)
- [Context Management Strategies](chapters/4-context/2-context-strategies.md) (45 min)

**Key Competencies:**
- Choose appropriate models for different tasks
- Understand context window constraints and management
- Apply basic context strategies (summarization, windowing)
- Recognize when context is causing agent failures

**Self-Assessment Checkpoint:**

Can you answer these questions?

- [ ] When should you use a frontier model vs. a smaller model?
- [ ] What are three strategies for managing context window limits?
- [ ] How does context affect model behavior during a session?
- [ ] What information should go in context vs. prompts vs. tool outputs?
- [ ] How do you detect context-related failures?

**Recommended Exercise:**

Build a simple chatbot that:
1. Maintains conversation history (context management)
2. Uses different models for different query types
3. Handles context window overflow gracefully

Track token usage and observe behavior as context fills.

### Stage 3: Tool Use (2-3 hours)

**Core Reading:**
- [Chapter 5: Tool Use](chapters/5-tool-use/_index.md) (30 min)
- [Tool Design](chapters/5-tool-use/1-tool-design.md) (45 min)
- [Tool Selection and Routing](chapters/5-tool-use/2-tool-selection.md) (45 min)
- [Tool Restrictions and Security](chapters/5-tool-use/3-tool-restrictions.md) (30 min)

**Key Competencies:**
- Design clear, well-documented tools
- Understand tool selection and routing strategies
- Apply appropriate tool restrictions
- Debug tool-related failures

**Self-Assessment Checkpoint:**

Can you answer these questions?

- [ ] What makes a tool description effective?
- [ ] When should tools be restricted or hidden?
- [ ] How do you handle tool errors gracefully?
- [ ] What's the difference between user-invoked and model-invoked tools?
- [ ] How many tools is "too many" for an agent?

**Recommended Exercise:**

Extend your chatbot with three tools:
1. File reader (read local files)
2. Web search (fetch external information)
3. Calculator (perform computations)

Observe how the agent chooses between tools. Add restrictions to prevent inappropriate tool use.

### Stage 4: First Project (1-2 hours)

**Build:** Single-agent file organization system

**Requirements:**
- Takes user instructions about file organization rules
- Scans a directory structure
- Proposes file moves/renames based on rules
- Executes changes with user confirmation
- Maintains log of actions taken

**Success Criteria:**
- Agent correctly interprets organization rules from natural language
- Proposals make sense given the rules
- System handles errors gracefully (permissions, conflicts)
- User can review and approve before execution

**Example Reference:**
- [Context Loading Demo](appendices/examples/context-loading-demo/) - Basic tool use patterns

**Validation Questions:**

After building your project:

- [ ] Does the agent correctly use all four pillars?
- [ ] Can you explain why each tool is designed the way it is?
- [ ] How does context accumulate during the session?
- [ ] What happens if you change the model? The prompt structure?
- [ ] Where are the failure points and how do you handle them?

---

## Developer Path: AI Experience

**Goal:** Build production-ready patterns and practices for multi-agent systems.

**Total Time:** 6-10 hours over 1-2 weeks

**Prerequisites:** Completed Beginner Path or equivalent. Experience building LLM applications.

### Stage 1: Pattern Foundations (2-3 hours)

**Core Reading:**
- [Chapter 6: Patterns](chapters/6-patterns/_index.md) (30 min)
- [Research-Plan-Build-Review Pattern](chapters/6-patterns/1-plan-build-review.md) (45 min)
- [Self-Improving Expert Commands](chapters/6-patterns/2-self-improving-experts.md) (45 min)
- [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md) (45 min)

**Key Competencies:**
- Recognize when to apply each pattern
- Understand pattern tradeoffs and composition
- Design workflows that separate concerns
- Avoid anti-patterns (emergency context rewriting, etc.)

**Self-Assessment Checkpoint:**

Can you answer these questions?

- [ ] When should you use Plan-Build-Review vs. a simpler pattern?
- [ ] How do self-improving experts accumulate knowledge?
- [ ] What are the coordination challenges in orchestrator patterns?
- [ ] Which patterns compose well together?
- [ ] What anti-patterns have you seen in production systems?

**Recommended Exercise:**

Refactor your Stage 4 project (file organizer) to use Plan-Build-Review:
1. Planning agent analyzes directory and generates move plan
2. Build agent executes moves with validation
3. Review agent summarizes changes and reports conflicts

Compare complexity, reliability, and debuggability vs. single-agent approach.

### Stage 2: Production Practices (2-3 hours)

**Core Reading:**
- [Chapter 7: Practices](chapters/7-practices/_index.md) (30 min)
- [Debugging Agents](chapters/7-practices/1-debugging-agents.md) (45 min)
- [Evaluation](chapters/7-practices/2-evaluation.md) (45 min)
- [Cost and Latency](chapters/7-practices/3-cost-and-latency.md) (30 min)
- [Production Concerns](chapters/7-practices/4-production-concerns.md) (45 min)

**Key Competencies:**
- Debug agent failures systematically
- Design evaluation criteria and measure performance
- Optimize for cost and latency
- Handle production concerns (rate limits, errors, monitoring)

**Self-Assessment Checkpoint:**

Can you answer these questions?

- [ ] How do you diagnose prompt vs. context vs. tool failures?
- [ ] What metrics matter for evaluating agent performance?
- [ ] How do you balance cost, latency, and quality?
- [ ] What monitoring and logging should production agents have?
- [ ] How do you handle agent failures gracefully in production?

**Recommended Exercise:**

Add production readiness to your Plan-Build-Review system:
1. Structured logging for each agent's decisions
2. Cost tracking (token usage per operation)
3. Evaluation metrics (accuracy, latency, cost per file)
4. Error handling and retry logic
5. Simple monitoring dashboard

Deploy and observe behavior under realistic workloads.

### Stage 3: Mental Models (1-2 hours)

**Core Reading:**
- [Chapter 8: Mental Models](chapters/8-mental-models/_index.md) (20 min)
- [Pit of Success](chapters/8-mental-models/1-pit-of-success.md) (30 min)
- [Prompt Maturity Model](chapters/8-mental-models/2-prompt-maturity-model.md) (30 min)
- [Specs as Source Code](chapters/8-mental-models/3-specs-as-source-code.md) (30 min)

**Key Competencies:**
- Design systems that guide agents toward success
- Assess prompt maturity and identify improvement paths
- Treat specifications as the primary programming surface

**Self-Assessment Checkpoint:**

Can you answer these questions?

- [ ] How does the pit of success apply to your agent designs?
- [ ] What maturity level are your prompts at?
- [ ] Where are agents "climbing" vs. "falling" into success?
- [ ] How could specs improve your development workflow?
- [ ] What constraints would improve agent reliability?

**Recommended Exercise:**

Redesign your system's prompts using pit-of-success principles:
1. Identify where agents currently fail or struggle
2. Add constraints that make correct actions easier
3. Remove options that lead to failure
4. Structure outputs to catch errors early

Measure reduction in failure rates after changes.

### Stage 4: Intermediate Project (2-3 hours)

**Build:** Multi-agent content processing pipeline

**Requirements:**
- Orchestrator coordinates three specialized agents
- Intake agent validates and preprocesses documents
- Analysis agent extracts insights and structure
- Output agent formats results in multiple formats
- System handles partial failures and retries
- Monitoring tracks cost, latency, and quality per stage

**Success Criteria:**
- Clear separation of concerns between agents
- Orchestrator delegates without doing specialist work
- Failures in one agent don't cascade
- Logging enables debugging each agent independently
- Cost per document is tracked and optimized

**Example Reference:**
- [KotaDB](appendices/examples/kotadb/) - Multi-agent coordination patterns

**Validation Questions:**

After building your project:

- [ ] Why did you choose the orchestrator pattern vs. alternatives?
- [ ] How do agents communicate and share context?
- [ ] What happens when one agent fails?
- [ ] Where are the cost and latency bottlenecks?
- [ ] How do you evaluate end-to-end performance?

---

## Practitioner Path: Experienced

**Goal:** Master advanced techniques and contribute to the field.

**Total Time:** 4-6 hours targeted deep-dives

**Prerequisites:** Developer Path or equivalent. Production agentic systems experience.

### Stage 1: Advanced Context & Multi-Agent Patterns (1-2 hours)

**Core Reading:**
- [Advanced Context Patterns](chapters/4-context/3-context-patterns.md) (45 min)
- [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md) (45 min)
- [Workflow Coordination for Agents](chapters/7-practices/5-workflow-coordination.md) (30 min)

**Key Competencies:**
- Apply progressive disclosure and context isolation
- Design context handoff protocols for multi-agent systems
- Coordinate complex workflows with state management
- Optimize context usage across agent boundaries

**Self-Assessment Checkpoint:**

Can you answer these questions?

- [ ] When does progressive disclosure pay off vs. adding complexity?
- [ ] How do you prevent context drift in multi-agent systems?
- [ ] What context should be shared vs. isolated between agents?
- [ ] How do you debug context handoff failures?
- [ ] What are the limits of context-based coordination?

**Recommended Exercise:**

Design a context isolation strategy for a system with 5+ specialized agents:
1. Map what context each agent needs vs. what it produces
2. Design explicit handoff protocols
3. Implement progressive disclosure for large knowledge bases
4. Measure context efficiency gains

### Stage 2: Toolkit Mastery (1-2 hours)

**Core Reading:**
- [Chapter 9: Practitioner Toolkit](chapters/9-practitioner-toolkit/_index.md) (20 min)
- [Claude Code](chapters/9-practitioner-toolkit/1-claude-code.md) (45 min)
- [Google ADK](chapters/9-practitioner-toolkit/2-google-adk.md) (45 min)
- [Scaling Tool Use](chapters/5-tool-use/4-scaling-tools.md) (30 min)
- [Skills and Meta-Tools](chapters/5-tool-use/5-skills-and-meta-tools.md) (30 min)

**Key Competencies:**
- Leverage platform-specific capabilities (Claude Code, Google ADK)
- Scale tool use through meta-tools and skill systems
- Design tool architectures for large agent systems
- Understand platform tradeoffs and constraints

**Self-Assessment Checkpoint:**

Can you answer these questions?

- [ ] How do Claude Code and Google ADK differ architecturally?
- [ ] When should you build custom tools vs. using platform tools?
- [ ] What are the cost/latency implications of skills vs. traditional tools?
- [ ] How do meta-tools reduce maintenance burden?
- [ ] What patterns work across platforms vs. are platform-specific?

**Recommended Exercise:**

Implement the same agent functionality on two platforms (e.g., Claude Code + Google ADK):
1. Compare development velocity
2. Measure runtime cost and latency differences
3. Identify platform-specific optimizations
4. Document portability challenges

### Stage 3: Meta-Framework Design (2-3 hours)

**Core Reading:**
- [Context as Code](chapters/8-mental-models/4-context-as-code.md) (30 min)
- [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md) (45 min)

**Key Competencies:**
- Design frameworks that support multiple agent architectures
- Build systems that accumulate and evolve expertise
- Create reusable patterns for your organization
- Balance flexibility with constraints

**Deep Dive Exercise:**

Design your own meta-framework for a specific domain:
1. How will you separate concerns across agents?
2. What makes it reusable across different problem types?
3. How will knowledge accumulate and evolve?
4. Where will you trade complexity for power?
5. What constraints will guide correct usage?

Document insights and iterate based on implementation experience.

### Stage 4: Mastery Validation

**Choose one path to demonstrate mastery:**

#### Option A: Contribute to This Book

Requirements:
- Identify gap in current content
- Research and document new pattern, practice, or mental model
- Follow style guide and structural standards
- Add cross-references and examples

**Validation:** Content accepted into book.

#### Option B: Audit External System

Requirements:
- Audit a production agentic system (yours or client)
- Apply book frameworks (four pillars, twelve leverage points, patterns)
- Identify failure modes and optimization opportunities
- Deliver actionable recommendations

**Validation:** Implement recommendations and measure improvements.

#### Option C: Teach Workshop

Requirements:
- Design 2-hour workshop teaching agentic engineering
- Use book content as foundation
- Include hands-on exercises
- Deliver to audience and gather feedback

**Validation:** Positive participant feedback and measurable learning outcomes.

#### Option D: Build Reference Implementation

Requirements:
- Implement a novel pattern or practice not yet documented
- Demonstrate across multiple use cases
- Document tradeoffs and when to use
- Open-source or publish findings

**Validation:** Adoption by other practitioners.

**Example References:**
- [KotaDB](appendices/examples/kotadb/) - Advanced production patterns

---

## Problem-Driven Index

Navigate directly to solutions for specific issues you're facing right now.

### Agent Failures & Debugging

| Symptom | Quick Check | Deep Dive | Example |
|---------|-------------|-----------|---------|
| Agent ignores instructions | [Prompt Types](chapters/2-prompt/1-prompt-types.md) - Check prompt level/authority | [Debugging Agents](chapters/7-practices/1-debugging-agents.md) | [Claude Code](chapters/9-practitioner-toolkit/1-claude-code.md) |
| Agent uses wrong tools | [Tool Selection](chapters/5-tool-use/2-tool-selection.md) - Review tool descriptions | [Tool Design](chapters/5-tool-use/1-tool-design.md) | [KotaDB](appendices/examples/kotadb/) |
| Agent repeats same failed action | [Evaluation](chapters/7-practices/2-evaluation.md) - Add failure detection | [Prompt Structuring](chapters/2-prompt/2-structuring.md) | [Context Loading Demo](appendices/examples/context-loading-demo/) |
| Agent loses track of task progress | [Context Strategies](chapters/4-context/2-context-strategies.md) - Implement explicit state | [Workflow Coordination](chapters/7-practices/5-workflow-coordination.md) | - |
| Agent gives inconsistent outputs | [Model Selection](chapters/3-model/1-model-selection.md) - Check model vs. task fit | [Prompt Structuring](chapters/2-prompt/2-structuring.md) | - |
| Agent can't find relevant information | [Context Fundamentals](chapters/4-context/1-context-fundamentals.md) - Improve information structure | [Context Patterns](chapters/4-context/3-context-patterns.md) | - |
| Multi-agent coordination failures | [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md) - Review delegation | [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md) | [KotaDB](appendices/examples/kotadb/) |
| Agent refuses valid requests | [Tool Restrictions](chapters/5-tool-use/3-tool-restrictions.md) - Check constraints | [Pit of Success](chapters/8-mental-models/1-pit-of-success.md) | - |

### Performance & Cost Issues

| Symptom | Quick Check | Deep Dive | Example |
|---------|-------------|-----------|---------|
| High token costs | [Cost and Latency](chapters/7-practices/3-cost-and-latency.md) - Audit token usage | [Context Strategies](chapters/4-context/2-context-strategies.md) | [KotaDB](appendices/examples/kotadb/) |
| Slow agent responses | [Model Selection](chapters/3-model/1-model-selection.md) - Consider smaller models | [Cost and Latency](chapters/7-practices/3-cost-and-latency.md) | - |
| Context window overflow | [Context Fundamentals](chapters/4-context/1-context-fundamentals.md) - Implement windowing | [Context Patterns](chapters/4-context/3-context-patterns.md) | - |
| Too many tool calls | [Tool Selection](chapters/5-tool-use/2-tool-selection.md) - Restrict or consolidate | [Scaling Tools](chapters/5-tool-use/4-scaling-tools.md) | - |
| Agent making unnecessary API calls | [Evaluation](chapters/7-practices/2-evaluation.md) - Add efficiency metrics | [Cost and Latency](chapters/7-practices/3-cost-and-latency.md) | - |

### Design & Architecture Questions

| Question | Check | Deep Dive | Example |
|----------|-------|-----------|---------|
| Should I use one agent or multiple? | [Patterns](chapters/6-patterns/_index.md) - Review pattern catalog | [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md) | - |
| How do I structure prompts? | [Prompt Types](chapters/2-prompt/1-prompt-types.md) | [Prompt Structuring](chapters/2-prompt/2-structuring.md) | [Claude Code](chapters/9-practitioner-toolkit/1-claude-code.md) |
| Which model should I use? | [Model Selection](chapters/3-model/1-model-selection.md) | [Cost and Latency](chapters/7-practices/3-cost-and-latency.md) | - |
| How should agents share information? | [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md) | [Context Patterns](chapters/4-context/3-context-patterns.md) | [KotaDB](appendices/examples/kotadb/) |
| What tools should my agent have? | [Tool Design](chapters/5-tool-use/1-tool-design.md) | [Scaling Tools](chapters/5-tool-use/4-scaling-tools.md) | - |
| How do I make agents learn from experience? | [Self-Improving Experts](chapters/6-patterns/2-self-improving-experts.md) | [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md) | - |
| When should I add human oversight? | [Patterns](chapters/6-patterns/_index.md) - Human-in-the-Loop | [Production Concerns](chapters/7-practices/4-production-concerns.md) | - |
| How do I prevent agent mistakes? | [Pit of Success](chapters/8-mental-models/1-pit-of-success.md) | [Tool Restrictions](chapters/5-tool-use/3-tool-restrictions.md) | - |

### Production & Operations

| Challenge | Quick Check | Deep Dive | Example |
|-----------|-------------|-----------|---------|
| Rate limiting issues | [Production Concerns](chapters/7-practices/4-production-concerns.md) - Retry logic | [Cost and Latency](chapters/7-practices/3-cost-and-latency.md) | - |
| Monitoring and logging | [Debugging Agents](chapters/7-practices/1-debugging-agents.md) - Structured logging | [Evaluation](chapters/7-practices/2-evaluation.md) | [KotaDB](appendices/examples/kotadb/) |
| Error handling | [Production Concerns](chapters/7-practices/4-production-concerns.md) | [Debugging Agents](chapters/7-practices/1-debugging-agents.md) | - |
| Evaluating improvements | [Evaluation](chapters/7-practices/2-evaluation.md) | [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md) | - |
| Scaling to production | [Production Concerns](chapters/7-practices/4-production-concerns.md) | [Cost and Latency](chapters/7-practices/3-cost-and-latency.md) | [KotaDB](appendices/examples/kotadb/) |

### Tool-Specific Guidance

| Tool/Platform | Getting Started | Advanced Topics | Examples |
|---------------|----------------|-----------------|----------|
| Claude Code | [Claude Code Overview](chapters/9-practitioner-toolkit/1-claude-code.md) | [Skills and Meta-Tools](chapters/5-tool-use/5-skills-and-meta-tools.md) | [Context Loading Demo](appendices/examples/context-loading-demo/) |
| Google ADK | [Google ADK Overview](chapters/9-practitioner-toolkit/2-google-adk.md) | [Multi-Model Architectures](chapters/3-model/4-multi-model-architectures.md) | - |
| Custom Agents | [Foundations](chapters/1-foundations/_index.md) | [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md) | - |

---

## Chapter Prerequisites Map

Understanding dependencies helps you read efficiently.

### Dependency Layers

```
Foundation Layer (no prerequisites):
├── Chapter 1: Foundations

Core Pillars (require Chapter 1):
├── Chapter 2: Prompt
├── Chapter 3: Model
├── Chapter 4: Context
└── Chapter 5: Tool Use

Pattern Layer (require multiple pillars):
└── Chapter 6: Patterns
    ├── Requires: Chapters 2, 4, 5
    └── Recommended: Chapter 3

Practice Layer (require patterns):
└── Chapter 7: Practices
    ├── Requires: Chapter 6
    └── Recommended: All pillars (2-5)

Meta Layer (can be read independently):
├── Chapter 8: Mental Models
│   └── Enhanced by: Any prior chapters
└── Chapter 9: Practitioner Toolkit
    └── Requires: Chapters 1-5 for full context
```

### Reading Strategies

**Linear (Beginner-Friendly):**
Read chapters 1-9 in order. Deepest understanding but longest path.

**Parallel (Faster):**
1. Read Chapter 1
2. Read Chapters 2-5 in any order (pillars are mostly independent)
3. Read Chapter 6
4. Read Chapters 7-9 in any order

**Targeted (Problem-Solving):**
1. Start with [Problem-Driven Index](#problem-driven-index)
2. Follow quick check links
3. Read prerequisites only if concepts are unclear
4. Return to deep dive for thorough understanding

**Quick Reference (Practitioner):**
Use chapters as reference material:
- Chapter 2: When designing prompts
- Chapter 4: When facing context issues
- Chapter 6: When choosing architectures
- Chapter 7: When debugging or optimizing

---

## Learning Resources by Chapter

### Chapter 1: Foundations

**Core Reading:** [Foundations](chapters/1-foundations/_index.md)

**Key Sections:**
- The Pillars in Plain Language (5 min)
- How the Pillars Interact (10 min)
- Common Mistakes (10 min)

**Additional Reading:** [Twelve Leverage Points](chapters/1-foundations/1-twelve-leverage-points.md) (30 min)

**Exercise:** Map a simple agent design (chatbot, file organizer, etc.) to the four pillars. Where does each component fit?

**Total Time:** 45-60 min

---

### Chapter 2: Prompt

**Core Reading:** [Prompt](chapters/2-prompt/_index.md)

**Key Sections:**
- [Prompt Types](chapters/2-prompt/1-prompt-types.md) - Seven levels from static to meta (30 min)
- [Prompt Structuring](chapters/2-prompt/2-structuring.md) - Output design and templates (45 min)

**Exercise:**
1. Identify the maturity level of prompts in your current project
2. Rewrite one prompt at the next maturity level
3. Compare effectiveness

**Total Time:** 2 hours

---

### Chapter 3: Model

**Core Reading:** [Model](chapters/3-model/_index.md)

**Key Sections:**
- [Model Selection](chapters/3-model/1-model-selection.md) - Choosing models for tasks (45 min)

**Exercise:** Run the same task on three different models (frontier, mid-tier, small). Compare quality, cost, and latency.

**Total Time:** 1.5 hours

---

### Chapter 4: Context

**Core Reading:** [Context](chapters/4-context/_index.md)

**Key Sections:**
- [Context Fundamentals](chapters/4-context/1-context-fundamentals.md) (45 min)
- [Context Management Strategies](chapters/4-context/2-context-strategies.md) (45 min)
- [Advanced Context Patterns](chapters/4-context/3-context-patterns.md) (30 min)
- [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md) (30 min)

**Recommended Example:** [Context Loading Demo](appendices/examples/context-loading-demo/)

**Exercise:** Implement three context strategies (summarization, windowing, progressive disclosure) for a growing conversation. Measure token usage vs. information retention.

**Total Time:** 3 hours

---

### Chapter 5: Tool Use

**Core Reading:** [Tool Use](chapters/5-tool-use/_index.md)

**Key Sections:**
- [Tool Design](chapters/5-tool-use/1-tool-design.md) (45 min)
- [Tool Selection and Routing](chapters/5-tool-use/2-tool-selection.md) (45 min)
- [Tool Restrictions and Security](chapters/5-tool-use/3-tool-restrictions.md) (30 min)
- [Scaling Tool Use](chapters/5-tool-use/4-scaling-tools.md) (30 min)
- [Skills and Meta-Tools](chapters/5-tool-use/5-skills-and-meta-tools.md) (30 min)

**Exercise:** Design a tool suite for a document processing agent. Include at least one meta-tool. Document routing logic.

**Total Time:** 3 hours

---

### Chapter 6: Patterns

**Core Reading:** [Patterns](chapters/6-patterns/_index.md)

**Key Sections:**
- [Research-Plan-Build-Review Pattern](chapters/6-patterns/1-plan-build-review.md) (45 min)
- [Self-Improving Expert Commands](chapters/6-patterns/2-self-improving-experts.md) (45 min)
- [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md) (45 min)

**Recommended Example:** [KotaDB](appendices/examples/kotadb/) - Multi-agent patterns

**Exercise:** Refactor a single-agent system into Plan-Build-Review. Compare debugging, extensibility, and reliability.

**Total Time:** 3 hours

---

### Chapter 7: Practices

**Core Reading:** [Practices](chapters/7-practices/_index.md)

**Key Sections:**
- [Debugging Agents](chapters/7-practices/1-debugging-agents.md) (45 min)
- [Evaluation](chapters/7-practices/2-evaluation.md) (45 min)
- [Cost and Latency](chapters/7-practices/3-cost-and-latency.md) (30 min)
- [Production Concerns](chapters/7-practices/4-production-concerns.md) (45 min)
- [Workflow Coordination for Agents](chapters/7-practices/5-workflow-coordination.md) (30 min)
- [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md) (30 min)

**Recommended Example:** [KotaDB](appendices/examples/kotadb/) - Production patterns

**Exercise:** Add production instrumentation to an existing agent: structured logging, cost tracking, evaluation metrics, error handling.

**Total Time:** 4 hours

---

### Chapter 8: Mental Models

**Core Reading:** [Mental Models](chapters/8-mental-models/_index.md)

**Key Sections:**
- [Pit of Success](chapters/8-mental-models/1-pit-of-success.md) (30 min)
- [Prompt Maturity Model](chapters/8-mental-models/2-prompt-maturity-model.md) (30 min)
- [Specs as Source Code](chapters/8-mental-models/3-specs-as-source-code.md) (30 min)
- [Context as Code](chapters/8-mental-models/4-context-as-code.md) (30 min)

**Exercise:** Audit an existing agent design for "pit of success" principles. Where are agents climbing vs. falling into correct behavior? Redesign to reduce friction.

**Total Time:** 2 hours

---

### Chapter 9: Practitioner Toolkit

**Core Reading:** [Practitioner Toolkit](chapters/9-practitioner-toolkit/_index.md)

**Key Sections:**
- [Claude Code](chapters/9-practitioner-toolkit/1-claude-code.md) (45 min)
- [Google ADK](chapters/9-practitioner-toolkit/2-google-adk.md) (45 min)

**Exercise:** Build the same agent on two platforms. Document platform-specific optimizations and portability challenges.

**Total Time:** 2-3 hours

---

## Next Steps

After completing your learning path:

1. **Apply to Real Projects:** Use patterns and practices on actual work
2. **Contribute Back:** Document new patterns or insights you discover
3. **Join Community:** Share experiences with other practitioners
4. **Stay Current:** Models and tools evolve—revisit fundamentals as capabilities change

**Remember:** Mastery comes from building, breaking, and debugging real systems. This curriculum provides the foundation—experience provides the expertise.
