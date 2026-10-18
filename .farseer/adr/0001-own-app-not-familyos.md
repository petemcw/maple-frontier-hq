# ADR-0001: Build Home HQ as our own app, borrowing FamilyOS ideas

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

FamilyOS (brainstorm section 16) already does much of what the prototype mocks: calendar, chores, lists, roles, live sync. It is AGPL-3.0, beta with no compatibility promise, written in Svelte, treats wall displays as read-only, and leaves smart-home control out of scope. Home HQ's centre is Home Assistant control from an interactive wall panel, and the sibling `ha-dashboard` is React. Settles brainstorm section 18 question 2.

## Decision

We write Home HQ as our own TypeScript/React app. FamilyOS is a source of ideas (`FOS-xx` requirements), not a dependency, fork or neighbouring service.

## Alternatives Considered

### Run FamilyOS alongside Home HQ

- **Pros**: family-organiser features exist today.
- **Cons**: two apps, two logins, two data stores; no smart-home control; read-only displays.
- **Why not**: splits the family's day across two products.

### Fork FamilyOS

- **Pros**: starts with a working Hono/SQLite backend and PWA.
- **Cons**: Svelte rather than React, AGPL obligations, tracking a beta upstream, display model to unpick.
- **Why not**: we would rewrite most of the UI and fight its display rules.

## Consequences

### Positive

- One app, one sign-in, one data model, designed around HA and the wall panel.
- React patterns and test setup carry over from `ha-dashboard`.

### Negative

- We build calendar, chores, lists and roles ourselves.

### Risks

- Scope creep from copying too much of FamilyOS. Mitigation: each borrowed feature enters through a plan that cites its `FOS-xx` ID.
