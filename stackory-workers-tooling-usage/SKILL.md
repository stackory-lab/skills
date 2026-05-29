---
name: stackory-workers-tooling-usage
description: Use when helping a repository maintainer configure or operate @stackory/workers-tooling for Cloudflare Workers topology validation, topology printing, ordered deploys, Worker secret rotation, or local D1 migrations.
---

# Stackory Workers Tooling Usage

## Purpose

Help consumers use `@stackory/workers-tooling` to keep Cloudflare Worker topology facts centralized and apply them through validation, deployment, secret rotation, and local D1 migration workflows.

## Install

```sh
pnpm add -D @stackory/workers-tooling wrangler
```

## Entry Points

- Config helper: `defineWorkersTopology` from `@stackory/workers-tooling/core`
- CLI binary: `workers-tooling`
- Main commands: `validate`, `print`, `deploy`, `secret rotate`, `d1 apply-local`

## Start Here

1. For topology config structure, read `references/topology-config.md`.
2. For validation and topology inspection, read `references/validate-and-print.md`.
3. For deployment, secret rotation, and local D1 migrations, read `references/operations.md`.

## Usage Rules

- Keep project-specific Worker, account, D1, KV, Queue, R2, and secret facts in the consumer's `workers-tooling.config.ts`, not in reusable package code.
- Use `validate` before deploys to compare each Worker's `wrangler.jsonc` against the central topology.
- Run deploys through `deploy <staging|production>` so requested Worker IDs are validated and executed in topology order.
- Use `--dry-run` before publishing deploys or rotating secrets.
- Declare every rotatable secret in each Worker's `requiredSecrets`; secret rotation only targets eligible workers.
- Store secret values outside committed files. Prefer `--from-env` or `--generate` over shell history-visible literal values.
- Use `d1MigrationPresets` for local D1 migration workflows; preset names are project-owned.
- Pass `--config <path>` for an explicit config file or `--root <path>` to choose a different starting directory for upward config discovery.
