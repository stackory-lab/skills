---
name: stackory-agent-context-usage
description: Use when helping an application or repository maintainer integrate @stackory/sync-agent-context to generate or check AI agent context files such as AGENTS.md, CLAUDE.md, GEMINI.md, or other Markdown instruction documents from shared fragments and agent-specific sections.
---

# Stackory Agent Context Usage

## Purpose

Help consumers configure and use `@stackory/sync-agent-context` to generate deterministic AI agent instruction files from reusable Markdown fragments.

## Install

```sh
pnpm add -D @stackory/sync-agent-context
```

## Entry Points

- CLI write mode: `sync-agent-context sync --config ai-context.config.mjs`
- CLI check mode: `sync-agent-context check --config ai-context.config.mjs`
- API: `defineConfig`, `syncAiContext`, `buildDocument`, `loadSpecificInstructions`

## Start Here

1. For a complete config file, read `references/config.md`.
2. For fragment file layout and heading behavior, read `references/fragments.md`.
3. For CLI and CI usage, read `references/cli-and-ci.md`.
4. For programmatic API usage, read `references/api.md`.

## Usage Rules

- Keep shared content in fragment files and generate final agent files from config.
- Use one document profile per generated output file.
- Use `specificPath` for agent-specific additions keyed by `specificKey`.
- Use `check` mode in CI when generated files must stay committed and up to date.
- Do not edit generated output files directly; edit fragments, specific sections, or config instead.

