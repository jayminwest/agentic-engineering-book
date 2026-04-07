---
title: Context at Codebase Scale
description: Architectural patterns for giving agents effective context in large, underdocumented brownfield codebases
created: 2026-04-07
last_updated: 2026-04-07
tags: [context, brownfield, enterprise, codebase-scale, patterns, graduated-adoption]
part: 1
part_title: Foundations
chapter: 4
section: 6
order: 1.4.6
---

# Context at Codebase Scale

*[2026-02-11]*: Multi-agent deployments fail in the majority of enterprises — not due to agent capability limitations but due to architectural misunderstanding of legacy systems and undocumented dependencies. The systems agents interact with were not designed for automated consumption. Undocumented APIs, implicit workflows, and tribal knowledge create failure surfaces that no amount of agent sophistication can navigate without explicit mapping. ([Multi-Agent Landscape](../6-patterns/10-multi-agent-landscape.md))

This section addresses that failure mode directly with a pattern catalog for managing agent context in codebases that are too large, too old, or too underdocumented for naive context loading.

---

## Why Brownfield Context Is Different

### The Width vs. Depth Distinction

Section 5 ([Context Management Architectures](5-context-management-architectures.md)) addresses *depth*: how agents manage context across long sessions. LCM and Sapling answer the question "what happens when a single session runs too long?" This section addresses *width*: how agents navigate codebases that are too large for any single context window, regardless of session length. The two challenges are distinct and compose. A brownfield enterprise deployment requires both.

### What Makes Enterprise Codebases Structurally Hostile to Agents

Brownfield codebases accumulate structural complexity that greenfield systems never face:

**Size.** Production codebases range from 100K to 5M+ lines of code across hundreds or thousands of files. Even summary-level representations of files and symbols may exceed available context budgets. Agents cannot simply "read the code."

**Tribal knowledge.** Critical decisions live in the heads of long-tenured engineers, not in documentation. Why does the payment service call the legacy auth endpoint instead of the new one? The code reveals the what; the reason is unrecorded. Agents that encounter this code have no access to the why.

**Historical decisions.** Architecture Decision Records (ADRs) are often absent or incomplete. Constraints accumulate silently: PCI-DSS requirements added without documentation, performance trade-offs made during an incident and never revisited, deprecated libraries kept because migration risk was deemed too high. Agents navigating this code are archaeologists without field notes.

**Undocumented APIs and implicit workflows.** Internal APIs evolve without changelog discipline. Implicit workflows — the sequence that must happen before deployment, the service that must be notified after a schema change — live in runbooks, Slack threads, or nowhere at all.

### The Two Failure Types

Brownfield context management has two distinct failure modes:

**"I can't see enough."** The agent lacks access to the relevant code, documentation, or system relationships. It makes decisions based on an incomplete view. This is a *coverage* problem.

**"I see noise, not signal."** The agent has access to large amounts of code but cannot distinguish what matters for the current task. Irrelevant files, stale comments, and misleading naming conventions pollute the signal. This is a *relevance* problem.

The patterns in this section address both failure types. Semantic indexing (Pattern 1) and dependency-graph queries (Pattern 4) address coverage. Hierarchical convention files (Pattern 2), decision records (Pattern 3), and tribal knowledge codification (Pattern 6) address relevance. Progressive codebase disclosure (Pattern 5) addresses both by controlling which tier of information agents see.

---

## Your Mental Model

**Brownfield context management is archaeological fieldwork, not search.** Search assumes the answer exists in an indexed document and can be retrieved by query. Archaeological fieldwork assumes the evidence is fragmentary, partially buried, and requires interpretation. An agent navigating a 10-year-old enterprise codebase is not searching; it is reconstructing intent from artifacts.

**The relevant context for an agent is rarely the code itself.** It is the constraint behind the code — the PCI requirement that forced that data flow, the performance incident that caused that denormalization, the deprecated library that couldn't be removed. These constraints are invisible to semantic search and invisible to the model unless they have been explicitly codified.

**The brownfield adoption path is staged, not simultaneous.** Teams cannot instrument a 5M-line legacy codebase with comprehensive context infrastructure overnight. Each pattern in this catalog delivers standalone value. The graduated adoption path (Pattern 7) provides a sequencing framework for incremental improvement.

---

## Pattern Catalog

### Pattern 1: Semantic Indexing

**What it is.** A pre-built representation of codebase symbols — functions, classes, interfaces, modules — ranked by reference frequency or dependency depth and fitted into a configurable token budget. Rather than loading file contents, agents receive a structured summary of what exists and where, then fetch specific content on demand.

**Why it applies to brownfield specifically.** In greenfield codebases, developers know the structure. In brownfield codebases, neither humans nor agents can hold the structure in working memory. Semantic indexing provides the table of contents that makes navigation possible without requiring full content load.

**How it works conceptually.** Symbol extraction operates at the parse level — understanding language grammar to identify function definitions, class declarations, import relationships — rather than at the text-search level. Symbols are ranked using graph-based metrics: how many other symbols reference this one? How deep in the dependency chain is it? High-reference, high-depth symbols are more likely to be relevant entry points. The result fits into a configurable token budget, making it composable with other context sources.

The same 85% token reduction principle documented for dynamic tool discovery in [Scaling Tool Use](../5-tool-use/4-scaling-tools.md) applies here: representing thousands of symbols as metadata is dramatically cheaper than representing them as full content, while maintaining the agent's ability to navigate to full content on demand.

The [Progressive Disclosure Pattern](../6-patterns/7-progressive-disclosure.md) provides the three-tier loading model that semantic indexing implements at the codebase level: the index is Tier 1 (metadata), retrieved file contents are Tier 2 (activated content), and referenced dependencies are Tier 3 (on-demand resources).

**Failure mode.** Semantic indexing fails when symbol names are misleading or when the codebase uses implicit conventions that the index cannot capture. A function named `processData` that actually handles PII scrubbing is indexed as `processData` — its constraint regime is invisible until an agent reads it and encounters the business logic. Semantic indexing addresses coverage; it does not address the tribal knowledge failure type.

**Implementation guidance.**
- Parse symbols at the grammar level, not the text level — tree-sitter provides a language-agnostic parsing standard for 130+ languages
- Rank by reference frequency and dependency depth to surface entry points an agent is most likely to need
- Fit to a configurable token budget: the index should consume a predictable fraction of context, leaving room for activated content
- Combine with on-demand file retrieval — the index is a navigation aid, not a replacement for reading code

---

### Pattern 2: Hierarchical Convention Files

**What it is.** Convention documentation nested within the directory structure it describes. A root file captures project-wide conventions; module or service directories contain files capturing module-specific constraints, ownership, and context.

**Why it applies to brownfield specifically.** Brownfield codebases accumulate heterogeneous context across modules. The payments service lives under PCI-DSS constraints. The legacy auth module was written in a different era and uses patterns explicitly deprecated everywhere else. The data engineering directory has different owners with different deployment conventions. A single root convention file cannot capture this diversity without growing unmanageable.

The insight is structural: convention files should mirror the directory hierarchy they document. A file at `services/payments/CLAUDE.md` containing PCI-DSS constraints serves a fundamentally different purpose than a root file containing overall project conventions. Agents navigating the payments service inherit both — the root context and the module-specific override.

The book's prior evidence for this pattern appears in [Production Concerns](../7-practices/4-production-concerns.md): "For large codebases, this is essential — agents can't infer all conventions from code alone."

**Failure mode.** Hierarchical convention files fail when they drift from the code they document. A module-level file that described the state of a service two years ago may actively mislead agents navigating the current state. Convention files require maintenance discipline — they are living documentation, not one-time artifacts.

**Implementation guidance.**
- Root file: project-wide conventions, dependencies, deployment patterns, ownership structure
- Module files: module-specific constraints, known limitations, contact points, exception regimes
- Service files: API contracts, data schema ownership, SLA expectations, downstream consumers
- Establish a review trigger: when a module's code changes significantly, the convention file is updated with it

---

### Pattern 3: Decision Records as Agent Context

**What it is.** Architecture Decision Records (ADRs) used as structured context that explains *why* the code is the way it is, not just *what* it does. ADRs capture the decision, the alternatives considered, the reasoning, and the constraints that shaped the outcome.

**Why it applies to brownfield specifically.** This pattern directly addresses the tribal knowledge failure type. Agents that encounter code without decision context will "helpfully" reverse decisions made for reasons no longer visible in the code — deprecated library? Agents may suggest migrating away from it, not knowing the migration was attempted and abandoned due to a data integrity issue. PCI constraint? Agents may refactor the data flow, not knowing the separation is legally required.

ADRs give agents the *why*, which is invisible to semantic indexing and invisible to the model without explicit encoding.

**The AI-assisted retrospective ADR pattern.** For legacy codebases that predate ADR practices, retrospective generation is viable: feed existing code, test files, and commit history to a model to generate candidate ADRs for documented decisions. The generated candidates require human review and correction, but the process surfaces implicit decisions and makes them explicit. This turns tribal knowledge into structured context.

The relationship to [Specs as Source Code](../8-mental-models/3-specs-as-source-code.md) is direct: ADRs are a specific form of specification — the decision itself as an artifact that agents can read and respect, not just code that embeds the decision invisibly.

**Failure mode.** ADRs fail when they are disconnected from the code they document. An ADR that references a decision no longer reflected in the code misleads more than no ADR at all. ADRs must be co-located with or explicitly linked to the code they constrain.

**Implementation guidance.**
- Structure ADRs with: Status (Accepted / Superseded), Context (the forces at play), Decision (what was chosen), Consequences (the resulting constraints)
- Tag ADRs with the modules they constrain — agents can discover relevant ADRs for a module without reading all ADRs
- For retrospective generation: use commit history, test names, and code comments as source material; treat generated ADRs as drafts requiring engineer review
- Link ADRs to convention files: the module file points to its relevant ADRs

---

### Pattern 4: Dependency-Graph Queries

**What it is.** Rather than loading files, agents query a pre-built dependency graph: "what calls this function?", "what does this module import?", "what breaks if I change this interface?" The graph is built ahead of time; agents query it during task execution.

**Why it applies to brownfield specifically.** Brownfield codebases have deep, non-obvious dependency relationships accumulated over years of development. Safe change impact analysis in a legacy system requires knowing the transitive consumer graph of any modified symbol. Loading all potentially affected files is infeasible at scale; querying a pre-built graph is targeted and cheap.

This pattern is the brownfield answer to change risk. The dependency graph makes blast radius visible before any change is made.

The Language Server Protocol (LSP) is the standard protocol for this capability — it defines the interface by which editors and other consumers can query "find references," "go to definition," and related operations against a running language server. LSP-backed dependency graph queries operate at the semantic level of the language, not at the text-search level, providing accurate results for dynamic dispatch and interface implementations that text search cannot resolve.

The KotaDB case study (`appendices/examples/kotadb/CASE_STUDY.md:361-397`) demonstrates this pattern through its `search_code` MCP tool — change impact analysis via dependency traversal as an agent-accessible tool.

**Failure mode.** Dependency-graph queries fail in polyglot codebases where modules cross language boundaries without explicit interface documentation. A Go service calling a Python service via HTTP has a dependency that LSP cannot trace — it requires API documentation or integration tests to surface. Cross-language dependency resolution remains an unsolved problem in current tooling.

**Implementation guidance.**
- Build the dependency graph as a pre-computed artifact, updated on CI or on-commit; agents query it, not build it
- Expose as a tool interface: `find_callers(symbol)`, `find_dependencies(module)`, `impact_radius(symbol)` — discrete query operations, not raw graph access
- For cross-language boundaries, document the interface contracts explicitly in convention files and ADRs — the tooling gap requires human documentation
- Most suitable for codebases with strong static typing where LSP-level analysis is accurate; dynamically typed codebases require additional investment

---

### Pattern 5: Progressive Codebase Disclosure

**Reference only — see [Progressive Disclosure Pattern](../6-patterns/7-progressive-disclosure.md).**

The three-tier loading model documented in that section (metadata index → activated content → on-demand resources) transfers directly to codebase navigation at scale. The adaptation:

- **Tier 1 (metadata index):** A semantic index of files, modules, and symbols — names, descriptions, and dependency relationships at summary granularity
- **Tier 2 (activated content):** The specific files and modules relevant to the current task, loaded in full
- **Tier 3 (on-demand resources):** Dependencies, referenced modules, and supporting files fetched as the agent determines they are needed

The original pattern was documented for loading expertise and skills into agent context; the codebase navigation version is structurally identical. The budget discipline is the same: Tier 1 consumes a fixed, predictable fraction of context; Tiers 2 and 3 are loaded based on task relevance, not preloaded speculatively.

---

### Pattern 6: Tribal Knowledge Codification

**What it is.** The phase that precedes or accompanies other patterns: translating undocumented system knowledge from human memory into agent-readable form. This pattern is about process as much as structure — it describes how teams surface and encode knowledge that currently exists only in heads and Slack threads.

**Why it applies to brownfield specifically.** Gas Town's 189K LOC Go system documented this failure mode directly: tribal knowledge and undocumented APIs create the most persistent failure surfaces in agent deployments. Semantic indexing can navigate the code; it cannot navigate knowledge that was never written down. Before agents can operate effectively in a brownfield system, humans must do the work of making implicit knowledge explicit.

**Three codification mechanisms.**

*Structured convention files.* Encoding workflow patterns and implicit conventions into module-level and root-level convention files. The content: recurring workflows (what steps must happen before deploying this service), implicit conventions (this module uses a naming scheme that predates the rest of the codebase), boundary conditions (this API has a rate limit that is not documented in the API itself).

*Domain expertise files.* Structured `expertise.yaml` files that capture domain-specific knowledge the book's expert agent pattern can load. These go beyond conventions into domain semantics: what does "subscriber" mean in this system? Which events are idempotent and which are not? What are the valid state transitions for an order?

*The thin-pointer model.* For large legacy codebases, the root convention file is a navigation hub rather than a knowledge repository. Domain knowledge lives in module-level expertise files, and the root file provides the map. This mirrors the pattern documented in the external-teacher expertise: a 31KB legacy system document replaced with a 104-line navigation hub pointing to domain-specific expertise files. The result is composable — agents load only the expertise relevant to their current task rather than loading all domain knowledge upfront.

The relationship to [Specs as Source Code](../8-mental-models/3-specs-as-source-code.md) is foundational: tribal knowledge codification is the process of creating the specs that agents will use as source. Without this phase, other patterns operate on an incomplete substrate.

**Failure mode.** Tribal knowledge codification fails when it is treated as a one-time project rather than a continuous practice. The knowledge base grows stale as the system evolves. Codification requires a maintenance discipline: when a tribal decision changes, the documentation changes with it.

**Implementation guidance.**
- Start with the highest-risk knowledge: constraints that agents might violate with significant consequences (PCI requirements, rate limits, data ownership rules)
- Use AI assistance for drafting: describe the domain to a model, let it generate structured documentation, review and correct the output
- Co-locate documentation with code: expertise files live in the modules they describe, not in a separate documentation repository
- Treat codification as debt reduction: each piece of tribal knowledge encoded is one fewer failure surface for agents

---

### Pattern 7: Graduated Adoption

**What it is.** A named-stage framework for teams that cannot adopt comprehensive context infrastructure simultaneously. Each stage delivers standalone value and does not require subsequent stages. Teams may be on multiple stages across different modules at the same time.

**Evidence basis.** This framework was validated in a coaching session with a non-coder team lead running a B2B SaaS pipeline (transcript → SEO landing pages). The session revealed that the barrier to agent effectiveness was not capability but context infrastructure — specifically, prompt drift between two copies of the same extraction prompt that had diverged into different schemas. The graduated path emerged as the sequencing that resolved the most critical failure mode first, then built progressively toward greater automation. The stages apply broadly; the specific evidence comes from a Director-archetype practitioner (non-coder lead, directing more than coding).

**The five stages.**

```
Encoding          Documenting        Consolidating       Specializing        Enforcing
────────────    ────────────────    ─────────────────    ──────────────    ──────────────
Slash commands    CLAUDE.md:          Single source        Expert agents     Hooks and
for repeatable    workflow patterns   of truth for         with persistent   guardrails for
workflows         prompt inventory    prompts that drift   domain knowledge  automated
                  decision frames                                            invariants
```

*Encoding.* Create slash commands as convention files for recurring workflows. A command that runs the generation pipeline, a command that runs tests, a command that sets up a new client — these encode the implicit workflow steps that currently require human memory. Implementation is entirely in markdown files, requiring no code changes. Immediate daily value; usable the next day.

*Documenting.* Upgrade convention files with three additions: workflow patterns (the recurring sequences that agents should follow rather than invent), a prompt inventory (a list of every prompt in the system, its location, its purpose, and when it was last reviewed), and a decision framework (when to use which model, cost/quality trade-offs, task-specific guidance). This phase makes implicit knowledge explicit. Agents learn what practitioners already know.

*Consolidating.* Establish a single source of truth for prompts that have drifted — multiple copies in multiple files that have diverged from each other. Consolidation creates a canonical prompt directory and eliminates duplication. This phase is the one that often requires code changes, but the changes are mechanical: move text from A to B, add an import. The prompt inventory from Documenting makes the problem visible; Consolidating eliminates it.

*Specializing.* Create expert agent domains with persistent knowledge — domain-specific expertise files that agents load rather than having humans re-explain context at the start of each session. A content pipeline expert that knows extraction schemas, model selection rules, and multi-client content variations. A database operations expert that knows the namespace conventions, migration ordering, and Pydantic model patterns. The improve agent pattern learns from each session, making expertise cumulative.

*Enforcing.* Add hooks that enforce invariants automatically: run tests before every commit, load project context at session start, block operations that violate constraints. This stage enables higher autonomy — the safeguards enforce the rules the practitioner would otherwise enforce manually. The calibration principle from the autonomy frameworks applies: autonomy level and safeguard depth must scale together.

**Key principles for graduated adoption.**

Each stage moves the practitioner from operator to architect. At Encoding, humans follow workflows manually but have commands to assist. At Enforcing, agents work within an automated constraint system and humans review results.

Teams should not wait for all five stages to be in place before beginning. Encoding alone improves daily effectiveness. Documenting alone improves agent response quality in the next session. The stages are not prerequisites for each other; they are investments with compounding returns.

**Failure mode.** Graduated adoption fails when stages are skipped in ways that undermine later stages. Enforcing without Documenting means hooks enforce rules that agents don't know. Specializing without Consolidating means expert agents inherit the prompt drift problem. The stage ordering is a dependency chain, not a rigid timeline — but the dependencies are real.

---

## Decision Matrix

Archetype definitions:
- **Director** — Non-coder lead who directs agents more than writing code; managing an inherited project
- **Platform/DevEx** — Infrastructure-focused team building context tooling for other developers
- **Senior Dev** — Individual practitioner without organizational buy-in; can only adopt what requires no shared infrastructure
- **Compliance Enterprise** — Regulated industry; air-gap, PCI, HIPAA, or SOC 2 requirements constrain tool choice

| Pattern | Director | Platform/DevEx | Senior Dev | Compliance Enterprise | Notes |
|---------|----------|----------------|------------|----------------------|-------|
| **1. Semantic Indexing** | S | P | S | S | Platform/DevEx builds the index; others consume it. Senior Dev can use OSS local tooling without org buy-in. Compliance Enterprise requires air-gap deployment. |
| **2. Hierarchical Convention Files** | P | S | P | P | Primary for all archetypes — requires only file creation. Director starts here. Compliance Enterprise uses for regulatory constraint encoding. No infrastructure dependency. |
| **3. Decision Records** | S | S | P | P | Senior Dev can adopt unilaterally for personal workflow. Compliance Enterprise uses for audit trail of architectural decisions under regulatory review. |
| **4. Dependency-Graph Queries** | L | P | L | S | High infrastructure investment; Platform/DevEx archetype is the natural owner. Director lacks technical depth to operate. Senior Dev cannot adopt without org buy-in. Compliance requires on-prem graph store. |
| **5. Progressive Codebase Disclosure** | S | P | S | S | Platform/DevEx designs the tier structure; others benefit. Composable with all other patterns. Requires semantic index (Pattern 1) as Tier 1 input. |
| **6. Tribal Knowledge Codification** | P | S | P | P | Director-first: encoding what the team already knows into agent-readable form is the highest-leverage activity for a non-coder lead. Compliance Enterprise: essential for documenting regulatory constraint rationale. |
| **7. Graduated Adoption** | P | — | P | S | Designed for teams that cannot adopt simultaneously. Platform/DevEx typically has capacity for more systematic adoption. Compliance Enterprise uses staged approach to manage audit risk during transition. |

**P** = primary recommendation for this archetype | **S** = supplementary, useful in combination | **L** = low fit | **—** = not applicable

---

## Open Questions

**Cross-language dependency tracing.** LSP-backed dependency analysis works within language boundaries. Production brownfield systems routinely cross language boundaries — Go services calling Python APIs, Node frontends consuming Java backends. No current tooling provides semantic-level dependency resolution across these boundaries. The gap means cross-language blast radius analysis requires human documentation rather than automated tooling.

**Cost at scale for high-frequency context queries.** Dependency-graph queries and semantic indexing queries are cheap per call, but high-frequency agent loops can invoke them thousands of times per day across a large development team. The cost model for enterprise-scale codebase context infrastructure is not well-documented in production deployments. The economic threshold at which managed cloud indexing becomes cheaper than Platform/DevEx-maintained local infrastructure is unknown.

**PII and secrets filtering at index time.** Semantic indices built over production codebases may index secrets, PII, or proprietary business logic that should not appear in model context. Current tooling does not handle PII filtering at index construction time — the pattern is to filter at the AI gateway layer, but this requires the filtered content to reach the gateway before filtering occurs. An indexing-time filter would be more appropriate architecturally.

**Maintenance discipline for tribal knowledge artifacts.** Convention files, ADRs, and expertise files require ongoing maintenance as codebases evolve. No established practice exists for detecting when these artifacts have drifted from the code they document. The failure mode — stale tribal knowledge documentation that misleads agents — may be worse than no documentation.

**Adoption sequencing for compliance-bound enterprises.** Regulated industries have additional constraints on graduated adoption: each stage may require security review, audit documentation, or vendor approval before deployment. The five-stage framework from Pattern 7 does not account for the compliance overhead that extends transition timelines. A compliance-aware sequencing framework remains an open gap.

---

## Connections

- **To [Context Management Architectures](5-context-management-architectures.md):** Where Section 5 addresses depth (long sessions, context compaction, LCM, Sapling), this section addresses width (large codebases, navigating scale). The two sections are designed to be read together for enterprise deployments — long-session management and codebase-scale context are distinct problems with distinct pattern catalogs.

- **To [Progressive Disclosure Pattern](../6-patterns/7-progressive-disclosure.md):** The three-tier loading model (metadata index → activated content → on-demand resources) is the structural template for codebase navigation at scale. Pattern 5 in this section applies that model to brownfield codebases without redefining it — the pattern transfers directly.

- **To [Scaling Tool Use](../5-tool-use/4-scaling-tools.md):** The 85% token reduction from dynamic tool discovery applies to semantic codebase indexing by the same mechanism — metadata representation is dramatically cheaper than full-content representation while preserving the agent's ability to navigate to full content on demand.

- **To [Production Concerns](../7-practices/4-production-concerns.md):** The CLAUDE.md as convention encoding principle cited there is the foundation for Pattern 2 (Hierarchical Convention Files) and Pattern 6 (Tribal Knowledge Codification). The production concerns chapter provides the operational context; this section provides the codebase-scale application.

- **To [Specs as Source Code](../8-mental-models/3-specs-as-source-code.md):** ADRs (Pattern 3) and tribal knowledge codification (Pattern 6) are direct applications of the specs-as-source-code mental model. Tribal knowledge that has not been codified is source code that has not been checked in — it exists only in human memory and is lost when that memory is unavailable.

- **To [Multi-Agent Landscape](../6-patterns/10-multi-agent-landscape.md):** The 60% enterprise failure rate documented there is the explicit motivation for this section. The patterns here answer the open question at that section's close: what architectural patterns address brownfield context failure surfaces?

- **To [Gas Town Case Study](../../appendices/examples/gastown/_index.md):** Gas Town's 189K LOC Go system and its documented failure surfaces — tribal knowledge, undocumented APIs, implicit workflows — are the concrete evidence base for the brownfield failure modes this section addresses. The system demonstrates at production scale what happens when agents navigate complex legacy code without the context infrastructure described here.
