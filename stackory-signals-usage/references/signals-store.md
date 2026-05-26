# User Signals Store

## Define Signals

```ts
import { UserSignalsStore, defineSignal } from '@stackory/signals-core';

export const signalRegistry = {
  onboarding_completed: defineSignal({ type: 'boolean', defaultValue: false }),
  session_count: defineSignal({ type: 'number', defaultValue: 0 }),
  last_seen_at: defineSignal({ type: 'timestamp' }),
  preferred_theme: defineSignal({ type: 'string', defaultValue: 'system' }),
} as const;
```

## Provide a Persister

```ts
import type {
  ISignalPersister,
  ISignalStorage,
} from '@stackory/signals-core';

const localStoragePersister: ISignalPersister<keyof typeof signalRegistry & string> = {
  persist(key, record) {
    localStorage.setItem(`signals:${key}`, JSON.stringify(record));
  },
  delete(key) {
    localStorage.removeItem(`signals:${key}`);
  },
  async load(): Promise<ISignalStorage<keyof typeof signalRegistry & string>> {
    const records: ISignalStorage<keyof typeof signalRegistry & string> = {};
    for (const key of Object.keys(localStorage)) {
      if (key.startsWith('signals:')) {
        const raw = localStorage.getItem(key);
        if (raw) {
          records[key.slice('signals:'.length)] = JSON.parse(raw);
        }
      }
    }
    return records;
  },
  async clear() {
    for (const key of Object.keys(localStorage)) {
      if (key.startsWith('signals:')) {
        localStorage.removeItem(key);
      }
    }
  },
};
```

## Create and Use Store

```ts
const store = new UserSignalsStore(signalRegistry, {
  persister: localStoragePersister,
});

await store.hydrate();
store.set('onboarding_completed', true);
store.set('session_count', (store.get('session_count') ?? 0) + 1);

const unsubscribe = store.subscribe(() => {
  console.log('signals changed');
});
```

## Plugins

Plugins receive synchronous events from store writes.

```ts
import type { ISignalPlugin } from '@stackory/signals-core';

const logPlugin: ISignalPlugin = {
  on(event) {
    console.log(event.type, event);
  },
};

const store = new UserSignalsStore(signalRegistry, {
  persister: localStoragePersister,
  plugins: [logPlugin],
});
```

## Gotchas

- `persister` is required.
- `persist`, `delete`, and plugin `on` handlers are synchronous by contract.
- Use `subscribe()` from UI adapters such as React `useSyncExternalStore`; do not mutate `signals` directly.
