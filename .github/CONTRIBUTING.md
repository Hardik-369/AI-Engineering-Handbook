# Contributing to the AI Engineering Handbook

Thank you for your interest in contributing! This handbook is a community-driven resource for AI engineers. Your contributions help make it better for everyone.

---

## Code of Conduct

This project adheres to the [Contributor Covenant Code of Conduct](./CODE_OF_CONDUCT.md). By participating, you agree to uphold this code. Report unacceptable behavior to the project maintainers.

---

## How to Contribute

### Types of Contributions

We welcome contributions in many forms:

| Type | Examples |
|------|----------|
| **Content** | New chapters, sections, examples, cheatsheets |
| **Corrections** | Fixing errors, outdated information, broken links |
| **Clarifications** | Improving explanations, adding context, better examples |
| **Patterns** | New prompt patterns, architecture patterns, code patterns |
| **Resources** | Adding papers, tools, courses, communities to the resource lists |
| **Code** | Better code examples, additional language support, scripts |
| **Translation** | Translating content into other languages |
| **Review** | Technical review of existing content |

### Contribution Workflow

1. **Fork the repository** to your GitHub account.
2. **Create a branch** for your work:
   ```
   git checkout -b fix/typo-in-chapter-3
   git checkout -b feature/agent-patterns-section
   git checkout -b resource/add-safety-papers
   ```
3. **Make your changes** following the style guide below.
4. **Test locally** if applicable (linting, link checking, code examples).
5. **Push and open a Pull Request** with a clear description.

---

## Style Guide

### General

- Use **American English** spelling (e.g., "optimize" not "optimise").
- Use **sentence case** for headings (e.g., "How to build an agent" not "How to Build an Agent").
- Keep lines under **100 characters** where possible for readability in diffs.
- Use **descriptive link text** (e.g., "See the LangChain docs" not "click here").
- Reference sources when presenting specific claims or data.

### Markdown

- Use ATX-style headings (``#``, ``##``, ``###``) with a space after ``#``.
- Leave a blank line before and after headings.
- Use ``|`` tables with aligned columns for structured data.
- Use `` ```language `` fenced code blocks with language identifier.
- Use ``**bold**`` for emphasis, `` `code` `` for inline code/commands.
- Use ``- `` for unordered lists, ``1. `` for ordered lists.

### Code Examples

All code examples must:

1. **Be complete and runnable** when possible.
2. **Use modern practices** (e.g., `async`/`await`, type hints, error handling).
3. **Include imports** explicitly.
4. **Use descriptive variable names**.
5. **Avoid comments** unless explaining a non-obvious design decision.
6. **Be idiomatic** to the language (Python: use f-strings, `pathlib`, type hints; JS/TS: use modern ESM, async).

Example:
```python
import asyncio
from openai import AsyncOpenAI

async def generate_summary(text: str) -> str:
    client = AsyncOpenAI()
    response = await client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "Summarize in 2-3 sentences."},
            {"role": "user", "content": text}
        ]
    )
    return response.choices[0].message.content
```

### Diagrams

- Use **Mermaid** for diagrams (```mermaid code blocks).
- Keep diagrams simple and readable.
- Add a brief description above the diagram explaining what it shows.

### Links

- Use relative links for internal references: `[Chapter 1](../chapters/01-foundations.md)`
- Use full URLs for external resources.
- Prefer `https://` over `http://`.
- Check that links are not broken before submitting.

---

## Pull Request Process

1. **Title**: Concise description of the change (e.g., "Fix typo in RAG chapter", "Add agent memory patterns section").
2. **Description**: Include:
   - What you changed and why
   - Related issue (if applicable)
   - Any decisions you made
   - Checklist of what you verified
3. **Labels**: Apply relevant labels (e.g., `content`, `fix`, `resource`, `improvement`).
4. **Review**: At least one maintainer must review and approve.
5. **CI**: All checks must pass (linting, link checking, etc.).
6. **Merge**: Squash and merge is preferred for clean history.

### PR Size Guidelines

- **Small PRs are preferred** (under 200 lines changed when possible).
- Focus on one topic per PR.
- Large additions (new chapters, major rewrites) should be discussed in an issue first.

---

## Review Checklist

Reviewers should verify:

### Content
- [ ] Factually accurate and up-to-date
- [ ] Appropriate level of detail for the intended audience
- [ ] No duplication of existing content without good reason
- [ ] Claims are supported by references or experience

### Style
- [ ] Follows the style guide (spelling, formatting, headings)
- [ ] Clear, readable, and well-organized
- [ ] Code examples are correct and idiomatic
- [ ] Links are valid and use proper formatting

### Structure
- [ ] Fits into the existing table of contents
- [ ] Cross-references related content where appropriate
- [ ] Table of contents / index is updated if needed

### Technical
- [ ] Code examples run correctly
- [ ] Mermaid diagrams render properly
- [ ] No broken links
- [ ] Screenshots/outputs are up to date

---

## File Organization

```
AI-Engineering-Handbook/
├── chapters/           # Chapter content
├── cheatsheets/        # Quick reference guides
├── resources/          # Curated resource lists
├── papers/             # Paper reading list
├── .github/            # Repository configuration
│   ├── ISSUE_TEMPLATE/ # Issue templates
│   ├── CONTRIBUTING.md # This file
│   └── FUNDING.yml     # Funding configuration
└── README.md           # Main handbook entry point
```

---

## Community Guidelines

- **Be respectful**: Disagreement is fine, personal attacks are not.
- **Assume good faith**: Most contributions are attempts to help.
- **Give constructive feedback**: Explain why something should change, not just that it should.
- **Credit others**: If you build on someone's work, acknowledge it.
- **Keep discussions focused**: Stay on topic in issues and PRs.

---

## Getting Help

- Open an issue for questions or discussions.
- Tag maintainers if a PR has been waiting for review (>1 week).
- Check existing issues and PRs before creating new ones.

Thank you for contributing to the AI Engineering Handbook!
