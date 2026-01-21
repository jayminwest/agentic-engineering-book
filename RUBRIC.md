# Agentic Engineering Competency Rubric

This rubric provides a framework for self-assessment and skill development in agentic engineering. It defines observable behaviors across five skill dimensions and four competency levels.

**How to use this rubric:**
- Complete the self-assessment for each dimension
- Identify the current level based on observable behaviors
- Use the progression guide to advance to the next level
- Map competencies to chapters and examples
- Revisit periodically to track growth

**Related Resources:**
- [Table of Contents](TABLE_OF_CONTENTS.md) - Complete book structure
- [Foundations](chapters/1-foundations/_index.md) - Core concepts for all dimensions
- [Mental Models](chapters/8-mental-models/_index.md) - Strategic thinking frameworks

---

## Rubric Framework

### Five Skill Dimensions

This rubric assesses competency across five distinct dimensions of agentic engineering practice:

1. **Technical Implementation** - Building working agent systems with prompts, tools, and workflows
2. **System Design** - Architecting multi-agent workflows and selecting appropriate patterns
3. **Diagnostic & Debugging** - Identifying and fixing failures across the four pillars
4. **Production Operations** - Running agents reliably at scale with cost and quality constraints
5. **Strategic Thinking** - Meta-framework design, pattern extraction, and knowledge evolution

### Four Competency Levels

Each dimension progresses through four observable levels:

- **Level 1: Foundational** - Can use with guidance, understands core concepts
- **Level 2: Functional** - Can build independently, applies patterns effectively
- **Level 3: Advanced** - Can design complex systems, handles edge cases and production constraints
- **Level 4: Expert** - Can create new patterns, mentor others, and influence the field

**Important**: Levels are dimension-specific. An engineer might be Level 3 in Technical Implementation but Level 2 in Production Operations. Balanced development across dimensions matters more than reaching Level 4 in any single area.

---

## Dimension 1: Technical Implementation

Building working agent systems with prompts, tools, and workflows.

### Level Descriptors

| Level | Observable Behaviors | Self-Assessment Questions |
|-------|---------------------|---------------------------|
| **Level 1: Foundational** | • Can write structured prompts with clear output formats<br>• Has built 1+ working agent system under guidance<br>• Understands the four pillars (prompt, model, context, tool)<br>• Can implement basic tool calling<br>• Recognizes when outputs fail to match expected format | • Have I built at least one working agent system?<br>• Can I explain what each of the four pillars controls?<br>• Can I write a prompt with a clear output format?<br>• Do I understand when to use tools vs. generate text? |
| **Level 2: Functional** | • Can design custom tools with appropriate interfaces<br>• Has built 3+ agent systems independently<br>• Implements plan-build-review pattern consistently<br>• Structures prompts using levels (L1-L7) appropriately<br>• Handles context window limitations with summarization<br>• Uses failure sentinels and validation | • Have I built multiple agent systems without guidance?<br>• Can I design a tool interface that prevents common failures?<br>• Do I structure multi-step tasks using plan-build-review?<br>• Can I handle context overflow gracefully?<br>• Do I validate tool outputs before using them? |
| **Level 3: Advanced** | • Can design multi-agent orchestration patterns<br>• Has shipped agent systems to production<br>• Implements meta-tools and skill discovery<br>• Optimizes prompts for cost/latency/quality tradeoffs<br>• Designs self-improving workflows<br>• Handles multi-turn context management | • Have I architected systems with multiple coordinating agents?<br>• Have I shipped an agent system used by others?<br>• Can I design tools that generate other tools?<br>• Do I systematically optimize for cost and latency?<br>• Can I build systems that improve from feedback? |
| **Level 4: Expert** | • Can design novel agentic patterns not documented elsewhere<br>• Has built meta-frameworks that generalize across domains<br>• Contributes patterns back to the field<br>• Mentors others in implementation techniques<br>• Creates implementation tools used by other engineers | • Have I created patterns that others now use?<br>• Have I built frameworks that work across multiple domains?<br>• Do other engineers learn from my implementations?<br>• Have I published or shared novel techniques?<br>• Have I built tools that help others build agents? |

### Chapter Mapping

- **Level 1**: [Foundations](chapters/1-foundations/_index.md), [Prompt Types](chapters/2-prompt/1-prompt-types.md), [Tool Design](chapters/5-tool-use/1-tool-design.md)
- **Level 2**: [Prompt Structuring](chapters/2-prompt/2-structuring.md), [Context Strategies](chapters/4-context/2-context-strategies.md), [Plan-Build-Review](chapters/6-patterns/1-plan-build-review.md)
- **Level 3**: [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md), [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md), [Skills and Meta-Tools](chapters/5-tool-use/5-skills-and-meta-tools.md)
- **Level 4**: [Self-Improving Experts](chapters/6-patterns/2-self-improving-experts.md), [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md)

### Example Mapping

- **Level 1**: [Context Loading Demo](appendices/examples/context-loading-demo/) - Basic tool use and prompting
- **Level 2**: [KotaDB](appendices/examples/kotadb/) - Multi-step workflow with tools
- **Level 3**: [KotaDB](appendices/examples/kotadb/) - Multi-agent orchestration

### Progression Guide

**From Level 1 to Level 2:**
1. Build three agent systems from scratch without templates
2. Implement the plan-build-review pattern in at least one system
3. Design a custom tool with proper error handling and output validation
4. Study [Prompt Structuring](chapters/2-prompt/2-structuring.md) and apply levels appropriately
5. Handle a context overflow scenario with summarization

**From Level 2 to Level 3:**
1. Ship an agent system to production with real users
2. Design a multi-agent system with orchestrator + specialist agents
3. Implement a meta-tool or skill discovery system
4. Optimize a system for 50% cost reduction without quality loss
5. Build a self-improving workflow that incorporates feedback

**From Level 3 to Level 4:**
1. Create a novel pattern not documented in existing literature
2. Build a meta-framework that works across 3+ different domains
3. Mentor 2+ engineers to Level 2 or beyond
4. Publish implementation techniques or tools for the community
5. Create implementation tooling used by other practitioners

---

## Dimension 2: System Design

Architecting multi-agent workflows and selecting appropriate patterns.

### Level Descriptors

| Level | Observable Behaviors | Self-Assessment Questions |
|-------|---------------------|---------------------------|
| **Level 1: Foundational** | • Can explain the four pillars framework<br>• Identifies which pillar is failing in simple cases<br>• Understands single-agent vs. multi-agent tradeoffs<br>• Recognizes when a task needs multi-step workflow<br>• Can describe plan-build-review at a high level | • Can I explain what each pillar controls?<br>• When something fails, can I identify which pillar is the issue?<br>• Do I understand when to use one agent vs. multiple?<br>• Can I recognize tasks that need multiple steps?<br>• Can I describe how plan-build-review separates concerns? |
| **Level 2: Functional** | • Designs single vs. multi-agent architectures appropriately<br>• Selects models based on task requirements<br>• Applies documented patterns (plan-build-review, orchestrator)<br>• Designs tool interfaces that prevent common failures<br>• Structures workflows with clear phase boundaries<br>• Uses the pit of success to guide design choices | • Do I choose the right architecture for the problem?<br>• Can I select models based on cost/latency/quality needs?<br>• Do I apply patterns consistently and appropriately?<br>• Do my tool designs prevent misuse?<br>• Can I structure workflows with clean separation?<br>• Do I design systems that make correct use easy? |
| **Level 3: Advanced** | • Architects for cost/latency/quality constraints simultaneously<br>• Designs systems with graceful degradation<br>• Creates context management strategies for multi-agent coordination<br>• Balances synchronous vs. asynchronous workflows<br>• Designs for observability and debugging from the start<br>• Applies prompt maturity model to assess design evolution | • Can I optimize for multiple constraints at once?<br>• Do my systems degrade gracefully under failure?<br>• Can I design context handoff protocols for multi-agent systems?<br>• Do I choose sync/async appropriately?<br>• Can I debug my systems in production?<br>• Do I assess and evolve my prompt maturity? |
| **Level 4: Expert** | • Designs meta-frameworks that generalize across domains<br>• Creates novel architectural patterns<br>• Balances all twelve leverage points effectively<br>• Designs for knowledge evolution from inception<br>• Influences architectural decisions in the field<br>• Mentors others in system design thinking | • Have I created frameworks used across multiple domains?<br>• Have I designed patterns that others now reference?<br>• Can I optimize all twelve leverage points together?<br>• Do my systems improve themselves over time?<br>• Do my architectural decisions influence others?<br>• Can I teach others to think architecturally? |

### Chapter Mapping

- **Level 1**: [Foundations](chapters/1-foundations/_index.md), [Twelve Leverage Points](chapters/1-foundations/1-twelve-leverage-points.md), [Model Selection](chapters/3-model/1-model-selection.md)
- **Level 2**: [Patterns](chapters/6-patterns/_index.md), [Pit of Success](chapters/8-mental-models/1-pit-of-success.md), [Tool Selection](chapters/5-tool-use/2-tool-selection.md)
- **Level 3**: [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md), [Workflow Coordination](chapters/7-practices/5-workflow-coordination.md), [Prompt Maturity Model](chapters/8-mental-models/2-prompt-maturity-model.md)
- **Level 4**: [Specs as Source Code](chapters/8-mental-models/3-specs-as-source-code.md), [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md)

### Example Mapping

- **Level 1**: Basic single-agent examples in [Context Loading Demo](appendices/examples/context-loading-demo/)
- **Level 2**: Workflow design in [KotaDB](appendices/examples/kotadb/)
- **Level 3**: Multi-agent architecture in [KotaDB](appendices/examples/kotadb/)

### Progression Guide

**From Level 1 to Level 2:**
1. Design three systems using different patterns (plan-build-review, orchestrator, single-agent)
2. Make explicit model selection decisions for different task types
3. Create tool interfaces with built-in guardrails
4. Study [Pit of Success](chapters/8-mental-models/1-pit-of-success.md) and apply to one design
5. Document architectural decisions and tradeoffs

**From Level 2 to Level 3:**
1. Design a system optimizing for cost, latency, and quality simultaneously
2. Implement graceful degradation for at least three failure modes
3. Create a multi-agent context handoff protocol
4. Design with observability (logging, metrics, debugging hooks) from the start
5. Assess a system using the prompt maturity model and evolve it

**From Level 3 to Level 4:**
1. Create a meta-framework applicable to 3+ different problem domains
2. Design and document a novel architectural pattern
3. Mentor 2+ engineers through complex architectural decisions
4. Publish architectural insights that influence field practices
5. Build systems that evolve their own architecture based on usage

---

## Dimension 3: Diagnostic & Debugging

Identifying and fixing failures across the four pillars.

### Level Descriptors

| Level | Observable Behaviors | Self-Assessment Questions |
|-------|---------------------|---------------------------|
| **Level 1: Foundational** | • Can identify which pillar is failing in simple cases<br>• Debugs basic prompt issues (output format, instruction clarity)<br>• Recognizes context window overflow<br>• Identifies tool calling failures from logs<br>• Understands the difference between model capability and prompt quality | • When something fails, can I identify the pillar?<br>• Can I fix basic prompt formatting issues?<br>• Do I recognize when context is overflowing?<br>• Can I read tool calling logs to find failures?<br>• Do I know when the model can't vs. won't do something? |
| **Level 2: Functional** | • Debugs multi-step failures systematically<br>• Traces failure through prompt → context → tool chain<br>• Uses logging and instrumentation effectively<br>• Identifies prompt ambiguity and fixes it<br>• Debugs context management issues (stale context, missing information)<br>• Recognizes model selection problems | • Can I trace failures through multiple steps?<br>• Do I use logging to understand what happened?<br>• Can I identify and fix ambiguous prompts?<br>• Can I debug context issues systematically?<br>• Do I recognize when I've chosen the wrong model?<br>• Can I reproduce failures consistently? |
| **Level 3: Advanced** | • Debugs multi-agent coordination failures<br>• Identifies production-specific failures (load, latency spikes)<br>• Debugs subtle context leakage between agents<br>• Recognizes emergent failure modes in complex systems<br>• Creates debugging tools for recurring failure patterns<br>• Performs root cause analysis for systemic issues | • Can I debug failures in multi-agent systems?<br>• Can I identify production-only failure modes?<br>• Can I trace context leakage between agents?<br>• Do I recognize patterns in seemingly random failures?<br>• Have I built tools to debug faster?<br>• Can I find root causes vs. symptoms? |
| **Level 4: Expert** | • Debugs novel failure modes not documented elsewhere<br>• Creates debugging methodologies for new patterns<br>• Builds debugging tools used by other engineers<br>• Identifies failure patterns across multiple systems<br>• Contributes debugging techniques to the field<br>• Mentors others in systematic debugging | • Can I debug failures with no documented solutions?<br>• Have I created debugging methods others use?<br>• Have I built debugging tools for the community?<br>• Can I identify cross-system failure patterns?<br>• Have I published debugging techniques?<br>• Can I teach others to debug systematically? |

### Chapter Mapping

- **Level 1**: [Foundations](chapters/1-foundations/_index.md), [Context Fundamentals](chapters/4-context/1-context-fundamentals.md), [Tool Design](chapters/5-tool-use/1-tool-design.md)
- **Level 2**: [Debugging Agents](chapters/7-practices/1-debugging-agents.md), [Context Strategies](chapters/4-context/2-context-strategies.md), [Model Selection](chapters/3-model/1-model-selection.md)
- **Level 3**: [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md), [Production Concerns](chapters/7-practices/4-production-concerns.md), [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md)
- **Level 4**: [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md), [Debugging Agents](chapters/7-practices/1-debugging-agents.md)

### Example Mapping

- **Level 1**: Basic debugging in [Context Loading Demo](appendices/examples/context-loading-demo/)
- **Level 2**: Workflow debugging in [KotaDB](appendices/examples/kotadb/)
- **Level 3**: Multi-agent debugging in [KotaDB](appendices/examples/kotadb/)

### Progression Guide

**From Level 1 to Level 2:**
1. Debug three multi-step failures using systematic tracing
2. Implement logging and instrumentation in all systems
3. Create a reproducible test case for a complex failure
4. Study [Debugging Agents](chapters/7-practices/1-debugging-agents.md) thoroughly
5. Fix three ambiguous prompt issues with evidence-based improvements

**From Level 2 to Level 3:**
1. Debug a multi-agent coordination failure
2. Identify and fix a production-only failure mode
3. Create a debugging tool for a recurring pattern
4. Perform root cause analysis on a systemic issue
5. Document debugging methodology for a complex pattern

**From Level 3 to Level 4:**
1. Debug a novel failure mode with no documented solution
2. Create debugging methodology used by other engineers
3. Build debugging tooling adopted by the community
4. Identify cross-system patterns and document them
5. Mentor 2+ engineers in systematic debugging approaches

---

## Dimension 4: Production Operations

Running agents reliably at scale with cost and quality constraints.

### Level Descriptors

| Level | Observable Behaviors | Self-Assessment Questions |
|-------|---------------------|---------------------------|
| **Level 1: Foundational** | • Can estimate token costs for simple tasks<br>• Measures basic success rate metrics<br>• Understands the cost/latency/quality tradeoff triangle<br>• Monitors simple agent executions<br>• Recognizes when costs are unexpectedly high | • Can I estimate how much a task will cost?<br>• Do I measure whether my agent succeeds?<br>• Do I understand cost vs. latency vs. quality tradeoffs?<br>• Can I monitor basic agent runs?<br>• Do I notice when costs spike unexpectedly? |
| **Level 2: Functional** | • Optimizes for cost/latency without sacrificing quality<br>• Implements retry logic and error handling<br>• Deploys agents to production environments<br>• Sets up basic monitoring and alerting<br>• Uses model selection to balance cost and capability<br>• Implements rate limiting and quota management | • Have I reduced costs by 30%+ without quality loss?<br>• Do my systems handle failures gracefully?<br>• Have I deployed agents to production?<br>• Do I get alerted when things break?<br>• Do I choose models based on cost/capability needs?<br>• Can I handle rate limits and quotas? |
| **Level 3: Advanced** | • Runs agents at scale (1000+ tasks/day)<br>• Implements comprehensive observability (logs, metrics, traces)<br>• Optimizes context strategies for production cost<br>• Manages multi-model deployments<br>• Implements A/B testing for prompt changes<br>• Handles production incidents effectively | • Do I run systems processing 1000+ tasks daily?<br>• Can I debug production issues quickly with observability?<br>• Have I optimized context for cost at scale?<br>• Do I manage deployments across multiple models?<br>• Do I test changes before full rollout?<br>• Can I respond to incidents in under 15 minutes? |
| **Level 4: Expert** | • Runs agents at extreme scale (100k+ tasks/day)<br>• Builds production tooling used by other teams<br>• Designs cost optimization strategies adopted widely<br>• Creates reliability patterns for the field<br>• Influences production best practices<br>• Mentors teams on production operations | • Do I run systems at 100k+ tasks daily?<br>• Have I built production tools used by others?<br>• Have my cost optimizations been adopted elsewhere?<br>• Have I created reliability patterns others use?<br>• Do my practices influence the field?<br>• Can I help teams scale to production? |

### Chapter Mapping

- **Level 1**: [Cost and Latency](chapters/7-practices/3-cost-and-latency.md), [Model Selection](chapters/3-model/1-model-selection.md), [Evaluation](chapters/7-practices/2-evaluation.md)
- **Level 2**: [Production Concerns](chapters/7-practices/4-production-concerns.md), [Context Strategies](chapters/4-context/2-context-strategies.md), [Tool Restrictions](chapters/5-tool-use/3-tool-restrictions.md)
- **Level 3**: [Advanced Context Patterns](chapters/4-context/3-context-patterns.md), [Multi-Model Architectures](chapters/3-model/4-multi-model-architectures.md), [Workflow Coordination](chapters/7-practices/5-workflow-coordination.md)
- **Level 4**: [Production Concerns](chapters/7-practices/4-production-concerns.md), [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md)

### Example Mapping

- **Level 1**: Basic cost estimation using examples
- **Level 2**: Production deployment patterns in [KotaDB](appendices/examples/kotadb/)
- **Level 3**: Scale operations in [KotaDB](appendices/examples/kotadb/)

### Progression Guide

**From Level 1 to Level 2:**
1. Deploy an agent system to production with monitoring
2. Optimize a system for 30% cost reduction while maintaining quality
3. Implement comprehensive error handling and retries
4. Set up alerting for failures, cost spikes, and latency issues
5. Study [Production Concerns](chapters/7-practices/4-production-concerns.md)

**From Level 2 to Level 3:**
1. Scale a system to 1000+ tasks/day
2. Implement full observability (structured logs, metrics, traces)
3. Run an A/B test comparing two prompt versions
4. Manage a multi-model deployment with routing logic
5. Handle a production incident from detection to resolution

**From Level 3 to Level 4:**
1. Scale a system to 100k+ tasks/day
2. Build production tooling adopted by other teams
3. Create and document cost optimization patterns used widely
4. Mentor 2+ teams through production scaling
5. Publish production best practices that influence the field

---

## Dimension 5: Strategic Thinking

Meta-framework design, pattern extraction, and knowledge evolution.

### Level Descriptors

| Level | Observable Behaviors | Self-Assessment Questions |
|-------|---------------------|---------------------------|
| **Level 1: Foundational** | • Can explain core mental models (pit of success, four pillars)<br>• Applies documented patterns appropriately<br>• Recognizes when current approach isn't working<br>• Understands the concept of leverage points<br>• Can articulate tradeoffs in simple scenarios | • Can I explain the pit of success concept?<br>• Do I use documented patterns when appropriate?<br>• Do I recognize when to change approaches?<br>• Can I explain what leverage points are?<br>• Can I articulate why I chose this approach? |
| **Level 2: Functional** | • Assesses prompt maturity and plans evolution<br>• Designs using pit of success principles<br>• Extracts patterns from implementation experience<br>• Documents learnings systematically<br>• Applies specs-as-source-code thinking<br>• Makes explicit tradeoff decisions | • Do I assess and evolve prompt maturity?<br>• Do I design systems that make correct use easy?<br>• Can I extract generalizable patterns from experience?<br>• Do I document what I learn systematically?<br>• Do I treat specs as executable artifacts?<br>• Can I articulate my tradeoff decisions? |
| **Level 3: Advanced** | • Extracts generalizable patterns from multiple systems<br>• Designs evolving knowledge systems<br>• Applies context-as-code thinking to architecture<br>• Balances competing leverage points<br>• Creates frameworks that span multiple domains<br>• Identifies meta-patterns across implementations | • Can I find patterns across different systems?<br>• Do my systems improve their own knowledge?<br>• Do I design context as a manageable artifact?<br>• Can I balance multiple competing constraints?<br>• Have I built frameworks for multiple domains?<br>• Can I identify meta-patterns in the field? |
| **Level 4: Expert** | • Creates novel meta-frameworks that generalize broadly<br>• Influences mental models in the field<br>• Designs self-improving knowledge systems<br>• Identifies new leverage points not previously documented<br>• Shapes strategic thinking across the community<br>• Mentors others in strategic framework design | • Have I created frameworks used across the field?<br>• Do my mental models influence others' thinking?<br>• Have I built systems that evolve their own strategy?<br>• Have I identified novel leverage points?<br>• Do my ideas shape community practices?<br>• Can I teach strategic thinking effectively? |

### Chapter Mapping

- **Level 1**: [Mental Models](chapters/8-mental-models/_index.md), [Twelve Leverage Points](chapters/1-foundations/1-twelve-leverage-points.md), [Pit of Success](chapters/8-mental-models/1-pit-of-success.md)
- **Level 2**: [Prompt Maturity Model](chapters/8-mental-models/2-prompt-maturity-model.md), [Specs as Source Code](chapters/8-mental-models/3-specs-as-source-code.md), [Patterns](chapters/6-patterns/_index.md)
- **Level 3**: [Context as Code](chapters/8-mental-models/4-context-as-code.md), [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md), [Self-Improving Experts](chapters/6-patterns/2-self-improving-experts.md)
- **Level 4**: [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md), [Twelve Leverage Points](chapters/1-foundations/1-twelve-leverage-points.md)

### Example Mapping

- **Level 1**: Mental model application in basic examples
- **Level 2**: Pattern extraction from [KotaDB](appendices/examples/kotadb/)
- **Level 3**: Framework design in [KotaDB](appendices/examples/kotadb/)

### Progression Guide

**From Level 1 to Level 2:**
1. Assess a system using the prompt maturity model
2. Design one system using pit of success principles
3. Extract and document three patterns from implementations
4. Create systematic documentation of learnings
5. Study [Specs as Source Code](chapters/8-mental-models/3-specs-as-source-code.md) and apply it

**From Level 2 to Level 3:**
1. Extract generalizable patterns from 3+ different systems
2. Design a knowledge system that improves itself
3. Apply context-as-code to a complex architecture
4. Balance five+ competing leverage points in one design
5. Create a framework applicable to multiple domains

**From Level 3 to Level 4:**
1. Create a meta-framework adopted across multiple organizations
2. Publish mental models that influence field thinking
3. Build a system that evolves its own strategic approach
4. Identify and document a novel leverage point
5. Mentor 3+ engineers in strategic framework thinking

---

## Self-Assessment Worksheet

### Instructions

1. Read each dimension's observable behaviors
2. Check boxes for behaviors consistently demonstrated
3. Identify current level (the highest level where most behaviors are checked)
4. Set target level for each dimension
5. Prioritize dimensions for development
6. Use progression guides to create action plan

### Dimension 1: Technical Implementation

**Level 1: Foundational**
- [ ] Can write structured prompts with clear output formats
- [ ] Has built 1+ working agent system under guidance
- [ ] Understands the four pillars (prompt, model, context, tool)
- [ ] Can implement basic tool calling
- [ ] Recognizes when outputs fail to match expected format

**Level 2: Functional**
- [ ] Can design custom tools with appropriate interfaces
- [ ] Has built 3+ agent systems independently
- [ ] Implements plan-build-review pattern consistently
- [ ] Structures prompts using levels (L1-L7) appropriately
- [ ] Handles context window limitations with summarization
- [ ] Uses failure sentinels and validation

**Level 3: Advanced**
- [ ] Can design multi-agent orchestration patterns
- [ ] Has shipped agent systems to production
- [ ] Implements meta-tools and skill discovery
- [ ] Optimizes prompts for cost/latency/quality tradeoffs
- [ ] Designs self-improving workflows
- [ ] Handles multi-turn context management

**Level 4: Expert**
- [ ] Can design novel agentic patterns not documented elsewhere
- [ ] Has built meta-frameworks that generalize across domains
- [ ] Contributes patterns back to the field
- [ ] Mentors others in implementation techniques
- [ ] Creates implementation tools used by other engineers

**My Current Level**: __________

---

### Dimension 2: System Design

**Level 1: Foundational**
- [ ] Can explain the four pillars framework
- [ ] Identifies which pillar is failing in simple cases
- [ ] Understands single-agent vs. multi-agent tradeoffs
- [ ] Recognizes when a task needs multi-step workflow
- [ ] Can describe plan-build-review at a high level

**Level 2: Functional**
- [ ] Designs single vs. multi-agent architectures appropriately
- [ ] Selects models based on task requirements
- [ ] Applies documented patterns (plan-build-review, orchestrator)
- [ ] Designs tool interfaces that prevent common failures
- [ ] Structures workflows with clear phase boundaries
- [ ] Uses the pit of success to guide design choices

**Level 3: Advanced**
- [ ] Architects for cost/latency/quality constraints simultaneously
- [ ] Designs systems with graceful degradation
- [ ] Creates context management strategies for multi-agent coordination
- [ ] Balances synchronous vs. asynchronous workflows
- [ ] Designs for observability and debugging from the start
- [ ] Applies prompt maturity model to assess design evolution

**Level 4: Expert**
- [ ] Designs meta-frameworks that generalize across domains
- [ ] Creates novel architectural patterns
- [ ] Balances all twelve leverage points effectively
- [ ] Designs for knowledge evolution from inception
- [ ] Influences architectural decisions in the field
- [ ] Mentors others in system design thinking

**My Current Level**: __________

---

### Dimension 3: Diagnostic & Debugging

**Level 1: Foundational**
- [ ] Can identify which pillar is failing in simple cases
- [ ] Debugs basic prompt issues (output format, instruction clarity)
- [ ] Recognizes context window overflow
- [ ] Identifies tool calling failures from logs
- [ ] Understands the difference between model capability and prompt quality

**Level 2: Functional**
- [ ] Debugs multi-step failures systematically
- [ ] Traces failure through prompt → context → tool chain
- [ ] Uses logging and instrumentation effectively
- [ ] Identifies prompt ambiguity and fixes it
- [ ] Debugs context management issues (stale context, missing information)
- [ ] Recognizes model selection problems

**Level 3: Advanced**
- [ ] Debugs multi-agent coordination failures
- [ ] Identifies production-specific failures (load, latency spikes)
- [ ] Debugs subtle context leakage between agents
- [ ] Recognizes emergent failure modes in complex systems
- [ ] Creates debugging tools for recurring failure patterns
- [ ] Performs root cause analysis for systemic issues

**Level 4: Expert**
- [ ] Debugs novel failure modes not documented elsewhere
- [ ] Creates debugging methodologies for new patterns
- [ ] Builds debugging tools used by other engineers
- [ ] Identifies failure patterns across multiple systems
- [ ] Contributes debugging techniques to the field
- [ ] Mentors others in systematic debugging

**My Current Level**: __________

---

### Dimension 4: Production Operations

**Level 1: Foundational**
- [ ] Can estimate token costs for simple tasks
- [ ] Measures basic success rate metrics
- [ ] Understands the cost/latency/quality tradeoff triangle
- [ ] Monitors simple agent executions
- [ ] Recognizes when costs are unexpectedly high

**Level 2: Functional**
- [ ] Optimizes for cost/latency without sacrificing quality
- [ ] Implements retry logic and error handling
- [ ] Deploys agents to production environments
- [ ] Sets up basic monitoring and alerting
- [ ] Uses model selection to balance cost and capability
- [ ] Implements rate limiting and quota management

**Level 3: Advanced**
- [ ] Runs agents at scale (1000+ tasks/day)
- [ ] Implements comprehensive observability (logs, metrics, traces)
- [ ] Optimizes context strategies for production cost
- [ ] Manages multi-model deployments
- [ ] Implements A/B testing for prompt changes
- [ ] Handles production incidents effectively

**Level 4: Expert**
- [ ] Runs agents at extreme scale (100k+ tasks/day)
- [ ] Builds production tooling used by other teams
- [ ] Designs cost optimization strategies adopted widely
- [ ] Creates reliability patterns for the field
- [ ] Influences production best practices
- [ ] Mentors teams on production operations

**My Current Level**: __________

---

### Dimension 5: Strategic Thinking

**Level 1: Foundational**
- [ ] Can explain core mental models (pit of success, four pillars)
- [ ] Applies documented patterns appropriately
- [ ] Recognizes when current approach isn't working
- [ ] Understands the concept of leverage points
- [ ] Can articulate tradeoffs in simple scenarios

**Level 2: Functional**
- [ ] Assesses prompt maturity and plans evolution
- [ ] Designs using pit of success principles
- [ ] Extracts patterns from implementation experience
- [ ] Documents learnings systematically
- [ ] Applies specs-as-source-code thinking
- [ ] Makes explicit tradeoff decisions

**Level 3: Advanced**
- [ ] Extracts generalizable patterns from multiple systems
- [ ] Designs evolving knowledge systems
- [ ] Applies context-as-code thinking to architecture
- [ ] Balances competing leverage points
- [ ] Creates frameworks that span multiple domains
- [ ] Identifies meta-patterns across implementations

**Level 4: Expert**
- [ ] Creates novel meta-frameworks that generalize broadly
- [ ] Influences mental models in the field
- [ ] Designs self-improving knowledge systems
- [ ] Identifies new leverage points not previously documented
- [ ] Shapes strategic thinking across the community
- [ ] Mentors others in strategic framework design

**My Current Level**: __________

---

### Summary

| Dimension | Current Level | Target Level | Priority (1-5) |
|-----------|---------------|--------------|----------------|
| Technical Implementation | __________ | __________ | __________ |
| System Design | __________ | __________ | __________ |
| Diagnostic & Debugging | __________ | __________ | __________ |
| Production Operations | __________ | __________ | __________ |
| Strategic Thinking | __________ | __________ | __________ |

### Next Steps

**Immediate actions (next 2 weeks):**
1. __________________________________________
2. __________________________________________
3. __________________________________________

**Medium-term goals (next 3 months):**
1. __________________________________________
2. __________________________________________
3. __________________________________________

**Resources to study:**
1. __________________________________________
2. __________________________________________
3. __________________________________________

---

## Competency Mapping

### Chapter-to-Competency Map

This table shows which dimensions each chapter primarily addresses:

| Chapter | Tech Impl | System Design | Debug | Production | Strategic |
|---------|-----------|---------------|-------|------------|-----------|
| [Foundations](chapters/1-foundations/_index.md) | ✓✓ | ✓✓✓ | ✓ | ✓ | ✓✓✓ |
| [Prompt](chapters/2-prompt/_index.md) | ✓✓✓ | ✓✓ | ✓✓ | ✓ | ✓ |
| [Model](chapters/3-model/_index.md) | ✓ | ✓✓✓ | ✓✓ | ✓✓✓ | ✓ |
| [Context](chapters/4-context/_index.md) | ✓✓ | ✓✓ | ✓✓✓ | ✓✓ | ✓ |
| [Tool Use](chapters/5-tool-use/_index.md) | ✓✓✓ | ✓✓ | ✓✓ | ✓ | ✓ |
| [Patterns](chapters/6-patterns/_index.md) | ✓✓ | ✓✓✓ | ✓ | ✓ | ✓✓ |
| [Practices](chapters/7-practices/_index.md) | ✓ | ✓ | ✓✓✓ | ✓✓✓ | ✓✓ |
| [Mental Models](chapters/8-mental-models/_index.md) | ✓ | ✓✓ | ✓ | ✓ | ✓✓✓ |
| [Practitioner Toolkit](chapters/9-practitioner-toolkit/_index.md) | ✓✓ | ✓ | ✓ | ✓✓ | ✓ |

**Legend:** ✓ = Relevant, ✓✓ = Significant, ✓✓✓ = Primary focus

### Example-to-Competency Map

| Example | Tech Impl | System Design | Debug | Production | Strategic |
|---------|-----------|---------------|-------|------------|-----------|
| [Context Loading Demo](appendices/examples/context-loading-demo/) | L1 | L1 | L1 | L1 | L1 |
| [KotaDB](appendices/examples/kotadb/) | L2-L3 | L2-L3 | L2-L3 | L2-L3 | L2-L3 |

### Learning Path by Target Level

**Target Level 1 (Foundational)**

**Core Reading:**
1. [Foundations](chapters/1-foundations/_index.md)
2. [Prompt Types](chapters/2-prompt/1-prompt-types.md)
3. [Context Fundamentals](chapters/4-context/1-context-fundamentals.md)
4. [Tool Design](chapters/5-tool-use/1-tool-design.md)
5. [Pit of Success](chapters/8-mental-models/1-pit-of-success.md)

**Practice:**
- Build three simple agent systems using [Context Loading Demo](appendices/examples/context-loading-demo/) as reference
- Implement basic tool calling
- Debug output format failures

**Target Level 2 (Functional)**

**Core Reading:**
1. [Prompt Structuring](chapters/2-prompt/2-structuring.md)
2. [Context Strategies](chapters/4-context/2-context-strategies.md)
3. [Plan-Build-Review](chapters/6-patterns/1-plan-build-review.md)
4. [Debugging Agents](chapters/7-practices/1-debugging-agents.md)
5. [Production Concerns](chapters/7-practices/4-production-concerns.md)
6. [Prompt Maturity Model](chapters/8-mental-models/2-prompt-maturity-model.md)

**Practice:**
- Build three independent agent systems
- Implement plan-build-review pattern
- Study [KotaDB](appendices/examples/kotadb/) implementation
- Deploy to production with monitoring

**Target Level 3 (Advanced)**

**Core Reading:**
1. [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md)
2. [Advanced Context Patterns](chapters/4-context/3-context-patterns.md)
3. [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md)
4. [Skills and Meta-Tools](chapters/5-tool-use/5-skills-and-meta-tools.md)
5. [Workflow Coordination](chapters/7-practices/5-workflow-coordination.md)
6. [Cost and Latency](chapters/7-practices/3-cost-and-latency.md)
7. [Context as Code](chapters/8-mental-models/4-context-as-code.md)

**Practice:**
- Design multi-agent orchestration
- Study [KotaDB](appendices/examples/kotadb/) architecture
- Run systems at 1000+ tasks/day
- Implement self-improving workflows

**Target Level 4 (Expert)**

**Core Reading:**
1. [Self-Improving Experts](chapters/6-patterns/2-self-improving-experts.md)
2. [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md)
3. [Specs as Source Code](chapters/8-mental-models/3-specs-as-source-code.md)
4. [Twelve Leverage Points](chapters/1-foundations/1-twelve-leverage-points.md)
5. All chapters for comprehensive understanding

**Practice:**
- Create novel patterns
- Build frameworks for multiple domains
- Mentor others
- Contribute to field knowledge

---

## Common Development Gaps

### Pattern 1: Strong Implementation, Weak Design

**Profile:**
- Level 2-3 in Technical Implementation
- Level 1 in System Design
- Can build working systems but struggles with architecture

**Symptoms:**
- Reaches for code before considering design alternatives
- Builds single solutions without pattern extraction
- Struggles to choose between architectural options
- Difficult to explain why design choices were made
- Systems work but don't scale or evolve well

**Remediation:**
1. Study [Patterns](chapters/6-patterns/_index.md) and [Mental Models](chapters/8-mental-models/_index.md)
2. Practice articulating architectural tradeoffs before implementing
3. Review [Twelve Leverage Points](chapters/1-foundations/1-twelve-leverage-points.md)
4. For each implementation, write design doc explaining alternatives
5. Study example architecture: [KotaDB](appendices/examples/kotadb/)

**Diagnostic Questions:**
- Can I explain three alternatives to my current design?
- Have I documented why I chose this architecture?
- Can I predict how this design will evolve?
- Do I use mental models to guide design?

---

### Pattern 2: Strong Theory, Weak Production

**Profile:**
- Level 2-3 in System Design and Strategic Thinking
- Level 1 in Production Operations
- Can design elegant systems but struggles to run them reliably

**Symptoms:**
- Prototypes work beautifully, production deployments fail
- No monitoring or observability in place
- Surprised by costs, latency, or failures in production
- Doesn't test at production scale
- Lacks operational expertise

**Remediation:**
1. Study [Production Concerns](chapters/7-practices/4-production-concerns.md) thoroughly
2. Read [Cost and Latency](chapters/7-practices/3-cost-and-latency.md)
3. Deploy three systems to production with full monitoring
4. Run a system at 1000+ tasks/day
5. Implement comprehensive error handling and retries
6. Practice incident response

**Diagnostic Questions:**
- Have I deployed systems used by others?
- Can I estimate costs before deployment?
- Do I have monitoring and alerting?
- Have I optimized for production constraints?
- Can I debug production failures quickly?

---

### Pattern 3: Strong Debugging, Weak Prevention

**Profile:**
- Level 2-3 in Diagnostic & Debugging
- Level 1-2 in System Design and Strategic Thinking
- Excellent at fixing failures but keeps encountering the same issues

**Symptoms:**
- Constantly firefighting the same problems
- Doesn't extract patterns from debugging sessions
- Reactive rather than proactive
- Good at root cause analysis but doesn't prevent recurrence
- Debugging skills mask design weaknesses

**Remediation:**
1. Study [Pit of Success](chapters/8-mental-models/1-pit-of-success.md)
2. For each debug session, document the design change that would prevent it
3. Read [Knowledge Evolution](chapters/7-practices/6-knowledge-evolution.md)
4. Create debugging tools that prevent future failures
5. Practice designing systems that make errors impossible

**Diagnostic Questions:**
- Do I see the same failures repeatedly?
- Have I created prevention patterns from debugging?
- Do my designs eliminate entire failure classes?
- Am I designing for pit of success?

---

### Pattern 4: Breadth Without Depth

**Profile:**
- Level 2 across all dimensions
- Can do everything competently but nothing excellently
- Struggles with complex or novel problems

**Symptoms:**
- Comfortable with documented patterns, uncomfortable with novel situations
- Doesn't push into Level 3 challenges
- Stays in comfort zone of known solutions
- Limited ability to handle edge cases or production scale
- Hasn't built anything truly complex

**Remediation:**
1. Choose one dimension to push to Level 3
2. Take on a challenging project requiring deep expertise
3. Study advanced chapters: [Multi-Agent Context](chapters/4-context/4-multi-agent-context.md), [Orchestrator Pattern](chapters/6-patterns/3-orchestrator-pattern.md)
4. Analyze [KotaDB](appendices/examples/kotadb/) deeply
5. Mentor someone to force articulation of deep knowledge

**Diagnostic Questions:**
- Have I built anything at production scale?
- Can I handle novel problems without documentation?
- Do I push beyond comfortable patterns?
- Have I specialized in any dimension?

---

### Pattern 5: Specialization Without Breadth

**Profile:**
- Level 3-4 in one dimension
- Level 1 in other dimensions
- Deep expertise in one area, but limited holistic capability

**Symptoms:**
- Solves every problem with the same approach (the hammer-nail problem)
- Blind spots in other dimensions limit effectiveness
- Struggles to communicate with those in other specializations
- Can't diagnose failures outside area of expertise
- Limited perspective on tradeoffs

**Remediation:**
1. Assess lowest dimension and commit to raising it
2. Work on a project requiring multiple dimensions
3. Study foundational chapters to build missing fundamentals
4. Pair with someone strong in weak dimension
5. Take on end-to-end responsibility for a system

**Diagnostic Questions:**
- Can I debug failures across all four pillars?
- Do I design, implement, and operate systems end-to-end?
- Can I make informed tradeoffs across dimensions?
- Am I at least Level 2 in all dimensions?

---

### Balanced Development Target

**Recommended minimum profile:**
- Level 2 in all five dimensions
- Level 3 in at least two dimensions
- Working toward Level 3 in remaining dimensions

**Why this matters:**
- Breadth prevents blind spots that cause systemic failures
- Depth in multiple areas enables novel synthesis
- Balance allows end-to-end ownership and understanding
- Prevents over-reliance on templates and known patterns

**Assessment question:**
"Can I design, implement, debug, deploy, and evolve an agent system independently?"

If the answer is yes across all dimensions, development is balanced. If no, identify which dimension is limiting and prioritize its development.
