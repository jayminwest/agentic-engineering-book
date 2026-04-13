# Changelog

All significant changes to the Agentic Engineering Knowledge Base are documented here.

Editions are dated (`Edition YYYY-MM-DD`) and tagged in git as `edition-YYYY-MM-DD`.
Categories: **Content** (chapters, sections), **Infrastructure** (agents, commands, adapters),
**Research** (external sources, monitoring), **Structure** (organization, frontmatter, TOC).

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
