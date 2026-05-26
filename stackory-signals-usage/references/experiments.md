# Experiments

## Define Experiment Signals

```ts
import {
  assignAllExperiments,
  defineExperimentSignal,
} from '@stackory/signals-core/experiment';
import { UserSignalsStore } from '@stackory/signals-core';

const registry = {
  home_redesign: defineExperimentSignal(
    { control: 50, treatment: 50 },
    { version: 1, description: 'Home page redesign' },
  ),
} as const;
```

## Assign Variants

```ts
const store = new UserSignalsStore(registry, { persister });

assignAllExperiments({
  store,
  registry,
  seed: userId ?? anonymousId,
});

const variant = store.get('home_redesign');
```

Assignments are sticky for the same experiment version. Bump `version` to rebucket.

## Track Exposure

Call exposure tracking when the user actually sees the variant.

```ts
store.trackExposure('home_redesign');
```

## Send Signal Events to Analytics

Use `@stackory/signals-core/analytics-signals` when signal events should become analytics events.

```ts
import { createSignalsAnalyticsPlugin } from '@stackory/signals-core/analytics-signals';

const store = new UserSignalsStore(registry, {
  persister,
  plugins: [
    createSignalsAnalyticsPlugin({
      analytics,
      mapEvent: (event) => {
        if (event.type === 'signal:set') {
          return {
            name: 'signal_set',
            properties: {
              key: event.key,
              source: event.source,
            },
          };
        }
        return null; // default mapping still handles experiment events
      },
    }),
  ],
});
```

## Gotchas

- Use stable seeds. Do not seed with random values unless you want reassignment.
- Weights are interpreted by the resolver as relative bucket weights.
- Call `trackExposure` at render/exposure time, not assignment time.
