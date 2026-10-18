# ADR-0011: House layout comes from HA areas and labels, with Home HQ overrides

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Home control needs rooms, tiles, scenes and cameras (section 10). HA already models areas and lets entities carry labels. `ha-dashboard` uses a mounted `home.json`, so every change means editing a file on the host.

## Decision

Rooms come from HA areas. Tiles come from entities that carry a Home HQ label in HA. Ordering, display names and hidden items are overrides stored in SQLite and edited by admins in the app.

## Alternatives Considered

### Config file like `home.json`

- **Pros**: explicit, versionable.
- **Cons**: changes need host access.
- **Why not**: admins should adjust the layout from the app.

### Layout entirely in Home HQ's database

- **Why not**: duplicates the room structure HA already has, and the two drift.

## Consequences

### Positive

- Adding a device to the dashboard is an HA label, not a deploy.

### Negative

- Reads area and label registries from HA, which the relay must subscribe to as well as states.

### Risks

- Overrides pointing at entities that were renamed or removed in HA. Mitigation: ignore and flag orphaned overrides rather than failing.
