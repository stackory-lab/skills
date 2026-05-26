---
name: stackory-signals-usage
description: Use when helping an application developer integrate Stackory signals packages: analytics clients and providers, PostHog web or node adapters, user behavior signal stores, signal persistence, feature flags, experiment assignment, or analytics-from-signal plugins.
---

# Stackory Signals Usage

## Purpose

Help consumers add typed analytics, user signals, and experiment assignment with Stackory packages.

## Package Map

- `@stackory/signals-core`: framework-neutral signal store, analytics client, experiment helpers, and plugin interfaces.
- `@stackory/signals-posthog/web`: browser PostHog provider. Requires app-installed `posthog-js`.
- `@stackory/signals-posthog/node`: Node or edge-compatible PostHog provider. Requires app-installed `posthog-node`.

## Install

Core only:

```sh
pnpm add @stackory/signals-core
```

PostHog web:

```sh
pnpm add @stackory/signals-core @stackory/signals-posthog posthog-js
```

PostHog node:

```sh
pnpm add @stackory/signals-core @stackory/signals-posthog posthog-node
```

## Start Here

1. For analytics fan-out and PostHog providers, read `references/analytics.md`.
2. For local user signals and persistence, read `references/signals-store.md`.
3. For experiment bucketing and exposure tracking, read `references/experiments.md`.

## Usage Rules

- Keep web and node PostHog imports separate.
- `@stackory/signals-core` has no UI framework dependency; connect React, Vue, MobX, or native UI through `subscribe()`.
- Plugins must tolerate synchronous event delivery and should handle their own async work.
- Use stable user/client IDs as experiment seeds so assignments remain sticky.

