# Utilities

## Crypto

```ts
import { cryptoUtils } from '@stackory/utils';

const digest = await cryptoUtils.sha256Hex('payload');
const signature = await cryptoUtils.hmac('secret', ['part-a', 'part-b']);

const token = await cryptoUtils.signHmacToken({ userId: 'u1' }, 'secret', 60);
const payload = await cryptoUtils.verifyHmacToken<{ userId: string }>(
  token,
  'secret',
);
```

Available helpers:

- `hex(data)`
- `sha256Hex(data)`
- `hmac(secret, parts, separator?)`
- `hmacRaw(secret, parts, separator?)`
- `timingSafeEqual(a, b)`
- `signHmacToken(payload, secret, ttlSeconds)`
- `verifyHmacToken<T>(token, secret)`

## Gzip

```ts
import { gzip } from '@stackory/utils';

const compressed = await gzip.gzipText(JSON.stringify(payload));
const text = await gzip.gunzipText(compressed.buffer);
```

## Gotchas

- Crypto helpers use Web Crypto (`crypto.subtle`); ensure the target runtime provides it.
- Gzip helpers use `CompressionStream` and `DecompressionStream`.
- `verifyHmacToken` throws for invalid format, invalid signature, and expired tokens.

