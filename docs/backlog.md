# Backlog

One flat list of Home HQ features, each sharp enough to hand to `/farseer:planner`. Area tickets in the brainstorm-to-backlog map append features under "Unordered"; the ordering ticket sorts them.

## Unordered

### Roles

Every signed-in family member or kiosk has one fixed role: admin, member or display. Both parents are admins and the four children are members. Admin-only actions are managing members and roles (including letting in an unmapped HA user), approving kiosks, and integrations: iCloud, Immich, the AI key and spending limit, and the weather location. Every other action is open to every member, and kiosks act as the household. Roles are checked in code, with no capability table.

- **Covers**: FOS-48 (narrowed), question 8 (approval part), question 20
- **Depends on**: sign-in with Home Assistant (ADR-0005)
- **ADRs**: ADR-0005, ADR-0006, ADR-0016

### Delete requests

Deleting a calendar event, household record, recipe, chore or task that someone else owns creates a delete request instead. The item stays until an owner (whoever added it, or the person it is for) or an admin approves. Unanswered requests expire after 7 days, or when the event ends. Home HQ records the creator and subject of every covered item. An event with no known creator belongs to the person whose calendar it's on, or to the household on the Family calendar. A kiosk deletes only household items. Shopping items aren't covered.

- **Covers**: FOS-48 (replaces Supervised drafts), question 8 (approval part)
- **Depends on**: Roles; the calendar, chores, tasks, kitchen and household records features it guards
- **ADRs**: ADR-0016

### Shopping lists

Named shopping lists shared by the family, starting with the permanent general list (Shopping) and Groceries. Members create, rename and delete the others. Open items group by category in one household order. Each item has a name, quantity, optional note, optional store and the adder's avatar badge (or a voice badge), and can be edited, moved to another list, or deleted with undo. A store filter shows one store's items with open counts. Ticked items go to the "Got it" drawer, where they can be added back. Anyone can clear it, and items drop out after 7 days. Taps work offline in the store through the intent queue.

- **Covers**: SHOP-01 to SHOP-04, SHOP-08, FOS-20, FOS-24, FOS-25, FOS-26, DATA-12 to DATA-14
- **Depends on**: Roles; the offline intent queue (ADR-0010)
- **ADRs**: ADR-0008, ADR-0010, ADR-0017

### Shopping add

One input adds one or several items: "milk, 2 kg rice, 1.5 l juice" splits on commas, semicolons and newlines and reads amounts and units. Suggestions come from items seen before. Adding something already listed merges into it (quantity up), a ticked-off item comes back, and undo restores the merge exactly. A new item takes its remembered category (falling back to a guess, then Other), its last store, and, when no list is chosen, its last shopping list or else the general list.

- **Covers**: SHOP-05, SHOP-07, FOS-21, FOS-22, FOS-23
- **Depends on**: Shopping lists; locale and units for parsing amounts
- **ADRs**: ADR-0017

### Usuals

Each shopping list offers its own usuals: the items most often bought lately from that list, learned from history rather than curated. With the add input empty, they show as one-tap tiles, and items already on the list show "On list ×n".

- **Covers**: SHOP-06, DATA-16
- **Depends on**: Shopping lists, Shopping add

### Voice inbox

HA's built-in shopping list (`todo.shopping_list`) is a one-way inbox. The server watches it, and each new item is parsed like a Shopping add, filed to its last shopping list (or the general list when new), merged, and removed from HA by uid once the Home HQ add commits. Inbox items carry a voice badge. Exposing the list to Assist is an HA config change to agree with the family first.

- **Covers**: FOS-20 (adding by voice), the shopping ticket's sync question
- **Depends on**: Shopping add; the HA connection (ADR-0004)
- **ADRs**: ADR-0004, ADR-0017
