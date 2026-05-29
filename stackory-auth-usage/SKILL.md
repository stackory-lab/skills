---
name: stackory-auth-usage
description: >
  Use when helping an application developer integrate Stackory auth packages: OAuth 2.0 PKCE login, token storage and refresh, RFC 8628 device flow, Better Auth client actions, Better Auth server plugins, OAuth provider consent, or auth URL configuration.
---

# Stackory Auth Usage

## Purpose

Help consumers wire Stackory auth packages into applications without importing the wrong runtime subpath.

## Package Map

- `@stackory/auth-core`: runtime-neutral OAuth orchestration. Use for PKCE state, token manager, device polling ports, token mapping, and structured `AuthError`.
- `@stackory/auth-better-auth`: Better Auth adapter package. Use subpath exports; do not import the package root.
- `@stackory/contracts`: provides the `IStorage<T>` shape used for token and pending-state storage.

## Install

For core-only integrations:

```sh
pnpm add @stackory/auth-core @stackory/contracts
```

For Better Auth integrations:

```sh
pnpm add @stackory/auth-core @stackory/auth-better-auth @stackory/contracts better-auth @better-auth/oauth-provider
```

`better-auth` and `@better-auth/oauth-provider` are peer packages owned by the consuming app.

## Start Here

1. For browser PKCE login and token refresh, read `references/pkce.md`.
2. For device-code login or device approval UI, read `references/device-flow.md`.
3. For Better Auth server plugin setup, read `references/better-auth-server.md`.

## Runtime Rules

- Browser/TUI code may import `@stackory/auth-better-auth/client`, `/client/pkce`, `/client/device`, `/client/session`, `/client-plugins`, and `/config`.
- Server Better Auth setup may import `@stackory/auth-better-auth/server-plugins`.
- Do not import `better-auth/api` or server plugin subpaths in browser bundles.
- Do not store tokens in module variables; pass an `IStorage<IStoredTokens>` implementation.
- PKCE redirect handling is application-owned: prepare URL before redirect, then exchange code in the callback route.

