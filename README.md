# Agentic Engineering: The Book

> [!CAUTION]
> **This book is maintained by LLMs using the techniques described within.**
> Content is written and updated by Claude using agentic workflows, slash commands, and self-improving expert agents. Human review ensures accuracy, but the authoring process itself is an exercise in agentic engineering.

A practical guide to building agentic systems—covering prompts, models, context, and tooling. Written from hands-on experience, not abstractions.

## Structure

```
├── PREFACE.md           # Book introduction
├── TABLE_OF_CONTENTS.md # Auto-generated from frontmatter
├── chapters/            # Main content organized by topic
│   ├── 1-foundations/   # Part 1: Foundations
│   ├── 2-prompt/
│   ├── 3-model/
│   ├── 4-context/
│   ├── 5-tool-use/
│   ├── 6-patterns/      # Part 2: Craft
│   ├── 7-practices/
│   ├── 8-mental-models/ # Part 3: Perspectives
│   └── 9-practitioner-toolkit/
├── appendices/          # Supplementary materials
│   └── examples/        # Real configs from projects
└── .claude/             # Claude Code configuration
```

## Commands

This book includes Claude Code slash commands for content maintenance:

- `/book:toc` - Regenerate TABLE_OF_CONTENTS.md from frontmatter
- `/knowledge:capture` - Store a new learning
- `/knowledge:expand` - Add questions to an entry
- `/review:questions` - Suggest follow-up questions based on content

## Content Conventions

- All content is markdown with YAML frontmatter
- Files prefixed with `_` are chapter/section indexes
- Ordering controlled by `order` field in frontmatter (e.g., `1.2.0`)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines. This book is developed through the [Agentic Engineering Skool community](https://www.skool.com/agentic-engineering).

## Author

Jaymin West
