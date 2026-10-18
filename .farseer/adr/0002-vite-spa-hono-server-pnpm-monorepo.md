# ADR-0002: Vite React SPA and a Hono server on Node 24, in a pnpm monorepo

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Home HQ needs a backend for state HA doesn't hold (chores, stars, lists) and to relay HA (ADR-0004). Its screens are long-lived and realtime: a wall panel that loads once and lives on WebSocket updates. It is private behind our network, so SEO and first-load SSR don't matter. A native iOS app may follow, which wants a plain API. Next.js and Vite were both candidates.

## Decision

- `apps/web`: React SPA built with Vite.
- `apps/server`: Hono on Node 24 LTS. It serves the built SPA, `/api` and one `/ws` WebSocket per client (`@hono/node-ws`). One container.
- `packages/shared`: zod schemas and domain types used by both (see ADR-0003).
- pnpm workspaces. `better-sqlite3` for SQLite (ADR-0008).

## Alternatives Considered

### Next.js (App Router)

- **Pros**: mature conventions, built-in routing and data loading, strong auth ecosystem.
- **Cons**: route handlers can't hold WebSockets, so a custom server is needed; server actions and RSC are web-only.
- **Why not**: its strengths (SSR, SEO, server components) don't help a private realtime dashboard, and its weak spots are our core needs.

### TanStack Start

- **Pros**: Vite-based, type-safe server functions.
- **Cons**: youngest option with the most API churn; still needs separate WebSocket handling.
- **Why not**: churn risk for no gain over a plain SPA + API.

### Bun runtime

- **Pros**: built-in SQLite, WebSockets and TypeScript; simpler image.
- **Cons**: less proven for months-long uptime; Vitest, Playwright and drizzle-kit are Node-first.
- **Why not**: on a box in the house, staying up matters more than convenience.

### npm workspaces or a single package

- **Why not**: pnpm's strict dependency isolation keeps `apps/web` from leaning on server-only packages. A single package blurs the client/server boundary.

## Consequences

### Positive

- Same Vite, Vitest, RTL and oxlint toolchain as `ha-dashboard`.
- The API is the product boundary from day one, ready for a native client.

### Negative

- We wire routing, data fetching and the client/server contract ourselves.
- Differs from `ha-dashboard`'s npm, so commands differ between the sibling projects.

### Risks

- Node's type stripping only supports erasable TypeScript syntax. Mitigation: the code standards already ban `enum`; keep namespaces and parameter properties out too.
