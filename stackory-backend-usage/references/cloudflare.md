# Cloudflare Platform Integration

## Install

```sh
pnpm add @stackory/backend-platform @stackory/backend-platform-cloudflare
```

If using Sentry:

```sh
pnpm add @sentry/cloudflare
```

## Build Platform Services

Use `createCloudflarePlatformServices` at Worker startup to wrap raw bindings into Stackory platform contracts.

```ts
import {
  CloudflareQueueBatch,
  SentryCloudflareMonitoring,
  createCloudflarePlatformServices,
} from '@stackory/backend-platform-cloudflare';

export interface Env {
  SESSIONS: KVNamespace;
  ASSETS: R2Bucket;
  JOBS: Queue<JobMessage>;
  SENTRY_DSN?: string;
  ENVIRONMENT: string;
}

export default {
  fetch(request: Request, env: Env) {
    const monitoring = new SentryCloudflareMonitoring();
    const platform = createCloudflarePlatformServices({
      kv: { sessions: env.SESSIONS },
      storage: { assets: env.ASSETS },
      queue: { jobs: env.JOBS },
      monitoring,
    });

    const app = buildApp(platform);
    const wrapped = monitoring.wrapApp(app, {
      dsn: env.SENTRY_DSN,
      environment: env.ENVIRONMENT,
      enabled: Boolean(env.SENTRY_DSN),
    });

    return wrapped.fetch(request, env);
  },

  async queue(batch: MessageBatch<JobMessage>) {
    const queueBatch = new CloudflareQueueBatch(batch);
    for (const message of queueBatch.messages) {
      try {
        await processJob(message.body);
        message.ack();
      } catch {
        message.retry({ delaySeconds: 60 });
      }
    }
  },
};
```

## Using Platform Services

```ts
import type { IPlatformServices } from '@stackory/backend-platform';

export function buildApp(platform: IPlatformServices) {
  const app = new Hono();

  app.get('/asset/:key', async (c) => {
    const object = await platform.storage?.assets.get(c.req.param('key'));
    if (!object) {
      return c.notFound();
    }
    const headers = new Headers();
    object.writeHttpMetadata(headers);
    return new Response(object.body, { headers });
  });

  return app;
}
```

## Gotchas

- Do not import `@stackory/backend-platform-cloudflare` in browser code or non-Cloudflare runtimes.
- Call `SentryCloudflareMonitoring.wrapApp` after the app is fully built.
- Construct `CloudflareQueueBatch` inside the `queue()` handler and do not keep it across invocations.
