# ADR-0003: API-first PWA with a zod/OpenAPI contract

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Home HQ will run on phones, tablets and a 21.5" panel, and may become an iOS app later. We don't want to pick native technology before an app is real, but we also don't want a web-only backend that must be rebuilt for it.

## Decision

Ship an installable PWA. Every feature goes through `/api` and `/ws`. Routes are declared with `@hono/zod-openapi`, with schemas in `packages/shared`, so the web app gets typed clients and the server publishes an OpenAPI document. Auth accepts a bearer token as well as the session cookie, so a native client can use it.

## Alternatives Considered

### Hono RPC (`hc`) only

- **Pros**: end-to-end TypeScript types with no ceremony.
- **Cons**: no language-neutral spec.
- **Why not**: a Swift client would have nothing to generate from.

### tRPC

- **Pros**: excellent TypeScript developer experience.
- **Cons**: RPC-shaped and TypeScript-only.
- **Why not**: poor fit for a later native app.

### Build for Expo/React Native now, or plan around Capacitor

- **Pros**: one codebase for web and iOS.
- **Cons**: heavier setup now; kiosk and web ergonomics suffer (Expo), or the web app gets shaped around a shell (Capacitor).
- **Why not**: pays a cost now for an app that may never ship. An API-first PWA keeps all three paths open.

## Consequences

### Positive

- Request and response shapes are validated at runtime and typed at build time from one source.
- A native app can be added as `apps/ios` against the same contract.

### Negative

- Route declarations are more verbose than plain Hono handlers.

### Risks

- The WebSocket protocol sits outside OpenAPI. Mitigation: define `/ws` message types as zod discriminated unions in `packages/shared`.
