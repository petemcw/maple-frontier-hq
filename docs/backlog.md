# Backlog

One flat list of Home HQ features, each sharp enough to hand to `/farseer:planner`. Area tickets in the brainstorm-to-backlog map append features under "Unordered"; the ordering ticket sorts them.

## Unordered

### Roles

Every signed-in family member or kiosk has one fixed role: admin, member or display. Both parents are admins and the four children are members. Admin-only actions are managing members and roles (including letting in an unmapped HA user), approving kiosks, and integrations: Google, Immich, the AI key and spending limit, and the weather location. Adding and removing calendar subscriptions is admin-only too (ADR-0024). Adding and viewing papers, viewing account numbers, and editing duty presets are admin-only too (ADR-0018), as are setting the task reminder hours and putting someone else on a task break (ADR-0023). Every other action is open to every member, and kiosks act as the household. Roles are checked in code, with no capability table.

- **Covers**: FOS-48 (narrowed), question 8 (approval part), question 20
- **Depends on**: sign-in with Home Assistant (ADR-0005)
- **ADRs**: ADR-0005, ADR-0006, ADR-0016, ADR-0018, ADR-0023, ADR-0024

### Delete requests

Deleting a household record, recipe or task that someone else owns creates a delete request instead. The item stays until an owner (whoever added it, or the person it is for) or an admin approves. Unanswered requests expire after 7 days. Home HQ records the creator and subject of every covered item. A kiosk deletes only household items. Calendar events aren't covered, because Home HQ never edits Google (ADR-0024). Shopping items aren't covered.

- **Covers**: FOS-48 (replaces Supervised drafts), question 8 (approval part)
- **Depends on**: Roles; the tasks, kitchen and household records features it guards
- **ADRs**: ADR-0016, ADR-0024

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
- **Depends on**: Duties; the HA mirror (ADR-0004); actionable Companion notifications (ADR-0012); Attention; kiosks (ADR-0006)
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
- **Depends on**: Recipe box; Telegram linking
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

The recipe in large type for cooking from a phone or tablet: ingredients alongside the steps (side by side on a tablet), each tickable, and the screen kept awake with the Screen Wake Lock API while open. Scaling and the US/metric toggle apply. A time in a step ("simmer 20 minutes") is tappable and starts a timer labeled with the recipe and step ("Chili · Step 4").

- **Covers**: none (the brainstorm notes FamilyOS has no cook mode, under "Kitchen (new area)")
- **Depends on**: Recipe scaling and units; Timers
- **ADRs**: ADR-0021, ADR-0028

### Tasks

A task is one-off or repeating. A one-off has an optional date, and a time only with a date. A repeating task repeats daily, or weekly on chosen days, from a first date, with an optional end date and an optional due time. Each task has a title, an emoji, notes, and who's up: a fixed person or Anyone. Any member adds, edits and archives tasks, and every task is family-wide. The owners are whoever added it and, for a fixed-person task, that person. Deleting someone else's task is a delete request. A one-line add at the top of the Tasks screen makes a one-off for Anyone with no date, and "Add details" opens the full form. The Tasks screen groups tasks into Overdue, Today, Coming up, Someday (one-offs with no date) and Done (ticked today), filtered Everyone or Mine, where Mine is my fixed tasks, my turns and Anyone tasks. Rows show a repeat mark on repeating tasks. Anyone may tick any task. A tick on a phone credits the signed-in member, with "Done by…" to credit someone else, and a kiosk always asks "Who did it?". A tick on a repeating task completes its next open occurrence, even before its day, and the schedule stays put. A dated task turns overdue after its due time, or at the end of its day without one. For a repeating task, the missed occurrence is dropped as missed when the next one comes due, so only one is ever open. A one-off stays open and overdue until it's ticked, moved or deleted. Tapping a done task undoes it, with no confirmation: a repeating task until its next occurrence comes due, a one-off until the end of that day. After that a one-off can be reopened from its history. Completions are one per occurrence, so offline replays are safe. No stars, points or routines.

- **Covers**: FOS-19 (one list with the routine work; no monthly repeats, Done for the day), FOS-26 (edit and archive), FOS-27 (without monthly repeats), FOS-30 (undo widened), FOS-32 (no suggestions), CHORE-04, CHORE-06, CHORE-07, CHORE-08 (reworked as tasks without stars or routines), question 8
- **Depends on**: Roles; Delete requests; Region format
- **ADRs**: ADR-0010, ADR-0016, ADR-0020, ADR-0023

### Task rotas

A repeating, take-turns task has an ordered rota of family members. Completing it passes the turn to whoever follows the turn holder, whoever did the work, so favors aren't tracked. A missed occurrence leaves the turn where it is. Undo gives the turn back. The task row shows whose turn it is. One-off tasks have no rota.

- **Covers**: FOS-28 (without "least done in 90 days"), FOS-29 (without kid rules)
- **Depends on**: Tasks
- **ADRs**: ADR-0023

### Task breaks

A member puts themselves on a task break until a date, and an admin may do it for anyone. During the break their turns are skipped on every rota and their fixed tasks fall to Anyone. A task whose whole rota is on a break becomes Anyone. Everything returns on the end date. A task break is separate from HA presence and from vacation mode.

- **Covers**: none in the brainstorm (the two oldest may be away for weeks)
- **Depends on**: Tasks; Task rotas
- **ADRs**: ADR-0023

### Task snooze

Anyone may snooze an open, dated task from its row or its attention card: +1 hour, to the evening check, or to tomorrow at the morning hour. Only choices still ahead, and for a repeating task before its next occurrence, are offered, so a daily task can go to tonight but not tomorrow. A one-off can be snoozed again and again. A snoozed task isn't due or overdue until the snooze ends, then its reminder fires again. The schedule and the turn don't move. Snoozes have no limit, and each one goes in the task's history. Someday tasks can't be snoozed. The reminder notification has no snooze action.

- **Covers**: FOS-05 (the postpone, for tasks)
- **Depends on**: Tasks; Task reminders
- **ADRs**: ADR-0023

### Task reminders

Whoever's up gets a Companion notification at the due time, or at the morning hour (default 8 AM) for dated tasks without one. If it's still open, they get one more at the due time, or at the evening check (default 7 PM) for tasks without one. Admins set both hours. Anyone tasks and Someday tasks never send notifications. Members on a task break get none.

- **Covers**: FOS-19 (reminder at the due time), FOS-31 (plus an overdue reminder)
- **Depends on**: Tasks; mapping each member to their Companion notify targets (ADR-0012)
- **ADRs**: ADR-0012, ADR-0023

### Task history

Every completion records the occurrence, who did it, who ticked it and when. Every snooze records who snoozed it and until when. Each task shows its history, done one-offs can be found and reopened there, and each member's completions can be listed. It replaces the prototype's static "This week" strip.

- **Covers**: CHORE-10 (history), replaces CHORE-09
- **Depends on**: Tasks
- **ADRs**: ADR-0023

### Google calendar connection

Home HQ reads the Family calendar and each member's free/busy from the family's Google Workspace, as a dedicated Home HQ Workspace user. It uses an Internal OAuth app with `calendar.calendarlist.readonly`, `calendar.events.readonly` and `calendar.freebusy`, and never writes. The OAuth client and refresh token live in a Docker secret, never in SQLite. Access tokens stay in memory, and auth headers are redacted from logs. Events sync with sync tokens every 1-2 minutes and whenever a client opens the calendar. A 410 triggers a full resync of that calendar. Each member maps to their Workspace address. An `invalid_grant` shows admins "Google access lost" and never retries. Setup is a checklist for the admin: create the Home HQ user and the Internal app, share the Family calendar with it as "See all event details", and set the domain's primary-calendar default to "Only free/busy". Before building, check the research's open items on a real account: whether `calendar.freebusy` alone is enough, how declined and "Available" invites show, whether attendee copies share `iCalUID`, and how far an endless series expands.

- **Covers**: question 6
- **Depends on**: Roles; the Docker secret (ADR-0009); a Workspace seat for the Home HQ user
- **ADRs**: ADR-0008, ADR-0009, ADR-0024

### Calendar

The Calendar screen shows the Family calendar in Today, 3 Day, Week and Month modes, with the prototype's header, arrows, swipe and Today button. Today mode has one lane per member. Everyone events span every lane, and an event with family attendees appears in each attendee's lane, colored by its first attendee, with dots for the others. Week is Sunday to Saturday, and Month is a Sunday-start grid. Repeating events show as their occurrences. Times are in the household time zone, and all-day events stay on their date. Each member has one color kept in Home HQ, and everyone events are black. On phones, a "+" opens Google Calendar's new-event screen. The wall panel has no "+".

- **Covers**: CAL-01 to CAL-18 (CAL-04 now opens Google; CAL-16 is Sunday to Saturday; CAL-18 starts on Sunday), FOS-13 (showing repeats only)
- **Depends on**: Google calendar connection; Region format
- **ADRs**: ADR-0020, ADR-0024

### Event sheet

Tapping an event opens its sheet: a band in the event's color with the day and date, the title, and the time range or "All day". It notes the event's own time zone when that differs from the household's ("10:00 AM MT"). Below that come the leave-by row, the location, attendee chips (everyone for an everyone event), the notes, and how it repeats. On phones, the sheet also has "Open in Google Calendar". Nothing on the sheet edits the event.

- **Covers**: CAL-19, CAL-20, CAL-21 (repeats now shown)
- **Depends on**: Calendar
- **ADRs**: ADR-0024

### Busy blocks

Each member's primary calendar shows as grey "Busy" blocks in their Today lane, in 3 Day and in Week, read through `freebusy.query` with no titles. Busy blocks never count for Next up, leave-by or Month's dots, and tapping one does nothing. A busy block that matches an event already in that member's lane is dropped, so an accepted invitation doesn't show twice.

- **Covers**: question 6 (primary calendars as free/busy)
- **Depends on**: Google calendar connection; Calendar
- **ADRs**: ADR-0024

### Calendar subscriptions

Admins add, rename and remove read-only ICS feeds, such as a school calendar or US holidays. Home HQ fetches each feed itself every 6 hours. Their events are everyone events, shown and counted like any other, but never editable.

- **Covers**: FOS-14 (narrowed: every feed is for everyone, with no reminder switch)
- **Depends on**: Calendar; Roles
- **ADRs**: ADR-0024

### Leave-by

A timed event with a physical location that isn't home gets a drive time from Google's traffic-aware Routes API. Home HQ calls it directly with its own key, kept in a Docker secret, and caches the result per event, recomputing as leave-by nears. A single-person event measures from that person's location in the HA mirror. If the location is unknown, stale, or HA is down, it uses home. Shared and everyone events measure from home. Leave-by is start minus drive time, and it drives the leave pill, the hatched travel block, Next up's order, and the sheet's "Leave by {time}" with "{n} min drive from {home | your location}". A quota cap in Google Cloud keeps costs inside the free tier.

- **Covers**: DATA-06, TODAY-06 (order), TODAY-08, CAL-10 (travel block), CAL-20 (leave-by row, now live)
- **Depends on**: Google calendar connection; the HA mirror (ADR-0004); a Google Maps API key with billing
- **ADRs**: ADR-0004, ADR-0024

### Time to leave

Ten minutes before leave-by, each of the event's people gets a Companion notification: "Leave in 10 min for {title} · {n} min drive". There's none if they are already at the event's location. It's the only calendar notification Home HQ sends, because Google sends the event reminders.

- **Covers**: TODAY-08 (as a notification), FOS-13 (replaces Home HQ's event reminders)
- **Depends on**: Leave-by; the Companion app on every member's phone and mapping each member to their notify targets (ADR-0012)
- **ADRs**: ADR-0012, ADR-0024

### Immich connection

Home HQ reads the Home HQ album from the family's Immich on the NAS, as its own Immich user that owns nothing. A parent owns the album, makes the other five family members editors, and shares it to Home HQ's user as viewer. The API key has only `album.read`, `asset.read` and `asset.view`, and lives in a Docker secret, never in SQLite. The server calls Immich through `@immich/sdk` behind one adapter. On connect it checks `GET /server/version`, needs v3.2 or later, and refuses an untested major. It re-reads the album every 15 minutes and when someone opens Photos, and skips the re-read when the album's change timestamp hasn't moved. Only stills are kept. Browsers get images only through the server's proxy, at `/api/photos/:id/:size?c=<thumbhash>`, and only for photos in the album that aren't hidden. They never see the key or an Immich URL. A disk cache of recent previews (about 150 MB, keyed by photo, size and thumbhash) keeps slideshows playing while Immich is down. With an empty cache, Photos says "Photos are unavailable". Admins see the connection's state. Setup is a checklist for the admin: create the Home HQ user and its key, create the album, and share it. Tests run against a fake Immich. Relicensing the repo to AGPL-3.0 is already done.

- **Covers**: question 5 (photo storage), question 9, PHOTO-10 (a real library)
- **Depends on**: Roles; the Docker secret (ADR-0009); Immich v3.2 or later on the NAS
- **ADRs**: ADR-0008, ADR-0009, ADR-0016, ADR-0025

### Slideshow

The Photos screen plays the Home HQ album full-bleed on black. Each slide lays itself out. A landscape fills the screen with a slow random pan, two portraits from the same day sit side by side, and a lone portrait sits over a blurred copy of itself, with the thumbhash painting the blur while the preview loads. A portrait counts as one when its height is greater than its width. On this day photos lead each round, found by Home HQ's own search on the local capture date across past years. The rest are shuffled. The caption reads "On this day · 3 years ago · Duluth", or "3 years ago" or "This year" with the place, above the date. It never says who added the photo. Tapping the left or right edge, or swiping, steps back or forward through history. A tap in the middle brings the caption to full strength and shows a bar with back, heart, Hide and next that fades after a few seconds. Slides change every 15 seconds, a manual step restarts the timer, and the slideshow pauses while Photos is hidden. The service worker preloads the next few slides.

- **Covers**: PHOTO-01 to PHOTO-03 (no layout picker), PHOTO-04 (on this day first), PHOTO-05 and PHOTO-06 (Cover and Pair only), PHOTO-07, PHOTO-09
- **Depends on**: Immich connection; Region format
- **ADRs**: ADR-0010, ADR-0020, ADR-0025

### Photo favorites

Each family member hearts photos for themselves, stored in Home HQ by Immich photo ID. A member's favorites come up more often in their own slideshow, and an All / Mine switch plays only their favorites, with an empty state when they have none. On kiosks, a photo anyone has hearted comes up more often. Kiosks show no heart and no switch. The toast reads "Added to favorites · it'll show up more often" or "Removed from favorites". Favorites are never written to Immich.

- **Covers**: PHOTO-04 (favorites weighting), PHOTO-08 (now saved, per member)
- **Depends on**: Slideshow
- **ADRs**: ADR-0008, ADR-0025

### Hidden photos

Anyone, a kiosk included, can hide the photo on screen from the slideshow bar. It leaves every Home HQ screen at once, with undo in the toast, and stays in Immich and in the album. Home HQ records who hid it and when. Only admins see the Hidden list, with thumbnails, and unhide from it. Hiding isn't a delete, so it needs no delete request. A hidden photo is never proxied, cached anew, or chosen as the photo of the day.

- **Covers**: PHOTO-10 (hiding a photo)
- **Depends on**: Slideshow; Roles
- **ADRs**: ADR-0006, ADR-0016, ADR-0025

### Photo of the day

The server picks one photo per family day, the same on every screen. An on this day photo wins when there is one. Otherwise it walks a shuffled round of the album, stored in SQLite, so every photo comes up once per round and none repeats on consecutive days. A photo anyone has hearted is likelier, and hidden photos never qualify. A photo hidden during its day is replaced at once. It doesn't fill the Today hero, which is the backdrop (ADR-0031). Whether it gets a Today tile belongs to the Today layout.

- **Covers**: FOS-08
- **Depends on**: Immich connection; Photo favorites; Hidden photos; Region format (the family day)
- **ADRs**: ADR-0008, ADR-0020, ADR-0025, ADR-0031

### Home control layout

Home control lists rooms from HA areas, grouped under HA floors (Top, Main, Lower, Lowest). Areas with no floor come last under "Outside". A room appears only if it holds a tile or a scene. Each room heading shows "{n} on" or "All off", plus a temperature where the room has a labeled temperature sensor. Three summary cards top the screen. Lights shows the count on and opens a sheet of every light, on first. Garage shows Open or Closed, tinted when open. Climate shows the temperature and what the ecobee is doing. Display names, order and hidden items use ADR-0011's admin overrides. With HA offline, every tile is disabled and no state shows as current.

- **Covers**: HOME-01 (no Security card; Media card is its own feature), HOME-02 (Lights sheet), HOME-05, DATA-17, RED-55 (temperature on rooms only)
- **Depends on**: the HA connection (ADR-0004); HA areas, floors and Home HQ labels; Stairs and Double Down given a floor in HA (a household change)
- **ADRs**: ADR-0004, ADR-0011, ADR-0026

### Tiles

Labeled lights, switches and fans show as tiles in their room. A tap toggles, and turning a light on restores its last level, or 100%. A horizontal drag of more than 10 px on a dimmable light turns the tile into a dimmer, and letting go commits. On/off lights and switches only toggle. A fan tile shows on, off and its speed. A long press without moving opens the tile's sheet. There's no visible cog and no keyboard or screen-reader way in (ADR-0026). A light's sheet holds what that light supports: brightness, warm to cool, color, and "On for {n} min". A fan's sheet sets its speed.

- **Covers**: HOME-06, RED-53, RED-37 (the sheet pattern), DATA-18 (lights, switches, fans)
- **Depends on**: Home control layout
- **ADRs**: ADR-0011, ADR-0026

### Pending changes

Every tile change shows at once with a pending dot, which stays until HA reports the new state. With no change after 5 s, the garage's after 30 s, the tile reverts and toasts "Couldn't reach {name} · it may be offline". Unavailable entities show as unavailable and can't be tapped. HA commands are never queued offline (ADR-0010).

- **Covers**: HOME-10
- **Depends on**: Tiles
- **ADRs**: ADR-0004, ADR-0010, ADR-0026

### All off

"Turn everything off" in the Lights summary sheet turns off every labeled light. Each floor heading has its own All off for that floor's lights. Both toast with Undo, which turns back on exactly the lights they turned off, at their earlier levels, from the HA mirror.

- **Covers**: HOME-02 ("Turn everything off"), plus All off per floor
- **Depends on**: Home control layout, Tiles
- **ADRs**: ADR-0026

### Scenes

Every HA scene or script with the Home HQ label is a scene button. A scene in an HA area shows in that room's section, and the rest share a whole-house row at the top. A tap runs it and toasts "{scene} on" with Undo, which restores the entities the scene touched from the HA mirror as they were just before. For a script, that's the entities it changed in the next few seconds. No scene stays highlighted. Home HQ stores no scene definitions, so a scene that pauses an automation is an HA script (RED-21).

- **Covers**: HOME-03 (HA's scenes in place of the five fixed ones, with no locking), HOME-04 (Undo only), RED-21, DATA-19 (replaced by HA)
- **Depends on**: Home control layout
- **ADRs**: ADR-0011, ADR-0026

### Climate

The climate panel shows the ecobee's temperature, humidity and what it's doing (Heating, Cooling, Holding, Off), with the setpoint ± in 0.5 °F steps within the thermostat's own limits. In Auto (heat_cool) it shows both setpoints, each with ±. ± is disabled while the mode is Off. A Heat / Cool / Auto / Off switch sets the mode. A long press opens the climate sheet, with presets (home, away, sleep), fan (auto, on) and the humidifier. Every member and kiosk may change it.

- **Covers**: HOME-11 (in °F), RED-37 (climate sheet), DATA-20; RED-15 and RED-56 dropped (one zone)
- **Depends on**: Home control layout, Pending changes; Region format
- **ADRs**: ADR-0020, ADR-0026

### Garage

The garage tile drives HA's garage cover. It shows Closed or Open, with a warning style when not closed, and Opening... or Closing... while the change is pending. A tap opens a confirm sheet ("Open the garage?" with "Make sure the driveway is clear.", or "Close the garage?" with "The door will stop and reverse if anything is in the way."). Sliding the knob all the way commits, and letting go early snaps back. It works the same away from home, and from kiosks.

- **Covers**: HOME-09, the Garage summary card in HOME-01
- **Depends on**: Tiles, Pending changes; an HA template cover built from the Shelly relay and the garage door sensor (a household change)
- **ADRs**: ADR-0006, ADR-0026

### Cameras

Labeled cameras show as tiles on every device and kiosk. A camera with a still image refreshes every 20 s while the tile is on screen and the screen is awake, and shows "{n}s ago" from the image's real time. A camera with no still image (the porch camera on wyzeapi) shows its name and "Tap for live". Opening a tile plays live video, muted, through HA's WebRTC, with the server relaying the signaling over its own HA connection. A camera with no image and no stream says "Camera unavailable". There's no package chip, motion badge, playback, two-way talk or siren.

- **Covers**: HOME-12, RED-39 (real refresh and live view only), DATA-22
- **Depends on**: Home control layout; docker-wyze-bridge (IDisposable fork) publishing the School Room and Double Down cameras to HA by MQTT discovery (a household change)
- **ADRs**: ADR-0004, ADR-0026

### Open at your room

On a phone, Home control opens scrolled to the room the member's room presence says they're in. If that room has no tiles, it opens at the room's floor, and with no reading at all, at the top. Kiosks and other shared devices always open at the top. Each member maps to their room presence sensor in HA, alongside their other HA mappings.

- **Covers**: the "open at the room the family member is in" part of the home control ticket
- **Depends on**: Home control layout; ESPresense receivers in the bedrooms and main rooms with every family member's phone enrolled, publishing a room sensor per person to HA (a household change)
- **ADRs**: ADR-0026

### Banners

A banner is a notice that shows across the top of every screen, kiosks and phones alike, until someone acts on it. The server holds each banner's state and pushes changes over `/ws`, so dismissing it on one device clears it everywhere, and a device that reconnects gets the current set. While Home HQ's HA connection is down, banners built from HA state are withdrawn instead of showing as current (ADR-0004).

- **Covers**: the acknowledge-across-devices part of ALERT-02 (applied to banners), section 12 Banners
- **Depends on**: the `/ws` channel and the HA mirror
- **ADRs**: ADR-0004, ADR-0027

### Garage left open

The garage left open banner fires when `binary_sensor.garage_door_status` has been open for 15 minutes, or for 3 minutes while `binary_sensor.household_nighttime` is on, and a door already open past 3 minutes at nightfall fires at once. It reads "Garage open for {n} min · Opened at {time}" with Close garage, which opens the garage slide sheet, and Leave open. Leave open quiets it on every device until the door closes, except that nightfall brings it back once. Closing the door clears it. When it fires, both admins get a Companion push whose Close garage action iOS runs only after Face ID or the passcode (`authenticationRequired`). The action can only close the door. Tests cover the day and night thresholds, nightfall re-arming, and that the push action never sends open.

- **Covers**: ALERT-03, RED-18 (the garage's open duration)
- **Depends on**: Banners, Garage, Companion notifications to admins (ADR-0012); the garage template cover (a household change)
- **ADRs**: ADR-0006, ADR-0012, ADR-0026, ADR-0027

### Parcels

Home HQ reads the Mail and Packages sensors and camera entities from HA. Mail and Packages is a HACS integration pinned to a release tag, reading the parcels mailbox in the family's Workspace, and each member's Gmail filter forwards carrier and shop emails there. Parcels belong to the household. While any are out for delivery, Today shows "{n} parcels today", and its sheet lists each one with its carrier and any Amazon locker or delivery code, on every screen. Each delivery raises a "Delivered · {carrier}" banner whose View photo opens the carrier's delivery photo, and OK on any device clears it. Deliveries send no push. Home HQ holds no mailbox credentials.

- **Covers**: ALERT-04, RED-19 (today's deliveries only)
- **Depends on**: Banners; Mail and Packages installed in HA with a parcels mailbox and a Gmail forwarding filter for each member (a household change)
- **ADRs**: ADR-0004, ADR-0027

### House status

The house status line on Today shows the garage open or closed, in the warning style when open, then "{n} lights on" from labeled light tiles and the inside temperature from the ecobee in the region format. Tapping it opens Home control. No lock or alarm wording appears anywhere, which retires DEF-02 and DEF-05.

- **Covers**: TODAY-17, DEF-02, DEF-05
- **Depends on**: Home control layout, Garage, Climate
- **ADRs**: ADR-0020, ADR-0026, ADR-0027

### House problems

The server derives house problems from the HA mirror: a battery below 20%, named with its device, room and battery-type label ("Laundry motion · Laundry Room · CR2450 · 14%"), a Shelly or TP-Link fault sensor turning on, and HA's daily backup failing. A problem clears itself when its cause clears. An unavailable tile is not a house problem. Problems show as attention to the admins and on every kiosk.

- **Covers**: RED-06 (the signals, not the pill)
- **Depends on**: the HA mirror
- **ADRs**: ADR-0027, ADR-0033

### Timers

The server holds each timer's label, duration, end time and state (running, paused, time's up) and pushes changes over `/ws`. Timers belong to the household, so every screen shows the running ones, kiosks and phones alike, ticking locally from the end time. Anyone may cancel a timer, add a minute, or pause and resume it, and the change shows everywhere at once. A timer starts from a time in a cook mode step, labeled "{recipe} · Step {n}", or from New timer on the timers sheet on any screen, which takes a duration and an optional label ("10 min timer" without one). Where running timers sit on Today and other screens is set by the Today layout ticket. Starting a timer never moves the wall panel to Today. Timers are Home HQ data, so they keep running while the HA connection is down. Home HQ doesn't read the kitchen Alexa's voice timers.

- **Covers**: RED-07 (Home HQ's own timers, without voice or speakers); RED-30's jump to Today is not adopted
- **Depends on**: the `/ws` channel
- **ADRs**: ADR-0028

### Time's up

When a timer reaches zero it becomes a "Time's up · {label}" banner. Every screen with Home HQ open chimes for 1 minute. Then the banner stays silent until someone dismisses it on any device, which clears it everywhere, or 30 minutes pass. +1 minute on the banner snoozes it into a running timer again. The family member who started the timer gets one time-sensitive Companion push. A timer started on a kiosk pushes nobody. Tests cover the chime stopping at 1 minute, the banner clearing at 30 minutes, snooze, and no push for kiosk-started timers.

- **Covers**: RED-07 (the "Time's up!" state)
- **Depends on**: Timers, Banners; Companion notifications to the starter (ADR-0012)
- **ADRs**: ADR-0012, ADR-0027, ADR-0028

### Day, evening and night

Home HQ reads night from `binary_sensor.household_nighttime` in the HA mirror (22:45 to 08:00 today), the same night the garage left open banner uses. Evening runs from 45 minutes before sunset, from `sun.sun`, until night. Every kiosk knows which of the three it is in, and the change reaches open kiosks over `/ws`. While the HA connection is down, a kiosk keeps the last phase it had and works out evening from the sun times it last saw. Tests cover the edges of each phase, a night that crosses midnight, and a dropped HA connection.

- **Covers**: SHELL-06's day and night windows, RED-24's time-of-day brightness
- **Depends on**: the HA mirror, the `/ws` channel
- **ADRs**: ADR-0027, ADR-0029

### Idle timers

On a kiosk, a touch anywhere resets the idle timer. After 60 s idle it closes any open sheet and goes to Today. After 120 s it shows ambient by day and the night clock at night, as that kiosk's settings allow. In the night dashboard, 20 s idle returns to the night clock. Phones never go idle this way. Tests cover each timeout and that a touch resets it.

- **Covers**: SHELL-04, SHELL-05, SHELL-06, SHELL-08; SHELL-07 is moot (no takeovers)
- **Depends on**: Day, evening and night; Kiosk settings
- **ADRs**: ADR-0029

### Ambient

A kiosk's idle screen by day: the slideshow at its 15 s pace, weighted by anyone's favorites, with each slide's place-and-date caption, under a dark gradient. Bottom left, a huge clock and the date. Bottom right, the weather and the next Family calendar event, as Next up counts it, with its person-color dot. Busy blocks never show. In the evening the gradient darkens and the kiosk dims. The text shifts by up to 4 px every minute. A tap goes to Today. A banner shows over ambient without leaving it.

- **Covers**: AMB-01 (captions shown; presence avatars wait for the presence decision), AMB-02, AMB-03, NFR-12
- **Depends on**: Slideshow, Idle timers, Weather, Calendar; Banners
- **ADRs**: ADR-0024, ADR-0025, ADR-0029

### Night clock

A kiosk's idle screen at night, on black in dim red: a big clock; the weather with "First up tomorrow: {title} {time} · {person}" from the Family calendar ("today" after midnight, or "Nothing early tomorrow"); and the house status line, such as "Garage closed · 3 lights on · 68°". The text shifts by up to 4 px every minute. A tap opens the night dashboard. A banner shows over the night clock without leaving it.

- **Covers**: NIGHT-01 (with the house status line in place of "House locked · alarm armed"), NFR-12, DEF-05
- **Depends on**: Day, evening and night; Idle timers; Calendar; House status; Banners
- **ADRs**: ADR-0027, ADR-0029

### Night dashboard

A tap on the night clock opens the full interface under a dim red-amber filter (invert, sepia, saturate, dim). Everything works as normal. After 20 s idle it goes back to the night clock.

- **Covers**: NIGHT-02, SHELL-08
- **Depends on**: Night clock, Idle timers
- **ADRs**: ADR-0029

### Screen sleep

On a kiosk running in Fully Kiosk, one small screen-control module listens for Fully's camera motion events and turns the screen on, off and dim through Fully's JavaScript interface. Outside Fully the module does nothing. By day the screen goes off after 10 minutes with no motion, and motion wakes it to ambient. At night it stays off, and motion or a touch brings up the night clock, which goes dark 1 minute after the last motion. In the evening and at night the screen dims. A new banner wakes the screen, which then sleeps by the same rules, with the banner still showing when it next wakes. Time's up chimes at night too. Tests run against a fake Fully interface.

- **Covers**: question 15, RED-24 (wake and dim by motion from the panel's own camera, not a room sensor); RED-25's and RED-46's screensavers are not adopted
- **Depends on**: Day, evening and night; Ambient; Night clock; Banners; Fully Kiosk Plus on the wall panel with motion detection, camera access and the JavaScript interface turned on (a household change)
- **ADRs**: ADR-0013, ADR-0029

### Kiosk settings

Each kiosk has settings an admin changes: theme (system, light or dark), and ambient, night clock and Time's up chime, each on or off. A new kiosk starts with everything on and the system theme. With ambient off the kiosk stays on Today when idle by day. With the night clock off, its nights look like its days, and with the chime off, Time's up shows silently there. Changes reach the kiosk over `/ws`. Where admins change them is set with the other admin surfaces.

- **Covers**: FOS-11 and RED-44, narrowed to four settings
- **Depends on**: Roles (admins), the `/ws` channel
- **ADRs**: ADR-0016, ADR-0029

### Telegram linking

Home HQ is the only reader of @MapleFrontierBot (ADR-0019). A family member taps Link Telegram in their settings and Home HQ shows a one-time code, valid for 10 minutes, which they send to the bot. From then on that chat is theirs, and everything it sends acts as them. Unlink removes it, from the member's own settings or by an admin. The bot answers an unlinked chat with one line on how to link and does nothing else. Admins see each member's link state and the bot's polling status, including a 409 conflict.

- **Covers**: FOS-41 (the Telegram way in)
- **Depends on**: Roles; the Telegram bot poller (ADR-0019)
- **ADRs**: ADR-0019, ADR-0030

### AI connection and cap

The server calls the Claude API with `@anthropic-ai/sdk`, using Sonnet 5.5, and the key is a server secret that's never in source, the bundle or a served file. Each call records its feature, model, tokens and cost. The AI cap, $5 a month by default, is changed by admins, who also see this month's spend by feature. Once the cap is reached, or the API fails, quick add and dinner ideas pause. The bot replies "AI is paused, try again later" and keeps nothing, the add box keeps the typed text, and dinner ideas say they're unavailable. The cap resets on the first of the month (household time zone). The Anthropic Console workspace spend limit is set as a backstop. Before build, an eval on real captures checks the model choice.

- **Covers**: FOS-47 (narrowed to one provider and one cap)
- **Depends on**: Roles (admins); Region format (the month boundary)
- **ADRs**: ADR-0016, ADR-0030

### Quick add

A family member sends text, photos (JPEG, PNG, WebP, HEIC converted) or a PDF to the bot, or puts them in the add box on their own device. Kiosks have no add box. Voice notes get a reply asking for text. The server resizes images, then asks Claude, with propose tools only, to read the capture into suggestions: tasks (who, one-off or repeating, date and time), shopping items (list, store, amount, merged as a shopping add would be), recipes and events. The model gets the household's member names, shopping lists, stores, categories and open tasks so it can match them. A suggestion it's unsure of marks the field to check. Nothing is saved until the sender adds it. A capture that yields nothing gets "I couldn't find anything to add."

- **Covers**: FOS-41 (text, photos and PDFs from the bot and the app; no voice, mailbox or share target), RED-11's menu-photo idea in general form
- **Depends on**: Telegram linking; AI connection and cap; Tasks; Shopping add
- **ADRs**: ADR-0017, ADR-0019, ADR-0023, ADR-0030

### Quick add tray

Suggestions from one capture show together in the bot's reply, each ticked, with Add selected and Dismiss buttons, and in the sender's quick add tray in the app, where a suggestion can be edited first. Add selected saves the ticked ones as the sender, through the same actions as their own taps, and offers one Undo for the batch. A suggestion that matches something already there ("Already on Shopping", an open task with the same name) starts unticked and says so. Ticking it adds it anyway. Suggestions expire after 7 days, or once their date has passed if that's sooner. The capture's attachments are deleted when its last suggestion is added, dismissed or expired. Only the sender ever sees their tray. A count of waiting suggestions is available to the Today layout.

- **Covers**: FOS-42; question 16's propose-or-write (the sender adds, no one else approves); not FOS-43
- **Depends on**: Quick add
- **ADRs**: ADR-0016, ADR-0030

### Event suggestions

An event read from a capture becomes a suggestion with title, start, end or all-day, and place, and an Add to Google Calendar button instead of a tick. The button opens Google Calendar's pre-filled event page (`calendar.google.com/calendar/render?action=TEMPLATE`) and names the Workspace address it expects. Saving there is the sender's own act in Google. Home HQ keeps read-only scopes, and the event reaches Home HQ when it next syncs the Family calendar or the sender's busy blocks. Add selected skips event suggestions, and they expire after their date.

- **Covers**: FOS-41 (events, as a hand-off)
- **Depends on**: Quick add tray; Google calendar connection
- **ADRs**: ADR-0024, ADR-0030

### Recipe from a capture

A recipe in a photo, screenshot or PDF (a cookbook page, a handwritten card) is read into title, ingredient lines, steps, servings and time, and becomes the sender's recipe draft, as a link sent to the bot does. The capture's photo stays with the draft as its recipe photo when it shows the dish. The bot reply and the tray link to the draft, which is saved from its own form. Ingredient lines are parsed as on save.

- **Covers**: FOS-33 (AI capture), FOS-41 (recipes)
- **Depends on**: Quick add; Recipe from a link (the recipe draft); Recipe scaling and units
- **ADRs**: ADR-0021, ADR-0030

### Dinner ideas

On request, on a phone or a kiosk, Home HQ suggests three dinners for today: mostly recipes from the recipe box, plus one new dish. Each shows a name, a one-line summary and total time. The model weighs today's weather (Open-Meteo, from Weather), recent dinners and the food notes, and avoids repeating a recent dinner. Other ideas runs again with the shown ones excluded. Make this counts the dish as today's dinner and opens the recipe, or the new dish's full recipe with ingredient lines that scale and go to shopping. A new dish also has Save to recipe box, owned by whoever saved it, or the household on a kiosk. Nothing runs on a schedule.

- **Covers**: RED-10 (adapted to an adult household with no meal plan)
- **Depends on**: AI connection and cap; Recipe box; Recipe scaling and units; Ingredients to shopping; Recent dinners and food notes; Weather
- **ADRs**: ADR-0021, ADR-0030

### Recent dinners and food notes

Home HQ keeps the household's recent dinners for 14 days: a dish chosen with Make this, and a recipe opened in cook mode, counted once per day. Food notes are one short household text ("no shellfish, quick on weeknights"), edited by any member, with who changed it last. Both feed dinner ideas only.

- **Covers**: RED-10 (avoid recent meals, the family's rules)
- **Depends on**: Recipe box; Cook mode
- **ADRs**: ADR-0030

### Search

One search box finds tasks, shopping items, recipes, records and duties by words, using SQLite full-text search, with no AI. Results are grouped by area and limited to what the searcher may see (papers stay admin-only). Kiosks search as the household. Where the box lives is set by the navigation ticket.

- **Covers**: FOS-45 (by words, not by meaning)
- **Depends on**: Tasks; Shopping lists; Recipe box; Records; Duties
- **ADRs**: ADR-0018, ADR-0030

### Weather

The server fetches the forecast from Open-Meteo every 15 minutes, with no key, for the weather location an admin sets: the hourly WMO code, temperature, rain chance and wind, plus the local day's high, low, sunrise and sunset, in the household's units. It picks hours by timestamp, never by index, so the strip crosses midnight (DEF-07). Today's weather, the weather advice line, ambient, the night clock, the backdrop and dinner ideas all read it. Once the last fetch is out of date, no screen presents the weather as current. A "Weather data by Open-Meteo.com" link sits next to it.

- **Covers**: TODAY-02, TODAY-03, TODAY-04 (the data; how they show is the Today layout's), DATA-23, DEF-07, question 13 (weather source)
- **Depends on**: Region format (units and the family day); Roles (the weather location is admin-only)
- **ADRs**: ADR-0020, ADR-0031

### Backdrop

The Today hero is a backdrop: a storybook painting of Maple Frontier for the current weather group, light phase and season, on the panel and on phones. The server picks the cell and pushes it over `/ws`, so every screen shows the same image. Weather groups come from the hourly WMO code, and windy means clear or partly cloudy with wind over 19 mph. Light phase comes from the sun's elevation: daylight above 20°, golden hour to 8°, twilight to −6°, then dark. Season goes by month, but snow and freezing rain use winter art. Ten holidays win all day with their own day and dark images per weather. When weather is out of date it shows the weatherless backdrop for the season and light phase. Images and a manifest (cell, text tone, focus point) live in a mounted backdrops folder, outside the repo. A missing image falls back from holiday to base, then to weatherless, then to a plain background, and admins see which cells have no image. Phones crop a short banner around the focus point. The clock and weather sit at upper left over a soft scrim, in the manifest's text tone. The full list of images and prompts is `docs/backdrops.md`.

- **Covers**: RED-01 (our own art), RED-02 (adapted: no kid schedule, temperature gates or birthdays), RED-03, question 13
- **Depends on**: Weather; the backdrop art set (household setup); HA's `sun.sun` elevation; Idle and night (kiosks dim in the evening)
- **ADRs**: ADR-0029, ADR-0031

### Moon on the backdrop

On clear, partly cloudy and windy dark backdrops, Home HQ places the painted sprite for today's moon phase at the manifest's moon spot, between moonrise and moonset. The server reads the phase and the moon times from wttr.in once a day for the weather location. With no answer, no moon shows. Holiday backdrops get no moon, and a new moon shows nothing.

- **Covers**: question 13 (moon, added in this ticket)
- **Depends on**: Backdrop; the 7 moon sprites
- **ADRs**: ADR-0031


### Attention

A "Needs attention" section on Today holds one attention card per item waiting on someone who can act. Each viewer sees only their own. Sensor prompts go to the duty's assignee (or the admins for Anyone) and every kiosk. Delete requests go to every owner of the item and both admins, never a kiosk, and the first answer clears them everywhere. Overdue tasks: a member sees their fixed tasks, their turns and Anyone tasks, and a kiosk sees every overdue task with who's up. Overdue duties go to the assignee (or both admins for Anyone) and every kiosk. House problems go to both admins and every kiosk. Banners, lights left on, waiting quick add suggestions and Someday tasks are not attention. Sensor prompts and delete requests come first, then overdue tasks and duties, most overdue first, then house problems. Up to the Today layout's cap, then "+{n} more" opens the full list. Each card has its actions: Yes or No on a prompt, Delete or Keep on a delete request, Done or Snooze on a task, Done or Move date on a duty, Snooze on a house problem. On a kiosk, Done asks "Who did it?". With nothing waiting, the section leaves Today, and no other screen shows attention. The server works out each viewer's attention and pushes changes over `/ws`, so an item comes and goes on every device as its cause changes. The server withdraws house problems while the HA connection is down (ADR-0004). Tests cover who sees each kind, member and kiosk alike.

- **Covers**: FOS-05 (adapted: no suggestions, shown on kiosks), RED-06 (the place, not the pill), RED-51 (ruled out: no notification center or "on now" list)
- **Depends on**: Roles; Delete requests; Sensor prompts; Duties; Tasks; Task snooze; House problems; the `/ws` channel; the Today layout for placement and the cap
- **ADRs**: ADR-0004, ADR-0006, ADR-0016, ADR-0018, ADR-0023, ADR-0027, ADR-0033

### Duty move date

Move date on an overdue duty's attention card moves only this due date later, never past the next period's date. A repeating duty stays anchored to its first due date, so the cycle doesn't shift. Anyone who sees the duty may move it, a kiosk included, which asks "Who moved it?". The move goes in the duty's log with who made it, and its reminders follow the new date.

- **Covers**: FOS-05 (postpone, applied to duties)
- **Depends on**: Duties; Attention
- **ADRs**: ADR-0018, ADR-0033

### House problem snooze

Admins and kiosks may snooze a house problem from its attention card. It leaves every screen at once until it gets worse: a battery comes back after another 5% drop, and a fault sensor or a failed backup comes back the next time it fires. A problem that clears and returns is a new one. Home HQ records who or what snoozed it. Tests cover the 5% step, a repeat failure and a cleared-then-returned problem.

- **Covers**: RED-06 (setting a problem aside)
- **Depends on**: House problems; Attention
- **ADRs**: ADR-0027, ADR-0033

### Speaker

The Living Room Sonos is a tile in its room. A tap plays or pauses it, and the tile shows the art, title and artist while something plays. Its sheet holds play/pause, previous, next, a volume slider that commits on release, and the position, which the browser works out from HA's last reported position and time. The server fetches the album art and serves it from its own URL, so no HA proxy token reaches a browser. Every member and kiosk may use it, at home or away. Changes show a pending dot until HA reports them (Pending changes). With HA down, the tile is disabled. No shuffle, repeat, seek or sleep timer.

- **Covers**: HOME-13, RED-08 (now playing, pause, skip, volume; no power button)
- **Depends on**: Tiles; Pending changes; the speaker labeled for Home HQ in HA
- **ADRs**: ADR-0004, ADR-0011, ADR-0026, ADR-0034

### Sonos favorites

The speaker's sheet shows the household's "My Sonos" list as it stands, each entry with its artwork, read through HA's media browser. A tap plays it on the speaker. The list is edited in the Sonos app and changes show as HA reports them. Home HQ adds no radio browsing and no admin-picked subset.

- **Covers**: RED-08 (one-tap stations), RED-38 (one-tap playlists)
- **Depends on**: Speaker
- **ADRs**: ADR-0034

### TVs

Each room's TV is one tile: the Family Room TV (Samsung TV, Denon receiver and Apple TV together) and the Double Down TV (its Apple TV). A TV shows Off, On, or Playing with the app and title its Apple TV reports. A tap opens its sheet, which holds Turn off. Turn off switches every part off with `media_player.turn_off`, never `remote.turn_off`, and needs no confirm. Every member and kiosk may use it, at home or away. Home HQ can't turn a TV on, and has no remote, volume or input control. Tests cover a TV whose parts disagree, such as the Denon on with the Samsung off.

- **Covers**: RED-08 ("TV & remote", narrowed to state and Turn off)
- **Depends on**: Tiles; Pending changes; the TV parts labeled for Home HQ in HA
- **ADRs**: ADR-0026, ADR-0034

### Media card

A fourth summary card on Home control, beside Lights, Garage and Climate. It shows "Quiet" when nothing plays, otherwise what's playing (the speaker's song first, then a playing TV), with "+{n}" when more than one player is busy. A tap opens a sheet with the speaker and every TV. With HA down, it shows no state as current.

- **Covers**: HOME-01 (the Media card), RED-50 (the "+1" for a second player)
- **Depends on**: Home control layout; Speaker; TVs
- **ADRs**: ADR-0004, ADR-0026, ADR-0034

### Synced lyrics

When the speaker's song changes, the server looks it up on LRCLIB by title, artist, album and duration, then without the album, then by search for the closest duration within ±2 s. It sends a `User-Agent`, honors `Retry-After` and caches each result in SQLite. The speaker's sheet gains a Lyrics view on every device, the panel included, that highlights the current line. A ± lyrics offset lines them up, and the server keeps it with that song for the household. The view appears only when there's a match. Radio and tracks without a duration get none. Tests run against a fake LRCLIB and cover each fallback, a miss and the offset.

- **Covers**: RED-50 (synced lyrics and the ± offset; no mini player)
- **Depends on**: Speaker
- **ADRs**: ADR-0008, ADR-0034
