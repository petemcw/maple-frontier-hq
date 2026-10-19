# Backlog

One flat list of Home HQ features, each sharp enough to hand to `/farseer:planner`. Area tickets in the brainstorm-to-backlog map append features under "Unordered"; the ordering ticket sorts them.

## Unordered

### Roles

Every signed-in family member or kiosk has one fixed role: admin, member or display. Both parents are admins and the four children are members. Admin-only actions are managing members and roles (including letting in an unmapped HA user), approving kiosks, and integrations: iCloud, Immich, the AI key and spending limit, and the weather location. Adding and viewing papers, viewing account numbers, and editing duty presets are admin-only too (ADR-0018), as are setting the chore reminder hours and putting someone else on a chore break (ADR-0022). Every other action is open to every member, and kiosks act as the household. Roles are checked in code, with no capability table.

- **Covers**: FOS-48 (narrowed), question 8 (approval part), question 20
- **Depends on**: sign-in with Home Assistant (ADR-0005)
- **ADRs**: ADR-0005, ADR-0006, ADR-0016, ADR-0018, ADR-0022

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
- **Depends on**: Shopping lists; Region format
- **ADRs**: ADR-0017, ADR-0020

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

### Region format

One region format for the whole household, read from HA's config (unit system, time zone, country, language) on every connect. `Intl` for that locale supplies the clock, week start and date styles: en-US today, so 12-hour, weeks starting Sunday, and "Sat, Oct 18" (numeric "10/18/2026" only where a numeric date can't be avoided). The server keeps the last one it read in SQLite and gives it to every client, falling back to en-US, America/Chicago and °F before the first connect. Every device shows times in the household time zone, even a phone in another zone. Values keep the unit HA reports and are never converted, and outside sources such as Open-Meteo are asked for the household's units. All dates, times, numbers and units are formatted through one shared formatter, never by hand. English only, American spelling, no translation layer.

- **Covers**: NFR-05 (replaced: the region format follows HA), FOS-53 (ruled out)
- **Depends on**: the HA connection (ADR-0004); SQLite storage (ADR-0008)
- **ADRs**: ADR-0020

### Recipe box

One family recipe box. A recipe has a title, ingredient lines (headings such as "For the sauce:" allowed), steps, servings, total time, source link, labels and one photo. Search matches titles and ingredients, and labels filter as chips. Cook notes are a dated log on the recipe showing who wrote each one. Anyone can add one, and only the writer or an admin deletes it. Whoever adds a recipe owns it, so deleting someone else's is a delete request. Kiosks can view recipes. No rating, no made-it log, no meal plan.

- **Covers**: FOS-33 (no AI capture here)
- **Depends on**: Roles; Delete requests
- **ADRs**: ADR-0015, ADR-0016, ADR-0021

### Recipe from a link

Paste a recipe page's URL and the server reads its schema.org Recipe data (JSON-LD, including `@graph`, HowToSection and ISO 8601 times) into the recipe form to check and save. A link shared to the Telegram bot does the same as a recipe draft that only the sender sees, and the bot replies with a link to it. When a page can't be read, the form opens with the URL as the source.

- **Covers**: FOS-33 (source link), FOS-41 (the link part, no AI)
- **Depends on**: Recipe box; the Telegram bot (ADR-0019) and linking each Telegram user to a family member
- **ADRs**: ADR-0019, ADR-0021

### Recipe scaling and units

Each ingredient line is parsed when saved into amount, unit and the rest: whole numbers, decimals, fractions, unicode fractions, mixed numbers and ranges, in US and metric units. The edit form shows how each line was read. A servings stepper (or ½× and 2×) scales amounts, shown as kitchen fractions, and tidies units within their system (6 tsp → 2 tbsp, 16 oz → 1 lb). A US/metric toggle converts amounts like for like, volume to volume and weight to weight, never volume to weight. Step text is never converted. Each device remembers its choice for all recipes. Unparsed lines never change.

- **Covers**: none in the brainstorm (FamilyOS lists scaling as a gap)
- **Depends on**: Recipe box; Region format
- **ADRs**: ADR-0020, ADR-0021

### Ingredients to shopping

Tick a recipe's ingredients ("Select all" or one by one) and add them to a chosen shopping list exactly as shown, scaled and in the units on screen. Only the recipe's own lines are accepted. An item already open on the list is never changed, and the result reports it with both amounts ("Chicken is already on the list (500 g; recipe needs 600 g)"). A ticked-off item in Got it comes back. New items take their remembered category.

- **Covers**: FOS-35
- **Depends on**: Recipe scaling and units; Shopping lists; Shopping add
- **ADRs**: ADR-0017, ADR-0021

### Cook mode

The recipe in large type for cooking from a phone or tablet: ingredients alongside the steps (side by side on a tablet), each tickable, and the screen kept awake with the Screen Wake Lock API while open. Scaling and the US/metric toggle apply. A time in a step ("simmer 20 minutes") is tappable and starts a kitchen timer.

- **Covers**: none (the brainstorm notes FamilyOS has no cook mode, under "Kitchen (new area)")
- **Depends on**: Recipe scaling and units; kitchen timers
- **ADRs**: ADR-0021

### Chores

A chore has a title, an emoji, notes, a repeat (once, daily, or weekly on chosen days, with an optional end date), an optional due time, and who's up: a fixed person or Anyone. Any member adds, edits and archives chores. The owners are whoever added it and, for a fixed-person chore, that person. Deleting someone else's chore is a delete request. The Chores screen groups chores into Overdue, Today, Coming up and Done, filtered Everyone or Mine. Anyone may tick any chore. A tick on a phone credits the signed-in member, with "Done by…" to credit someone else, and a kiosk always asks "Who did it?". A tick completes the next open occurrence, even before its day, and the schedule stays put. A chore turns overdue after its due time, or at the end of its day without one. When the next occurrence comes due, the missed one is dropped as missed, so only one is ever open. Tapping a done chore undoes it, with no confirmation, until the next occurrence comes due. Completions are one per occurrence, so offline replays are safe. No stars, points or routines.

- **Covers**: FOS-26 (edit and archive), FOS-27 (without monthly repeats), FOS-30 (undo widened), FOS-32 (no suggestions), CHORE-04, CHORE-06, CHORE-07, CHORE-08 (reworked without stars or routines), question 8
- **Depends on**: Roles; Delete requests; Region format
- **ADRs**: ADR-0010, ADR-0016, ADR-0020, ADR-0022

### Chore rotas

A take-turns chore has an ordered rota of family members. Completing it passes the turn to whoever follows the turn holder, whoever did the work, so favors aren't tracked. A missed occurrence leaves the turn where it is. Undo gives the turn back. The chore row shows whose turn it is.

- **Covers**: FOS-28 (without "least done in 90 days"), FOS-29 (without kid rules)
- **Depends on**: Chores
- **ADRs**: ADR-0022

### Chore breaks

A member puts themselves on a chore break until a date, and an admin may do it for anyone. During the break their turns are skipped on every rota and their fixed chores fall to Anyone. A chore whose whole rota is on a break becomes Anyone. Everything returns on the end date. A chore break is separate from HA presence and from vacation mode.

- **Covers**: none in the brainstorm (the two oldest may be away for weeks)
- **Depends on**: Chores; Chore rotas
- **ADRs**: ADR-0022

### Chore snooze

Anyone may snooze an open chore from its row or its attention card: +1 hour, to the chores check, or to tomorrow at the morning hour. Only choices still ahead and before the next occurrence are offered, so a daily chore can go to tonight but not tomorrow. A snoozed occurrence isn't due or overdue until the snooze ends, then its reminder fires again. The schedule and the turn don't move. Snoozes have no limit, and each one goes in the chore's history. The reminder notification has no snooze action.

- **Covers**: FOS-05 (the postpone, for chores)
- **Depends on**: Chores; Chore reminders
- **ADRs**: ADR-0022

### Chore reminders

Whoever's up gets a Companion notification at the due time, or at the morning hour (default 8 AM) for chores without one. If it's still open, they get one more at the due time, or at the chores check (default 7 PM) for chores without one. Admins set both hours. Anyone chores never send notifications. Members on a chore break get none.

- **Covers**: FOS-31 (plus an overdue reminder)
- **Depends on**: Chores; mapping each member to their Companion notify targets (ADR-0012)
- **ADRs**: ADR-0012, ADR-0022

### Chore history

Every completion records the occurrence, who did it, who ticked it and when. Every snooze records who snoozed it and until when. Each chore shows its history, and each member's completions can be listed. It replaces the prototype's static "This week" strip.

- **Covers**: CHORE-10 (history), replaces CHORE-09
- **Depends on**: Chores
- **ADRs**: ADR-0022
