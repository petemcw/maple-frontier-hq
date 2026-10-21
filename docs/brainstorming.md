# Home HQ brainstorming

Reverse-engineered from the photo-frame dashboard prototype in `photo-frame-dashboard/dashboard/` (Vite, plain JS, mock data). This file records what the prototype does today, so we can decide what the real Home HQ keeps, changes, or drops. Section 16 adds candidate features from FamilyOS (`family-os/`), a self-hosted family organizer with a lot of what the prototype only mocks. Section 17 adds candidate features from other people's dashboards posted on Reddit.

## How this was gathered

- Read every file under `dashboard/js/`, the CSS that changes behavior, `index.html`, `vite.config.js`, `public/manifest.webmanifest`, and the Android kiosk app in `dashboard/kiosk/`.
- Ran `npm run dev` and drove every screen in Firefox at 1280x800: each rail tab, every calendar mode, the event, overlap and new-event sheets, chores (kid, adult and "Anyone" lanes, undo, routine tabs), shopping (search, add, quantity, check off, drawer, store filter, clear), photos (all five layouts, favorite, caption toggle), home control (tap, drag, hold-to-unlock, slide-to-open garage, scenes, summary sheets, thermostat, cameras, media), agents, the demo drawer, all four alerts, ambient, night, the night dashboard, the idle timers, and the `?diag` overlay.
- FamilyOS was not run. A background agent read its source, migrations and user guide and wrote a code-grounded inventory with file paths, saved as `family-os-inventory.md` next to this file. I checked a sample of its claims against the code (smart home out of scope, no chore stars, the three rotation modes, day/night following the device's dark-mode setting, Open-Meteo weather) and they held.
- Reddit threads were read from the original post and its gallery screenshots. Comments are read only when asked for, and then only the original poster's ("OP"). Features seen only in a screenshot are marked "(screenshot)", details that come from the OP's comments are marked "(OP comment)", and a purpose I inferred rather than read is marked "(inferred)". Every image from a thread, including ones the OP posted in comments, is saved under `docs/references/reddit/<post id>-<short name>/`.
- Each requirement has an ID (`SHELL-03`, `CAL-12`) so we can refer to it when we plan.
- "Mock" means the prototype fakes it. "Not wired" means the UI exists but does nothing. Section 13 lists defects found during testing.

## Contents

1. Product summary
2. Platform and non-functional requirements
3. App shell
4. Shared data model
5. Today (home overview)
6. Calendar
7. Chores
8. Shopping
9. Photos
10. Home control
11. Agents
12. Ambient, night and alerts
13. Prototype gaps and defects
14. Kiosk app (Android)
15. Data and integration needs by feature
16. Candidate features from FamilyOS
17. Candidate features from Reddit
18. Questions for the Home HQ plan

---

## 1. Product summary

Home HQ is a wall-mounted family dashboard for a 10.1" landscape photo frame (1280x800, Android WebView 106) in the kitchen. It is glanceable first and touch second. The family is two adults and two kids (ages 10 and 6), each with a fixed color. The screens are:

| Rail tab | View id | Purpose |
| --- | --- | --- |
| Today | `home` | One non-scrolling overview: clock, weather, presence, next event with leave-by countdown, each person's day, photo, chores, shopping and house status |
| Calendar | `calendar` | Family calendar. Today-by-person lanes, 3-day, rolling week, month |
| Chores | `tasks` | Per-person chore lanes for the current time-of-day routine, with star rewards for kids |
| Shopping | `shopping` | Shared grocery list with store filters, aisle-ordered categories, autocomplete and "usuals" |
| Photos | `photos` | Full-bleed family photo slideshow with layout modes |
| Home | `control` | Smart-home control: lights, blinds, locks, garage, scenes, climate, cameras, media |
| Agents | `agents` | AI plan usage meters (Claude, Codex, Cursor, Gemini, Copilot), display only |

When nobody touches it, the screen returns to Today, then turns into a photo frame (ambient mode) by day or a dim red clock (night mode) at night. Doorbell and water-leak events take over the whole screen. Garage and package notices appear as banners.

## 2. Platform and non-functional requirements

### Display and device

- **NFR-01.** The design canvas is a fixed 1280x800 landscape stage (`#stage`). On any other screen size the stage scales uniformly to fit (`fit()` in `app.js`), so desktop browsers show a letterboxed preview. The viewport meta is `width=1280, user-scalable=no`. _Superseded by ADR-0013: fluid layout, no fixed stage._
- **NFR-02.** The target engine is Android WebView / Chromium 106. JS builds to `chrome106`. CSS builds to `chrome61` on purpose, because that WebView rejects 4- and 8-digit hex colors and a newer target lets the minifier emit them (`vite.config.js`). The CSS avoids nesting, `color-mix()`, `oklch()` and `dvh` (`css/tokens.css`). _Superseded by ADR-0013: evergreen browsers only._
- **NFR-03.** Touch only. Minimum tap target is 64px (`--tap`). There is no hover-only UI.
- **NFR-04.** Today never scrolls. Calendar grids, the shopping list and the home-control rooms scroll inside their own panels.
- **NFR-05.** Units are Celsius and 12-hour time. Copy uses Canadian spelling and places ("Favourite", Toronto venues). The real system will need locale settings.

### Performance (measured on the frame's hardware)

- **NFR-06.** The rail switches tabs on `pointerdown`, not on click, to avoid tap delay. Keyboard activation still works.
- **NFR-07.** Each view is built once and kept laid out but hidden with `content-visibility`, so switching tabs only paints. A view rebuilds only when data changed (`data` event), the demo time changed (`time` event), or, for minute-sensitive views, when the minute rolled over.
- **NFR-08.** Hidden views are pre-built in the background 1.5 s after start and again 1.5 s after any data change, one view every 400 ms.
- **NFR-09.** Actions repaint only what they touched. A full home-control repaint rebuilds about 270 nodes and costs about 350 ms on the tablet. A full shopping repaint costs about 500 ms. The device notes record a toggle dropping from 366-448 ms to 160-178 ms after this change (`docs/biuframe.md`).
- **NFR-10.** The Today clock and next-up card update every minute. Weather and the per-person lanes refresh on the hour.

### Resilience and operations

- **NFR-11.** If the JS bundle has not started within 10 s (a stale page after a deploy, or a network blip), the page reloads itself (inline script in `index.html`).
- **NFR-12.** Ambient and night screens nudge their text by up to 4 px every minute to avoid image retention.
- **NFR-13.** Hosting is a static Vite build deployed to Cloudflare Workers static assets (`npm run deploy`, `wrangler.jsonc`). There is no backend, auth, or persistence. All state lives in memory and resets on reload. _Superseded by ADR-0002, ADR-0008, ADR-0009: Hono server, SQLite, Docker on the HA host._
- **NFR-14.** A PWA manifest makes it installable as a fullscreen landscape app named "Home HQ". _Kept; see ADR-0003._
- **NFR-15.** External runtime dependencies are Google Fonts (Archivo variable-width sans, Instrument Serif) and `picsum.photos` for every image (mock photos, camera snapshots, album art). A broken image gets a fallback style, and a slide shows after 2.5 s even if its image never loads.

### Diagnostics

- **NFR-16.** `?diag` turns on an on-screen overlay and remembers it in `localStorage`. `?diag=0` turns it off. The overlay shows the user agent, viewport, DPR, stage scale, fps, frames over 50 ms, DOM node count, image count and megapixels, JS heap, and for the last 7 tab switches the tap-to-click delay plus JS, layout, paint and total time.
- **NFR-17.** `&exp=` runs A/B experiments in the overlay: `font` (system fonts), `shadow` (no shadows), `img` (hide images), `has` (drop `:has()` and `[style*=]` rules), `scale` (no stage scaling), `noframe` (no frame timing).

### Visual design

- **NFR-18.** Warm paper-and-ink palette (`--paper #F2EEE6`, `--ink #111`), signal colors (alert red, amber, ok green, warm yellow, night red), serif display type for headlines and condensed sans for numbers and labels. Each person has a color, an RGB triple, and an ink color for text on that color. The family pseudo-person is black.
- **NFR-19.** Tone is friendly and editorial: "Nothing else scheduled. Enjoy the quiet.", "All done!", "Everyone crushed it."

## 3. App shell

### Navigation rail

- **SHELL-01.** A left rail with seven icon-and-label buttons (Today, Calendar, Chores, Shopping, Photos, Home, Agents) and a clock at the bottom (`h:mm` plus AM/PM). The active tab is inverted.
- **SHELL-02.** The URL hash holds the current view (`#calendar`). It updates with `replaceState`, so it adds no history entries. Loading with a hash deep-links to that view. An unknown view falls back to Today.
- **SHELL-03.** Pressing and holding the rail clock for 700 ms opens the demo drawer (SHELL-14).

### Idle behavior

- **SHELL-04.** Any `pointerdown` anywhere resets the idle timer.
- **SHELL-05.** After 60 s idle on any other view, close any open sheet and go to Today.
- **SHELL-06.** After 120 s idle, close sheets, go to Today, and enter ambient mode between 06:00 and 22:00 or night mode between 22:00 and 06:00.
- **SHELL-07.** While a full-screen takeover (doorbell, leak) is open, idle timers do nothing.
- **SHELL-08.** In the night dashboard (section 12), 20 s idle returns to the night clock.

### Feedback components

- **SHELL-09. Toasts.** One at a time at the bottom center. A new toast replaces the old one. Default life 5 s, with an optional icon, an error tone, and an optional Undo button that runs a callback.
- **SHELL-10. Sheets.** A modal card over a scrim. It closes from the X button, a scrim tap, Escape, idle, or a takeover. Variants are default, wide, event (colored header band) and camera (large image). Only one sheet is open at a time.
- **SHELL-11. Banners.** Stack at the top of the content area. Each has an icon, title, subtitle, an optional solid action button, a dismiss button with custom text, an amber or ink tone, and optional auto-dismiss.
- **SHELL-12. Takeovers.** Full-screen alert dialogs. Opening one closes any sheet and forces day mode.
- **SHELL-13. Press-and-hold.** A shared helper with a configurable duration and visible progress. Releasing early can show a hint ("Keep holding to unlock"). Uses: rail clock (700 ms), lock tiles (1 s), doorbell unlock (1 s), water shut-off (1.2 s).

### Demo controls (prototype only)

- **SHELL-14.** A right-side drawer titled "Demo controls":
  - Time of day: 7:10 AM, 3:18 PM, 7:45 PM, 11:40 PM, Real time. The demo clock is an offset from real time, so it keeps ticking. The app boots at 3:18 PM.
  - Modes: Ambient, Night, and "Idle: fast", which drops the idle timers to 8 s and 15 s.
  - Alerts: Doorbell, Water leak, Garage open, Package.
  - Calendar data: Normal day, Empty day (removes today's timed events), Overloaded (adds 8 extra events today).
- **SHELL-15.** Keyboard shortcuts: `D` toggles the demo drawer, `A` ambient, `N` night, `1` to `7` switch views, `Esc` closes the drawer and any sheet. Keys are ignored while typing in an input.
- **SHELL-16.** A tip toast on load says to hold the clock for demo controls.

## 4. Shared data model

All mock data is in `js/data.js` and dated relative to today, so the demo always looks current.

### People

- **DATA-01.** `person { id, name, color, rgb, ink, age?, home: bool, status: string, eta?: minutes }`. Four people: two adults (Wes, Claudia) and two kids (Buzzsaw 10, Skidmark 6). The internal ids (`alex`, `jordan`, `maya`, `otto`) differ from the display names.
- **DATA-02.** Two pseudo-people: `family` (black, star avatar, used when an event includes everyone) and `any` ("Anyone", for chores anyone can do).
- **DATA-03.** Avatars are colored circles with the first initial (a star for family). Away people get a hollow avatar style.

### Calendar events

- **DATA-04.** `event { id, title, people: personId[], start, end, allDay?, loc?, travel?: minutes, notes? }`. All-day events run midnight to midnight and can span several days.
- **DATA-05.** An event is a family event when its attendee count is at least the number of people. Its color is black. Otherwise it takes the first attendee's color.
- **DATA-06.** Leave-by time is `start - travel`. Events without `travel` have no leave-by.
- **DATA-07.** The mock repeats weekly background events from 40 days back to 45 days ahead (piano Mondays, soccer Tuesdays and Thursdays, and so on) so the month view looks full.

### Chores

- **DATA-08.** `routine { id, name, range label, from hour, to hour }`: Morning (label "6 - 11 AM", active 00:00-11:00), After school (11:00-17:00), Evening (label "5 - 10 PM", active 17:00-24:00).
- **DATA-09.** `chore { id, r: routineId, who: personId | 'any', title, emoji, stars: int, done: bool, doneBy?, overdue? }`.
- **DATA-10.** `stars { personId: balance }` exists only for kids. Having a balance is what makes someone a kid in the chore logic.
- **DATA-11.** `week { personId: [Mon..Sun] }` holds a 1/0 per day for "all chores done", kids only.

### Shopping

- **DATA-12.** `store { id, name }`: FreshCo (`grocer`) and Costco.
- **DATA-13.** `catOrder { all | storeId: category[] }` gives the aisle order per store. Categories are Produce, Bakery, Meat & Fish, Dairy & Eggs, Pantry, Frozen, Snacks, Household, plus "Other" for unknown items.
- **DATA-14.** `grocery { id, name, cat, qty, store, done, note }`.
- **DATA-15.** `catalog` is about 70 known `[name, category]` pairs used for autocomplete and auto-categorizing.
- **DATA-16.** `usuals` is 12 frequently bought item names.

### Home

- **DATA-17.** `room { id, name }`: Kitchen, Living Room, Entry + Garage, Upstairs, Backyard.
- **DATA-18.** `device { id, room, type, name, ... }` by type:
  - `light { on, level 0-100 }`
  - `blind { pos 0-100 }` (0 is closed)
  - `lock { locked }`
  - `garage { state: closed | opening | open | closing }`
  - optional `flaky: true` simulates an unreachable device
- **DATA-19.** `scene { id, name, icon, set: { deviceId: level } }`. A light at 0 turns off, above 0 turns on at that level. A blind value is its position.
- **DATA-20.** `climate { current, set, mode: heat | cool | auto | off, humidity, outside }`. One zone.
- **DATA-21.** `media { room, title, artist, playing, volume, art }`. One player.
- **DATA-22.** `camera { id, name, image }`: Front Porch, Driveway, Backyard.

### Weather

- **DATA-23.** `weather { hi, lo, feels, humidity, wind, t[24], c[24], p[24] }`: hourly temperature, condition (`sun`, `moon`, `partly`, `cloud`, `rain`) and precipitation chance, indexed by hour of today. `feels`, `humidity`, `wind` and `outside` are never shown.

### Photos

- **DATA-24.** `photo { src, o: 'l' | 'p' (orientation), date: YYYY-MM-DD, place, from: personId, tag (album or outing), fav? }`. Two photos are dated today's month and day in past years so "On this day" always has content. `from` is never shown.

### Agents

- **DATA-25.** `agent { id, name, plan, color, status: ok | degraded, incident?, meters[], tokens (M today), cost ($ today), week[7] (M tokens) }`.
- **DATA-26.** `meter { label, win (window length, min), used (%), left (min until reset), detail? }`. Windows are 5 h, 1 day, 1 week or 30 days.

## 5. Today (home overview)

Layout: clock top left, weather top center, presence top right; next-up card and four person lanes on the left; photo, chores, and a shopping plus house-status pair on the right.

### Clock

- **TODAY-01.** Large `h:mm` with AM/PM, and the weekday plus date below. Tapping it opens Calendar.

### Weather

- **TODAY-02.** Current condition icon, temperature, label (Sunny, Clear, Partly cloudy, Overcast, Rain), today's high and low.
- **TODAY-03.** A one-line summary plus advice, built from the hourly data. Rain means a precipitation chance of 50% or more.
  - Raining now: "Rain until {first dry hour}." with "Umbrella weather."
  - Rain later today: "Rain from {hour}." with "Pack a rain jacket for soccer." when a soccer event remains today, otherwise "Grab a rain jacket."
  - No rain and under 12 degrees: "Clear and cool." with "Jacket weather."
  - Otherwise: "Dry all day." with "Light layers."
- **TODAY-04.** A 7-hour strip: Now plus the next 6 hours, each with a label, icon and temperature. Icons sit higher for warmer hours, a small built-in temperature curve. Wet hours are highlighted.

### Presence

- **TODAY-05.** One row per person: avatar, name, and "Home", "ETA {n} min" (bold) or a free-text status ("At the office"). Away people get the hollow avatar. Display only.

### Next up

- **TODAY-06.** The next event is today's timed event, starting after now, with the earliest leave-by time. Sorting by leave-by means a far-away event can come before a nearer one that starts earlier.
- **TODAY-07.** The card takes the event's person color (black for family) and shows "Next up · {names joined with +}" or "Everyone", the title, time range and location, attendee dots for multi-person events, and a big countdown. The countdown is minutes when under 100 minutes away, otherwise the start time.
- **TODAY-08.** For events with travel time, a leave pill: "Leave in {duration}", styled as urgent at 15 min or less, and a pulsing red "Leave now" once leave-by has passed.
- **TODAY-09.** With nothing left today: "Rest of today. Nothing else scheduled. Enjoy the quiet." plus "Tomorrow starts with {title} at {time}". Tapping opens Calendar.
- **TODAY-10.** Tapping the card opens the event sheet (CAL-20).

### Person lanes

- **TODAY-11.** Four columns, one per person, with a color bar, avatar and name. Each shows that person's remaining timed events today (end after now), at most 3. An in-progress event shows "Now" instead of a start time.
- **TODAY-12.** Extra events show "+{n} more", which opens Calendar. An empty lane says "Free the rest of the day".
- **TODAY-13.** Family events appear in every lane. Tapping an event opens its sheet.

### Side tiles

- **TODAY-14. Photo.** A slideshow tile (cover layout, 12 s per photo, no on-photo captions) with a caption bar: "On this day · {n} years ago today" or "{n} years ago", and the place. Tapping opens Photos.
- **TODAY-15. Chores.** "Chores · {current routine}" and a progress ring per person plus Anyone, each showing done/total or a check when complete. People with no chores in the routine are skipped. Tapping opens Chores.
- **TODAY-16. Shopping.** Count of open items and the 3 most recently added names. Tapping opens Shopping.
- **TODAY-17. Home.** Lights-on count, security line ("All locked", "Garage open", or "{lock} unlocked" with a warning style), and inside temperature. Tapping opens Home control.

## 6. Calendar

### Header

- **CAL-01.** Previous and next arrows, a serif title, and a small label:
  - Today mode: "Monday 12" with "October 2026 · Today", or the relative day name.
  - 3 Day: "Mon 12 - Wed 14" with the month and year.
  - Week: "Oct 12 - Oct 18" with "Next 7 days".
  - Month: "October" with the year.
- **CAL-02.** A "Today" button that appears only when the selected date is not today (in month mode, when the month is not the current one). It resets to today and scrolls to now.
- **CAL-03.** A segmented control: Today, 3 Day, Week, Month.
- **CAL-04.** A round "+" button that should open the new-event sheet (CAL-22). It does not work. See DEF-01.
- **CAL-05.** Arrows step 1 day, 3 days, 7 days, or 1 month by mode. A horizontal swipe on the body does the same: more than 90 px across, less than 60 px vertical, under 700 ms, not starting on the header. It is the only swipe gesture in the app.

### Today mode (lanes by person)

- **CAL-06.** An all-day row above the grid. Single-person all-day events sit in that person's column, family ones span the full width. A multi-day event shows "Day {n} of {m}", with a left arrow when it started earlier.
- **CAL-07.** A time grid from 6 AM to 11 PM at 72 px per hour, with an hour gutter ("Noon" at 12) and one lane per person. The lanes have no person header. You tell them apart by event color and position only (DEF-11).
- **CAL-08.** Family events span all lanes, drawn as one wide block on an overlay.
- **CAL-09.** A shared event (two people, not everyone) appears in each attendee's lane, colored by its first attendee, with a dot for the other person.
- **CAL-10.** Event blocks show the title, time range and "Leave {time}" with a car icon when there is travel. Travel time is a hatched block of matching height above the event. Past events are faded, the current event is outlined, and events shorter than 45 min use a compact one-line layout.
- **CAL-11.** Overlaps inside a lane use at most 2 columns. When a cluster needs 3 or more, the first column stays and the rest collapse into a dashed "+{n} more" tile that opens an "Overlapping · {n} more events" sheet. Each row in it opens the event sheet.
- **CAL-12.** A red now line with the time. It moves every minute and only shows when today is on screen between 6 AM and 11 PM.
- **CAL-13.** On open, the grid scrolls so now sits about 170 px from the top. Scroll position survives re-renders and resets when you navigate.
- **CAL-14.** With no timed events: "Nothing on the calendar. A rare, quiet day."

### 3 Day

- **CAL-15.** Three day columns starting at the selected date. Each header has the date number, Today/Tomorrow/weekday and the short month, and today's header is inverted. Every person's events share each column, colored by person, with the same overlap rule. All-day events are bars across the columns they cover, with continuation arrows at either edge.

### Week

- **CAL-16.** A rolling 7 days from the selected date, not a fixed Mon-Sun week. Each column has a weekday and date header, all-day chips, then up to (7 minus the all-day count) timed events showing start time, a color dot and the title. Past events are faded.
- **CAL-17.** "+{n} more" or a tap on the column opens Today mode for that date. An empty day says "Free".

### Month

- **CAL-18.** A Monday-start 6x7 grid. Each cell has the date number and up to 5 colored dots (one per event, all-day first), then "+{n}". Days outside the month are dimmed and today is inverted. Tapping a day opens Today mode on that date.

### Event sheet

- **CAL-19.** Opens from any event in Today, Calendar or the overlap sheet.
- **CAL-20.** A colored band with "{Today | Tomorrow | Yesterday | weekday} · {Month} {d}", the title, and the time range or "All day". Below it:
  - Leave-by row: "Leave by {time}" and "{n} min drive in current traffic". The traffic part is static copy.
  - Location row with "Tap to send directions to phone" (not wired).
  - Attendee chips with avatars. A family event lists everyone.
  - Notes in quotes.
- **CAL-21.** There is no edit, delete, RSVP or recurrence display.

### New event sheet

- **CAL-22.** Reached only through the broken "+" button. Fields:
  - A large "What's happening?" title input plus idea chips that fill it: Soccer, Playdate, Dentist, Dinner out, Swim, Birthday party.
  - Who: a multi-select of the four people plus "Everyone", which toggles all four.
  - When: Today, Tomorrow, and the next 3 dated days.
  - Time presets: 8 AM, Noon, 3:30 PM (default), 4:30 PM, 6 PM, 7:30 PM.
  - Length: 30m, 1h (default), 2h.
  - Cancel and "Add to calendar".
- **CAL-23.** On save: an empty title becomes "New event", no attendees defaults to Claudia (`jordan`), the calendar jumps to that day (switching from Month to Today mode), and a toast "Added {title} · {day} {time}" offers Undo.
- **CAL-24.** You can't set location, travel time, notes, all-day, recurrence, a custom time or a custom length.

## 7. Chores

- **CHORE-01.** The header shows "Right now · {range}" (or just the range for a non-current routine), the routine name, a Morning / After school / Evening switcher, and "{done}/{total} done" with a progress bar.
- **CHORE-02.** The current routine comes from the clock (DATA-08). Picking another tab pins it until the demo time changes. Picking the current one unpins.
- **CHORE-03.** Five lanes: Wes, Claudia, Buzzsaw, Skidmark, Anyone. Each lane header has an avatar, a name, a ring with done/total (a check when complete, a dash when empty), and a subtitle:
  - kids: star balance
  - adults: "{done} of {total}" or "Nothing"
  - Anyone: "Up for grabs"
- **CHORE-04.** A chore row shows the emoji (which turns into a check when done), the title, and one of: "Overdue" in red with a red outline, the star value, "{stars} · {name}" when done by someone, or "by {name}". Rows sort open first, overdue first among open, then done (struck through).
- **CHORE-05.** Tapping an open chore plays a 350 ms check animation and marks it done. Stars go to the owner if they are a kid. The toast says "{title} done · +{n}★ {kid}" with Undo, which reverses completion and stars.
- **CHORE-06.** Tapping an open "Anyone" chore opens a "Who did it?" sheet with four large avatars. The pick is recorded as `doneBy`. Stars go to them only if they are a kid.
- **CHORE-07.** Tapping a done chore un-completes it straight away and takes back its stars. No confirmation.
- **CHORE-08.** A lane where everything is done shows "All done!" and a completed style. When the whole routine is done, the toast says "{routine} routine complete! Everyone crushed it." with no Undo.
- **CHORE-09.** Kid lanes end with a "This week" Mon-Sun strip: past days are hit or miss from `week`, today is outlined, future days are empty. The strip is static and does not respond to completions.
- **CHORE-10.** Missing: adding or editing chores, schedules, rotation, a daily reset, spending stars, and any history beyond the static week strip.

## 8. Shopping

- **SHOP-01.** The header shows "Shared list · Groceries", "{n} to get" for the current filter, and a store switcher (All, FreshCo, Costco), each with its open-item count.
- **SHOP-02.** The left list groups open items by category in the selected store's aisle order, with a category header and count. Each row has a large checkbox hit area, name, optional note, a "Costco" badge (All view only) and a quantity stepper. The minimum quantity is 1, and the stepper cannot remove an item.
- **SHOP-03.** Tapping an item marks it as "checking" for 1.2 s. A second tap within that window cancels. The toast "{name} checked off" offers Undo. The item then moves to the "Got it ({n})" drawer at the bottom of the list.
- **SHOP-04.** The drawer opens and closes on tap. Inside, each item has "Add back", and "Clear {n} checked items" deletes them all, with Undo.
- **SHOP-05.** The right panel has an "Add an item..." input. With text typed, it shows up to 5 catalog suggestions (prefix matches first, then substring matches). Each shows "On list · ×{qty}" or its category. Below them is an `Add "{text}"` row with a guessed category, unless the text exactly matches a catalog item. Enter adds the typed text.
- **SHOP-06.** With no text, it shows "Usuals · tap to add", a 3-column grid of the 12 usuals. Items already on the list are inverted with "On list ×{n}". The others show their category.
- **SHOP-07.** Add rules:
  - Already on the list: quantity goes up by one, toast "Already on the list · now ×{n} {name}".
  - Checked off earlier: it comes back with quantity 1, toast "Added back {name}".
  - New: the catalog spelling if known, otherwise the typed text capitalized. The category comes from an exact catalog match, then a substring match, then any word over 3 letters, otherwise "Other". The store is the filtered store, or FreshCo when viewing All. Toast "Added {name} to {category}".
  - Every add clears the input, flashes the row and scrolls it into view.
- **SHOP-08.** Missing: delete, edit name, note or store, choosing a store when adding, who added an item, multiple lists, and sorting other than by category.

## 9. Photos

- **PHOTO-01.** A full-bleed black stage, a glass segmented control at top right (Auto, Cover, Pair, Collage, Polaroid), and bottom controls: previous, Favorite, next.
- **PHOTO-02.** Three invisible tap zones: left 22% goes back, center 56% toggles captions to full opacity (they sit at 70% by default), right 22% goes forward.
- **PHOTO-03.** Slides advance every 15 s here, 12 s on the Today tile and 14 s in ambient mode. Manual next or previous restarts the timer. Previous walks back through history.
- **PHOTO-04.** Queue: each cycle shuffles all photos, adds favorites a second time, and puts "on this day" photos first.
- **PHOTO-05.** Layouts:
  - Cover: a landscape photo full-bleed with a slow random Ken Burns pan, or a portrait photo fitted over a blurred copy of itself.
  - Pair: two portraits side by side, preferring the same tag (outing).
  - Collage: three photos.
  - Polaroid: one photo as a tilted print with a handwritten-style caption "{place}, {Month Year}" on a blurred background.
- **PHOTO-06.** Auto picks per slide. A portrait gets Pair if another photo shares its tag, else Polaroid. Landscapes are mostly Cover, with every 5th slide Collage (slot 3) or Polaroid (slot 4).
- **PHOTO-07.** Captions read "On this day · {n} years ago today · {place}" (or "{n} years ago" or "This year") above a big serif date. Multi-photo slides show the month and year.
- **PHOTO-08.** Favorite toggles the current slide's first photo. Toast "Added to favourites · it'll show up more often" or "Removed from favourites". Not persisted.
- **PHOTO-09.** The slideshow pauses when Photos is hidden and resumes on return.
- **PHOTO-10.** Missing: a real library or albums, uploads, contributor display, hiding a photo, and video.

## 10. Home control

Layout: the left column has summary cards, scenes and scrolling rooms. The right column has climate, cameras and media.

### Summary cards

- **HOME-01.** Four cards:
  - Lights: count on and "on in {n} rooms", warm tint when any are on.
  - Security: "Locked" and "All doors + garage", or a count with names and a warning tint.
  - Climate: current temperature and "Heating / Cooling / Holding to {set}" or "System off".
  - Media: "Playing" or "Paused" and "{room} speaker".
- **HOME-02.** Lights opens a wide sheet of every light (on first) as live tiles, with "Turn everything off" (Undo restores the same lights). Security opens doors plus garage tiles with "Lock all doors". Climate and Media pulse their panel on the right.

### Scenes

- **HOME-03.** Five scene buttons: Morning, Dinner, Movie, Away, Goodnight. A scene applies its levels to lights and blinds. Away and Goodnight also lock every lock.
- **HOME-04.** The last applied scene stays highlighted until any manual device change. The toast "{scene} scene on" offers Undo, which restores a full snapshot of every device and the previous active scene.

### Rooms and tiles

- **HOME-05.** One section per room with a serif name and "{n} on" or "All off". Tiles sit in a 4-column grid.
- **HOME-06. Light tile.** Tap toggles. Turning on restores the last level, or 100%. A horizontal drag of more than 10 px makes the whole tile a dimmer: the fill and label follow the finger, 0 means off, release commits. Shows "{level}%" or "Off".
- **HOME-07. Blind tile.** Tap toggles fully open or closed. Drag sets the position. Labels are "Closed", "Open" or "{n}% open".
- **HOME-08. Lock tile.** Locked: hold 1 s to unlock, with a progress ring and the hint "Hold to unlock". A short tap shows "Hold for a second to unlock {name}". Unlocked: warning style, "Tap to lock". Unlock toasts "{name} unlocked".
- **HOME-09. Garage tile.** Shows Closed, Opening..., Open or Closing..., with a warning style when not closed. Tap opens a confirm sheet ("Open the garage?" with "Make sure the driveway is clear.", or "Close the garage?" with "The door will stop and reverse if anything is in the way."). Dragging a knob all the way commits. The door moves for 4 s, then toasts "Garage open" or "Garage closed". Letting go early snaps back.
- **HOME-10. Optimistic updates.** Every change applies at once and shows a pending dot. After 600 ms it confirms. A device marked unreachable (the backyard String Lights) reverts after 2.6 s with an error toast "Couldn't reach {name} · it may be offline".

### Climate

- **HOME-11.** A dark "Climate · Main floor" panel: a large setpoint with minus and plus buttons in 0.5 degree steps, clamped to 15-28. Pressing either while Off switches to Heat. It shows current temperature, a verb badge (Heating, Cooling, Holding, Off), humidity, and a Heat / Cool / Auto / Off switch.

### Cameras

- **HOME-12.** "Cameras · snapshots every 20s" with two tiles (Front Porch, Driveway), each showing a name and "{n}s ago". The age counts 1 to 20 and wraps, a fake refresh. Tapping opens a large snapshot sheet. The Backyard camera exists in data but is never shown.

### Media

- **HOME-13.** A now-playing card: album art, room, title, artist, play/pause. No volume, skip, source or room selection.

## 11. Agents

- **AGENT-01.** The header shows "AI plans · usage limits", "Agents", "Updated {n} min ago" and a refresh button. Refresh only resets the timestamp.
- **AGENT-02.** One card per provider: color swatch, name, plan name, and status "Operational" or "Degraded". An incident line shows when degraded. Otherwise "Near the session limit" shows when any meter is at 85% or more.
- **AGENT-03.** Each meter shows its label, percent used, a bar with an "even pace" marker, optional detail ("$45.60 of $60", "320 of 2,000"), "Resets in {d h | h m | m}", and a pace verdict. Pace is the share of the window already elapsed. Within 3 points of it is "On pace", otherwise "{n}% over pace" (highlighted) or "{n}% under pace". A meter at 85% or more is "hot".
- **AGENT-04.** Each card footer shows tokens today ("3.1M"), API cost today or "Incl. in plan", and a 7-day token sparkline.
- **AGENT-05.** A dark summary card: "All agents · Last 7 days", total tokens, API cost today, a stacked daily bar chart by provider (today highlighted), and a legend.
- **AGENT-06.** Every minute the countdowns, pace markers and pace text update in place without a rebuild.

## 12. Ambient, night and alerts

### Ambient (photo frame)

- **AMB-01.** A full-screen slideshow (cover layout, 14 s, no captions) under a dark gradient. Bottom left: a huge clock and the date. Bottom right: weather icon, temperature and summary, a next-event pill with a person-color dot ("{title} · {time}"), and presence avatars.
- **AMB-02.** It refreshes every minute and shifts its text up to 4 px every minute.
- **AMB-03.** Any tap returns to day mode on Today.

### Night

- **NIGHT-01.** Black screen with dim red text: a big clock, a line with the weather icon and temperature plus "First up tomorrow: {title} {time} · {person}" ("tomorrow" becomes "today" before 6 AM, or "Nothing early tomorrow"), and "House locked · alarm armed". That last line is hardcoded and stays even when the garage is open (DEF-05).
- **NIGHT-02.** A tap switches to the night dashboard: the full UI under a red-amber filter (invert, sepia, saturate, dim). It works normally and returns to the night clock after 20 s idle.

### Takeovers

- **ALERT-01. Doorbell.** A full-screen live camera image with "LIVE · Front Porch", "Doorbell · {time}", "Someone's at the front door", and three buttons:
  - Talk toggles to "Talking..." (UI only, no audio).
  - "Hold to unlock" (1 s) unlocks the front door. Its toast promises "relocks in 2 min", but nothing relocks (DEF-06).
  - Dismiss.
  - It auto-dismisses after 45 s.
- **ALERT-02. Water leak.** A full-screen red critical alert: "Critical alert · {time}", "WATER LEAK", "Laundry room sensor detected water on the floor." "Hold to shut off water" (1.2 s) only changes its label to "Water main closed". Acknowledge closes it with the toast "Leak acknowledged by the kitchen display", which suggests acknowledgment syncs across displays. No auto-dismiss.

### Banners

- **ALERT-03. Garage left open.** Amber banner: "Garage open for 15 min" and "No one is in the garage · Wes is 12 min away" (static copy). "Close garage" opens the slide-to-close sheet. "Leave open" dismisses. Triggering it also sets the garage to open.
- **ALERT-04. Package delivered.** Ink banner: "Package delivered" and "Front porch · {time}". "View camera" opens the porch snapshot. "OK" dismisses. Auto-dismisses after 9 s.

## 13. Prototype gaps and defects

Found while testing. Most are expected in a mock-data prototype, but each one marks something the real system must decide or build.

### Defects

- **DEF-01. The calendar "+" button never opens the new-event sheet.** `#stage` carries `data-mode="day"`, and the calendar click handler checks `closest('[data-mode]')` before `[data-add]`. A tap on the "+" button, or on any empty part of the calendar, matches the stage and switches the calendar to Today mode. Confirmed in the browser: from Week mode, tapping "+" switched to Today and opened nothing. The sheet works when that attribute is out of the way.
- **DEF-02.** The security summary reads "Garage open open" when only the garage is open.
- **DEF-03.** "Lock all doors" does not close an open garage, though the sheet is titled "Doors + garage".
- **DEF-04.** The climate verb compares setpoint to current temperature and ignores the mode. In Cool with the setpoint above the room temperature, it still says "Heating".
- **DEF-05.** The night screen always says "House locked · alarm armed". There is no alarm in the data model.
- **DEF-06.** Doorbell unlock promises a relock in 2 minutes, but no relock exists.
- **DEF-07.** The hourly weather strip wraps past midnight using today's early-morning hours instead of tomorrow's forecast.
- **DEF-08.** The Morning routine label says 6-11 AM but the routine is active from midnight.
- **DEF-09.** The mock week data assumes a particular weekday. On a Monday, "Sunday brunch" lands on Friday.
- **DEF-10.** A new event with no attendee defaults to Claudia without saying so.
- **DEF-11.** Today mode in the calendar has no person headers on its lanes. CSS for a header row exists (`.lanes-head`) but the day view never renders it.

### Not wired (UI only)

- "Tap to send directions to phone" on the event location.
- Doorbell Talk.
- Water shut-off.
- Agents refresh.
- The camera "seconds ago" counter.
- "{n} min drive in current traffic".
- Static alert copy ("Garage open for 15 min", "No one is in the garage", "Wes is 12 min away").

### Missing capabilities

- **Persistence.** Nothing survives a reload.
- **Sync.** Nothing is shared between displays or phones, though the leak toast implies it.
- **Auth and profiles.** No accounts, settings or per-device identity.
- **Editing.** No event edit or delete, chore management, or shopping item edit or delete.
- **Recurrence.** No recurring events or chores.
- **Rewards.** No star redemption, chore streaks, or history.
- **Photos.** No photo source or upload.
- **Offline.** No offline handling beyond the boot-time reload.
- **Locale.** No settings for units, language, or the 12/24-hour clock.

## 14. Kiosk app (Android)

The `com.wesbos.homehq` app in `dashboard/kiosk/` builds without Gradle (`build.sh`, `build.ps1`).

- **KIOSK-01.** A full-screen immersive WebView: hidden status and nav bars, screen kept on, landscape, JavaScript and DOM storage on, media autoplay without a user gesture.
- **KIOSK-02.** It loads `DEFAULT_URL`, or a URL passed in the launch intent. A new intent with a URL reloads.
- **KIOSK-03.** A failed main-frame load retries every 5 s, which covers booting before Wi-Fi is up.
- **KIOSK-04.** Holding two fingers still (within 60 px) for 1.5 s opens a native menu: "Open launcher" (another Home app, else Android Settings), "Android settings", "Reload dashboard". All touches still reach the page.
- **KIOSK-05.** Back steps back in WebView history and never exits.
- **KIOSK-06.** WebView remote debugging is on only when launched with `--ez debug true`. The next normal launch turns it off, because every app on a rooted frame has root.
- **KIOSK-07.** An optional Accessibility service ("Home HQ navigation") adds floating Back and Dashboard buttons outside the kiosk, for frames without a nav bar. They are draggable, remember their position, and hide while the kiosk is in front. The service requests no accessibility events.
- **KIOSK-08.** The app is not the default Home app and does not launch at boot.

## 15. Data and integration needs by feature

What each feature reads and writes once the mock data goes away. Sources are left open for the plan.

| Feature | Reads | Writes | Notes |
| --- | --- | --- | --- |
| Presence (Today, ambient) | Per person: home or away, a status line, ETA when heading home | None | ETA needs location plus routing |
| Weather (Today, ambient, night) | Current conditions, hourly temperature, condition and precipitation chance for 7+ hours across midnight, daily high and low | None | The rain advice also reads today's events |
| Next up, lanes, calendar | Events for every person: title, attendees, start and end, all-day and multi-day, location, notes, travel time | Create event (edit and delete missing) | Travel time "in current traffic" needs a routing source per event. Attendees must map to household members |
| Chores | Chore definitions by routine and person, done state, who did it, overdue, star values and balances, weekly completion | Complete, undo, record who did it, adjust stars | Needs a daily reset and a definition of "overdue" |
| Shopping | Items with quantity, note, category and store; catalog; usuals; per-store aisle order | Add, change quantity, check, revive, clear | Must be shared live with phones in the store |
| Photos | A library with date, place, orientation, album or tag, contributor; favorites | Favorite | "On this day" needs reliable capture dates |
| Home control | Lights (on, brightness), covers (position), locks, garage door state and movement, one thermostat (current, set, mode, humidity), one media player, camera snapshots, scene definitions | Turn on and off, brightness, cover position, lock and unlock, open and close garage, setpoint and mode, play and pause, run scene | Needs device reachability for HOME-10 and confirmed state after optimistic updates. Overlaps the existing `ha-dashboard` HA work |
| Home status (Today, night) | Lights-on count, open locks and doors, inside temperature, alarm state | None | The night line needs a real alarm or security state |
| Doorbell | Ring event, live camera, two-way audio | Unlock with auto-relock | Must wake the screen and override ambient or night |
| Water leak | Leak sensor and location, water valve | Close the valve, acknowledge across displays | Critical alert |
| Garage left open | How long the garage has been open, whether anyone is in the garage, nearest person's ETA | Close garage | |
| Package | Delivery detection and camera | None | |
| Agents | Usage per AI provider: meters, reset times, tokens, cost, status and incidents | None | CodexBar-style polling of each provider |

## 16. Candidate features from FamilyOS

FamilyOS is a self-hosted family organizer: one Docker image, a Hono API on Node 24, SQLite through Kysely, and a SvelteKit PWA for phones, desktops and paired wall displays. It is AGPL-3.0 and marked beta, with no compatibility promise before 1.0. Every feature is a first-party extension compiled into the build. Only language and theme packs install at runtime. The full inventory, with file paths, is in `family-os-inventory.md`. Paths below are relative to `family-os/`.

### Principles that force a decision

FamilyOS commits to five rules. Each is a choice Home HQ has to make on purpose.

1. **AI proposes, people decide.** No model changes data by itself. AI changes, and changes by "Supervised" members (kids), become drafts that an adult approves. Every approval stores an inverse action, so it can be undone.
2. **Access is capabilities, not role names.** Each feature declares `view`, `use`, `edit` and `manage` capabilities, and roles (Admin, Member, Supervised, Guest, custom) map to them. Someone without edit rights can still suggest a change ("propose instead of refuse").
3. **Wall displays are view-only devices.** A display is paired by code, holds only the read grants an admin picks, never gets PIN sign-in, and cannot run actions. By default it hides names and colors.
4. **Region and language are data.** The code is English only. Languages and palettes are installable packs, and date formats follow the family's region separately from each person's language.
5. **Smart-home control is out of scope** because Home Assistant already does it (`docs/architecture.md`). Its `home` extension is household records, not devices.

Rule 3 conflicts with the prototype head-on. The prototype's wall screen is the main control surface: it ticks chores, edits the grocery list, unlocks doors and opens the garage.

### Overlap with the prototype

| Prototype area | FamilyOS equivalent | Difference |
| --- | --- | --- |
| Today | Today page in three styles (Broadsheet, Lanes, Wall) | Adds an evening flip to tomorrow, a one-line status per person, a "Needs you" band, and a "family or mine" next-up toggle |
| Calendar | Day, Week and Month boards on tablets. Week strip and month on phones | Adds repeats, reminders, tags, ICS in and out, edit and delete. Phones have no Day view and no person filter |
| Chores | Rotating chores with due dates | No stars, rewards or time-of-day routines |
| Shopping | Several named lists | No per-store aisle order and no usuals. Adds a one-line multi-item parser, category memory and offline taps |
| Photos | Uploaded photos, a deterministic photo of the day, ambient slideshow | No albums, captions, "on this day" or layout modes |
| Home control | None | Out of scope, left to Home Assistant |
| Agents | Admin-only AI usage page (calls, tokens, USD per month, budgets) | Covers FamilyOS's own AI calls, not your personal AI subscriptions |
| Ambient and night | Idle slideshow. Dark mode follows the device's `prefers-color-scheme` | No night clock, dimming, pixel shift, wake lock or fullscreen. The docs say "follows the sun". The code does not |
| Takeovers and banners | None | The only display interrupt is a pill with the count of waiting suggestions |

### Today and wall displays

- **FOS-01. Evening flip.** From 20:00, or when the last timed item ends if that is later, the hero goes away, Today shows tomorrow, and a "done today" line sums up the day.
- **FOS-02. Status per person.** One clause each: "Piano lesson in 25 min", "Piano lesson until 16:30", "1 chore left", "Nothing planned".
- **FOS-03. Next-up scope.** Each person picks whether the hero shows the family's next thing or their own. The countdown shows minutes under an hour, a short duration under two hours, then the start time, and "now, until..." once started.
- **FOS-04. Weather sentence tied to events.** It names the event a rain window overlaps ("Rain from 15:00, during Football"), using fixed templates and no LLM. The prototype's version only knows about soccer (TODAY-03).
- **FOS-05. Needs you.** Cards for waiting suggestions, overdue tasks and overdue records, with a one-tap "Tomorrow" postpone for tasks. Shown to people, never on displays.
- **FOS-06. Next days.** On a light day (fewer than 3 items), Today adds up to 4 items from the next 6 days.
- **FOS-07. Never-scroll fit planner.** When content does not fit, it drops in a fixed order: fewer items per person, then widget items, then whole blocks (photo, weather hours, extras, chores), then zoom down to a floor. Ended items go first. It measures offscreen so nothing visibly reflows.
- **FOS-08. Photo of the day.** One photo per family day, the same on every screen, never repeating on consecutive days, and every photo shown once per round.
- **FOS-09. Hide blocks.** Any Today block can be hidden from its own menu (with Undo) or in settings, per person and per display.
- **FOS-10. Display pairing.** The screen shows an 8-character code and a QR code. An admin confirms it on a phone and picks what the screen may show. The token is single-use, stored hashed, and kept in an httpOnly cookie. Five wrong codes end every open pairing. Revoking a screen takes effect on its next request.
- **FOS-11. Per-display grants and look.** Each screen has its own read grants (calendar, chores, lists, meals, weather, photos, board, records, waiting count), its own style, palette and hidden blocks, and a "show names and colors" switch that is off by default.
- **FOS-12. Display upkeep.** Realtime events carry ids only. A 60 s self-check and a full re-read every 5 minutes back them up. The service worker checks for a new build every 15 min and reloads once. An offline display keeps the last data and shows "Cannot reach FamilyOS. Trying again..." in a corner.

### Calendar

- **FOS-13. Repeats and reminders.** Repeats are daily, weekly on chosen days, or monthly, with an end date. Each event has one reminder, sent to its members or everyone, through quiet hours. All-day reminders fire at 09:00, or 21:00 the evening before.
- **FOS-14. Subscriptions in.** Read-only ICS feeds (school, clubs, holidays, Google or iCloud secret addresses), refreshed every 6 hours, each with a color, a member, and a reminder switch.
- **FOS-15. Feeds out.** A secret ICS feed per person ("my events plus family events") for the phone's own calendar app. Tokens are hashed and shown once, labeled per device, show "last used", and can be revoked.
- **FOS-16. Tags.** Event categories with family-wide colors, rename, merge and delete, a per-device tag filter, and round-trip through ICS `CATEGORIES`.
- **FOS-17. Editing.** Event detail, edit and delete. Tapping empty grid space adds an event at that time, snapped to 30 minutes. Overnight events work (an end before the start means the next day). Events of 6 hours or more draw hollow behind the others, which suits work shifts.
- **FOS-18. Overlap layout.** Columns grow into free space to the right, with 2 to 6 columns depending on width before collapsing into "+N more". The prototype stops at 2 (CAL-11).

### Tasks (new area)

- **FOS-19. Family task list, separate from chores.** One-off or repeating to-dos with an assignee or "Anyone", a due date and time, and a reminder at the due time. Tasks group into Overdue, Today, This week, Later and No date, and timed tasks appear on the Today timeline. Done tasks stay visible for 7 days.

### Shopping

- **FOS-20. Several named lists** (groceries, pharmacy, hardware) that the AI can address by name.
- **FOS-21. One-line multi-item add.** "milk, 2 kg rice, 1,5 l juice" splits on commas, semicolons and newlines and parses amounts, fractions, ranges and units.
- **FOS-22. Merge on add.** Adding something already on the list merges into it, a bought item comes back, and Undo restores the merged rows exactly. The prototype's add rules (SHOP-07) are close but only bump the quantity.
- **FOS-23. Category memory.** The section chosen for a product is remembered for the whole family.
- **FOS-24. In the cart.** Ticked items stay visible in a cart column until someone clears it, and drop out of view after 7 days anyway.
- **FOS-25. Offline taps in the store.** Taps queue in IndexedDB with idempotency keys, show "Waiting to send", and replay safely. Tasks and chores work the same way.
- **FOS-26. Edit and delete items,** with Undo on delete.

### Chores

- **FOS-27. Schedules.** Once, daily, weekly on chosen days, or monthly, with an end date. Missed occurrences are skipped instead of piling up after a holiday, "the 31st" clamps to the month's end, and finishing early keeps the schedule.
- **FOS-28. Rotation.** Fixed person, take turns, or whoever did it least in the last 90 days, with an ordered rota. "Anyone may do it" credits whoever taps it.
- **FOS-29. Turn rules.** A kid can tick only their own turn or an anyone chore, and the disabled button says "{name}'s turn". An adult ticking a kid's turn credits the kid, so the rota stays fair.
- **FOS-30. Completion and undo.** One live completion per occurrence (safe for offline replays). Undo works only for your own completion, on the same day, and only the latest one, and it restores the previous turn.
- **FOS-31. Reminder** on the due day to whoever is up, sent once.
- **FOS-32. Chore management.** Add, edit, archive. Kids can suggest a chore for approval.

FamilyOS has no stars or rewards, which the prototype has (CHORE-05). Rotation and stars can coexist: the prototype's `doneBy` already decides who earns.

### Kitchen (new area)

- **FOS-33. Recipe box.** Photo, servings, time, labels, source link, an ingredient checklist and numbered steps. The AI can turn a photo of a cookbook page or a message into a draft recipe.
- **FOS-34. Weekly meal plan.** Breakfast, lunch, dinner and other slots, several meals per slot, a saved recipe or a free-text dish, and a warning for duplicates.
- **FOS-35. Ingredients to the shopping list.** A line parser handles fractions, ranges and preparation notes. It never overwrites what someone already put on the list, and it reports differences ("you have 500 g, the recipe needs 600 g").
- **FOS-36. Today's meals** on Today and the display, placed at nominal hours and never shown as "next up".

FamilyOS leaves out a cook mode (large steps, timers, keep-awake). That would suit a kitchen wall screen.

### Household records (new area)

- **FOS-37. Vehicles, pets and the home** as records with recurring duties (insurance, inspection, vaccinations, warranty end, seasonal upkeep). Each duty has a due date, a repeat in months or years, a reminder some days before (default 14), an assignee, a completion history, and presets ("Usually also: rabies, check-up").
- **FOS-38. Papers.** Files attached to a record with a label and a "valid until" date. The pill turns amber within 30 days and red once expired. Papers can be filed without the AI reading them.
- **FOS-39. Per-domain extras.** Home: appliances by room with brand, model, serial and warranty dates, plus utilities and subscriptions. Pets: treatment history that completes a duty in the same transaction. Vehicles: a service log.

### Family board (new area)

- **FOS-40. Short notes.** Up to 500 characters, an optional photo, a paper color, and an end day (until taken down, today, tomorrow, end of week, a chosen day). Notes expire by themselves. Admins pin. The family drags to reorder. "Seen by" receipts replace reactions. Notes show on Today and on displays, and posting notifies everyone except the author.

### Capture and AI

- **FOS-41. Quick add.** One inbox for text, photos, PDFs and voice notes from the app, the phone share sheet (PWA share target, HTTPS required), a Telegram bot or a forwarding mailbox. The AI turns each capture into suggestion cards across calendar, tasks, shopping, chores, meals, board and records.
- **FOS-42. Tick and add.** Cards start ticked. "Add selected (n)" approves them with one Undo for the batch. Uncertain fields say "Check this", possible duplicates need a second tap, and date-bound suggestions expire after their day.
- **FOS-43. Scoped auto-approval** for routine kinds such as shopping items and tasks. It approves as the admin who turned it on and switches off if that admin loses the right.
- **FOS-44. Assistant chat.** Plain-language questions and changes ("what's on this week?", "put milk on the list"). Every change it proposes is an approvable card with Undo. It keeps private memory per person and is reachable from Telegram.
- **FOS-45. Family search by meaning** across captures, recipes, home items, events and records, limited to what each person may see.
- **FOS-46. Bring your own agent.** An MCP server with personal app keys (read, or read and suggest), which expire and can be revoked. An agent never sees more than its member.
- **FOS-47. AI providers and limits.** One OpenAI-compatible client for OpenRouter, Ollama or any compatible server, with no native Anthropic or Google client. Each feature area can be set to Off, Local only (checked against the home network) or Any provider. A family-wide monthly USD budget or call cap pauses all AI when reached, and an admin page shows usage per month, feature and model.

### People, access and notifications

- **FOS-48. Roles.** Admin, Member, Supervised and Guest, plus custom roles defined as a base role with overrides.
- **FOS-49. Sign-in.** PIN sign-in only on devices an admin marks as family devices, with lockout per account. Every request re-checks device and role, so revoking a lost tablet takes effect immediately.
- **FOS-50. Notifications.** In-app, web push (its own VAPID and encryption) and Telegram with approve and reject buttons. Quiet hours per person delay pushes instead of dropping them. Reading a notice in the app cancels its push. Approval prompts clean themselves up once a draft is decided.

### Platform

- **FOS-51. Live sync** over a websocket, so lists and chores update on every device.
- **FOS-52. Backups.** Nightly `VACUUM INTO` and one before each upgrade, optionally age-encrypted and copied to S3 or WebDAV.
- **FOS-53. Languages and region** as packs, with region formats separate from each person's language.
- **FOS-54. Looks.** Today has three styles. Palettes are data packs with a WCAG contrast check. The look resolves display, then person, then family, then default, and applies before first paint so wall tablets never flash the default.

### What FamilyOS lacks

These prototype features have no FamilyOS equivalent and would be ours to build or keep:

- Star rewards and time-of-day routines (CHORE-01 to CHORE-09).
- Smart-home control, cameras, scenes and climate (section 10).
- Night clock, night dashboard, pixel shift, wake lock, fullscreen (AMB, NIGHT, NFR-12).
- Doorbell and leak takeovers, garage and package banners (ALERT-01 to ALERT-04).
- Photo layouts, captions, "on this day", favorites and albums (section 9).
- Per-store aisle order and usuals (SHOP-02, SHOP-06).
- Usage meters for personal AI subscriptions (section 11).
- Interaction on the wall screen itself. FamilyOS displays are read-only by design.
- Tuning for an old Chromium 106 frame. FamilyOS targets current browsers (it uses `dvh`, for example), so its UI is unlikely to run well on the frame as is.

### FamilyOS defects worth knowing

From the inventory's docs-versus-code check:

- Saving a repeating series from the UI brings back dates that were skipped.
- The monthly AI call cap counts failed calls and MCP tool calls, so an active agent can use up the family's budget. The USD budget never trips for providers that do not report cost (direct OpenAI, Groq).
- A role once held by a removed member can never be deleted. A demoted adult keeps a working password.
- Signing out on a shared tablet leaves push notifications subscribed.
- A display that starts while the server is down seems never to retry. A cold start offline cannot render Today.
- The 52 px display touch target in the design docs is never applied.

## 17. Candidate features from Reddit

Features pulled from dashboards other people have posted. Each thread gets its own subsection with IDs that carry on from the last one (`RED-01`, `RED-02`...). Where a feature already exists in this doc, the entry points to that ID and notes only what is different.

### Kitchen tablet with a weather frog

- **Source:** [r/homeassistant, "I finally built a kitchen tablet dashboard that actually helps us run the house (and a weather frog with 161 scenes)"](https://www.reddit.com/r/homeassistant/comments/1x5mx00/), posted 2026-10-14, read 2026-10-18. Post text plus 9 screenshots, and the OP's 18 replies in the comments, including one more photo of the tablet. 70 of the 76 comments loaded. The other 6 are probably deleted.
- **Screenshots:** `references/reddit/1x5mx00-kitchen-frog/` (all 10 images: full dashboard, frog scenes, the banner cards, the kid's room page, and the OP's photo of the tablet).
- **Design source (OP comment):** the look was inspired by [another r/homeassistant dashboard](https://www.reddit.com/r/homeassistant/comments/1wuhxk0/finally_have_a_workable_dashboard_that_i_actually/), which the OP asked Claude to recreate. It is a possible next thread to read.
- **Household:** two adults and a 3-year-old in Bucharest. The design is built around the family's day (kindergarten run, dinner, laundry, weekend plans), not around showing every entity.
- **Hardware:** Samsung Galaxy Tab A9+ in the kitchen, always charging (the photo shows it in landscape, in a USB-C dock on a shelf), running the HA companion app as its only screen. A Sonoff SNZB-06P mmWave presence sensor in the kitchen. A Google Nest Hub in the living room casting the same dashboard. Google/Nest speakers around the house. An iDotMatrix 16x16 pixel panel used as a clock.
- **Software:** Home Assistant 2026.10 in Docker on Debian. ZHA for Zigbee, plus Tuya Local, Govee LAN, Tapo, Sonoff, SmartThings and Plex. About 1,460 entities and 130 automations. The dashboard is one custom card in vanilla JS (about 5,300 lines, no build step) plus a small panel card for the other views. The author says Claude helped write it.
- **Layout (screenshot):** a wide illustrated banner across the top, with two columns of cards below it. The left column has Lights, TV & remote, Light & presence, Water sensors and Tablet controls. The right column has Running, Heating, Speakers & radio, Doors & windows, Water filter and "For {kid}". A second view uses a top tab bar with 8 icon tabs (home, kid, energy, stats and others) and a kid's room page. The layout changed after posting (RED-30).

#### Hero banner

- **RED-01. Illustrated weather scene as the hero.** The banner art is the Google weather frog, with 161 scenes: 100 of Google's originals, picked by the Open-Meteo weather code for the current hour, and 61 scenes the author generated with Gemini in the same style, showing the family's own life (the walk to kindergarten in fog, rain, snow or wind; bedtime; weekend zoo, market, pancakes; seasonal and holiday scenes; everyone's birthday). The clock, date and weather text sit on top of the art. In the prototype this space is a photo (TODAY-14) and the weather is its own block (TODAY-02). The Google panoramas come from [github.com/sga-noud/google-weather-frog](https://github.com/sga-noud/google-weather-frog) (`images/wide`) (OP comment).
- **RED-02. Scene rules.** The scene is picked from several inputs. When several scenes fit, they rotate every 15 minutes. The inputs are:
  - weather for the current hour: Open-Meteo's hourly WMO weather code, which the OP prefers to HA's weather states because it is more precise (OP comment)
  - wind: kite scenes only above 20 km/h, windy scenes only above 30 km/h (OP comment)
  - sun phase, from `sun.sun` elevation while the sun is setting: below 8° is dusk (lamp or candle lit), below 20° is golden hour, otherwise day; then night (OP comment)
  - temperature gates: no beach or swimming below 22°C, ice cream above 28°C, fireflies only at 14°C or more, picnics and camping at 15°C or more, the summer wheat field only above 10°C
  - month: fireflies May to August, orchard in season, autumn leaves September to November
  - the kid's schedule: kindergarten run on weekdays 07:00-08:30, with the frog dressed for the actual weather (umbrella, sled); bedtime 20:30-21:30; weekend mornings 08:00-10:30
  - dates: holidays (Christmas, Easter, Halloween, Children's Day) and birthdays beat everything else

  If the season filter rules out every match, it falls back to the unfiltered matches. The 15-minute rotation is deterministic: the current 15-minute slot number picks the scene from the list, so every screen shows the same one without storing anything (OP comment). Text color: the post says each scene stores a color measured from screenshots. A comment says the card samples the image under the clock and weather text and switches between white and dark text. It may do both. Each scene also has its own vertical framing, so the frog is never cut off. The OP posted the scene-picking JavaScript (weather code to scene, season filter, light phase, pick) [in a comment](https://www.reddit.com/r/homeassistant/comments/1x5mx00/comment/pgqhhqj/).
- **RED-03. Exhaustive rule test.** A script runs every combination (28 weather codes × every half hour × every month × realistic local temperatures, about 86,000 cases) to check that the rules always pick a sensible scene. The same idea would work for any rule-driven content we add, such as weather advice (TODAY-03) or the idle and mode timers (SHELL-05, SHELL-06).
- **RED-04. Context cards over the banner (screenshot).** Timers, now playing, kindergarten, dinner ideas, kids' shows and parcels appear as white cards over the right half of the banner, each with its own X to dismiss. Two parcel cards can sit side by side. Round icons in the header (a teddy bear and a plate) open the kid and dinner cards; the plate icon is ringed while its card is open. This is a lighter alternative to our sheets (SHELL-10) and banners (SHELL-11): the clock and weather stay visible. The OP says everything time-sensitive goes in the banner so nobody has to scroll for it, including a "Laundry is done" card (RED-09) (OP comment).
- **RED-05. Header strip (screenshot).** Humidity and a CAQI air-quality index with a colored dot; current temperature, condition, high and low, and rain chance; two blurred person avatars with a green percentage badge (85%, 99%), most likely each phone's battery (inferred); live household power draw ("179 W"); and the "All good" pill (RED-06). The prototype collects but never shows humidity (DATA-23) and has no air quality or power data.
- **RED-06. "All good" health pill.** One pill at the top that turns orange or red for low batteries, offline critical sensors, an open window while the heating is on, or a leak. It is a broader version of the Today home status line (TODAY-17), which only covers lights, locks and temperature.

#### Kitchen helpers

- **RED-07. Voice timers on the wall.** "Ok Google, set a timer for 15 minutes" said to any speaker in the house shows on the tablet as a huge countdown with the timer's label and source speaker ("Pasta · Kitchen speaker"), a progress bar and a dismiss button, readable from the stove. We have no timers at all. Details from the OP's comment:
  - Up to three timers show at once, soonest first. A timer reads "Time's up!" while it rings and "Paused" when paused. The ✕ cancels the timer on the speaker, not just on screen. Starting a timer also sends the tablet back to Home (RED-30).
  - The data comes from the [ha-google-home](https://github.com/leikoilja/ha-google-home) HACS integration, which creates a `sensor.<device>_timers` per speaker (fire time, duration, label, status) and a service to delete timers. Setup needs a Google "master token", which is the fiddliest part.
  - The integration polls every 180 s by default, so a new timer can take 3 minutes to appear. The OP lowered it to 3 s.
  - The countdown runs locally in the card from the fire time, so it ticks every second without polling. It is shifted by about 3.5 s because the speaker rings slightly late, so both reach zero together.
  - It relies on Google's unofficial local API, so a Google update could break it.
- **RED-08. Now playing with full controls.** Album art, title, artist, room, pause, skip and a volume slider, shown as a banner card. The Speakers & radio card also has a power button per speaker and a row of favorite radio stations, each one tap ("Vanilla", "Smooth", "OFF") (screenshot). This extends HOME-13, which has play/pause only.
- **RED-09. Running appliances.** Washing machine (from SmartThings) and dishwasher (detected from power draw), each with % done, the current phase ("Washing", "Rinsing") and time left on a progress bar. When a cycle ends, a "Laundry is done" card appears in the banner with the finish time ("13:05, 5 min ago") and the hint "Tap ✕ once it's emptied", so it stays until someone deals with it (OP's tablet photo). How the dishwasher works (OP comment):
  - The hardware is a Tuya smart plug with power metering, run locally through `tuya_local`, plus a Zigbee door contact sensor. There is no dishwasher integration.
  - A cycle starts when the plug reads above 10 W for 3 minutes. The delay ignores the panel lighting up when the door opens or a program is picked.
  - It is "really washing" once the plug reads above 40 W for 30 s (the pump draws 50-65 W, the heater about 1,850 W). Without that flag, short pulses caused false "done" alerts.
  - The cycle is finished after 15 minutes below 5 W. Idle is about 2.2 W and the lit panel 6-8 W. The 15 minutes also cover the short pulses after a cycle ends.
  - Cycle length is learned from the last real cycle, and that drives the progress bar and "minutes left" (also shown on the pixel clock).
  - "Dishes are clean" stays on until someone opens the door. Opening the door also resets the state, so closing it after loading never counts as a cycle.
  - The OP's lesson: base it on measured power plus the door, not on fixed timers.
- **RED-10. Dinner ideas from an LLM.** Claude suggests three dinners for the kid. They must not repeat what she ate at kindergarten that day, should suit the weather (something warm when it's cold), avoid recent meals, and follow her likes and the family's rules. Each card shows a dish name, one-line summary and cook time, with an "Other ideas" refresh (screenshot). Each has a full recipe. On the recipe screen you tick the ingredients you don't have, and they go into a new Google Keep shopping list shared with the other parent. Liked recipes are saved, with a ♥ ("Liked?") for the ones the kid actually ate. FamilyOS has a recipe box, a meal plan and ingredients-to-list (FOS-33 to FOS-35) but makes no suggestions. Details from the OP's comments:
  - **Engine:** Claude Sonnet through HA's AI Task integration (`ai_task.generate_data`). Ideas are generated fresh every day, not pulled from recipe sites. Each run costs 1.5-2 cents, about $1-1.50 a month.
  - **Scope:** dinner only, because the kid has breakfast and lunch at kindergarten.
  - **Prompt rules:** at most 30 minutes; only chicken, turkey or fish (never beef or pork), salmon at most every two weeks; little salt, nothing spicy, soft textures; a list of dishes she likes, "go in that direction, but vary". The three ideas must be different types: one soup or cream, one vegetable-based, one with chicken, turkey, fish or egg. Each comes with a full recipe for two child portions, with ingredients and amounts and short steps.
  - **Prompt context:** the month; today's high, low and current temperature (if it's cold, at least one warm idea); the last 15 suggested dishes, to avoid them and close variants; and today's kindergarten menu, so dinner avoids lunch's main ingredient and is lighter after a heavy lunch.
  - **Storage:** results live in a trigger-based template sensor's attributes, driven by `dinner_start`, `dinner_done` and `dinner_error` events. Its `history` attribute keeps the last 15 dish names.
  - **Gotchas:** AI Task's structured output didn't accept a list of objects, so the OP used flat fields (`t1`, `d1`, `m1`, `k1`, `i1`, `p1` and so on for each idea: title, description, minutes, kind, ingredients, steps). The token limit had to be raised so all three recipes came back complete.
  - **Saved recipes:** a ☆ saves the recipe to an HA Local To-do list, which the dashboard reads back as "Saved recipes".
  - **Shopping list hand-off:** there is no official Keep API for personal accounts, and the Google Keep Sync HACS integration can't create lists. Instead, a Python script on the server uses `gkeepapi` (unofficial) with a Google master token. It creates a Keep list named "🛒 {recipe}", shares it with the chosen parent(s), and only adds the missing items if the list already exists. HA runs it through a `shell_command` wrapped in a script that returns the result (`response_variable`), so the dashboard can show "✓ 5 added", and a phone notification opens Keep. On the tablet you open a recipe, tap the missing ingredients, pick who gets the list, and tap "Add to Google Keep". It reaches both phones in about 6 seconds. The master token gives full access to the Google account, so it has to be kept safe.
  - The full prompt, AI Task script and template sensor are [in a comment](https://www.reddit.com/r/homeassistant/comments/1x5mx00/comment/pgqchso/). The comments don't explain the "Other ideas" button.

#### Kid and family

- **RED-11. School card.** In the morning: the weather at drop-off time (temperature at 08:00, feels like, today's max, rain chance) and what the kid eats at kindergarten today (breakfast, lunch, snack, each with an emoji). In the evening it switches to tomorrow, like FamilyOS's evening flip (FOS-01). The weekly menu arrives as a photo from the teacher on WhatsApp. The author forwards it to a Discord channel and Claude reads it into an HA sensor. That is a narrow version of FamilyOS's capture inbox (FOS-41).
- **RED-12. Kids' events finder.** Every Monday it reads the local children's theater listings and keeps shows for next weekend that suit the kid's age and don't start before 10:00 ("we're never ready by 9"). Shown as a paged card ("Kids shows 3/8") with poster, title, day and time, price, age rating and venue (screenshot). When nothing matches, the kid's section says "none next weekend".
- **RED-13. Per-kid section (screenshot).** A "For {kid}" card collects the kid features: Kids shows, Weekly menu (showing today's dish), Recipes, and a saved recipes list with a count and a "4 more recipes" expander.
- **RED-14. Kid's room page (screenshot).** Room temperature and humidity, and a sleep light with an on/off schedule (on at 20:30, off at 07:30) and an on/off switch for that schedule.

#### Home status and control

- **RED-15. Heating by room.** Temperature for three rooms, the central heating thermostat with ± buttons, and its state ("Idle", or "Not asking · today 2 min", which is heating run time today) (screenshot). The boiler will not start while a window in that room is open. One-zone climate in the prototype (HOME-11, DATA-20) would need to grow to several rooms.
- **RED-16. Water safety that acts.** Three leak sensors, the dishwasher door state, and a motorized main valve that closes by itself on a leak and cuts power to the machines. The valve tile shows Open with a manual Close button. In the prototype the shut-off is UI only (ALERT-02).
- **RED-17. Consumables tracker.** Each water filter cartridge shows days left, the replace-by date, a progress bar and a reset button for when it is replaced. This is a smaller, kitchen-specific form of FamilyOS's recurring duties on household records (FOS-37).
- **RED-18. How long a door has been closed (screenshot).** Door and window tiles say "Closed for 36 min", and the card header summarizes "all closed". The garage banner in the prototype has duration copy, but it is static (ALERT-03).
- **RED-19. Parcels from courier app notifications.** The tablet reads the notifications of the two courier apps the family uses (Sameday and FAN Courier). Cards show a locker pickup countdown ("1d 7h left to pick up, until Thu 19:42") and a courier delivery window with the amount to pay on delivery (screenshot). The prototype's package banner (ALERT-04) only covers a porch drop. How it works (OP comment):
  - The couriers have no official API. The HA Android companion app's "last notification" sensor is limited to the two courier apps.
  - When a notification arrives ("Your parcel ... has arrived at easybox ..."), an automation sends its text to a Python script that pulls out the tracking number (AWB) and the locker. The script then checks the courier's public tracking page every 30 minutes.
  - The tablet only shows two states: "courier coming today" (with the time window and the cash-on-delivery amount) and "in the locker" (with the time left to pick up).
  - Email was tried first, but the shop's order emails (eMAG) have only the order number, not the tracking number. The app notification has it.
- **RED-20. Tonight's movie.** In the evening, a pick from Plex shows on the dashboard. Tapping it plays the trailer.
- **RED-21. Scene that pauses an automation.** "Movie Night" turns on the kitchen LED strip and stops the motion sensor from turning it off. Our scenes (HOME-03) only set device levels.
- **RED-22. Automation settings on the dashboard (screenshot).** The kid's room page shows the AC rules as plain controls: a night shut-off rule with its time and time left, other rules as on/off tiles ("Automatic temperature control", "Motion → bedroom light"), thresholds with ± buttons ("Start cooling above 27.3°", "Stop cooling below 26.9°", "Cooldown between switches 15 min"), and a "Freeze automation" button. The family can tune automations without opening the HA editor.
- **RED-23. Light and presence readout (screenshot).** Kitchen light level in lux and kitchen presence ("Empty"). Probably mainly for debugging the wake and dim rules (inferred).

#### Screen behavior

- **RED-24. Presence-driven wake and dim.** The mmWave sensor wakes the screen when someone walks into the kitchen and dims it when the room is empty. Brightness also follows the time of day. The prototype only uses touch idle timers (SHELL-04 to SHELL-06) and a time-based night mode.
- **RED-25. Presence-based screensaver.** After 15 minutes with nobody in the kitchen, daytime only, the tablet switches to a screensaver: eyes that follow you, a drawn frog, or an animated frog. Our ambient mode (AMB-01) starts after 120 s without a touch, whether or not anyone is in the room.
- **RED-26. Tablet controls card (screenshot).** Tiles for Clock Sync (to the iDotMatrix pixel clock), Parental (on), On charge (on), Screensaver (off), Frog scene ("Automatic", so a manual pick is possible) and Reset ("On schedule", most likely a scheduled reload, inferred).
- **RED-27. Unplugged means hands off.** When the tablet is unplugged, every automation leaves it alone, because it means someone is using it for something else. This only matters if the frame is ever carried away from the wall.
- **RED-28. One card, a layout per device.** The Nest Hub in the living room casts the same dashboard, which detects the Hub and switches to a slimmer layout. The OP says the same card also has its own layouts for the kitchen tablet, phones and desktop browsers. The Hub uses a different section order and leaves some parts out entirely. Because it is all one codebase, a change reaches every device at once. The full scrolling page in the screenshots is mostly the phone and browser view (OP comment). This relates to question 3 (one app on wall and phones, or wall only).
- **RED-30. Tablet layout rebuilt with a side bar (OP comment).** After a commenter pointed out how much scrolling the dashboard needed, the OP rebuilt the tablet layout. It now has a side bar, and each section fits on one screen. Home holds only what the family uses all the time: the banner (weather, timers, dinner, parcels, laundry), the main lights, and the speakers and radio. Everything else has its own section: Lights (by room), Climate (heating and both ACs), Media, Water & doors, the kid's section, Energy, and Tablet settings. A garage button is always at the bottom of the side bar. The tablet goes back to Home after a few minutes without a touch, or as soon as a timer starts. This is close to our rail (SHELL-01) and idle return (SHELL-05). The new parts are the always-there garage button and jumping to Home when a timer starts. There are no screenshots of the new layout.

#### Build approach

- **RED-29. One custom card with no build step.** The whole dashboard is a single vanilla JS custom card of about 5,300 lines, plus a small panel card for the other views, served inside HA and run through the companion app (contrast RED-47, which is a standalone page). It is a data point for question 2: Home HQ could live inside HA as custom cards and not as a separate app. The dashboard was only about two weeks old when posted and was mostly written with Claude Code. Most of the integrations already existed. The OP says the slow part is testing and living with it: catching a scene that doesn't match the weather, a card that shows up at the wrong time, a timer a few seconds off from the speaker (OP comment).

### Walnut-and-brass living room dashboard

- **Source:** [r/homeassistant, "Finally have a workable Dashboard that I actually like looking at"](https://www.reddit.com/r/homeassistant/comments/1wuhxk0/), posted 2026-09-30, read 2026-10-18. Post text, 1 screenshot, and the OP's 82 replies in the comments (30 with real content), including one more screenshot. 237 of the 258 comments loaded.
- **Screenshots:** `references/reddit/1wuhxk0-walnut-brass/` (the full dashboard, and a climate tile opened to its controls).
- **Relation to the kitchen-frog thread:** this is the dashboard the kitchen-frog OP copied (RED-29), so the two share a layout: header strip, cards in columns, a warm palette, serif numbers. The kitchen-frog OP thanked this OP in this thread's comments, and this OP said they planned to borrow the Plex "next episode" idea back.
- **Household:** two adults, two kids and two French bulldogs, near NYC.
- **Hardware:** a wall iPad and an LG OLED TV in the living room, each opening the page in a fullscreen browser. UniFi network and cameras (Dream Machine, G4 Doorbell Pro, Protect). A Synology NAS that runs HA. A Dreame robot vacuum. Fi dog collars. A Rivian R1S and an Audi RS e-tron GT. Sonos speakers.
- **Software:** one standalone HTML/JS page, built in about a day with Claude (Opus 5.5, medium effort), styled to match a photo of the living room: walnut, brass, serif numbers. Light during the day, dark at night.
- **Layout (screenshot):** a header strip (quote, family and dog avatars, flight badge, "All clear" pill, theme toggle, clock), then three columns. Left: Systems, Coming up, Home (vacuum, dogs). Middle: Today (weather, watch and fragrances), Climate, Sonos. Right: Cameras, Garage (cars). It doesn't scroll. Controls open in a sheet when you tap a tile, not on separate pages (OP comment).

#### How much overlaps

Most of the screen maps to something we already have. The new parts are the household-specific extras (planes, pets, cars, vacuum, network health, the watch and fragrance pick) and the implementation details in the OP's comments.

| This dashboard | Already in this doc | What's new here |
| --- | --- | --- |
| Family avatars with presence dots | Presence (TODAY-05) | A badge for a named zone (a graduation cap at school), and pets alongside people (RED-32) |
| "All clear" pill | Home status (TODAY-17), "All good" pill (RED-06) | A network and backup health chip (RED-34) |
| Clock and date, top right | Clock (TODAY-01) | Nothing |
| Coming up: today's events with a "Now" marker and durations, "See all" | Next up and person lanes (TODAY-06 to TODAY-13) | Less than ours. A single list, no per-person lanes or leave-by |
| Today: temperature, condition, high and low, sunset, feels like, wind, UV, an 8-hour strip | Weather (TODAY-02 to TODAY-04) | Shows feels like and wind, which our data has but never shows (DATA-23). Adds sunset and UV |
| Climate: two zones, mode badge; the tile opens a full control sheet | Climate (HOME-11), heating by room (RED-15) | The control sheet's content (RED-37) |
| Sonos: now playing, transport, volume, room chips to group speakers | Media (HOME-13), now playing (RED-08) | Speaker grouping and one-tap playlists (RED-38) |
| Cameras: 4 snapshot tiles, live on tap, "No recent activity" | Cameras (HOME-12) | Real refresh and live-stream behavior, and a package chip (RED-39) |
| Doorbell pop-up | Doorbell takeover (ALERT-01), package banner (ALERT-04) | The trigger, suppress and auto-close rules (RED-40) |
| Theme toggle, light by day and dark at night | Day, ambient and night modes (AMB, NIGHT) | A manual toggle and a second, serif theme (RED-45) |
| OLED burn-in handling | Pixel shift (NFR-12) | An art screensaver after one minute (RED-46) |
| One HTML page served by HA over its WebSocket API | Static Vite app on Cloudflare (NFR-13), custom card (RED-29) | A third way to host Home HQ (RED-47, RED-49) |
| Daily quote | Nothing | RED-31 |
| Plane overhead | Nothing | RED-33 |
| Systems card | Nothing | RED-34 |
| Robot vacuum | Nothing | RED-35 |
| Watch and fragrance pick | Nothing | RED-36 |
| Cars | Garage door only (HOME-09) | RED-41 |
| Dog page | FamilyOS pet records (FOS-37), which hold no live data | RED-32 |

#### Header

- **RED-31. Daily quote.** A Stoic quote with its source ("Marcus Aurelius · Meditations IV.7") at the top left. About 75 quotes (Marcus Aurelius, Seneca, Epictetus) are hard-coded in a JS file, taken from public-domain translations and checked against the original text. The quote changes every 4 hours and the whole list cycles in about a week and a half (OP comment). For us, the same slot could hold a family message or a board note (FOS-40).
- **RED-32. People, pets and places.** Four family avatars and two dog avatars, each with a green presence dot. The kids get a little graduation cap while they are at school. Presence comes from Life360. The OP's one complaint is that it sometimes says someone left home while they are asleep in the house. Commenters suggested the HA companion app's location instead, and the OP said they'd look into it (OP comment). Each dog has its own page with steps, rest and collar battery from a Fi collar, read through Fi's API, which needs the base subscription (OP comment). The screenshot shows rest, battery % and steps for each dog. Our presence (TODAY-05) has no zone badges and no pets.
- **RED-33. Plane overhead.** When a plane is actually over the house, the header shows a badge with the airline logo, flight, route and altitude ("Delta 218 · JFK › CPH · 10k ft"). Tapping it opens a radar view of every plane within 10 miles. How it works (OP comment):
  - A small Python script (standard library only) runs as an HA `command_line` sensor every 10 s. The sensor's state is the number of planes overhead. Its attributes hold the closest plane and a list (callsign, airline, origin and destination, aircraft type, altitude, distance).
  - Positions come from adsb.lol (`/v2/point/<lat>/<lon>/10`, free, no key). Route and airline come from adsbdb.com by callsign, cached 12 h. Aircraft type comes from adsbdb by airframe, cached 30 days. AeroDataBox (RapidAPI free tier) is the fallback, only for a plane that is really overhead with an unknown route, capped at 6 calls a day with a 6 h back-off after a 429.
  - "Overhead" means at least 30° above the horizon, under 25,000 ft and within 4 miles, worked out from the home's latitude and longitude. A fixed radius would keep showing planes landing at the 3 nearby airports. It also projects each plane along its track and flags one that will be overhead within 75 s, so the badge is ready when you look up.
  - Airline logos were downloaded once as SVGs into `/config/www` and load by airline code, so nothing is hotlinked.
  - Lessons: anonymous OpenSky rate-limited too hard. Polling AeroDataBox every 2 minutes used up the free monthly quota fast. Use the free ADS-B sources for anything polled, keep the paid API for rare lookups, and cache everything.
- **RED-34. Network and system health.** A Systems card (screenshot) shows "Internet up", router uptime (19 d), live throughput, the last speed test (via Fast.com), devices online and data used today, the last backup time, and pending updates. It also has a load bar and temperature for each server (named chaos, cosmos, Protect). The OP says most of the value is one chip that answers "is it the internet or is it us?" (OP comment):
  - WAN up/down from the UniFi integration. A tiny script pings 1.1.1.1 every few seconds and reports a 10-minute median and packet loss. The baseline is about 4 ms, and it turns yellow over 40 ms or 2% loss.
  - Access points online and clients per AP. NAS volume status, disk space and the last good backup.
  - All of it rolls into one chip: normally "All systems healthy · 4 ms", otherwise exactly what's wrong ("Internet down", "Loss 5%", "Outdoor AP offline", "NAS needs attention"). Tap it for details.

  This could feed our health pill (RED-06), but it is more of an admin tool than a family one, like Agents (section 11).

#### Today column

- **RED-35. Robot vacuum with "rooms fresh".** The Home card shows the Dreame's battery, status ("Heading home"), pause and dock buttons, and "3/6 rooms fresh" with one dot per room (screenshot). The post says the full view has the cleaning map, which rooms were cleaned and when, and lets you tap a room to clean it. A room is "fresh" when it has been both vacuumed and mopped in the last 24 hours. Dreame has no rooms-cleaned entity, so the dashboard builds one (OP comment): it pulls 10 days of history for the vacuum, its current-room sensor and its state sensor, and replays them as one timeline. A room counts as cleaned when the robot was cleaning, on the floor (not at the dock washing the mop), and spent at least 90 seconds in that room, so driving through the hallway doesn't count. Each room's color comes from its last clean: green under 24 h, yellow 1-3 days, gray after that. Room names come from the `rooms` attribute of the Tasshack Dreame integration.
- **RED-36. What to wear today.** Under the weather: a watch for the day ("Datejust 36") and a morning and evening fragrance. The pick comes from the OP's own collections in Notion, and checks the weather and calendar (meetings, dinners, flights). It rotates through what they own, and rain means no leather straps. The OP only says it is "a mix of Notion databases, news articles, weather lookups" (OP comment). The idea that carries over for us: a pick from things the family owns, filtered by weather and the day's events, like kids' clothes or which jacket to take. It is a step beyond our weather advice line (TODAY-03).
- **RED-37. Controls inside the tile.** No separate controls page: tapping a tile opens its controls in a sheet (OP comment). The climate sheet shown has a zone switcher (Upstairs, Basement), setpoint ±, now/humidity/outside, Mode (Off, Cool, Heat, Auto), Fan (Auto, On, Circulate) with circulate time %, a dehumidify toggle with a target %, humidity mode (Normal, Max, Climate IQ), schedule presets (Schedule IQ, Summer, Winter, Spring/fall, Save energy), and Away and Smart Away toggles (screenshot). This is our sheet pattern (SHELL-10, HOME-02) applied to every tile. The richer climate sheet goes past HOME-11.
- **RED-38. Speaker grouping.** The Sonos card shows what's playing and its source ("TV audio · From the TV · Living Room"), previous, pause and next, and volume. Below that are room chips that add or remove a speaker from the group (Kitchen +, Living Room ✓, Main Bedroom +, Pool...) (screenshot). The post also mentions one-tap playlists. The OP said the card is still a work in progress (OP comment). Our media (HOME-13) and RED-08 have no grouping.

#### Right column

- **RED-39. Cameras that are actually live.** Four UniFi Protect cameras as snapshot tiles that refresh every 15 s, with a "No recent activity" badge. Tapping one opens a live view. A package detection doesn't pop up. It shows a "Package · delivered 12m ago" chip on the camera card for 6 hours (OP comment). Our cameras (HOME-12) fake the refresh and have no live view.
- **RED-40. Doorbell pop-up rules (OP comment).** The page subscribes to state changes and opens the pop-up itself, without Browser Mod:
  - It triggers on the doorbell ring event or the doorbell's person-detected sensor turning on.
  - It is skipped if someone in the family got home in the last 3 minutes, so you don't get a pop-up of yourself walking in.
  - It opens the camera sheet and wakes the screensaver. A snapshot shows at once and refreshes every 2 s, then switches to live HLS through HA's camera stream once that is ready.
  - It closes 30 s after the porch is clear and nobody has rung for a minute, 3 minutes at most. Touching the screen keeps it open.
  - Ring, person and TV are each a setting, so it can be kept off the living room TV.

  Our doorbell takeover (ALERT-01) has a fixed 45 s auto-dismiss and no rule to suppress it when family arrives.
- **RED-41. Cars.** Each car has a photo, its name ("Rivianna", R1S), battery %, range ("124 mi"), lock state, and home or away. A car that can't be reached shows "Not connected" (screenshot). The OP confirmed in a comment that Audi access broke recently (app store integrity checks). There is nothing like this in the prototype, where the garage is only a door (HOME-09).

#### Display and look

- **RED-42. Design system shared (OP comment).** The OP posted their tokens and CSS for someone copying the look. The main points:
  - Fonts: Unbounded for the clock, titles and numbers, Plus Jakarta Sans for everything else, and Fraunces for the serif theme. All from Google Fonts.
  - Small uppercase letter-spaced labels instead of big bold headers.
  - Big radii (26 px cards, 20 px tiles, 15 px buttons) and a soft two-layer shadow.
  - List rows as rounded tiles, and cards that hug their content instead of stretching to the column height.
  - A warm background (`#f1eee7`), navy for the selected pill, brass for the brand color, and press feedback by scaling down slightly.
  - Turning off transitions for `prefers-reduced-motion`.

  The full token block and CSS are [in a comment](https://www.reddit.com/r/homeassistant/comments/1wuhxk0/comment/peba885/). It is close to our paper-and-ink palette (NFR-18).
- **RED-43. Matched to the room.** The OP photographed the living room and had Claude match the palette to it (walnut, brass, serif numbers). Wanting the dashboard to look like part of the room, not an app, is a design goal we could adopt.
- **RED-44. Same page on a tablet and a TV.** The wall iPad and the LG OLED both open the same URL fullscreen. Each pop-up type can be turned off per display, for example doorbell pop-ups off on the TV (OP comment). This is like FamilyOS's per-display settings (FOS-11) and the per-device layouts in RED-28.
- **RED-45. Theme toggle.** A button at the top right switches light and dark, and the page also goes light by day and dark at night. There is also a serif theme (Fraunces). The OP mentions a separate OLED screen for the evening. We switch modes by time only (SHELL-06), with no manual toggle.
- **RED-46. OLED burn-in care (OP comment).** The dashboard stays up about a minute before an art screensaver takes over. The OP is also testing a pixel shift every few minutes. We already shift text on ambient and night screens (NFR-12).

#### Build approach

- **RED-47. Standalone page served by HA.** One HTML/JS file copied into `/config/www/` with a small deploy script. HA serves it at `http://<ha-ip>:8123/local/<file>.html`, so there is no separate web server. On load it opens a WebSocket to HA for live states, subscriptions and service calls, so HA is both the host and the backend. The OP says this has been more stable than custom Lovelace cards, because it only uses the WebSocket API that the official apps use, which HA is careful not to break, and frontend updates can't affect it. The OP's advice to someone trying to copy the look with Lovelace: "Give up on Lovelace". Remote access is over a VPN, not exposed to the internet (OP comment). The code is in the OP's private git, and they are considering publishing it.
- **RED-48. Building it with Claude.** Claude's desktop app was given the dashboard folder and an HA token, so it could see the entities. The OP suggests exporting the entity list instead if that's a concern. The loop: ask for one feature at a time; Claude builds it, tests it against fake data and screenshots it; the OP deploys with one script. Everything is in git so any change can be rolled back (OP comment).
- **RED-49. Three ways to host Home HQ.** Between the two threads and the prototype there are now three options: a separate static app (the prototype, NFR-13), a custom card inside HA's frontend (RED-29), or a standalone page served by HA and talking to it over the WebSocket API (RED-47). Question 18 covers the choice.

### Phone-first HTML dashboard with themes

- **Source:** [r/homeassistant, "Responsive Dashboard Part 2"](https://www.reddit.com/r/homeassistant/comments/1x6i98v/), posted 2026-10-15, read 2026-10-18. The post is a 4 min 53 s portrait screen recording of the phone version. I reviewed it as frames taken every 3 seconds. I also read the post text and the OP's 20 replies in the comments, which include one desktop screenshot. 68 of the 69 comments loaded. The title implies there is a Part 1, and the OP said a tablet version would follow.
- **Screenshots:** `references/reddit/1x6i98v-responsive-html/`. There are 13 contact sheets of the video (8 frames 3 s apart each; file names give the start time and the screens shown) and the desktop screenshot.
- **Household:** New Zealand (GeoNet alerts, NZ public holidays). Two adults with their own calendars, and young children (a baby room, a play room, and a bed LED for each child).
- **Devices seen:** a heat pump with a unit in each room, Zigbee2MQTT lights (OP comment), Spotify Connect speakers, cameras with person, vehicle and line-crossing detection plus two-way audio, an alarm panel with a siren, heated towel rails, room presence sensors, three energy-monitoring smart plugs, soil moisture sensors and irrigation valves with flow meters, and a Mercedes with a connected-car integration.
- **Layout:** on the phone, a header with the page title, an alarm-state shield and the clock; a bottom bar (Home, Rooms, Scenes, Camera, More); and a mini player pinned above the bar on every page. "More" opens a grid: Climate, Weather, Energy, Irrigation, Media, Events, Vehicle, Theme, Settings. On a desktop the same pages sit in a left side bar, and the header shows "DISARMED", a "LIVE" connection badge and the outdoor temperature (screenshot).

#### How much overlaps

There is a lot of overlap, with this doc and with the walnut-and-brass thread. The pages we have no equivalent for are Energy, Irrigation and the theme picker. The overlapping pages add useful detail: a calendar form that has the fields our new-event sheet lacks (CAL-24), camera clip playback, an alarm keypad, and a notification center with an "Active" tab.

| This dashboard | Already in this doc | What's new here |
| --- | --- | --- |
| Greeting and weather card with animated rain | Weather (TODAY-02), illustrated scene (RED-01) | A personal greeting ("Evening, Rey!") and animated weather (RED-64) |
| Now playing with "+1" for a second player, expandable, lyrics | Media (HOME-13), RED-08, grouping (RED-38) | Synced lyrics and a mini player on every page (RED-50) |
| Heat pump card, 5-day forecast | Climate (HOME-11), weather strip (TODAY-04) | Per-room units on the Climate page (RED-56) |
| Weather page | TODAY-02 to TODAY-04, header metrics (RED-05) | Charts per metric with a one-line summary, and hazard alerts (RED-52) |
| Notifications: Alerts and Active tabs | Health pills (RED-06, RED-34), FamilyOS "Needs you" (FOS-05) | An "Active" list of what's on or open right now (RED-51) |
| Light detail sheet | Light tile (HOME-06) | Color temperature, colors, "turned on 19 minutes ago" (RED-53) |
| Scene gallery | Scenes (HOME-03) | Image-based scenes applied per room (RED-54) |
| Rooms | Rooms (DATA-17, HOME-05) | Filters, temperature-tinted cards, per-room automation toggles (RED-55) |
| Alarm | None (DEF-05 notes we have no alarm) | A PIN keypad and siren (RED-57) |
| Cameras | HOME-12, ALERT-01, RED-39 | Clip playback by date, a stream-quality switch, a garage button in the camera view (RED-58) |
| Energy | Live power draw only (RED-05) | Tariff-aware cost and insights (RED-59) |
| Irrigation | Nothing | RED-60 |
| Events | Calendar (section 6), new-event sheet (CAL-22), FamilyOS repeats (FOS-13) | A list by day with a color legend for each calendar, and a full add form (RED-61) |
| Vehicle | Cars (RED-41) | A fuel car's eco score, and window and boot status (RED-62) |
| Theme page | Theme toggle (RED-45), FamilyOS looks (FOS-54) | Style × font × mode × accent (RED-63) |
| Phone and desktop layouts | Per-device layouts (RED-28) | A bottom bar on phones, a side bar on desktop (RED-65) |
| Standalone page served by HA | RED-47 | Signing in through HA's login page, and a separate config file (RED-66) |

#### New or more detailed

- **RED-50. Mini player and synced lyrics.** A now-playing bar sits above the bottom bar on every page. On Home it expands into a card with shuffle, previous, pause, next and repeat, and a "+1" shows a second player is active. Swiping the card shows lyrics, highlighting the current line. The Media page lists each Spotify Connect speaker with its art, controls and volume, has an "Open Spotify" button, and shows a full lyrics view labeled "Synced" with a ± offset to line the lyrics up (screenshot). The post says lyrics show "when available" and are mostly synced. The source isn't named.
- **RED-51. Notification center (screenshot).** A sheet with two tabs:
  - **Alerts:** hazard alerts (RED-52); "towel heating" listing the heated towel rails that are on; low batteries with each sensor's %; and controls for room occupancy (rooms turned off), presence status, and driveway camera detection (person, vehicle).
  - **Active:** lights that are on, with their brightness and an "All off" button, and open windows.

  The Active tab answers "what's on or open?" before you leave or go to bed, which our home status line (TODAY-17) only counts.
- **RED-52. Hazard alerts.** A GeoNet alert ("Volcano Ruapehu · Minor volcanic unrest · 15 October at 11:21 pm") tops the notification sheet and the Weather page. For us this would mean official local weather and emergency warnings.
- **RED-53. Light detail sheet (screenshot).** On/off, "turned on 19 minutes ago", a brightness ring, a warm-to-cool slider, color swatches, and that light's scenes ("Midwinter", "All scenes").
- **RED-54. Picture scenes per room (screenshot).** Scenes look like Philips Hue's: a gallery of photos grouped by mood (Refreshing, Peaceful, Lake, Coastal), each with its color dots, plus a Gradient tab. You pick a scene and then a room to apply it to, from tabs for Living area, Bedroom, Bathroom and Outdoor, listing that room's lights. Our scenes (HOME-03) are whole-house buttons.
- **RED-55. Rooms (screenshot).** Filters for Occupied, Main, Utility and Outdoor. Each room card shows temperature, humidity, a count of open windows and lights on, and a "Heat" badge, and is tinted by temperature. A room page has light tiles that fill to their brightness, "1 of 8 on", and "All off". A per-room automations sheet has switches such as "Movie mode" and "Presence sensor" (compare RED-21 and RED-22).
- **RED-56. Climate by room (screenshot).** The heat pump card has Off, Cool, Heat, Fan and Dry modes and a fan speed. Below it, each room's unit has a card with its temperature, on/off and a target with ±, tinted warmer or cooler by temperature.
- **RED-57. Alarm with a PIN (screenshot).** The Security page shows the alarm state ("DISARMED"), an "Arm away" button, and siren controls ("Siren", "Siren · 30 s"). Arming opens a numeric keypad for the PIN. The alarm state also shows in the header on every page. We have no alarm (DEF-05), and our only protection for risky actions is press-and-hold (SHELL-13).
- **RED-58. Cameras with clip playback.** Live tiles with a "LIVE" badge. The full view has mute, two-way talk, fullscreen and "Playback", and a garage door control ("Closed · tap to open") at the bottom. Playback lists days, then each day's clips with time, length and detection type ("Motion Person", "Motion Vehicle Crossline"), and plays a clip in place (screenshot). The post says you can switch between a light "fluent" stream to save resources and the "clear" 2K stream, and that playback and two-way audio were a game changer.
- **RED-59. Energy with tariff-aware cost (screenshot).** Day, week, month and year views with date stepping:
  - Electricity cost split by the actual tariff: a charged period (midnight to 9 pm at its rate), a free period (9 pm to midnight), and the daily charge.
  - A usage chart.
  - Usage per device from three smart plugs (portable heater, rice cooker, Dyson).
  - Insights: change vs last month ("↓ 23%"), the busiest hour and its biggest consumer, and how much of the free hours were used, with a saving tip ("running Portable Heater there instead could save up to $0.83").

  The OP says this is their favorite page and they plan to add more plugs.
- **RED-60. Irrigation adviser (screenshot).**
  - The current water restriction ("No water restriction · use water sensibly").
  - A rain check for the next 12 hours with a verdict ("Rain on the way, likely enough to skip watering"), expected millimeters, peak chance, time to first rain, and hourly bars.
  - Soil moisture for the last 72 hours for each zone, with wet/moist/dry bands and its own verdict ("Soil is still wet, no need to water yet").
  - Each zone shows its flow rate ("0.4 m³/h · moderate flow"), soil moisture and soil temperature, with run buttons for 5 m, 15 m, 30 m, 1 h and 2 h.

  The pattern of a plain-language verdict on top of the numbers would work for any decision card we build.
- **RED-61. Household calendar (screenshot).** The Events page has month stepping, "Today" and "+ Add event", and a color legend for each calendar (Birthdays, each parent, House maintenance, New Zealand holidays). Events are listed by day. Bin days and chores are all-day events ("Recycle", "Rubbish", "7:00 pm Bring the purple library bag"). The add form has a title, a calendar picker, an all-day switch, start and end dates and times, repeat, location and notes. Those are exactly the fields our new-event sheet is missing (CAL-24).
- **RED-62. Fuel car (screenshot).** Fuel %, range, odometer, lock, ignition and park brake; an eco score with rings for acceleration, constant speed and coasting; trip stats (km, km/h average, L/100 km); and a grid of windows, sunroof and boot, with an unlocked boot highlighted in red. This extends RED-41 to a fuel car.
- **RED-63. Theme picker (screenshot).**
  - Four styles: Default; Liquid glass ("iOS style: frosted, translucent layers"); Material ("Google style: soft tonal surfaces, pill buttons"); Minimal ("flat, thin lines, no effects").
  - Six fonts: Plus Jakarta Sans, Roboto Flex, Roboto Flex with Roboto Mono labels, Outfit, Manrope, and "Classic", the previous look.
  - Dark, light, or follow the device.
  - Twelve accent colors (violet, blue, teal, green, indigo, cyan, lime, amber, orange, red and more).

  The video shows the whole UI re-tinting as soon as an accent is picked. The OP was "doing final checks on the colour visibility on all the themes" (OP comment). This goes well past RED-45 and FOS-54.
- **RED-64. Color and motion from live data.** The post says there are many animations, and that color follows the data (temperature, on, off, playing). In the video, the weather card animates rain, room and climate cards are tinted by temperature, and lit light tiles glow in their light's color.
- **RED-65. Phone, desktop, then tablet.** One page that changes layout: a bottom bar plus a "More" grid on phones, a full side bar on desktop (screenshot). The tablet version is still to come. Compare RED-28 and question 3.
- **RED-66. Hosting, sign-in and build (OP comment).**
  - **Files:** two files in HA's `www` folder, served at `/local/dash.html`. `dash.html` is the "engine": look, behavior and a version number. `config.js` holds the HA token, pages, rooms, devices and helper entities.
  - **Connection and speed:** the page runs in the browser on a phone, tablet or PC and connects straight to HA over the WebSocket. It loads about 3 seconds faster than opening the dashboard in the companion app. The OP keeps the companion app only for notifications.
  - **Access:** it is reached through Nabu Casa (HTTPS, no open port). The OP called access the weak point, then changed the page to send people to HA's own login page first. The token is saved on each device and cleared when the user logs out.
  - **Security note:** files under `/config/www` are served at `/local/` without authentication. A long-lived token kept in `config.js` would be readable by anyone who can reach the HA address. Signing in through HA's login flow, which the OP moved to, avoids that. This applies to RED-47 too.
  - **Build:** made in one weekend with Claude Pro, migrating the OP's YAML dashboards. Claude was never connected to HA: it wrote code, tested it against a fake HA in a sandbox, and the OP pasted files and tokens across by hand. Anything that touched real devices (like Zigbee2MQTT light topics) had to be tried by the OP. A commenter pointed to an HA add-on that runs Claude in the browser inside HA, and the OP put it on their to-do list. The OP may share a cleaned-up version later.

## 18. Questions for the Home HQ plan

1. Which screens stay? Agents is a personal developer tool, not a family one. **Agents ruled out: ADR-0015. The rest of the screen list is still open.**
2. Do we build on FamilyOS (run it, fork it, or write extensions for it) or copy its ideas into our own app? It is AGPL-3.0, beta, Svelte, and built for read-only displays, while `ha-dashboard` is React and talks to Home Assistant directly. **Decided: ADR-0001.**
3. Is Home HQ one app on both the wall frame and phones (like `ha-dashboard`, which targets phones first), or a wall-only app? **Decided: one app for phones, tablets and the wall panel (ADR-0002, ADR-0013).**
4. Does the wall screen stay interactive (tick chores, edit lists, unlock doors), or become read-only as FamilyOS requires? **Decided: ADR-0006.**
5. Where does shared state live (chores, stars, shopping, photos, favorites), given the prototype has no backend? **Decided for app data: ADR-0008. Photo storage: ADR-0025 (photos stay in Immich; favorites, hides and the photo of the day live in Home HQ).**
6. Which calendar provider holds the family's events, and how do events map to people? **Decided: ADR-0024 (Google Workspace, view-only, one Family calendar plus busy blocks from primary calendars).**
7. Which FamilyOS areas matter to us: tasks, kitchen and meal plan, household records, the board, Quick add, the assistant, MCP access? **Decided: ADR-0015.**
8. Do kids use stars, a rotation, or both? Do chores need recurrence, a daily reset and star redemption on day one? Do kids' changes need an adult's approval? **Decided: ADR-0016 (no kid role; only deletes of others' items need approval) and ADR-0023, which supersedes ADR-0022 (no stars; chores and family tasks are one list of tasks, with rotation, recurrence and overdue roll-over).**
9. Where do photos come from, and who curates them? **Decided: ADR-0025 (one shared Immich album that every family member adds to).**
10. Which devices does the house have for each Home control tile, alert and status line? Is there an alarm system, a water valve, a doorbell camera with two-way audio? **Home control decided: ADR-0026 (no locks, covers, valve or doorbell; the garage becomes an HA template cover; cameras through a Wyze bridge). Alerts and status lines decided: ADR-0027 (no takeovers; garage and parcel banners; parcels from Mail and Packages in HA).**
11. Should the wall display get its own credentials and permissions (for example no unlock without a PIN) separate from phones? **Decided: ADR-0005, ADR-0006.**
12. Do we keep the 1280x800 fixed stage and the Chromium 106 limits, or does the frame change? **Decided: ADR-0013.**
13. Is the Today hero a family photo (TODAY-14) or an illustrated scene that reacts to weather and the family's schedule (RED-01)? If it is a scene, where does the art come from? The frog panoramas are Google's artwork reposted on GitHub, so we would need to check whether we can use them. The OP made extra scenes in the same style with an image model.
14. Should short-lived things (timers, now playing, parcels, the school card) show as cards over the hero (RED-04) instead of as sheets and banners? **Timers partly decided: ADR-0028 (Time's up is a banner; where running timers sit is still open). Now playing and the school card are still open.**
15. Is there a presence sensor in the kitchen, so the screen can wake, dim and go to ambient by room occupancy (RED-24, RED-25) and not only by touch? **Decided: ADR-0029 (no new sensor; the wall panel, which is in the entry, wakes and sleeps by its own camera through Fully Kiosk, and dims in the evening).**
16. Which LLM features do we want (dinner ideas, reading a menu photo into data, finding events), and do they write data directly or propose changes for an adult to approve, as FamilyOS requires? **Decided: ADR-0030 (quick add and dinner ideas on the Claude API; the AI only suggests, and the sender adds what they want; no assistant chat or search by meaning).**
17. Should automation settings (thresholds, schedules, freeze) be adjustable from the wall (RED-22), or stay in the HA editor? **Decided: ADR-0026 (they stay in the HA editor; an HA script used as a scene can pause one).**
18. How is Home HQ hosted: as its own static app (NFR-13), as custom cards inside HA's frontend (RED-29), or as a standalone page served by HA over its WebSocket API (RED-47)? **Decided: ADR-0002, ADR-0004, ADR-0014.**
19. Which household-specific extras earn space on a family screen? The two Reddit dashboards each carry things only their owners care about (planes, cars, network health, a watch pick), much like our Agents tab. **Decided: ADR-0015.**
20. If Home HQ is a page served by HA, how does each device sign in? Files in `/config/www` are public, so a long-lived token can't live in the page's files (RED-66). The options are HA's login flow on each device, or something like FamilyOS's display pairing (FOS-10). **Decided: ADR-0005 (Home HQ is not served by HA; members and kiosks sign in with HA OAuth).**
