# ADR-0024: Home HQ shows the family's Google calendars, view-only

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Question 6 asks which calendar provider holds the family's events and how events map to people. The family uses iCloud today, with Family Sharing's Family calendar. The oldest child has left the family plan, though, and Family Sharing caps a group at six, so it can't hold a Home HQ account as well. Every family member is a user in the family's Google Workspace, and Peter administers it. Some family members don't use a calendar at all, and those who do use different apps on their iPhones. The prototype's new-event sheet was broken and missing most of its fields (CAL-22..24). Research: "Reading Google calendars from the server" (`research/google-calendars`), and the iCloud research it replaces (`research/icloud-calendars`).

## Decision

- The family's calendars live in **Google Workspace**. Home HQ doesn't use iCloud. Google starts empty, with no import, and anyone who wants an iCloud event re-adds it.
- Home HQ is **view-only**. It never creates, edits or deletes an event. Owners make changes in their own apps. On phones, a "+" opens Google Calendar's new-event screen, and the event sheet has "Open in Google Calendar". The wall panel has neither.
- One shared **Family calendar** holds the events the family sees in full. A parent owns it and shares it "Make changes to events" with the other five, and "See all event details" with the Home HQ user.
- Each person's **primary calendar** shows only as **busy blocks**. The admin sets the domain's internal sharing default for primary calendars to "Only free/busy", and Home HQ reads them through `freebusy.query`.
- Home HQ reads Google as a dedicated **Home HQ Workspace user**, through an Internal OAuth app with `calendar.calendarlist.readonly`, `calendar.events.readonly` and `calendar.freebusy`. The client secret and refresh token are in a Docker secret (ADR-0009), never in SQLite (ADR-0008). Home HQ polls with sync tokens every 1-2 minutes and when a client opens the calendar.
- An event on the Family calendar is for the family members invited to it. With no family attendees, or with all six, it is an **everyone event**. A busy block that matches an event already in that person's lane is dropped.
- Busy blocks appear in the Today lanes, 3 Day and Week. They don't count for Next up, leave-by or Month's dots, and tapping one does nothing.
- The views are Today, 3 Day, Week and Month (CAL-03). Week is Sunday to Saturday, not a rolling 7 days, and Month starts on Sunday (ADR-0020). Repeating events show as their occurrences, and the event sheet says how the event repeats.
- Times show in the household time zone (ADR-0020). When an event's own zone differs, the sheet notes it ("10:00 AM MT"). All-day events stay on their date.
- **Leave-by** comes from a computed **drive time**. The drive time is for timed events with a physical location that isn't home, from Google's traffic-aware Routes API, called directly with Home HQ's own key in a Docker secret and cached per event. A single-person event measures from that person's location in the HA mirror, and falls back to home when the location is unknown, stale or HA is down (ADR-0004). Shared and everyone events measure from home.
- **Time to leave**: a Companion notification (ADR-0012) to the event's people 10 minutes before leave-by, unless they are already there. It's the only calendar notification Home HQ sends, and Google sends the event reminders.
- **Calendar subscriptions**: admins add read-only ICS feeds (FOS-14), which Home HQ fetches itself every 6 hours. Their events are everyone events.
- Each member has one color, kept in Home HQ, and everyone events are black. Google's calendar colors differ per viewer, so Home HQ doesn't use them.
- Out: feeds out (FOS-15), tags (FOS-16), editing in Home HQ (FOS-17, CAL-22..24), and Home HQ's own event reminders (FOS-13's reminder half).

## Alternatives Considered

### iCloud over CalDAV with a dedicated Apple Account

- **Pros**: the family's calendars are there today, and research had a plan.
- **Cons**: the app-specific password opens the whole account and never expires. Family Sharing's six-person cap and the oldest's departure break the Family calendar.
- **Why not**: everyone already has a Workspace account, so Google gives one admin-controlled domain instead.

### Create, edit and delete in Home HQ

- **Pros**: covers FOS-17 and the new-event sheet, and lets kiosk users add events.
- **Cons**: needs write scopes, conflict handling, recurrence edits and delete requests for events.
- **Why not**: owners already edit in their own apps, and a read-only token can't damage a calendar.

### One calendar per person plus Family

- **Pros**: a person's events need no attendees.
- **Why not**: more calendars for people who barely use one. Attendees are the normal Google habit within a domain.

### Primary calendars in full, or by opt-in

- **Why not**: private events would reach the wall panel. Busy blocks say who is free without saying why.

### Service account with domain-wide delegation

- **Pros**: no Workspace seat and no refresh token.
- **Cons**: its key can read all six calendars, private events included, and newer orgs block keys by default.
- **Why not**: a leaked Home HQ user token exposes only the Family calendar and busy times.

### Drive time from HA's Google Travel Time or Waze integration

- **Pros**: HA resolves person and zone origins itself, and Waze needs no key.
- **Cons**: an HA config change, and Waze relies on an unofficial library.
- **Why not**: Home HQ already has the locations from the HA mirror, and calling the API directly keeps caching under its control.

### Typed travel time in the event notes

- **Why not**: easy to forget, and nobody types it in Google Calendar.

### Rolling 7-day Week

- **Why not**: the family wanted a fixed week, the same as Google and Apple.

## Consequences

### Positive

- No write path, no edit conflicts, and no delete requests for events. A leaked token can only read.
- Family members keep the apps they already use, and Home HQ shows only what's on the Family calendar plus busy times.
- One admin controls access in the Workspace Admin console.
- Leave-by works without anyone typing a travel time.

### Negative

- A Workspace seat for the Home HQ user, and a Google Maps API key with billing enabled.
- ADR-0016's delete requests no longer cover calendar events, and ADR-0016's owners for events with no known creator are moot.
- A kiosk can't add an event, and quick add (FOS-41) can't create one unless a later decision adds a write path.
- FOS-13, FOS-17 and CAL-16 are narrowed or changed, and FOS-15, FOS-16 and CAL-22..24 are dropped.
- The iCloud research (`research/icloud-calendars`) no longer applies.

### Risks

- The refresh token dies after 6 months unused or on revocation. Mitigation: show "Google access lost" to admins on `invalid_grant`, and never retry it.
- Routes API costs go past the free tier. Mitigation: cache per event, recompute only as leave-by nears, and set a quota cap in Google Cloud.
- Free/busy may count declined or "Available" events, or duplicate invited events. Mitigation: check on real devices before building, as research listed. Matching drops duplicates.
- A person's location may be wrong (stale GPS). Mitigation: fall back to home, and show "from home" or "from your location" on the sheet.
