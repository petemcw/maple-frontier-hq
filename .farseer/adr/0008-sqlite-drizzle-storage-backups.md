# ADR-0008: SQLite with Drizzle for Home HQ's own data, with nightly snapshots

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Home HQ owns chores, stars, lists, members, roles, layout overrides and attribution records. About six people and a few kiosks use it, from one server. Settles brainstorm section 18 question 5 for where shared state lives (photo storage is still open).

## Decision

- SQLite in one file on a Docker volume, accessed through Drizzle ORM with drizzle-kit migrations, using `better-sqlite3`.
- The server writes a `VACUUM INTO` snapshot nightly and before every migration into a backups volume, keeping N days. The host's existing backup copies that folder off the box.

## Alternatives Considered

### SQLite with Kysely

- **Pros**: a lean query builder (FamilyOS's choice).
- **Cons**: migrations written by hand; no schema-as-code.
- **Why not**: Drizzle gives typed schema and generated migrations.

### Postgres container

- **Why not**: another service to run for a load SQLite handles easily.

### Litestream to S3/R2

- **Pros**: near-zero data loss.
- **Cons**: a sidecar and a bucket.
- **Why not**: losing at most a day of chore ticks is acceptable.

### Host-level backups only

- **Why not**: copying a live SQLite file risks a torn copy.

## Consequences

### Positive

- Nothing extra to run; backups are plain files.

### Negative

- A single writer process. The server cannot be scaled to several replicas.

### Risks

- Snapshots on the same host die with the host. Mitigation: confirm the host backup includes the backups volume.
