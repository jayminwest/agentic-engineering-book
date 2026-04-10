# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A book on agentic engineering—covering prompts, models, context, and tooling. Technical practitioner guide organized in chapters with structured frontmatter.

**Writing Style:** See [STYLE_GUIDE.md](STYLE_GUIDE.md) for voice, evidence standards, and structural requirements.

## Output Style

**Be concise.** Prefer:
- Bullet points over paragraphs
- Tables over prose comparisons
- Code blocks over descriptions
- 3-5 line summaries over exhaustive reports

**Structure findings as:**
```
## Topic
- Key point 1
- Key point 2

**Recommendation:** One sentence.

**Next:** What to do now.
```

**Skip:** Preamble, hedging, exhaustive context, restating the question.

## Book Structure

Content is organized under `chapters/` with book-style frontmatter (`part`, `chapter`, `section`, `order`).

### Part 1: Foundations

| Chapter | Directory | Contents |
|---------|-----------|----------|
| 1. Foundations | `chapters/1-foundations/` | Twelve leverage points in agentic systems |
| 2. Prompt | `chapters/2-prompt/` | Types, structuring, language patterns |
| 3. Model | `chapters/3-model/` | Selection, behavior, limitations, multi-model architectures, evaluation |
| 4. Context | `chapters/4-context/` | Fundamentals, strategies, patterns, multi-agent context |
| 5. Tool Use | `chapters/5-tool-use/` | Design, selection, restrictions, scaling, meta-tools |

### Part 2: Craft

| Chapter | Directory | Contents |
|---------|-----------|----------|
| 6. Patterns | `chapters/6-patterns/` | Plan-build-review, self-improving experts, orchestrator pattern |
| 7. Practices | `chapters/7-practices/` | Debugging, evaluation, cost/latency, production, workflow coordination, knowledge evolution |

### Part 3: Perspectives

| Chapter | Directory | Contents |
|---------|-----------|----------|
| 8. Mental Models | `chapters/8-mental-models/` | Pit of success, prompt maturity model, specs as source code, context as code |
| 9. Practitioner Toolkit | `chapters/9-practitioner-toolkit/` | Claude Code, Google ADK |

### Part 4: Appendices

| Section | Directory | Contents |
|---------|-----------|----------|
| Examples | `appendices/examples/` | Production examples and case studies |

## Slash Commands

### Book Management
- `/book:toc` - Regenerate TABLE_OF_CONTENTS.md from frontmatter
- `/book:chat <question>` - Have a conversation with the book content

### Knowledge Management
- `/knowledge:capture <insight>` - Store a learning in the appropriate location
- `/knowledge:expand <file> <topic>` - Add leading questions to an entry
- `/knowledge:research-external <topic>` - Research external sources

### Review
- `/review:questions <file>` - Suggest follow-up questions based on content
- `/review:clarity <file>` - Review for clarity without changing voice

## Frontmatter Schema

### Book Content

```yaml
---
title: Entry Title
description: Brief description
created: 2025-01-21
last_updated: 2025-01-21
tags: [tag1, tag2]
contributor: Your Name (Optional - for community contributions)
part: 1                    # Part number
part_title: Foundations    # Part name
chapter: 2                 # Chapter number
section: 0                 # 0 = chapter intro, 1+ = sections
order: 1.2.0              # Sort key (part.chapter.section)
---
```

### Agent Frontmatter

```yaml
---
name: agent-name
description: Brief action-oriented description
tools: Read, Glob, Grep
model: sonnet
color: yellow
output-style: concise-reference  # Optional
---
```

**⚠️ CRITICAL:** Never use colons in description values. Use `Expects SPEC` not `Expects: SPEC`. Colons cause silent agent discovery failures.

## Content Conventions

- Files prefixed with `_` are chapter/section indexes
- `_index.md` - Chapter introduction and overview
- Internal links use relative paths
- Timestamps in frontmatter: `YYYY-MM-DD`
- Ordering via `order` frontmatter field
- **Style:** Third-person, evidence-grounded, technical practitioner voice

## When Adding Content

1. Check if an existing entry fits using Glob/Grep
2. Prefer editing existing files over creating new ones
3. Follow [STYLE_GUIDE.md](STYLE_GUIDE.md) (third-person, evidence-grounded, required elements)
4. Update `last_updated` in frontmatter
5. Add book frontmatter (`part`, `chapter`, `section`, `order`)
6. Run `/book:toc` to regenerate table of contents
7. For community contributions, see [CONTRIBUTING.md](CONTRIBUTING.md)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- Contribution process
- Style guidelines
- Attribution conventions
- Code of conduct
