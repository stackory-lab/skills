---
name: stackory-backend-usage
description: Use when helping an application developer integrate Stackory backend packages in server or edge applications: Hono middleware, internal HMAC auth, typed service-to-service calls, platform service abstractions, Cloudflare Workers KV/R2/Queue adapters, or Sentry Cloudflare monitoring.
---

# Stackory Backend Usage

## Purpose

Help consumers use Stackory backend packages correctly in application code. Focus on public imports, runtime wiring, peer packages, and common integration recipes.

## Package Map

- `@stackory/backend-platform`: runtime-agnostic server interfaces such as `IPlatformServices`, `ILogger`, `IKVStore`, `IObjectStorage`, `IServiceCaller`, and queue contracts.
- `@stackory/backend-utils`: response helpers, internal request signing, HMAC verification, and service-call interceptors.
- `@stackory/backend-hono`: Hono middleware and typed Hono RPC helpers. Requires consumer-installed `hono >=4.12.0 <5`.
- `@stackory/backend-platform-cloudflare`: Cloudflare Workers implementations for platform interfaces. Requires consumer-installed `@sentry/cloudflare >=10.53.0 <11` only when using Sentry monitoring.

## Start Here

1. For Hono app middleware, read `references/hono.md`.
2. For Cloudflare Workers platform bindings, read `references/cloudflare.md`.
3. For internal service calls and request signing, read `references/service-calls.md`.

## Usage Rules

- Register `errorHandlerMiddleware` only with top-level `app.onError(...)`.
- Register request logging middleware before route handlers that read the child logger.
- Use `userIdMiddleware` only behind trusted/authenticated upstreams; it trusts the `X-User-Id` header.
- Pair `createInternalAuthMiddleware` on the receiving service with `createHmacSigningInterceptor` on the calling service.
- Keep Cloudflare-specific imports out of Node.js services and browser bundles.
- Treat `IPlatformServices` as the injected application boundary; pass concrete implementations at app startup.

