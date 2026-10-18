# Backlog

One flat list of Home HQ features, each sharp enough to hand to `/farseer:planner`. Area tickets in the brainstorm-to-backlog map append features under "Unordered"; the ordering ticket sorts them.

## Unordered

### Roles

Every signed-in family member or kiosk has one fixed role: admin, member or display. Both parents are admins and the four children are members. Admin-only actions are managing members and roles (including letting in an unmapped HA user), approving kiosks, and integrations: iCloud, Immich, the AI key and spending limit, and the weather location. Adding and viewing papers, viewing account numbers, and editing duty presets are admin-only too (ADR-0018). Every other action is open to every member, and kiosks act as the household. Roles are checked in code, with no capability table.

- **Covers**: FOS-48 (narrowed), question 8 (approval part), question 20
- **Depends on**: sign-in with Home Assistant (ADR-0005)
- **ADRs**: ADR-0005, ADR-0006, ADR-0016, ADR-0018

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

### Records

The home (one fixed record, Maple Frontier), things, services, vehicles and pets. A thing has a room, brand, model, serial, bought-on date and warranty end. A service has a kind (electricity, internet, insurance, streaming and so on), a provider and an account number. Vehicles and pets have a name and notes. Records carry dates only, never money. Any member can add or edit records. Account numbers are shown only to admins. A record that leaves the household is archived, and its duties stop while its log and papers stay viewable. Deleting someone else's record is a delete request.

- **Covers**: FOS-37 (records), FOS-39
- **Depends on**: Roles, Delete requests
- **ADRs**: ADR-0016, ADR-0018

### Duties

Each record lists its open duties, soonest first, with due labels ("3 days overdue", "Tomorrow"). A duty has a title, a due date, an optional repeat every N weeks, months or years anchored to the first due date, a lead time (default 14 days), an assignee or Anyone, and notes. Done takes an optional note, adds a log entry, and moves the duty one period on (or past every missed period when late), with exact undo. A stale or repeated completion changes nothing. Reminders go out on the lead day and the due day through the Companion app, to the assignee or, for Anyone, to the admins. End dates (warranty ends, paper valid-until) remind the admins 30 days ahead and are never overdue. "Usually also" presets per kind prefill a duty, and admins edit the preset list. Each record keeps a dated log of what was done to it, and entries can also be added by hand.

- **Covers**: FOS-37 (duties, reminders, presets), FOS-39 (service log, treatment history as one log), RED-17
- **Depends on**: Records; notifications through the Companion app (ADR-0012)
- **ADRs**: ADR-0012, ADR-0018

### Sensor prompts

A duty can link one HA entity and the state that triggers it, such as the HVAC filter tray contact opening. On that state the duty either raises a sensor prompt ("Was the furnace filter changed?") or, if set to auto-complete, completes itself with a log entry "by HA sensor" and undo on the duty. A prompt goes to the assignee's (or the admins') Companion app as a notification with Yes and No, shows as an attention card, and appears on every kiosk. It stays open until it's answered or the duty is done, one answer clears it everywhere, and repeat triggers never add a second one. Yes completes the duty. Once the furnace filter and the fridge water and air filters run as duties here, HA's filter helpers, reset scripts and detection automation are retired, an HA config change to agree with the family at the time.

- **Covers**: RED-17 (consumables fed by HA sensors)
- **Depends on**: Duties; the HA mirror (ADR-0004); actionable Companion notifications (ADR-0012); what needs attention; kiosks (ADR-0006)
- **ADRs**: ADR-0004, ADR-0012, ADR-0018

### Papers

Files attached to a record with a label (policy, invoice, card, receipt, manual, other) and an optional valid-until date, which adds an end date to the record and shows an amber pill within 30 days and red once expired. Only admins add or open papers. They upload a file or a phone photo on the record page, or drop a file into the record's NAS folder. Papers live on a NAS share mounted into the container as a readable tree, `Records/<kind>/<record>/`. A scan turns hand-dropped files into unfiled papers on that record, follows renames and moves by content, and flags a paper whose file was deleted by hand as missing. While the NAS is unreachable, papers show as unavailable, never as missing. Kiosks never show papers.

- **Covers**: FOS-38
- **Depends on**: Records; a NAS share, mount and NAS account for Home HQ, with credentials kept out of the repo
- **ADRs**: ADR-0018
