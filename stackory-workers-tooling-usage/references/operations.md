# Operations

## Deploy Workers

Deploy every Worker in topology order:

```sh
workers-tooling deploy staging
workers-tooling deploy production
```

Deploy a validated subset. The input order does not control execution order; the tool filters the requested IDs into topology order.

```sh
workers-tooling deploy staging --workers api,webhook
workers-tooling deploy production --workers api,worker-b,webhook
```

Preview the plan without publishing:

```sh
workers-tooling deploy staging --dry-run
workers-tooling deploy staging --workers webhook,api --dry-run
```

The dry-run output includes `command`, `args`, `cwd`, `packageDir`, and `workerId`. A real deploy stops on the first failed Worker and prints a JSON summary containing `successful`, `failed`, and `skipped` Worker IDs.

## Rotate Worker Secrets

List eligible Workers for a secret:

```sh
workers-tooling secret rotate --secret API_TOKEN --list-workers
```

Dry-run a generated rotation:

```sh
workers-tooling secret rotate --secret API_TOKEN --env staging --generate --dry-run
```

Rotate across all eligible Workers:

```sh
workers-tooling secret rotate --secret API_TOKEN --env production --from-env API_TOKEN_NEXT
```

Rotate a subset:

```sh
workers-tooling secret rotate --secret WEBHOOK_SECRET --env production --workers api,webhook --from-env WEBHOOK_SECRET_NEXT
```

Secrets are eligible only when the Worker includes the secret name in `requiredSecrets`. Exactly one value source is required:

- `--from-env <VAR_NAME>`
- `--generate`
- `--value <secret>`

Prefer `--from-env` or `--generate` because literal `--value` arguments can appear in shell history and process listings.

## Apply Local D1 Migrations

Local D1 migration presets are read from `topology.d1MigrationPresets`.

```sh
workers-tooling d1 apply-local app-db
workers-tooling d1 apply-local app-db workers/api
```

Example preset:

```ts
export default defineWorkersTopology({
  // ...
  d1MigrationPresets: {
    'app-db': {
      applyMode: 'materialize',
      databaseName: 'app-db-local',
      defaultWorkerDir: 'workers/api',
      migrationsDir: 'drizzle',
    },
  },
});
```

Use the optional `workerDir` argument when the preset's default Worker directory is not the command target.

