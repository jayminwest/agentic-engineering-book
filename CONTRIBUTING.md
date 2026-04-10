# Contributing to Agentic Engineering Knowledge Base

We welcome contributions from practitioners who are actively practicing agentic engineering.

## Who Can Contribute

- Practitioners with production experience
- Contributors following the [STYLE_GUIDE.md](STYLE_GUIDE.md)

## Types of Contributions

### Production Stories
Real-world examples of agentic systems in production:
- System architecture and design decisions
- Performance characteristics and metrics
- Lessons learned and failure modes
- Cost and latency optimizations

### Debugging Traces
Documented debugging sessions showing:
- Problem identification process
- Hypothesis formation and testing
- Solution implementation
- Verification steps

### Patterns and Practices
Reusable patterns from your work:
- Prompt engineering techniques
- Tool design patterns
- Context management strategies
- Evaluation approaches

### Corrections and Improvements
- Technical inaccuracies
- Outdated information
- Clarity improvements
- Missing context or examples

### Questions
Well-formed questions that would benefit the community:
- Edge cases not covered
- Trade-offs needing exploration
- Integration scenarios
- Production concerns

## Contribution Process

### 1. Fork and Branch
```bash
# Fork the repository on GitHub
# Clone your fork
git clone https://github.com/YOUR_USERNAME/agentic-engineering-book.git
cd agentic-engineering-book

# Create a feature branch
git checkout -b contrib/your-topic-name
```

### 2. Make Changes
- Follow the [STYLE_GUIDE.md](STYLE_GUIDE.md) conventions
- Use third-person voice
- Ground claims in evidence
- Include required elements (Problem/Solution/Patterns)
- Add proper frontmatter

### 3. Submit Pull Request
- Write a clear PR description
- Reference any related issues or discussions
- Link to production examples (if public)

### 4. Review Process
- Maintainers will review for style and technical accuracy
- Revisions may be requested
- Attribution will be preserved

## Style Guidelines

See [STYLE_GUIDE.md](STYLE_GUIDE.md) for complete requirements. Key points:

**Voice:**
- Third-person technical writing
- Evidence-grounded claims
- No marketing language or hype
- Specifics over generalities

**Structure:**
- Problem statement
- Solution description
- Implementation patterns
- Trade-offs and constraints
- Evidence from production

**Examples:**
- Real code, not pseudocode
- Complete enough to understand
- Annotated with context
- Include failure modes

## Frontmatter Requirements

All content files must include proper frontmatter:

```yaml
---
title: Your Entry Title
description: Brief description of the content
created: 2025-01-21
last_updated: 2025-01-21
tags: [relevant, tags]
contributor: Your Name (Optional - for community contributions)
part: 1                    # Part number (if applicable)
part_title: Foundations    # Part name (if applicable)
chapter: 2                 # Chapter number (if applicable)
section: 1                 # Section number (if applicable)
order: 1.2.1              # Sort key (if applicable)
---
```

### Contributor Field

The optional `contributor` field allows attribution:
```yaml
contributor: Jane Smith
contributor: Jane Smith (Company Name)
contributor: Jane Smith - @github_username
```

Maintainers will review contributions before merging with attribution.

## Content Placement

### New Patterns or Practices
- Add to existing chapter sections when possible
- Propose new sections in Chapter 6 (Patterns) or Chapter 7 (Practices)
- Include in PR description why new section is needed

### Examples
- Add to `appendices/examples/` if substantial
- Include inline if illustrating a specific point
- Follow existing example structure

### Corrections
- Edit existing files directly
- Update `last_updated` in frontmatter
- Note correction in PR description

## Code of Conduct

- Be respectful and professional
- Assume good intent
- Focus on technical merit
- No self-promotion or marketing
- No plagiarism - cite sources
- Respect confidential/proprietary information

## Questions?

- Open a GitHub Discussion for clarification
- Tag maintainers in your PR if urgent

## License

By contributing, you agree that your contributions will be licensed under the same terms as the project.

---

Thank you for contributing to the Agentic Engineering Knowledge Base.
