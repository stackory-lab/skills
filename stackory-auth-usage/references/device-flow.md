# Device Flow

Use device flow for CLIs, TV-style login, TUI apps, or devices that cannot complete a normal browser redirect.

## Client Login

```ts
import type { IStoredTokens } from '@stackory/auth-core';
import type { IPendingOAuthState } from '@stackory/auth-core';
import { createPkceSetup } from '@stackory/auth-better-auth/client/pkce';
import {
  createDeviceFlowProvider,
  createDeviceLogin,
} from '@stackory/auth-better-auth/client/device';
import { deviceFlowClient } from '@stackory/auth-better-auth/client-plugins';
import { createFileStorage } from '@stackory/storage/node';
import { createAuthClient } from 'better-auth/client';
import { homedir } from 'node:os';
import path from 'node:path';

const configDir = path.join(homedir(), '.my-app');

const { tokenManager } = createPkceSetup({
  baseURL: 'https://api.example.com',
  basePath: '/api/auth',
  clientId: 'cli-client',
  audience: 'https://api.example.com',
  scope: 'openid profile email offline_access',
  redirectURI: 'http://localhost/callback',
  tokenStorage: createFileStorage<IStoredTokens>(path.join(configDir, 'tokens.json')),
  pendingStorage: createFileStorage<IPendingOAuthState>(path.join(configDir, 'pending.json')),
});

const authClient = createAuthClient({
  baseURL: 'https://api.example.com',
  basePath: '/api/auth',
  plugins: [deviceFlowClient()],
});

const provider = createDeviceFlowProvider(authClient);
const login = createDeviceLogin({
  provider,
  tokenManager,
  clientId: 'cli-client',
  scope: 'openid profile email offline_access',
  resource: 'https://api.example.com',
});

await login({
  onUserPrompt: ({ userCode, verificationUri }) => {
    console.log(`Open ${verificationUri} and enter ${userCode}`);
  },
  onVerificationUri: async (url) => {
    console.log(url);
  },
  signal: abortController.signal,
});
```

## Verification Page Actions

Use these actions in the web page where a user enters the device code.

```ts
import { createDeviceActions } from '@stackory/auth-better-auth/client/device';

const actions = createDeviceActions({
  baseURL: 'https://api.example.com',
  basePath: '/api/auth',
});

const info = await actions.getDeviceInfo({ userCode });
await actions.approveDevice({ userCode });
await actions.denyDevice({ userCode });
```

## Session and OAuth Consent Actions

```ts
import { createSessionActions } from '@stackory/auth-better-auth/client/session';

const session = createSessionActions({ baseURL, basePath });
await session.getSession();
await session.signOut();
await session.signInSocial({ provider: 'google', callbackURL: '/auth/callback' });
await session.getOAuthClientPublic(clientId);
await session.submitOAuthConsent({ accept: true, scope: 'openid profile' });
```

## Gotchas

- The device flow client needs the `deviceFlowClient()` Better Auth client plugin.
- Use the same `clientId`, `scope`, and `resource` expected by the server plugin.
- Pass an `AbortSignal` when login should be cancellable.
