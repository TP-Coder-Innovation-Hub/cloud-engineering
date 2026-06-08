# AGENTS.md

## Repository Structure

Cloud engineering curriculum organized as directory-per-topic. Each `.md` file is a self-contained lesson.

## Style Conventions

- 200-500 words per file
- YAML configs and pseudocode for examples (language-agnostic)
- Mermaid diagrams for architecture and flows
- Level badges: `[Entry]`, `[Mid]`, `[Senior]`
- No emojis. Direct, concise tone.
- Assumes reader has programming experience

## File Naming

- `NN-topic-slug.md` where NN is zero-padded sequence number
- Directories: `NN-category-slug/`

## Build / Lint Commands

```bash
# Markdown lint (if configured)
npx markdownlint-cli2 "**/*.md"
```

## Commit Style

Conventional commits: `feat:`, `docs:`, `fix:`, `refactor:`
