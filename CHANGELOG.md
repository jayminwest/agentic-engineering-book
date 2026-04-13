# Changelog

All significant changes to the Agentic Engineering Knowledge Base are documented here.

Editions are dated (`Edition YYYY-MM-DD`) and tagged in git as `edition-YYYY-MM-DD`.
Categories: **Content** (chapters, sections), **Infrastructure** (agents, commands, adapters),
**Research** (external sources, monitoring), **Structure** (organization, frontmatter, TOC).

---

## Edition 2026-04-13

_Changes since edition-2026-04-12 (5 commits)_

### Content

- **New Chapter 6 — Harnesses**: Added as fifth foundational pillar (7 sections: what-is-a-harness, harness-stack, harness-categories, harness-as-control-system, harness-engineering, security-permissions-trust, designing-for-your-context)
- **Chapter renumbering**: Patterns 6→7, Practices 7→8, Mental Models 8→9, Practitioner Toolkit 9→10 — all frontmatter updated atomically
- **Part 1 Foundations** now covers five pillars: Prompt, Model, Context, Tool Use, Harnesses

### Structure

- **CLAUDE.md**: Updated chapter tables for new 10-chapter layout with Harnesses at ch6
- **TABLE_OF_CONTENTS.md**: Regenerated to reflect chapter renumbering
- All `_index.md` files updated with corrected part/chapter/order frontmatter

### Expertise Learnings (Mulch)

| Domain | Type | Learning |
|--------|------|---------|
| knowledge | failure | Cross-references to chapters that have been renumbered will silently break if up |
| knowledge | decision | core-five-pillars-harness-addition |
| knowledge | decision | harness-chapter-section-ordering |
| knowledge | pattern | proposal-as-spec |
| knowledge | pattern | directory-rename-reverse-order-execution |
| knowledge | pattern | content-migration-elevate-conceptual-leave-implementation |
| knowledge | pattern | new-chapter-index-structure |
| knowledge | pattern | definitional-crystallization-sourcing |
| knowledge | pattern | ascii-diagram-update-for-pillar-expansion |
| orchestration | pattern | parallel-proposal-builds |
| claude-config | convention | Edition-based changelog versioning: use 'edition-YYYY-MM-DD' lightweight git tag |
| github | convention | Public repo sync filters CHANGELOG.md to strip Infrastructure sections via Pytho |
| knowledge | convention | When renaming chapter directories, frontmatter batch updates must be atomic: all |
| knowledge | convention | When a new foundational chapter is inserted into Part 1, update CLAUDE.md, READM |
| knowledge | guide | inserting-new-chapter-into-existing-sequence |

_15 records captured_

---

## Edition 2026-04-12 --- Initial Release

This edition establishes the baseline for the changelog system.
All content prior to this point (123 commits) constitutes the initial release.

### Content

- **9 chapters across 4 parts** --- Foundations (ch1-5), Craft (ch6-7), Perspectives (ch8-9), Appendices
- **60 chapter files** covering prompts, models, context, tool use, patterns, practices, mental models, and practitioner toolkit
- **Chapter 6 --- Patterns**: 11 sections including plan-build-review, self-improving experts, orchestrator pattern, autonomous loops, ReAct, human-in-the-loop, progressive disclosure, expert swarm, multi-agent collaboration, multi-agent landscape, production multi-agent systems
- **Chapter 7 --- Practices**: 7 sections including debugging agents, evaluation, cost/latency, production concerns, workflow coordination, knowledge evolution, operating agent swarms
- **Chapter 8 --- Mental Models**: 7 sections including pit of success, prompt maturity model, specs as source code, context as code, execution topologies, design as bottleneck, software factories
- **Chapter 9 --- Practitioner Toolkit**: 5 sections covering Claude Code, Google ADK, IDE integrations, agent frameworks, multi-agent workspace managers
- **Appendices**: Production examples from kotadb, gastown, overstory, pi-mono

### Research

- 12 monitor-sourced proposals integrated across 18 chapter files
- External source coverage: Anthropic blog/changelog, frontier lab SDKs, arxiv (cs.AI, cs.LG, cs.CL), Hacker News (50+ points)
- Individual voice feeds via sources.yaml

### Structure

- Edition-based versioning introduced with `edition-YYYY-MM-DD` git tags
- Dual-repo publishing: private source repo syncs filtered content to jayminwest/agentic-engineering-book
- Web version at jayminwest.com/agentic-engineering-book (updated daily at 6am)
- CHANGELOG.md added to project with filtered public sync (Infrastructure sections stripped)
