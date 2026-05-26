# Better Auth Server Plugin

## Install

```sh
pnpm add @stackory/auth-better-auth better-auth @better-auth/oauth-provider
```

## Plugin Setup

```ts
import { betterAuth } from 'better-auth';
import { oauthProvider } from '@better-auth/oauth-provider';
import { deviceFlowPlugin } from '@stackory/auth-better-auth/server-plugins';

export const auth = betterAuth({
  plugins: [
    oauthProvider({
      // Configure Better Auth OAuth provider options here.
    }),
    deviceFlowPlugin({
      verificationUri: 'https://app.example.com/auth/device',
      codeTtlSec: 600,
      defaultIntervalSec: 5,
      allowedClientIds: ['cli-client'],
      defaultResources: ['https://api.example.com'],
      validResources: ['https://api.example.com'],
    }),
  ],
});
```

## Custom Token Issuing

If the default Better Auth adapter path is not enough, provide hooks:

```ts
deviceFlowPlugin({
  verificationUri: 'https://app.example.com/auth/device',
  issueAuthorizationCode: async (params) => {
    return await createCodeForUser(params.userId, {
      clientId: params.clientId,
      scopes: params.scopes,
      resource: params.resource,
      codeChallenge: params.codeChallenge,
    });
  },
  issueAccessToken: async (params) => {
    return await exchangeAuthorizationCode({
      clientId: params.clientId,
      authorizationCode: params.authorizationCode,
      codeVerifier: params.codeVerifier,
      resource: params.resource,
    });
  },
});
```

## Exports

Use only the server subpath for Better Auth server setup:

```ts
import {
  DEVICE_AUTHORIZATION_MODEL,
  deviceAuthorizationSchema,
  deviceFlowPlugin,
} from '@stackory/auth-better-auth/server-plugins';
```

## Gotchas

- `verificationUri` must be an absolute `http` or `https` URL.
- The plugin adds device authorization endpoints under the Better Auth API path.
- Keep browser code on `/client/*` subpaths; do not import `/server-plugins` into browser bundles.

