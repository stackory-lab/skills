# Storage

## Contract

`@stackory/contracts` defines the shared synchronous storage shape:

```ts
import type { IStorage } from '@stackory/contracts';

const storage: IStorage<TokenRecord> = {
  get: (key) => null,
  set: (key, value) => {},
  remove: (key) => {},
  clear: () => {},
  getAllKeys: () => [],
};
```

## Memory Storage

Use for tests, temporary state, and process-local caches.

```ts
import { createMemoryStorage } from '@stackory/storage/memory';

const storage = createMemoryStorage<{ accessToken: string }>();
storage.set('tokens', { accessToken: 'abc' });
```

## Web Storage

Use in browser apps with `localStorage` or `sessionStorage`.

```ts
import { createLocalStorage, createSessionStorage } from '@stackory/storage/web';

const tokenStorage = createLocalStorage<AuthTokens>({ prefix: 'auth:' });
const pendingStorage = createSessionStorage<PendingState>({ prefix: 'auth:' });
```

`prefix` isolates keys and makes `clear()` remove only matching keys.

## Node File Storage

Use in CLI, TUI, or local Node apps.

```ts
import { createFileStorage } from '@stackory/storage/node';

const tokenStorage = createFileStorage<AuthTokens>(
  `${process.env.HOME}/.my-app/tokens.json`,
);
```

## Gotchas

- `IStorage<T>` is synchronous. Do not pass async storage APIs directly without an adapter.
- Web storage adapters use `globalThis.localStorage` and `globalThis.sessionStorage`; only use them in browser-like runtimes.
- File storage writes a JSON object keyed by storage key.

