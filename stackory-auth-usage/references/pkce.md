# PKCE Login

## Choose Storage

Use any implementation of `IStorage<T>` from `@stackory/contracts`. Stackory provides storage helpers in `@stackory/storage`:

```ts
import { createLocalStorage, createSessionStorage } from '@stackory/storage/web';
```

For CLI or local Node apps:

```ts
import { createFileStorage } from '@stackory/storage/node';
```

## Browser Setup

```ts
import type { IPendingOAuthState, IStoredTokens } from '@stackory/auth-core';
import { createPkceSetup } from '@stackory/auth-better-auth/client/pkce';
import { createLocalStorage, createSessionStorage } from '@stackory/storage/web';

const { pkceFlow, tokenManager } = createPkceSetup({
  baseURL: 'https://api.example.com',
  basePath: '/api/auth',
  clientId: 'web-client',
  audience: 'https://api.example.com',
  scope: 'openid profile email offline_access',
  redirectURI: () => `${window.location.origin}/auth/callback`,
  tokenStorage: createLocalStorage<IStoredTokens>({ prefix: 'auth:' }),
  pendingStorage: createSessionStorage<IPendingOAuthState>({ prefix: 'auth:' }),
});
```

## Start Login

```ts
const url = await pkceFlow.prepareAuthorizationURL('/dashboard');
window.location.assign(url.toString());
```

## Callback Route

```ts
const params = new URLSearchParams(window.location.search);
const redirectTo = await pkceFlow.exchangeCode({
  code: params.get('code') ?? '',
  state: params.get('state'),
});

window.location.replace(redirectTo);
```

## Fetch With Token

```ts
const token = await tokenManager.getValidToken();
const res = await fetch('/api/profile', {
  headers: token ? { Authorization: `Bearer ${token}` } : {},
});
```

## Gotchas

- `scope` is required and must be a space-separated OAuth scope string.
- `audience` is sent to the token endpoint and must match the auth server expectation.
- `pendingStorage` should survive the redirect round trip but should not be long-lived; session storage is usually correct.
- `tokenManager.getValidToken()` refreshes when the access token is close to expiry.

