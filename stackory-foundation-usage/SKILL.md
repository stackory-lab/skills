---
name: stackory-foundation-usage
description: Use when helping an application developer use Stackory foundation packages: shared storage contracts and adapters, standard errors and error codes, API response shapes, API client helpers, crypto utilities, gzip helpers, and common type exports.
---

# Stackory Foundation Usage

## Purpose

Help consumers use the small shared packages that other Stackory packages build on.

## Package Map

- `@stackory/contracts`: shared TypeScript contracts, currently including `IStorage<T>`.
- `@stackory/storage`: concrete `IStorage<T>` adapters for memory, web storage, and Node file storage.
- `@stackory/constants`: standard error codes plus `StandardError` and `NetworkError`.
- `@stackory/utils`: Web API based crypto helpers, gzip helpers, and a small fetch API client.

## Start Here

1. For token/session persistence or generic app storage, read `references/storage.md`.
2. For standard errors and API client usage, read `references/errors-and-api.md`.
3. For crypto and gzip helpers, read `references/utilities.md`.

## Usage Rules

- Use `@stackory/contracts` for shared interfaces and `@stackory/storage/*` for implementations.
- Choose storage subpaths by runtime: `/web`, `/node`, or `/memory`.
- `@stackory/utils` crypto and gzip helpers rely on Web APIs such as `crypto.subtle`, `CompressionStream`, and `DecompressionStream`.
- Throw `StandardError` when code consumers need stable machine-readable error codes.

