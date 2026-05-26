# CLI and CI

## Commands

Write generated files:

```sh
sync-agent-context sync --config ai-context.config.mjs
```

Check generated files without writing:

```sh
sync-agent-context check --config ai-context.config.mjs
```

Use an explicit root directory:

```sh
sync-agent-context sync --root-dir . --config ai-context.config.mjs
```

`--config` defaults to `ai-context.config.mjs`.

## Behavior

- `sync` writes files when generated content differs.
- `check` compares generated content to disk.
- `check` exits non-zero when any generated file is missing or out of date.
- Both commands print the generated output paths relative to `rootDir`.

## package.json Scripts

```json
{
  "scripts": {
    "agent-context:sync": "sync-agent-context sync --config ai-context.config.mjs",
    "agent-context:check": "sync-agent-context check --config ai-context.config.mjs"
  }
}
```

## GitHub Actions Example

```yaml
name: Agent Context

on:
  pull_request:

jobs:
  check-agent-context:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm agent-context:check
```

## Gotchas

- Commit generated files if CI runs `check`.
- Do not edit generated files directly; run `sync` after changing fragments or config.
- If generated output changes unexpectedly, compare fragment order, `fragmentTitles`, `specificKey`, and `separator`.

