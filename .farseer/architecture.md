# Architecture

Home HQ is the family and home hub for Maple Frontier. All six family members will use it, from various devices. It builds on Home Assistant for the house and adds family features (calendar, chores, shopping, photos and more).

The infrastructure and stack are settled in `.farseer/adr/` (ADR-0001 to ADR-0013). There is no code yet; the directory layout below is what the first plan scaffolds. Product questions in `docs/brainstorming.md` section 18 that the ADRs don't cover are still open.

## Directory Structure

```text
apps/
  web/        Vite + React SPA (PWA, service worker, offline intent queue)
  server/     Hono on Node 24: serves the SPA, /api, /ws, HA relay, SQLite
packages/
  shared/     zod schemas, /api and /ws message types, domain types
docs/         Brainstorm, FamilyOS inventory, Reddit references
.farseer/     Agent config, ADRs, plans
```

Existing content:

- `docs/brainstorming.md` - Requirements from the photo-frame prototype, FamilyOS candidate features, and candidate features from Reddit dashboards, with requirement IDs (`TODAY-06`, `FOS-19`, `RED-40`).
- `docs/family-os-inventory.md` - Code-grounded inventory of FamilyOS.
- `docs/references/reddit/` - Screenshots from the Reddit threads, one folder per thread.

## Patterns Used

- **One container, API-first** (ADR-0002, 0003). The SPA talks only to the Home HQ server. Every feature goes through `/api` (zod + `@hono/zod-openapi`, OpenAPI published) or `/ws` (zod discriminated unions in `packages/shared`).
- **HA relay with a live mirror** (ADR-0004). One `subscribe_entities` connection held in RAM, rebuilt on every connect, never persisted. Clients get role-filtered states over `/ws`. When HA drops, clients show "HA offline" and disable house controls.
- **Identity from HA, sessions and roles in Home HQ** (ADR-0005). HA OAuth, mapped to a member; ~90-day sliding session (cookie, or bearer for native). Roles: admin, member, kid, display.
- **Kiosks act as the household** (ADR-0006). No PINs; risky actions need hold or slide gestures; chores credit their owner or ask who did an "Anyone" chore.
- **Layout from HA** (ADR-0011). Rooms from HA areas, tiles from entities with a Home HQ label, overrides in SQLite.
- **Offline** (ADR-0010). Cached app shell; shopping and chore intents queued with client IDs, last write wins, 24-hour expiry. HA commands are never queued.

## Conventions

- Requirement IDs from `docs/brainstorming.md` are stable. Refer to them in plans and issues; don't renumber them.
- Decisions that settle a brainstorm question get an ADR.
- Evergreen browsers only (Safari/iPadOS 17+, current Chrome), fluid layout from phone to 1920x1080 (ADR-0013).

## Key Integrations

- **Home Assistant** (Container install, 10.10.20.98, same Compose stack): WebSocket API via the non-admin "Home HQ" service user for state and commands; OAuth for sign-in; area and label registries for layout; `notify.mobile_app_*` for push (ADR-0012).
- **Network** (ADR-0007): Caddy serves `hq.<domain>` with a Let's Encrypt cert via Cloudflare DNS-01. Away devices reach the LAN address through a Tailscale subnet route. Kiosks run Tailscale too.
- **Storage** (ADR-0008): SQLite on a Docker volume, nightly and pre-migration `VACUUM INTO` snapshots.
- **Deploy** (ADR-0009): GitHub Actions builds `linux/amd64` and pushes `ghcr.io/petemcw/home-hq`; the iot Compose stack pulls it. Secrets in Docker secrets or a host `.env`.
- **Wall panel** (ADR-0013): 21.5" Android all-in-one running Fully Kiosk Browser.

## Business-Critical Paths

Areas where a change carries the highest test and review burden:

- **Anything that changes the real house**: locks, garage, alarm, climate, water valve, lights. A bug can unlock a door or turn off the heat. The server's per-role command allowlist is the guard (ADR-0004).
- **Auth and tokens**: HA OAuth, Home HQ sessions, the HA service token. Tokens must never reach source, the bundle, logs, git, or a publicly served file.
- **Family data and privacy**: presence, location, children's details, photos.
- **Offline replay**: queued intents must stay idempotent, and nothing that commands HA may ever enter the queue.

Before changing one of these, check `.farseer/adr/` for a decision covering that area.
