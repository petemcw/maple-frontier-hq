# Architecture Decision Records

| ADR | Title | Status | Date |
| --- | --- | --- | --- |
| [0001](0001-own-app-not-familyos.md) | Build Home HQ as our own app, borrowing FamilyOS ideas | accepted | 2026-10-18 |
| [0002](0002-vite-spa-hono-server-pnpm-monorepo.md) | Vite React SPA and a Hono server on Node 24, in a pnpm monorepo | accepted | 2026-10-18 |
| [0003](0003-api-first-pwa-openapi-contract.md) | API-first PWA with a zod/OpenAPI contract | accepted | 2026-10-18 |
| [0004](0004-server-relays-ha-live-mirror.md) | The server relays Home Assistant through a non-admin service user and a live in-memory mirror | accepted | 2026-10-18 |
| [0005](0005-ha-oauth-identity-home-hq-sessions.md) | Sign in with Home Assistant; Home HQ owns sessions and roles | accepted | 2026-10-18 |
| [0006](0006-kiosks-act-as-household.md) | Kiosks are interactive and act as the household, without PINs | accepted | 2026-10-18 |
| [0007](0007-lan-domain-caddy-tailscale-away.md) | A LAN domain behind Caddy, with Tailscale for away access | superseded by [0014](0014-tailscale-serve-hostname.md) | 2026-10-18 |
| [0008](0008-sqlite-drizzle-storage-backups.md) | SQLite with Drizzle for Home HQ's own data, with nightly snapshots | accepted | 2026-10-18 |
| [0009](0009-github-actions-ghcr-deploy.md) | Build images in GitHub Actions and deploy from GHCR | accepted | 2026-10-18 |
| [0010](0010-offline-shell-and-intent-queue.md) | Cached app shell and an offline intent queue for app data only | accepted | 2026-10-18 |
| [0011](0011-layout-from-ha-areas-labels.md) | House layout comes from HA areas and labels, with Home HQ overrides | accepted | 2026-10-18 |
| [0012](0012-notifications-via-ha-companion.md) | Push notifications go through the HA Companion app | accepted | 2026-10-18 |
| [0013](0013-evergreen-browsers-fluid-layout.md) | Evergreen browsers and a fluid layout; the wall panel is Android with Fully Kiosk | accepted | 2026-10-18 |
| [0014](0014-tailscale-serve-hostname.md) | Serve Home HQ at maplefrontier.alpine-ling.ts.net through a Tailscale sidecar | accepted | 2026-10-18 |
| [0015](0015-feature-scope.md) | Feature scope for Home HQ | accepted | 2026-10-18 |
| [0016](0016-fixed-roles-delete-requests.md) | Three fixed roles, and delete requests for others' items | accepted | 2026-10-18 |
| [0017](0017-ha-shopping-list-voice-inbox.md) | HA's shopping list is a one-way voice inbox for Home HQ's shopping lists | accepted | 2026-10-18 |
| [0018](0018-household-records-duties-papers.md) | Household records keep duties in Home HQ, linked to HA sensors, with papers on the NAS | accepted | 2026-10-18 |
