# ADR-0004: The server relays Home Assistant through a non-admin service user and a live in-memory mirror

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Browsers need live device state and must send commands. HA's own permissions are only admin or non-admin, so it can't stop a kid's account from unlocking a door. Long-lived HA tokens must never reach a device's bundle or storage. The original rule "Home HQ never keeps its own copy of device state" exists to prevent two sources of truth, and a cache only causes that if it outlives its HA connection or is written to disk. Settles brainstorm section 18 questions 18 and 20 for HA access.

## Decision

- A dedicated non-admin HA user, "Home HQ", owns one long-lived token, held only by the server (Docker secret). Browsers never connect to HA for device state or commands.
- The server keeps one `subscribe_entities` connection and holds current states in RAM. The mirror is rebuilt from scratch on every (re)connect and is never persisted.
- Each client gets the states and commands its role allows over its `/ws` connection. Home HQ enforces per-member rules.
- When the HA connection drops, the server tells every client and they show "HA offline" and disable house controls, never old values as current.
- Rule: _HA is the source of truth for device state. Home HQ may hold a live in-memory mirror rebuilt from HA on every connect, never persists device state, and never shows device state as current while its HA connection is down._
- App data about the house (who acted from a kiosk, who acknowledged a leak, which entities a chore watches) is Home HQ's own and lives in SQLite.

## Alternatives Considered

### Each device signs in to HA and talks to it directly

- **Pros**: no relay code; HA logbook names each person.
- **Cons**: HA credentials on every device; no per-entity limits.
- **Why not**: can't keep kids or kiosks away from locks.

### Pure pass-through, one HA subscription per client

- **Pros**: the most literal reading of the old rule.
- **Cons**: about ten HA connections; filtering, derived summaries and alert triggers computed per client.
- **Why not**: more load and duplicated logic for no gain in correctness.

### Mirror plus a persisted snapshot

- **Pros**: instant cold start.
- **Cons**: shows stale device state, which is exactly the drift the rule prevents.
- **Why not**: a wrong "locked" is worse than a short "connecting".

### Admin service user, or a personal admin token

- **Why not**: larger blast radius if leaked; a personal token mislabels actions and is painful to revoke.

## Consequences

### Positive

- One place enforces who may do what, and one place spots alert triggers (leak, doorbell) for every kiosk at once.
- HA stays private to the server.

### Negative

- HA's logbook attributes every Home HQ action to "Home HQ". Per-person attribution lives in Home HQ's own records.
- The server is on the path for every tap, so its uptime is the house UI's uptime.

### Risks

- A non-admin user can still call any service. Mitigation: the server's command allowlist per role, covered by tests against the fake HA.
