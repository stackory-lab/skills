# Workers Topology Config

## Config Discovery

Commands discover topology config from the current directory upward by default. Supported default file names are:

- `workers-tooling.config.ts`
- `workers-tooling.config.mjs`
- `workers-tooling.config.js`

Use `--config <path>` for an explicit config file or `--root <path>` to choose a different starting directory for upward discovery. When config is discovered, Worker paths resolve from the directory containing the discovered config file.

The config must export an `IWorkerTopology` directly, or an object containing one of these exports:

- `default`
- `repoWorkerTopology`
- `topology`
- `workerTopology`

## Minimal Config

```ts
import { defineWorkersTopology } from '@stackory/workers-tooling/core';

export default defineWorkersTopology({
  platform: {
    compatibilityDate: '2026-03-03',
    compatibilityFlags: ['nodejs_compat'],
    accountIdByEnv: {
      staging: 'example-account-id',
      production: 'example-account-id',
    },
  },
  workers: [
    {
      id: 'api',
      dir: 'workers/api',
      entry: 'src/index.ts',
      devPort: 8787,
      workersDevByEnv: {
        staging: true,
        production: false,
      },
      localVarsFiles: ['.dev.vars'],
      requiredSecrets: ['API_TOKEN'],
    },
  ],
  durableObjects: [],
  externalDurableObjectBindings: [],
  serviceBindings: [],
  queues: [],
  queueProducerBindings: [],
  d1: [],
  r2: [],
  kv: [],
  globalVars: [],
  workerVars: [],
});
```

## Topology Sections

- `platform`: shared compatibility date, compatibility flags, and Cloudflare account IDs by environment.
- `workers`: Worker IDs, directories, entries, dev ports, `workers_dev` settings, local vars files, required secrets, and optional deploy overrides.
- `durableObjects`: Durable Object classes owned by Workers.
- `externalDurableObjectBindings`: Durable Object bindings consumed from another Worker script.
- `serviceBindings`: Worker-to-Worker service bindings.
- `queues` and `queueProducerBindings`: Queue resources, consumers, and producer bindings.
- `d1`, `r2`, `kv`: Cloudflare resource bindings by owner or consumer Worker.
- `globalVars` and `workerVars`: expected environment vars for staging and production.
- `d1MigrationPresets`: optional local D1 migration presets for `d1 apply-local`.

## Deploy Overrides

Each Worker can override deploy behavior:

```ts
{
  id: 'api',
  dir: 'workers/api',
  entry: 'src/index.ts',
  devPort: 8787,
  workersDevByEnv: { staging: true, production: false },
  localVarsFiles: ['.dev.vars'],
  requiredSecrets: ['API_TOKEN'],
  deploy: {
    command: 'pnpm',
    args: ['--filter', '${packageName}', 'run', 'deploy:${env}'],
    cwd: '.',
    packageName: '@worker/api',
    requirePackageScript: true,
  },
}
```

Supported deploy tokens in `args`:

- `${env}`: `staging` or `production`
- `${packageName}`: explicit `deploy.packageName` or default `@worker/<workerId>`
- `${workerId}`: the Worker topology ID

Without a deploy override, the default command is `pnpm exec wrangler deploy --env <env>` from the Worker directory.
