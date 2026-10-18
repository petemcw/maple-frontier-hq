# ADR-0006: Kiosks are interactive and act as the household, without PINs

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

FamilyOS makes wall displays read-only. The prototype's wall screen is the main control surface: it ticks chores, edits the grocery list, unlocks doors and opens the garage. Our kiosks don't select a person. Settles brainstorm section 18 questions 4 and 11.

## Decision

- Kiosks (display role) can do everything the household does: chores, lists, lights, scenes, locks, garage.
- Kiosk actions are recorded as "household". There are no PINs.
- Risky actions (unlock, garage, alarm, water valve) need a deliberate gesture: hold to unlock (HOME-08), slide to open (HOME-09).
- A chore assigned to a person credits that person. An "Anyone" chore asks who did it (CHORE-06). This is attribution, not sign-in.

## Alternatives Considered

### Keep a member PIN for risky actions

- **Pros**: stops a guest or child at the wall from unlocking doors; records who unlocked.
- **Cons**: friction on the most-used screen; a PIN system to build.
- **Why not**: the household accepts the wall panel as trusted, like a key on a hook.

### Read-only kiosks (FamilyOS rule)

- **Why not**: loses the kitchen-screen workflow the prototype is built around.

### No special gestures

- **Why not**: an accidental tap could open the garage.

## Consequences

### Positive

- The wall panel works for anyone in the kitchen with no sign-in step.

### Negative

- Anyone inside the house, guests included, can unlock doors from a kiosk.
- HA's logbook and Home HQ both record kiosk unlocks as household, not a person.

### Risks

- A kiosk visible from outside or reachable by visitors. Mitigation: revisit with a PIN or a role restriction if that changes; the gesture components are the natural place to add one.
