# Errors and API Client

## Standard Errors

```ts
import { error } from '@stackory/constants';

throw new error.StandardError({
  code: error.errorCode.INVALID_ARGUMENT,
  message: 'projectId is required',
  data: { field: 'projectId' },
});
```

Useful common codes include:

- `INVALID_ARGUMENT`
- `UNAUTHORIZED`
- `PERMISSION_DENIED`
- `RATE_LIMIT_EXCEEDED`
- `INTERNAL_SERVER_ERROR`
- `SERVICE_UNAVAILABLE`
- `AUTH_INTERNAL_MISSING`
- `AUTH_INTERNAL_EXPIRED`
- `AUTH_INTERNAL_SIG_ERROR`
- `AUTH_USER_ID_MISSING`
- `AUTH_BEARER_ERROR`
- `AUTH_CONNECT_TOKEN_ERROR`

## API Client

```ts
import { ApiError, createApiClient } from '@stackory/utils';

const api = createApiClient({
  baseUrl: 'https://api.example.com',
  hooks: {
    beforeRequest: [
      async (request) => {
        const headers = new Headers(request.headers);
        headers.set('Authorization', `Bearer ${await getToken()}`);
        return new Request(request, { headers });
      },
    ],
  },
});

try {
  const profile = await api<Profile>('/v1/profile');
} catch (error) {
  if (error instanceof ApiError) {
    console.error(error.status, error.data);
  }
}
```

## Response Shape

Backend responses often use the shared request-result shape from `@stackory/backend-utils`:

```ts
interface IRequestResult<T> {
  code: number;       // 0 means success
  message: string;    // "success" for successful responses
  data: T;
}
```

When consuming APIs, inspect the target service contract; `createApiClient` only parses response bodies and throws on non-2xx status.
