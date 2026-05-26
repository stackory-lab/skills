# Analytics

## Typed Analytics Client

```ts
import { createAnalyticsClient } from '@stackory/signals-core/analytics';

type Events = {
  signed_in: { method: 'google' | 'github' };
  project_created: { projectId: string };
  page_viewed: { path: string };
};

const analytics = createAnalyticsClient<Events>({
  context: {
    platform: 'web',
    appVersion: '1.0.0',
  },
  defaultProperties: {
    source: 'app',
  },
  onError: ({ providerName, method, error }) => {
    console.warn('analytics provider failed', providerName, method, error);
  },
});

analytics.track('signed_in', { method: 'google' });
analytics.identify({ userId: 'user_123', traits: { plan: 'pro' } });
analytics.screen({ name: '/dashboard' });
analytics.group({ groupId: 'org_123', traits: { tier: 'enterprise' } });
await analytics.flush();
```

## PostHog Web

```ts
import posthog from 'posthog-js';
import { createAnalyticsClient } from '@stackory/signals-core/analytics';
import { PosthogAnalyticsProvider } from '@stackory/signals-posthog/web';

posthog.init(import.meta.env.VITE_POSTHOG_KEY, {
  api_host: 'https://app.posthog.com',
});

const analytics = createAnalyticsClient({
  providers: [new PosthogAnalyticsProvider(posthog)],
});
```

## PostHog Node

```ts
import { PostHog } from 'posthog-node';
import { createAnalyticsClient } from '@stackory/signals-core/analytics';
import { PosthogNodeAnalyticsProvider } from '@stackory/signals-posthog/node';

const posthog = new PostHog(process.env.POSTHOG_KEY!, {
  host: 'https://app.posthog.com',
});

const analytics = createAnalyticsClient({
  providers: [new PosthogNodeAnalyticsProvider(posthog)],
});

analytics.track('job_completed', {
  jobId: 'job_123',
  durationMs: 1200,
});

await analytics.flush();
await posthog.shutdown();
```

## Gotchas

- The PostHog providers accept an existing PostHog client; the application owns SDK initialization and shutdown.
- `PosthogNodeAnalyticsProvider` requires a distinct id from context for `track`. Provide `userId`, `anonymousId`, or `sessionId`.
- Provider failures are isolated and reported through `onError`; they should not break app workflows.

