# Internal Service Calls

## Caller Side

Create an `IServiceCaller` from a transport and add HMAC signing with `createHmacSigningInterceptor`.

```ts
import { createServiceCaller } from '@stackory/backend-platform';
import { createHmacSigningInterceptor } from '@stackory/backend-utils';
import { createCloudflareFetcherTransport } from '@stackory/backend-platform-cloudflare';

const serviceCaller = createServiceCaller(createCloudflareFetcherTransport(env), [
  createHmacSigningInterceptor(() => Promise.resolve(env.INTERNAL_SECRET)),
]);

const response = await serviceCaller.call('USER_SERVICE', {
  path: '/internal/users/123',
  method: 'GET',
});
```

For Hono RPC clients, wrap the `IServiceCaller` with `createInternalRpcClient`.

```ts
import {
  createInternalRpcClient,
} from '@stackory/backend-hono';
import type { AppType } from './remote-service';

const client = createInternalRpcClient<AppType>({
  baseUrl: 'https://internal.local',
  serviceCaller,
  serviceName: 'USER_SERVICE',
  getHeaders: () => ({ 'X-Trace-Id': crypto.randomUUID() }),
});
```

## Receiver Side

Protect internal routes with `createInternalAuthMiddleware`.

```ts
app.use(
  '/internal/*',
  createInternalAuthMiddleware({
    getSecret: (c) => c.env.INTERNAL_SECRET,
    ignorePaths: [/\/health$/],
  }),
);
```

## Response Helpers

Use shared response helpers for consistent API envelopes.

```ts
import { successResponse, errorResponse } from '@stackory/backend-utils';

return c.json(successResponse({ ok: true }));
return c.json(errorResponse(error.errorCode.INVALID_ARGUMENT, 'Bad input'), 400);
```

## Gotchas

- Both caller and receiver must use the same internal secret and Stackory signing helpers.
- Do not hand-roll signature string construction.
- `createInternalRpcClient` throws `InternalRpcError` for non-2xx responses.
