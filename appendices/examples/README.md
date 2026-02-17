---
title: Examples
description: Real-world Claude Code configurations demonstrating patterns from the book
created: 2026-01-30
last_updated: 2026-01-30
tags: [examples, reference, kotadb]
part: 4
part_title: Appendices
chapter: 10
section: 0
order: 4.10.0
---

# Examples

Real-world Claude Code configurations demonstrating patterns from the book.

## Available Examples

### kotadb

A production database project (Bun + TypeScript + Supabase) with comprehensive agentic infrastructure. Demonstrates mature patterns from multiple book chapters.

**Location:** `kotadb/.claude/`

#### Key Components

| Component | Path | Demonstrates |
|-----------|------|--------------|
| Agent Registry | `agents/` | Multi-agent architecture with capability-based routing |
| Slash Commands | `commands/` | Hierarchical command organization with template categories |
| Expert System | `commands/experts/` | Domain specialists with plan/review/improve pattern |
| Orchestrator | `commands/workflows/orchestrator.md` | End-to-end workflow automation |
| Conditional Docs | `commands/docs/conditional_docs/` | Layer-specific documentation loading |

#### Pattern Mappings

| Book Chapter | kotadb Implementation | Key Files |
|--------------|----------------------|-----------|
| [Plan-Build-Review](../../chapters/6-patterns/1-plan-build-review.md) | Expert system with plan/review/improve commands | `experts/*/` |
| [Orchestrator Pattern](../../chapters/6-patterns/3-orchestrator-pattern.md) | Multi-agent workflow coordination | `agents/orchestrator-agent.md`, `workflows/orchestrator.md` |
| [Tool Design](../../chapters/5-tool-use/1-tool-design.md) | Agent tool restrictions and capability scoping | `agents/*.md` (tool lists) |
| [Context Strategies](../../chapters/4-context/2-context-strategies.md) | Conditional docs for layer-specific context loading | `commands/docs/conditional_docs/` |
| [Multi-Agent Context](../../chapters/4-context/4-multi-agent-context.md) | Context isolation via agent boundaries | `agents/scout-agent.md` vs `agents/build-agent.md` |

#### Agent Architecture

```
┌─────────────────┐
│  orchestrator   │ (opus) - coordinates, delegates
└────────┬────────┘
         │
    ┌────┴────┬────────────┐
    ▼         ▼            ▼
┌───────┐ ┌───────┐  ┌───────────┐
│ scout │ │ build │  │  review   │
│(haiku)│ │(sonnet)│  │  (haiku)  │
└───────┘ └───────┘  └───────────┘
 read-only  full     read-only
            access   + analysis
```

**Model Selection Rationale:**
- `haiku` for read-only tasks (fast, cheap)
- `sonnet` for implementation (balanced)
- `opus` for orchestration (complex coordination)

#### Expert System

Seven domain experts, each with three commands:

| Expert | Domain | Files |
|--------|--------|-------|
| architecture | Path aliases, component boundaries | `architecture-expert/` |
| testing | Antimocking, test patterns | `testing-expert/` |
| security | RLS, authentication, input validation | `security-expert/` |
| integration | MCP, Supabase, external APIs | `integration-expert/` |
| ux | Accessibility, component usage | `ux-expert/` |
| cc_hook | Pre-commit hooks, automation | `cc_hook_expert/` |
| claude-config | CLAUDE.md, settings.json | `claude-config/` |

**Command Pattern (per expert):**
- `*_plan` - Analyze requirements from domain perspective
- `*_review` - Review code changes from domain perspective
- `*_improve` - Self-improve by analyzing git history

#### Slash Command Categories

Commands organized by template category (output format):

| Category | Description | Example Commands |
|----------|-------------|------------------|
| Message-Only | Single text output | `automation/generate_branch_name.md` |
| Path Resolution | Returns file path/URL | `automation/find_plan_file.md` |
| Action | Performs operations, returns summary | `git/commit.md`, `workflows/build.md` |
| Structured Data | Returns JSON | `tasks/query_phase.md` |

#### Notable Patterns

**Conditional Documentation Loading:**
```
commands/docs/conditional_docs/
├── app.md        # Backend/API layer
├── automation.md # ADW workflows, agent orchestration
└── web.md        # Frontend/UI (placeholder)
```
Agents load only relevant documentation for their layer, reducing context usage.

**Orchestrator State Management:**
- Checkpoint-based recovery for resume-after-failure
- State files at `automation/agents/<adw_id>/orchestrator/state.json`
- Supports `--resume`, `--dry-run`, `--skip-cleanup` flags

---

## Additional Examples

### gastown

A Go-based multi-agent workspace manager demonstrating production-grade daemon-based coordination. Validates patterns from the Expert Swarm, Production Multi-Agent Systems, and Operating Agent Swarms chapters.

**Location:** `gastown/`

### overstory

A TypeScript/Bun-based swarm coordination system demonstrating session-based multi-agent orchestration. Provides a companion case study to Gas Town, validating patterns as language-agnostic.

**Location:** `overstory/`

---

## Using These Examples

1. **Study the structure** - Note how `.claude/` directories organize agents, commands, and documentation
2. **Follow the mappings** - Use the pattern tables to connect examples to book concepts
3. **Adapt, don't copy** - Examples are production configurations; extract principles, not verbatim code
4. **Check the READMEs** - Each example contains its own documentation explaining the rationale

## Adding Examples

When contributing new examples:

1. Create a subdirectory under `appendices/examples/`
2. Include a `README.md` explaining the example's purpose
3. Add pattern mappings showing which book chapters apply
4. Update this file with the new example summary
5. Run `/book:toc` to regenerate the table of contents
