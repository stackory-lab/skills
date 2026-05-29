# Validate And Print

## Validate Wrangler Configs

Use `validate` to compare each Worker's `wrangler.jsonc` with the central topology.

```sh
workers-tooling validate
workers-tooling validate --config workers-tooling.config.ts
workers-tooling validate --root .
```

Validation checks expected Worker runtime facts, including:

- `compatibility_date`, `compatibility_flags`, and `account_id`
- `main` entrypoint and `workers_dev`
- service bindings
- Durable Object bindings and owned Durable Object migrations
- Queue producers and consumers
- D1, R2, and KV bindings
- expected staging and production vars from `globalVars` and `workerVars`

If validation fails, update the central topology or the Worker's `wrangler.jsonc` so they describe the same facts. Do not duplicate new resource facts across multiple places without also adding them to the topology.

## Print Resolved Topology

Use `print` to inspect the loaded topology as JSON.

```sh
workers-tooling print
workers-tooling print --config workers-tooling.config.ts
```

Use this before debugging deploy, validation, secret, or D1 commands. It confirms which config file was loaded and what the tool sees after TypeScript or module evaluation.
