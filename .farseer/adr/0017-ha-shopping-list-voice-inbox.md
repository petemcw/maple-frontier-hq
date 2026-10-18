# ADR-0017: HA's shopping list is a one-way voice inbox for Home HQ's shopping lists

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

The family has no shared shopping system yet. HA's built-in `todo.shopping_list` exists but is empty. Assist and Siri (through the Companion app's Assist shortcut) can add to any HA to-do list, but an HA to-do item holds only a title, a done flag, a due date and a note. Home HQ's shopping lists need quantity, category, store and who added an item, and there are several lists while HA's shopping list is a single one. Creating more HA lists needs an HA admin, which the Home HQ service user isn't. Apple Reminders has no server API: upgraded lists aren't served over iCloud CalDAV, and EventKit runs only on Apple devices.

## Decision

- Home HQ's own database (ADR-0008) is the only home for shopping lists and their items.
- `todo.shopping_list` is a one-way voice inbox. The server watches it (`todo/item/subscribe`). Each new item is parsed like a one-line add, goes to the shopping list it was last on, or to the permanent Shopping list when it's new, merges like any add, and is then removed from HA by uid.
- Inbox items carry a voice badge, since HA doesn't record which user added them.
- Nothing syncs back to HA, and Apple Reminders is ruled out.

## Alternatives Considered

### Two-way mirror of one list

- **Pros**: the HA app and its To-do panel show the list.
- **Cons**: HA flattens quantity, category and store into text, and edits from both sides must be merged.
- **Why not**: the family uses Home HQ to read the list. HA only adds value as a way in.

### Two-way mirror of every list

- **Why not**: all of the above, plus an HA admin must create a Local To-do list by hand for each Home HQ list.

### HA holds the items

- **Pros**: one store of items.
- **Cons**: the extras still live in Home HQ, and the list is unavailable whenever HA is down, including in the store.
- **Why not**: conflicts with offline use in the store (ADR-0010).

### Apple Reminders sync

- **Why not**: no supported server path. Bridges need a Mac running a daemon or polling Shortcuts.

## Consequences

### Positive

- "Hey Siri, add milk to the shopping list" works with no extra app and no sync conflicts.
- The full model (quantity, category, store, avatar) stays in one place.

### Negative

- HA's list is always empty, so the HA app never shows the family's list.
- `todo.shopping_list` must be exposed to Assist, an HA config change to make with the family's agreement.

### Risks

- Removing an item from HA after copying it is a write through the service user. Mitigation: remove by uid only, never by name (HA matches names too), and only after the Home HQ add commits.
- HA down: inbox items wait in HA and arrive on reconnect.
