# ADR-0010: Cached app shell and an offline intent queue for app data only

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Phones lose signal in shops, Wi-Fi blips, and the server or HA can restart. FamilyOS can't cold-start a display while its server is down. The prototype only reloads itself (NFR-11).

## Decision

- A service worker caches the app shell, so every device, kiosks especially, boots without the server.
- The last-loaded app data shows with an offline banner. House controls are disabled.
- Shopping and chore actions made offline are queued as intents (add item, +1 quantity, check off, complete chore) with client-generated IDs, so replays are idempotent. On reconnect the server applies them in order, last write wins per field. Queued intents expire after 24 hours.
- HA commands are never queued.

## Alternatives Considered

### Cached shell and read-only offline

- **Pros**: no sync logic.
- **Why not**: phones in a store with weak signal need to check items off.

### Ask the user on conflict

- **Why not**: lists and chores rarely conflict in a way worth interrupting someone in a store.

### Queue HA commands too

- **Why not**: a stale "unlock" or "open garage" could fire hours later.

### No offline support

- **Why not**: a kiosk that can't boot during a server restart is a blank screen in the kitchen.

## Consequences

### Positive

- Kiosks always render; shopping works in a dead zone.

### Negative

- Every queued action needs an idempotent intent and a server-side apply rule.
- Last write wins can silently drop an offline edit to the same field.

### Risks

- Clock skew or long offline periods. Mitigation: the server orders by arrival, not client time, and the 24-hour expiry bounds the window.
