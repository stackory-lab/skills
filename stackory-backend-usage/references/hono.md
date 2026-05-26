# Hono Integration

## Install

Install the Hono adapter and its peer:

```sh
pnpm add @stackory/backend-hono hono
```

Add `@stackory/backend-platform` if the app uses `ConsoleLogger`, `IPlatformServices`, or service callers.

## App Setup

```ts
import { Hono } from 'hono';
import {
  createInternalAuthMiddleware,
  createRequestLoggerMiddleware,
  errorHandlerMiddleware,
  userIdMiddleware,
} from '@stackory/backend-hono';
import type { ILogger } from '@stackory/backend-platform';

type Env = {
  Bindings: {
    INTERNAL_SECRET: string;
  };
  Variables: {
    logger?: ILogger;
    userId?: string;
  };
};

const app = new Hono<Env>();

app.onError(errorHandlerMiddleware);

app.use(
  '*',
  createRequestLoggerMiddleware({
    getLogger: (c) => c.get('logger'),
    setLogger: (c, logger) => c.set('logger', logger),
  }),
);

app.use(
  '/internal/*',
  createInternalAuthMiddleware({
    getSecret: (c) => c.env.INTERNAL_SECRET,
    maxTimeDiff: 60,
    ignorePaths: [/\/health$/],
  }),
);

app.use('/api/*', userIdMiddleware);
```

## Error Handling

Throw `StandardError` from `@stackory/constants` when callers need a stable error code. `errorHandlerMiddleware` converts known errors to the shared request-result shape and converts unknown errors to `INTERNAL_SERVER_ERROR`.

```ts
import { error } from '@stackory/constants';

throw new error.StandardError({
  code: error.errorCode.INVALID_ARGUMENT,
  message: 'Missing projectId',
});
```

## Gotchas

- Do not install a second incompatible Hono version in a framework package. The application owns `hono`.
- Do not use `userIdMiddleware` on public routes.
- Body signing is skipped for multipart and octet-stream requests by the signing/verification pair. Keep both sides on Stackory helpers.

