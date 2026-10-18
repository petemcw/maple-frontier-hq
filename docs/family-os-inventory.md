# FamilyOS feature inventory (input for the Home HQ requirements)

- **Source:** `/Users/prm/Projects/ha/family-os` (FamilyOS / "FamOS", AGPL-3.0, self-hosted). All paths below are relative to that folder.
- **Method:** I read the source, migrations, web UI, message catalogs, tests and docs. Nothing was installed, built or run, and nothing in the repo was changed. Each claim points to the file it comes from. Where the user guide and the code disagree, the code is treated as the truth and the difference is noted.
- **Compiled:** 2026-10-12.

Each feature section uses the same subsections: *What the user can do*, *Data model*, *Rules and logic worth copying*, *Integrations*, *Today / display relevance*, *Docs vs code*, *Unfinished or stubbed*, *Beyond Home HQ (net-new ideas)* and *Evidence*.

---

## 1. At a glance

### What FamilyOS is

FamilyOS is a single-container, self-hosted family organiser built for modest hardware such as a Raspberry Pi. One installation serves one family. It is a SvelteKit PWA (phone, desktop and paired wall display) over a Hono API on Node 24, with SQLite (Kysely) as the only database. Every feature is a first-party extension compiled in at build time. Five principles shape the product. Each one is a requirement decision Home HQ should make explicitly:

1. **AI proposes, people decide.** No model changes data on its own. Every AI change, and every change by a Supervised member, becomes a *draft* that a person approves. Approved drafts carry an *undo* (a declared inverse action).
2. **Access is capabilities, not role names.** Each feature declares `view` / `use` / `edit` / `manage` capabilities. Roles (Admin, Member, Supervised, Guest, plus custom roles) map those capabilities to levels. "Propose instead of refuse": a person without the edit right can still suggest, and an adult approves.
3. **Wall displays are view-only devices.** A display is paired by code, holds only the `view` grants an admin picks for it, never gets PIN sign-in or profiles, and cannot run actions. By default it hides names ("hallway privacy").
4. **Region-neutral and language-neutral.** The code holds English only. Languages and colour palettes are installable data packs, and date/number formats follow the family's region, separately from each person's language.
5. **Smart-home control is explicitly out of scope** (`docs/architecture.md` §1, "Home Assistant already does it well"). The `home` extension covers household *records* (warranties, utilities, upkeep), not devices.

### Home HQ prototype vs FamilyOS

| Home HQ prototype feature | What FamilyOS has | Section |
|---|---|---|
| Today overview | Today page with a "next thing" countdown hero (family's or mine), one lane per person, one-clause status per person, chore rings, a weather sentence that names the event the rain overlaps, "Needs you" cards, photo of the day, evening flip to tomorrow. Three styles: Broadsheet, Lanes, Wall | 2 |
| Calendar: Today by person / 3-day / week / month | Calendar with per-person colours, tags, repeats (daily, weekly on chosen days, monthly), one reminder per event, ICS subscriptions in (6 h refresh) and secret per-person ICS feeds out. Phones have no Day view or person filter | 3 |
| Chores with star rewards and time-of-day routines | Rotating chores (fixed, take turns, least done in 90 days), "anyone may do it", turn-aware permissions, credit to whoever's turn it was, rings on Today. **No points, stars or rewards; no time-of-day routines** | 5 |
| Grocery list with stores, categories, usuals | Several named shopping lists, merge-on-add (no duplicates; bought items come back), remembered section per product, one-line parser for many items and amounts, "in the cart" state, offline taps. Plus a separate general **task list** | 4 |
| Photo slideshow | Uploaded family photos, a deterministic "photo of the day", and ambient mode (idle slideshow) on displays. No albums, captions or per-display photo sets | 2 |
| Smart-home controls | Out of scope by design | n/a |
| AI-plan usage meters | Admin-only **Usage & limits** page (`/settings/ai/usage`): calls, errors, input/output tokens and USD per calendar month (family time zone), by feature and by model, against a family-wide monthly USD budget and/or call cap that pause all AI when reached. A metadata-only **activity log** of every call (`/settings/ai/activity`). Cost is recorded only when the provider reports it (in practice OpenRouter). Ollama counts as 0. Not shown on Today or on displays | 11 |
| Ambient / night modes | Ambient mode is idle-triggered. Day/night follows the device's `prefers-color-scheme` (the docs say "follows the sun", the code does not). No dimming, screen-off schedule, burn-in protection, wake lock or fullscreen | 2, 14 |
| Doorbell / leak takeovers | None. The only display "interrupt" is a pill showing how many suggestions are waiting (count only) | 2 |

### Candidate additions for Home HQ (things FamilyOS has that the prototype lacks)

Grouped by theme. Each is detailed, with evidence, in its section.

- **Capture and AI**
  - *Quick add*: one inbox for text, photos, PDFs and voice notes from the app, the phone share sheet, a Telegram bot or a forwarding mailbox. The AI turns them into pre-ticked draft cards: events, tasks, shopping items, chores, recipes, meal-plan entries, board notes, home/pet/vehicle records and their duties, and memory notes. Papers can be filed onto a record without the AI reading them. Tick and add, undo per batch, draft expiry for date-bound suggestions, a per-field "Check this" for uncertain fields.
  - *Scoped auto-approval* for routine kinds (shopping items, tasks). It approves "as" the admin who enabled it and switches itself off when that admin loses rights.
  - *Assistant chat* that can only propose (every change is a card with Undo). It has a name-matching guard, per-person private memory and thread continuity, and can be reached from Telegram.
  - *Semantic family search* across captures, recipes, home items, events and records, with per-person visibility.
  - *Bring-your-own-agent MCP server* with personal app keys (read, or read-and-suggest; expiring; revocable). An agent never sees more than its member.
  - *Per-area AI sandbox* (Off / Local only / Any provider), with "Local only" enforced by checking that the provider's address is on the home network.
  - *AI providers*: one OpenAI-compatible client (Vercel AI SDK) for OpenRouter, Ollama or any compatible server. There is no native Anthropic or Google client. Providers map to model roles. Keys are sealed with `FAMOS_SECRET`. Monthly budget and call caps apply.
- **Household records**
  - Vehicles, pets and the home as records with recurring *duties* (insurance, inspection, vaccinations, warranty end, upkeep), reminders some days ahead and on the due day, and filed *papers*. Repeats count from the first due date, and late completion skips missed periods.
  - Home: things with warranty dates kept in sync, plus utilities/services. Pets: treatment history. Vehicles: service log.
- **Family board:** short notes with an optional photo, colour and end day. Notes expire automatically, admins can pin them, the family can drag to reorder, and "seen by" receipts replace reactions. Shown on Today and on displays.
- **Kitchen:** a recipe box, a weekly meal plan (breakfast / lunch / dinner / other) and "ingredients to the shopping list" with a robust ingredient-line parser. Today's meals appear on the display's day view.
- **Tasks:** a family to-do list separate from chores, with assignee or "Anyone", due date and time, repeats that skip missed dates, and due-time reminders. Timed tasks appear on Today's timeline.
- **Calendar sync:** a secret per-person ICS feed out ("my events plus family events" in the phone's own calendar app; hashed tokens shown once; per-device label; last used; revoke), and read-only ICS subscriptions in (school, holidays, Google/iCloud secret addresses) with colour, member and reminder switches.
- **Displays:** device-code pairing (RFC 8628 style, QR deep link, single-use hashed token in an httpOnly cookie, five wrong codes end all open pairings). Per-screen view-only grants, a hallway-privacy default, a never-scroll fit planner that drops content in priority order, per-screen style/palette/hidden blocks, and a live-update model (id-only events, a 60 s self-check and a 5-minute full re-read).
- **People and access:** Supervised members whose changes become drafts, PIN sign-in only on admin-marked family devices (lockout per account), custom roles as a base role plus overrides, and Guests.
- **Notifications:** in-app records rendered per recipient language, web push (own VAPID / Web Push encryption) and Telegram with approve/reject buttons. Quiet hours per person *defer* pushes rather than drop them. A notification already read in the app suppresses the push. Approval prompts clean themselves up once a draft is decided.
- **Running it:** nightly `VACUUM INTO` backups and a backup before each upgrade, optionally age-encrypted and copied to S3 or WebDAV. Offline queue for shopping, tasks and chores (idempotency keys and IndexedDB). Installable language and theme packs, with a WCAG contrast gate on themes.

### Doc/code mismatches and likely bugs worth knowing

These show where FamilyOS's own spec drifted from what it built. They are useful as "decide explicitly" items.

- Displays: the docs and UI say screens "follow the sun"; the code follows the device's dark-mode setting. The photo hint says "same month in another year"; the code rotates through all photos. The 52 px display touch targets are never applied. A display that starts while the server is down seems never to retry. The weather credit is a tappable link on displays. (§2)
- Calendar: saving a series from the UI brings back dates that were skipped (likely bug). The feed scope choice is gated by a manage capability, not "admins". Phones lack the Day view and the person filter. (§3)
- Chores: `docs/guide/chores.md` says the AI "never ticks chores off", but the code lets it propose `markDone` as a draft. (§5)
- Records: Today shows 4 duties (the docs say 5). Undoing a pet treatment leaves its duty advanced. Vehicle log entries never complete duties. Home services reach family search despite "never sent to AI". (§7)
- Board: no character counter, and a note's photo cannot be removed. (§8)
- Roles: a role once held by a removed member can never be deleted. The password route does not check the role, so a demoted adult keeps a working password. Sign-out leaves push subscribed on shared tablets. Broadcast reminders also reach Guests. (§12, §13)
- Notifications: `notifications.md` leaves out board notifications. Notifications are never pruned. The 09:00 reminder time is hard-coded. (§13)
- AI: the monthly **call** limit also counts failed calls and MCP tool calls (which call no model), so an active MCP agent can use up the family's AI budget. The **USD** budget never trips for direct OpenAI, Groq and similar providers, because their cost is recorded as null. With no model set, text typed to the Telegram bot does not reach Quick add. The `fast` model role is never used. The "Finance" area named in the docs does not exist, and no bundled extension is marked sensitive. Expired but unrevoked app keys still count toward the 20-key cap. (§9, §11)

### Screen map (web routes)

- Core routes in `apps/web/src/routes`:
  - `/` (Today), `/display` (paired screen), `/login`, `/setup`, `/notifications`, `/connections`, `/more`, `/more/[section]`, `/assistant/memory`, `/calendar/settings`.
  - Settings: `/settings/` → `account`, `profile`, `members`, `roles`, `devices`, `features`, `notifications`, `region`, `languages`, `appearance`, `theme`, `display-content`, `photos-weather`, `email`, `telegram`, `connected-apps`, `auto-approval`, `backups`, `ai`, `ai/usage`, `ai/activity`, `ai/privacy`.
- Pages contributed by extensions (`extensions/*/src/web/index.ts`):
  - `/add` and `/share` (Quick add and the share target), `/assistant`, `/board`, `/calendar`.
  - `/chores`, `/chores/:id`, `/lists`, `/lists/shopping`, `/lists/tasks`.
  - `/kitchen`, `/kitchen/plan`, `/kitchen/recipes/:id`.
  - `/home`, `/home/:id`, `/home/items/:id`, `/home/services/:id`, `/pets`, `/pets/:id`, `/vehicles`, `/vehicles/:id`.
  - `/settings/calendars`, `/settings/calendar-feed`, `/settings/memory`.
- Slots extensions fill:
  - `dashboard.card` (Today cards from assistant, board, chores, inbox, lists, records);
  - `display.widget` (board, lists, records); `display.ambient` and `today.atmosphere` (ambient); `display.status` (inbox waiting count);
  - `today.capture` and `nav.action` (inbox); `record.section` (records); `settings.display` and `setup.step` (ambient).

---

## Contents

2. Today, family displays (kiosk) and ambient
3. Calendar
4. Lists (tasks and shopping)
5. Chores
6. Kitchen
7. Household records: records, home, pets, vehicles
8. Family board
9. Quick add (inbox), drafts and approval (including Telegram and email)
10. Assistant and semantic search
11. MCP server, app keys and AI providers
12. Members, roles, permissions and sign-in
13. Notifications, quiet hours, region and settings
14. Appearance: styles, palettes, themes and design system
15. Platform: extensions and packs, real-time sync, offline, i18n, backups, jobs
16. Tech stack
17. Unfinished, stubbed or TODO (repo-wide scan)

---

## 2. Today, family displays (kiosk) and ambient

Three parts work together:

- **Today** (`/`) is the home page for a signed-in person.
- **The family display** (`/display`) is a browser paired by code. It shows the same Today composition in "family mode": no person, view-only, and only what an admin granted.
- **The `ambient` extension** owns everything a display pulls in that is not family data: uploaded photos, the Open-Meteo weather, the weather sentence, and ambient mode (the idle slideshow).

There is one data layer, `TodayData` (`apps/web/src/lib/today/today-data.svelte.ts`). Each of the three styles (Broadsheet, Lanes, Wall) has a composition component that only arranges what `TodayData` provides (`apps/web/src/lib/today/compositions.ts`).

### What the user can do

**On Today (a person, `apps/web/src/routes/+page.svelte`)**

- **Head.** A clock and date, and a one-sentence greeting that summarises the day, for example "Good morning, Deniz. **3 things need you**, …" (`TodayData.sentence`, `composeSentence` in `apps/web/src/lib/today.ts`). The weather sits in the head too, through the `today.atmosphere` slot.
- **The next thing (the "hero").** It sits on the theme's single accent surface and shows:
  - a countdown;
  - a pill reading "Until 16:30", "At <place>" or "Starts 15:45";
  - a toggle between **the family's** next thing and **mine**, saved per person by `core.setMyNextUp` (`packages/core/src/next-up.ts`).
- **The rest of the day by person.** There is a column, lane or row per member and one for "anyone". It holds all-day and timed items, but not meals, chores or untimed tasks (`memberDays` in `packages/ui/src/famos/today-v3.ts`).
- **Where everyone is.** Each person gets a one-clause status, such as "Piano lesson in 25 min", "Football 17:00", "Piano lesson until 16:30", "1 chore left" or "Nothing planned" (`statusOf` / `statusText`, `today-v3.ts`).
- **Chores as rings.** Each member gets a done/total ring, plus an "anyone" ring when anyone-chores exist (`TodayData.rings`). Lanes uses bars instead of rings (`lanes/ChoreBars.svelte`).
- **The weather.**
  - A big temperature, the sky's state, and the high/low with the rain chance.
  - A sentence linking the weather to the day's timed items, for example "Rain from 15:00, during Football."
  - The next hours as tiles (Broadsheet), bars (Wall) or a row on the chart's time axis (Lanes).
  - An animated sky behind the block (`extensions/ambient/src/web/TodayWeather.svelte`, `packages/ui/src/famos/WeatherBlock.svelte`).
  - When the weather is off, a person holding `ambient.manage` sees a "Set your city to see the weather" link instead.
- **A photo of the day.** One uploaded photo per family day, labelled "From the photos · <month added>" (`packages/ui/src/famos/PhotoTile.svelte`).
- **Needs you.** These are the `dashboard.card` contributions in the `needs` band:
  - Quick add suggestions waiting (`inbox.waiting`);
  - overdue tasks (`lists.overdue`);
  - overdue records (`records.overdue`);
  - an "AI not connected" row for admins.

  It is person-only and never shown on a display.
- **Other cards.**
  - Shopping as a number, a "glance" card (`lists.shopping`).
  - Tasks (`lists.tasks`) and the board (`board.today`).
  - Coming up (`records.comingUp`), Ask the assistant (`assistant.ask`) and today's meals (the kitchen timeline).
  - The full list of `dashboard.card` contributions comes from `extensions/*/src/web/index.ts`.
- **Next days.** On a light day (fewer than 3 items, `LIGHT_DAY` in `apps/web/src/lib/today.ts`), the page adds up to 4 items from the 6 days after the day shown (`aheadWindow`, `nextDays`).
- **Evening mode.**
  - It starts at 20:00 or when the last timed item of the day ends, whichever is later. Meal-plan items do not push it back (`eveningStart` / `isEvening`, `apps/web/src/lib/today.ts`).
  - From then on the hero disappears and the day shown becomes **tomorrow**.
  - A "done today" line lists what the day held.
  - The evening text is either "free tomorrow" or counts of today's and tomorrow's items.
- **Quiet day.** With nothing timed and nothing planned, the page shows a sentence such as "Nothing is on the calendar today (or in the coming week)."
- **Hide parts.** Any block can be hidden from the **⋯** at the end of its label ("Hide from Today", with Undo) or from Settings. See section 14.
- **New families** see first steps and a capture box (Quick add inline).

**Admins, for displays (`Settings → Family → Devices`, `apps/web/src/routes/settings/devices/`)**

- **Add a device → "A screen for the wall or a TV".** This opens four guided steps (`AddDevice.svelte`):
  1. Open `<origin>/display` on the screen. A QR code of that address is shown.
  2. Type the code the screen shows (`ABCD-EFGH`).
  3. Name the screen and untick what it must not show. Everything is ticked at first; "Show names and colors" is off at first.
  4. Done.
- **Pair from the screen.** The screen shows its own QR code, of `<origin>/settings/devices?code=XXXX-XXXX`. Scanning it on a phone opens Add a device at step 3 with the code already filled (`PairScreen.svelte`, `devices/+page.svelte`).
- **Edit or remove a screen.** Tap its row to rename it, change its grants or toggle names. **Remove** asks for a second tap, then revokes the screen at once.
- **Per-display look**, under `Settings → Family → Theme → Displays` (`apps/web/src/routes/settings/theme/+page.svelte`):
  - a Style (Broadsheet / Lanes / Wall);
  - a Theme;
  - "What Today shows" (hidden blocks).

  Each falls back to "Same as the family".
- **Photos and weather** (`Settings → Family → Photos and weather`, the `settings.display` slot rendered by `extensions/ambient/src/web/DisplayContent.svelte`):
  - **Photos.** Upload JPEG/PNG/WebP. Each picture is shrunk in the browser to a 1920 px longest side and re-encoded as JPEG, which drops EXIF and location. Photos show in a grid, and "Remove" asks "Remove for good?".
  - **Weather.** A switch, plus a place search through Open-Meteo geocoding ("Use this place").
  - **Ambient timing.**
    - "Start after (minutes)": 1–240, default 5.
    - "Each photo stays (seconds)": 5–600, default 30.
    - Both are saved when the field loses focus.
- **First-run setup** offers "Where do you live?" (the weather place), which can be skipped (`SetupPlaceStep.svelte`, the `setup.step` slot).

**On the screen itself (`apps/web/src/routes/display/`)**

- **Unpaired.** The screen shows "Connect this screen" with the same four steps from its side: a large code, a 10-minute countdown, the address and a QR code. A new code is fetched automatically when the old one expires.
- **Paired.** It shows Today in family mode in its own style, palette and hidden-block choice:
  - one type step larger;
  - never scrolling;
  - status pills next to the clock (the waiting-suggestions count);
  - a band of widgets at the foot: board notes, open tasks and records "coming up".
- **Ambient mode.** After N idle minutes the screen shows a full-screen photo slideshow (or the animated weather sky), the clock, the date, the next events today and the weather. Any touch, pointer or key press brings Today back.
- **A browser where a member is signed in** (or a trusted family device) refuses to become a display. It shows "This browser is used by the family" and checks again every 10 s.

### Data model

**Core tables** (`packages/core/src/migrations/0001_init.ts`, types in `packages/core/src/tables.ts`)

- **`core_devices`.** Family devices and displays share this table.

  | Column | Meaning |
  |---|---|
  | `id` | Primary key |
  | `name` | Up to 60 characters |
  | `kind` | CHECK `'family' \| 'kiosk'` |
  | `token_hash` | Unique; the SHA-256 hex of a 256-bit token |
  | `created_by` | Member id |
  | `created_at` | Epoch ms |
  | `last_seen_at` | Epoch ms; updated at most hourly |
  | `revoked_at` | Null while active |
  | `display_capabilities` | Kiosk only: a JSON array of the granted view-capability ids |
  | `display_show_names` | 0/1 |

- **`core_display_pairings`.** Short-lived rows in the RFC 8628 style.

  | Column | Meaning |
  |---|---|
  | `id` | Primary key |
  | `device_code_hash` | Unique |
  | `user_code` | `XXXX-XXXX`, indexed |
  | `expires_at` | Creation + 10 minutes |
  | `attempts` | Wrong codes counted against this pairing |
  | `status` | `pending \| confirmed \| consumed` |
  | `name`, `capabilities` (JSON), `show_names` | What the admin chose at confirm |
  | `confirmed_by` | The confirming admin |
  | `device_id` | Set once the display collects its token |
  | `created_ip`, `created_at` | Where and when the pairing started |

- **`core_settings`** is a key/value store of JSON values (`packages/core/src/themes.ts`, `next-up.ts`). The keys that matter here:

  | Key | Shape |
  |---|---|
  | `themes.choices` | `{ family, members: {id: theme}, displays: {deviceId: theme} }` |
  | `styles.choices` | The same shape, for styles |
  | `today.hidden` | `{ family, members, displays }`, each an array of hidden block ids |
  | `today.nextUp` | `{ members: { memberId: 'me' } }`; only members who chose "me" are stored |
  | `themes.installed`, `themes.fetch` | Installed packs and the state of a pack fetched from an address (section 14) |

**Ambient tables** (`extensions/ambient/src/server/store.ts`, migration `0001_init`)

- **`ambient_settings`.** A single row, `id = 'main'`.
  - `data`: JSON validated by `StoredSettingsSchema`, shaped `{ weather: { enabled, place: { name, latitude, longitude } | null }, timing: { idleMinutes, slideSeconds } }`.
  - `version`: bumped on every change; it invalidates the in-memory caches.
  - `updated_at`.
- **`ambient_photos`.**
  - `id`: an `amb_photo_…` UUIDv7.
  - `file_key`: `ambient/photos/<amb_file id>.<jpg|png|webp>` in the file store.
  - `content_type`, `width`, `height`, `byte_size`.
  - `uploaded_by`: a member id or null, with no cross-extension foreign key.
  - `created_at`, indexed.

**Not persisted**

- **Weather cache.** An in-memory map per extension context. Its key is rounded lat,lon plus the unit; it lasts 30 minutes, de-duplicates requests in flight, and is dropped when the settings `version` moves (`caches.ts`).
- **Pairing poll timestamps.** An in-memory map used for `slow_down` (`devices.ts`).
- **On the device (`localStorage`).**
  - `famos.mode`: `system` / `day` / `night`.
  - `famos.theme`: `{ id, version, css }`, the last stylesheet.
  - `famos.style`: the last style, applied before first paint by the inline script in `apps/web/src/app.html`.

**The display actor** (`displayActor`, `packages/core/src/devices.ts`): `{ kind: 'display', deviceId, capabilities: string[], showNames: boolean }`.

**`GET /api/v1/core/displays/me`** gives the display page everything it needs (`apps/server/src/http/devices.ts`):

- its name;
- its effective capabilities (filtered to extensions that are on, plus `core.members.view` when names are shown) and `showNames`;
- the family's time zone, locale, format locale, week start, hour cycle and temperature unit;
- the extensions that are switched off;
- the resolved `theme`, `style` and `hidden` blocks.

### Rules and logic worth copying

**Pairing security** (`packages/core/src/devices.ts`, `apps/server/src/http/devices.ts`)

- **Two codes.**
  - A **device code**: 32 random bytes in base64url. Only its SHA-256 is stored, and only the display ever holds it.
  - A **user code**: 8 characters from `ABCDEFGHJKMNPQRSTUVWXYZ23456789` (no 0/O or 1/I/L), drawn by rejection sampling so every character is equally likely. It is shown as `XXXX-XXXX` and typed on a phone. Input is normalised (case, spaces, dashes) by `normalizeUserCode`.
- **Starting a pairing is public, with limits:**
  - 10 starts per IP per 10 minutes (`createRateLimiter`);
  - at most 100 open pairings server-wide;
  - expired rows are pruned on each start;
  - a new user code is redrawn (up to 5 tries) if it clashes with an open one.
- **Polling.**
  - The display polls every 5 s. A poll less than 4 s after the previous one answers `slow_down`, and the client adds 5 s to its interval.
  - Answers are `pending`, `expired` or `paired`.
  - **Exactly one poll wins.** A conditional `UPDATE … WHERE status='confirmed'` moves the row to `consumed`, then creates the `kiosk` device with a fresh 256-bit token.
  - The token only ever travels as the `famos_device` cookie: httpOnly, SameSite=Lax, Secure on https, max-age 400 days, refreshed on every request. It never appears in a JSON body.
- **Confirming.**
  - It needs a member session with `core.devices.manage`. That capability is `critical`, so only the Admin role holds it.
  - The code must be pending, unexpired and have fewer than 5 attempts.
  - **A wrong code increments `attempts` on every open pairing.** Five misses in total therefore end all open codes, which makes guessing pointless. The screens simply fetch new codes.
- **Grants are validated on the server** (`checkGrant`):
  - only `view`-level capabilities;
  - only from extensions that are switched on;
  - nothing from core except `core.drafts.waiting` (a count only).

  Names are a separate flag.
- **Two conflict guards (409):**
  - pairing refuses a browser that has a member session or a family-device cookie (`refuseFamilyBrowser`);
  - "trust this device" refuses a browser that is already a paired display, so the display cannot be silently unpaired.

**View-only enforcement, in layers**

1. **Actor resolution** (`resolveActor`, `apps/server/src/http/app.ts`).
   - A member session always wins over the device cookie.
   - Otherwise the device is looked up on **every request**, and the actor is a display only if `kind='kiosk'`, `revoked_at IS NULL` and the grants are non-null.
2. **`access.can`** (`packages/core/src/access.ts`). For a display it returns false unless:
   - the capability's level is `view`;
   - it was granted;
   - its extension is on.

   `core.members.view` depends on `showNames` only.
3. **The action runner refuses every action from a display,** even read-only ones (`packages/core/src/action-runner.ts`, `authorize`).
4. **Route access levels.** `signedIn` and `trusted-device` routes refuse displays, and `public` routes treat a display as an anonymous caller. The device middleware sets `device` only for `kind='family'`, so a kiosk never opens the profile picker or PIN sign-in.
5. **Private realtime events** (assistant answers) go only to the owning member's sockets, never to displays (`apps/server/src/http/realtime.ts`).
6. **The client does the same.** The display page provides an `AppContext` in which:
   - `runAction` always rejects, `toast` is a no-op, `me()` is null and `isAppPath` is false;
   - `TodayData` in family mode strips every `href`, and mounts only `dashboard.card` contributions marked `glance: true`;
   - person-only blocks (greeting, Needs you, first steps, the quiet line, the capture box, the next-up toggle) are never rendered.

**Privacy mode: "show names" off (the default)** (`TodayData`, `#forFamily`, `rings`)

- Items lose `members` and `subtitle`, so nothing names a person.
- The day is one column, lane or row.
- Chores become one "Family" total ring.
- The hero says what and when, not for whom.
- No member list is fetched: `GET /core/members` would be refused anyway without `core.members.view`.
- The page re-creates `TodayData` when the flag changes (`{#key me.showNames}`), so names loaded earlier never linger.

**Revocation and grant changes propagate at once**

- Remove sets `revoked_at`, ends the sessions on that device and publishes `core.device.changed`.
- The realtime hub first broadcasts the event, then closes that device's sockets with code 4401.
- The display handles the event by calling `/displays/me`. On 401 it returns to the pairing screen with a fresh code.
- A 60 s self-check is the fallback if the event is missed.
- Changing grants takes the same path: the reconnecting socket gets the new rights.

**Fit-to-screen planner: a display never scrolls** (`apps/web/src/routes/display/FamilyToday.svelte`, `apps/web/src/lib/today/display-fit.ts`)

- The composition renders hidden (`visibility: hidden`). It is measured once it has settled and its size has been quiet for 200 ms (capped at 1.5 s).
- Plans are tried in order. Each one is applied with `flushSync` and measured in the same task, so nothing in between is ever painted. Each plan is also tried a second time with `shift` (the style's smaller blocks moved to the column with room).
  1. `tight` 0, 1, 2: fewer items per person.
  2. Keep only one widget of the foot band (each in turn), then none.
  3. Drop blocks in order: `photo` → `atmosphere` (the weather's hours, the status list) → `extras` (shopping, kitchen, next days, ahead) → `chores`.
  4. `tight` 3.
  5. Zoom down in steps of 0.0625, never below 1 (a person's own size).
- After a fit is found, the planner relaxes `tight` again where the freed room allows.
- It re-runs on ResizeObserver and window resize events (a rotated tablet, a live update).
- The chosen plan is exposed as `data-fit` for tests and screenshots.
- Each style reads `tight` with its own limits (`byTightness`):

  | Style | Limit | Values for tight 0 → 3 |
  |---|---|---|
  | Broadsheet | Items per person | 3, 3, 2, 1 |
  | Broadsheet | Items for anyone | 9, 6, 4, 3 |
  | Wall | Day list (names off) | `DAY_LIST_MAX`, 5, 4, 3 |
  | Lanes | Rows per lane (names on) | 2, 2, 1, 1 |

  When a list is cut, items that have already ended leave first (`endedFirst`), and the rest show as "+n more".
- The base zoom gives the larger type: 1 below 1024 px, 1.125 from 1024 px, 1.5 from 1800 px (`baseZoom`).

**Next-up selection and countdown** (`packages/ui/src/famos/today-v3.ts`)

- **Which item.** The hero is the first timed, not-done item that **starts after now**. Only if none is left today, it is the item under way (start ≤ now < end).
  - "Me" mode keeps items where the person is a member or that have no members.
  - Without a person (a display), the mode is always the family's.
  - There is no hero in the evening.
- **The countdown:**

  | Time to start | Shown as |
  |---|---|
  | Under 60 minutes | Minutes, rounded up so it never says 0 before the start: "25 min" |
  | 60 to 119 minutes | A compact duration: "1 hr 46", "to go" |
  | 120 minutes or more | The start time, "starts" |
  | Started | "now", with "until <end>" |

  Wall draws it inside a ring that empties over the last hour (`Countdown.svelte`).
- **The pill:** "Until <end>" once started, "At <place>" if there is a location, "Starts <time>" unless the countdown already shows the time (`pillOf`).
- **The accessible name** reads "Piano lesson, 15:45, in 25 minutes".

**Person status** (`statusOf`). The first match wins:

1. The next timed item: "now/until", "soon" (under 60 min, as "in N min") or "at <time>".
2. An all-day item.
3. Chores left.
4. "Nothing planned".

Chores never join another clause; the rings carry them.

**Weather sentence: a template, not an LLM** (`extensions/ambient/src/contract/sentence.ts`)

- The function is pure. Its inputs are the hourly forecast, today's timed items (title and start only, read through the calendar's route), now, and the family's formats.
- It picks one of 10 catalog keys and returns pre-formatted parameters. The rules, in order:
  1. **Evening.** From 20:00, or from 17:00 once the day's last timed item has started: "{{temp}}° now, down to {{min}}° tonight". The night runs to 06:00.
  2. **Wet window.** The first run of hours that have a rain/snow/storm WMO code or a precipitation probability of at least 50%:
     - any storm hour → "Storms from T";
     - snow hours ≥ rain hours → "Snow from T";
     - wet now and to the end of the day → "Rain all day, max°";
     - wet now but ending → "Rain until T, then <clear|partly cloudy|cloudy|foggy|dry>";
     - later, and a timed item starts inside the window → "Rain from T, during <title>". The link is **by time only**; it never claims the item is outdoors.
     - otherwise → "Rain from T until E".
  3. **Cold morning.** Before 10:00, if the day warms by at least 8 °C (14 °F): "min° now, max° by T".
  4. **Dry day.** If every remaining hour is clear (codes 0 or 1): "Sunny and dry all day, max° by T". Otherwise "Dry all day, max° by T".
- Words live in the catalog (`extensions/ambient/src/messages/en.json`), so the template is translatable.

**Photo of the day** (`extensions/ambient/src/server/photo-of-the-day.ts`)

- The photos are taken in a stable order (sorted by id).
- "Rounds" of n days each get their own shuffle, seeded by the round number (mulberry32 + Fisher-Yates).
- Every photo shows once per round, and the same photo never shows two days running: if a round's first photo equals the previous round's last, the first two are swapped.
- With 1 or 2 photos the rotation is a simple modulo.
- The day is the family's local date, so the photo changes at local midnight. It is the same id all day on every screen.

**Slideshow** (`extensions/ambient/src/web/Ambient.svelte`, `server/photos.ts`)

- `GET /ambient/photos` returns up to 200 ids, freshly shuffled on each request, together with the timing.
- The client loads each image before showing it, so the crossfade (1.5 s; 0 under `prefers-reduced-motion`) never waits on the network.
- It also preloads the next image, and skips images that fail. It stops after one full lap of failures.
- **The clock face.** Without photos it shows a calm clock with the agenda. If weather is on, the animated `WeatherSky` (size `screen`) fills the background.

**Settling: "Today composes once"** (`TodayData` constructor)

- The head shows at once. The body waits until every first read has reported, then appears in one go:
  - every card's first report and the atmosphere's report;
  - the timeline and the look-ahead;
  - chores, the photo and the setup state.
- The wait is capped at 1.2 s (`SETTLE_CAP_MS`).
- Later live updates change the page in place.

**Refresh cadence** (all from code)

| What | Cadence | Where |
|---|---|---|
| Display self-check (`/displays/me`: grants, look, revocation) | Every 60 s; on `core.device.*`, `core.features.*`, `core.themes.*`, `core.family.*` events | `display/+page.svelte` |
| Full re-read (Today data reload, widgets and pills remounted) | Every 5 min (every 5th tick) | `display/+page.svelte`, `FamilyToday.svelte` |
| Members (when names are shown) | On every self-check and on `core.member.*` events | `display/+page.svelte` |
| Language pack | On `core.languages.*` events | `display/+page.svelte` |
| Display clock | 1 s (used by ambient mode) | `display/+page.svelte` |
| Today's clock and "now" (`TodayData.now`) | 15 s; the day rolls over from it | `today-data.svelte.ts` |
| Timeline and look-ahead | On a day change and on each contribution's `invalidateOn` events | `today-data.svelte.ts` |
| Chores, photo of the day | On `chores.*` / `ambient.*` events and on a day change | `today-data.svelte.ts` |
| Weather block on Today | Every 10 min and on `ambient.*` / `calendar.*` events. The server answers from its 30-min cache. | `TodayWeather.svelte`, `weather.ts` |
| Ambient mode content (photos, weather) | Every 8 min and on `ambient.*` events | `Ambient.svelte` |
| Ambient agenda | Every 60 s while active and on `calendar.*` events | `Ambient.svelte` |
| Widgets | Board every 5 min + `board.*`; tasks on `lists.task.*`; records on `records.*`; waiting pill on `core.draft.*` / `inbox.*`; each has an internal 60 s clock | `extensions/*/src/web/Display*.svelte` |
| WebSocket reconnect | Backoff from 1 s, doubling to a 30 s maximum. On reconnect, shared reads are forgotten and subscribers resync. | `apps/web/src/lib/realtime.ts` |
| App update (service worker) | Checked every 15 min and on visibility change. A waiting worker gets `SKIP_WAITING`; the page reloads once if its app version is older. | `apps/web/src/lib/sw-update.ts` |
| Pairing screen | Polls every 5 s (+5 s on `slow_down`); a new code on expiry; retries the start every 15 s on failure | `PairScreen.svelte` |

- Realtime payloads carry IDs only, and clients re-fetch.
- `sharedRead` de-duplicates identical reads that start within 2 s of each other.
- Every event and every write clears that cache first, so a stale answer is never reused (`packages/ui/src/famos/shared-read.ts`).

### Integrations

**Weather: Open-Meteo** (`extensions/ambient/src/server/weather.ts`)

- **Forecast request.** `https://api.open-meteo.com/v1/forecast` with:
  - `current=temperature_2m,weather_code,is_day`;
  - `daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max`;
  - `hourly=temperature_2m,weather_code,precipitation_probability`;
  - `timezone=auto`, `timeformat=unixtime`, `forecast_days=2` (so the evening can talk about the night);
  - `temperature_unit=fahrenheit` when the family's unit is Fahrenheit.
- **No API key.** The licence is CC BY 4.0, and the attribution "Weather data by Open-Meteo.com" / "Weather: Open-Meteo" is shown wherever the data appears.
- **Place.** Only one place per family is supported. Coordinates are rounded to 2 decimals (about 1 km) before they are stored or sent (`roundCoordinate`).
- **Geocoding.** `https://geocoding-api.open-meteo.com/v1/search?name=…&count=5&language=<family locale>`. Results are labelled "name, admin1, country". Searching needs `ambient.manage`.
- **Fetching.**
  - It goes through `ctx.http.get`, the SSRF-guarded port: 256 KB cap, 15 s timeout.
  - It runs **inside the route handler** (a read-through), not in a job.
  - Concurrent displays share one upstream call (`shared`).
  - Errors map to `FamosError('unavailable', 'weather_<code>')`.
  - A badly shaped reply becomes `weather_bad_response`.
- **Off.** Weather that is switched off, or has no place, returns `{ enabled: false }` rather than an error, and the widgets stay away.
- **Displays never call outside.** The server fetches; the display asks the server. Switching the ambient extension off in Settings → Features stops every outbound call.

**Photos**

- Uploads only. There is no photo-service integration; architecture.md says this is deliberate ("no upstream API to follow").
- **Upload** (`server/photos.ts`, `server/image.ts`):
  - `POST /ambient/photos` takes multipart, a 5 MB cap and one file.
  - The server reads the type and size from the magic bytes and headers (JPEG SOF markers, PNG IHDR, WebP VP8/VP8L/VP8X). The client's claimed type is not trusted.
  - Images larger than 8192 px on either side are refused.
  - The file is stored first. Then the `ambient.addPhoto` action runs, which enforces a library cap of 500, and the stored file is deleted if the action fails.
  - The action accepts only keys matching `^ambient/photos/…`, so it cannot be pointed at other files.
- **Delete.** `ambient.deletePhoto` removes the row and enqueues the `ambient.deleteFile` job.
- **Serving.** `GET /ambient/photos/{id}` with `cache-control: private, max-age=3600`.

**Other extensions, read through their own routes and view capabilities**

| Capability | What it feeds |
|---|---|
| `calendar.view` | `GET /calendar/occurrences` (the day, the hero, the weather sentence, the ambient agenda) |
| `chores.view` | `GET /chores/today` (rings) |
| `lists.view` | Shopping glance, tasks widget |
| `kitchen.view` | Meals |
| `board.view` | Board widget |
| `records.view` | Coming-up widget |
| `core.drafts.waiting` | `GET /core/drafts/waiting` (the pill) |

The capabilities offered at pairing are computed as `familyModeCapabilities()` plus every `requires` of the `display.widget`, `display.status` and `display.ambient` slots (`devices/+page.svelte`).

### Display specifics

| Topic | What the code does | Evidence |
|---|---|---|
| Route and phases | `/display` has four phases: `loading` → `unpaired` (pairing screen), `paired` (Today), or `signedIn` (refuses: "This browser is used by the family"). The layout skips session, navigation and the member theme on `/display*`. | `apps/web/src/routes/display/+page.svelte`, `apps/web/src/routes/+layout.svelte` |
| What is shown | Today in family mode in the display's style: clock, date, weather block, the hero with countdown, the day (by person only if names are on), chore rings, shopping number, meals, next days, photo of the day. Status pills sit in the head; a widget band sits at the foot. A tiny label (11 px, 60% opacity) in the bottom corner shows the display's name and, when offline, "Cannot reach FamilyOS. Trying again…". | `FamilyToday.svelte`, `Broadsheet.svelte`, `Lanes.svelte`, `Wall.svelte` |
| Gating | Without `calendar.view` there is no hero and no day. Without `chores.view` there are no rings. Without `ambient.weather` there is no weather; without `ambient.photos` no photo. Nothing granted at all gives "Nothing to show yet. An admin can choose…". | `today-data.svelte.ts`, `display/+page.svelte` |
| Widgets | Board notes: pinned first, at most 2, "+n". Open tasks: due today or overdue first, at most 7. Records coming up: at most 2, hidden when empty. Each is always mounted and shown or hidden by the fit plan; the grid is `minmax(18rem,1fr)`. | `extensions/board/src/web/DisplayBoard.svelte`, `extensions/lists/src/web/DisplayTasks.svelte`, `extensions/records/src/web/DisplayComingUp.svelte` |
| Rotation and slideshow | No rotation of Today's layout. Ambient mode is a photo slideshow (shuffled; slide time 5–600 s; 1.5 s crossfade; preloads the next image). The photo of the day changes daily. | `Ambient.svelte`, `photo-of-the-day.ts` |
| Ambient mode trigger | Any `pointerdown`, `keydown` or `touchstart` resets the idle timer. A 1 s ticker enters ambient mode after `idleMinutes`. It is a full-screen overlay (z-50). The display page renders `display.ambient` contributions filtered only by disabled extensions; the component itself runs only if the display holds `ambient.photos` **or** `ambient.weather`. | `Ambient.svelte`, `display/+page.svelte` |
| Ambient content | Clock (text-9xl, or text-8xl over a photo), the long date, "Next today": up to 3 timed events still to come (with a 15-min grace for ones just started; only 1 over a photo), else "Nothing else planned today". Weather: icon, temperature, sky label, low/high, rain chance when at least 20%, attribution. Text over photos sits on a bottom scrim (`--scrim`) with a drop shadow, forced to the day palette. | `Ambient.svelte`, `agenda.ts` |
| Day/night | `.dark` on `<html>` follows that browser's `famos.mode`. The default is "system", meaning the OS `prefers-color-scheme`, followed live. It is applied before first paint. There is **no sun-based switching** in code, and the display has no UI to set the mode. The weather sky uses Open-Meteo `is_day` for sun or moon. The photo tile dims a little at night. | `apps/web/src/lib/theme.svelte.ts`, `apps/web/src/app.html`, `packages/ui/src/famos/PhotoTile.svelte` |
| Dim or night mode | None. There is no brightness or dim schedule and no screen-off schedule. The docs tell the family to use the tablet's own brightness schedule. | `docs/guide/displays.md` (Tips) |
| Burn-in protection | None: no pixel shift, no moving clock. Content changes come from the slideshow, the animated sky (CSS keyframes, at most 11 animated elements, still under reduced motion) and the daily photo. | `packages/ui/src/famos/WeatherSky.svelte` |
| Auto-refresh and reload | Data: see the cadence table. Code: the service worker checks every 15 min and reloads the page once when a newer app is active ("kiosk tabs are never closed"). The shell, manifest and service worker are served `no-cache`; hashed assets are immutable. There is no periodic full page reload. | `apps/web/src/lib/sw-update.ts`, `apps/server/src/http/static.ts` |
| Offline | A paired display keeps showing the last data (every loader keeps its old state on failure) and flags "Cannot reach FamilyOS" in the corner. The socket reconnects with backoff; the self-check repeats every 60 s. Displays never queue writes. The service worker caches the shell, but `/core/displays/me` is **not** in its offline-read allow-list, so a cold start offline cannot render Today. | `display/+page.svelte`, `apps/web/src/service-worker/index.ts` |
| Per-display style and palette | Style, theme and hidden blocks are set per display (`core.setDisplayStyle`, `core.setDisplayTheme`, `core.setDisplayTodayBlocks`, all needing `core.devices.manage`). Resolution: the display's own → the family's → (style only) the theme's suggestion → the default. A person's own choice or trial never affects a display. | `packages/core/src/themes.ts`, `apps/server/src/http/devices.ts` |
| Language and formats | The display always uses the family's language and region formats from `/displays/me`, never a member's. It reloads on `core.family.*` and `core.languages.*` events. | `display/+page.svelte` |
| Revocation | Takes effect immediately on HTTP (the device is verified per request). The socket is closed with 4401; the screen returns to pairing with a new code. | `realtime.ts`, `devices.ts`, `app.ts` |
| Wake lock and fullscreen | None in code (no Screen Wake Lock API, no `requestFullscreen`). The docs advise "Add to Home Screen" (PWA manifest) and switching off auto-lock or using guided access. | `docs/guide/displays.md` |
| Screen size and orientation | `h-dvh overflow-hidden`; never scrolls. Padding is 20/16/24 px, and 28/40 px from 1024 px. The fit planner and zoom are described above. For a tablet held upright (640–1023 px), each style keeps its per-person structure (Broadsheet's columns, Lanes' horizontal chart, Wall's names). Phones in portrait are supported. | `FamilyToday.svelte`, the style components |
| Touch targets | The docs promise `--ctl: 52px` and 52 px targets on displays. The CSS hook (`[data-ctl="kiosk"]`) exists but is never set. In practice nothing on a display is meant to be tapped. | `packages/ui/src/theme.css`, `docs/design/design-system.md` |
| Multiple displays | Each is its own `core_devices` row with its own grants, name, style, theme and hidden blocks. Ambient timing and the weather place are family-wide. | `devices.ts`, `ambient/server/settings.ts` |

### Docs vs code

- **"Displays follow the sun" / "switch to night at sunset" is not implemented.**
  - The claim appears in `docs/guide/appearance.md`, `docs/design/design-system.md` principle 12, the UI string `devices_display_theme_hint` ("it still follows the sun", in `apps/web/src/messages/en.json`), the comment in `packages/ui/src/theme.css`, and the comment in `display/+page.svelte`.
  - In code, `/display` uses the same `followDayNight()` as every page: the per-browser `famos.mode`, defaulting to the OS colour scheme.
  - No sunrise or sunset is computed. Open-Meteo `is_day` only drives the weather icon and sky.
- **The photo-of-the-day hint is wrong.** `block_photo_hint` says "A family photo from the same month in another year." The code (`pickForDay`) walks a seeded shuffle, one photo per day, with no date matching. The label shows the month the photo was *added*.
- **52 px kiosk controls** (design-system §3.1, §9) are not applied. Nothing sets `data-ctl="kiosk"`, and the `kiosk` button size is unused.
- **`docs/architecture.md` differs from code in two places:**
  - It says the weather "also appears as a `dashboard.card` on Today". In code it is the `today.atmosphere` slot.
  - It says `display.widget` has "an optional `column`". `packages/ui/src/slots.ts` has no such field.
- **Calling `ctx.http` from a route.** AGENTS.md says to call `ctx.http.get` "from job handlers only". `docs/architecture.md` explicitly allows a bounded read-through in routes, and the weather and place search do that. Theme-from-URL, by contrast, uses a job.
- **"The display tries again by itself when the network comes back" (displays.md)** holds once the display is paired. From the code, a display that starts while the server is unreachable shows "Cannot reach FamilyOS. Trying again…", but nothing re-runs `check()` in the `loading` phase. The intervals exist only for the `paired` and `signedIn` phases. Worth testing; see the next subsection.
- **The ambient slot's `requires` is not applied.** `display.ambient` declares `requires: 'ambient.photos'`, but the display page does not filter ambient contributions by `requires`; the component self-gates on photos **or** weather.
  - So a display granted weather but no photos still gets ambient mode (sky and clock). This matches the docs.
  - A display with neither grant never enters ambient mode, which the docs do not say.
- **Otherwise docs and code agree:** the pairing flow and code lifetime, the 500/200 photo caps, the timing ranges, the fit order and the zoom floor, names off by default, everything ticked at first, and revocation.

### Unfinished or stubbed

- **Missing kiosk features** (all left to the device OS):
  - night switching at sunset;
  - a scheduled dim or brightness reduction;
  - a screen-off schedule;
  - burn-in mitigation (pixel shift);
  - Screen Wake Lock;
  - fullscreen.
- **Possible bug: no cold-start retry.** If the first `/displays/me` fails, the display appears to stay on "Cannot reach FamilyOS. Trying again…" until someone reloads it (`display/+page.svelte`; there is a single `$effect(() => void check())`). This is a likely real-world problem when the tablet boots before the server. I found it by reading the code, not by running it.
- **Possible bug: the weather attribution is a live link on displays.** `AtmosphereLine.svelte` does not pass the `familyMode` slot prop through to `TodayWeather`, so on a display the Open-Meteo attribution renders as an `<a target="_blank">` link instead of plain text. That breaks the "nothing on a display is a link" rule, and a tap would leave the kiosk.
- **Slideshow quirks.**
  - `GET /ambient/photos` reshuffles on every call. The 8-minute content refresh therefore replaces the id list, which restarts the slide effect and advances the photo at once, so order and repeats are not controlled across refreshes.
  - There are no captions, albums, per-display photo sets or face/people filters.
- **Weather limits.** Only one place per family and only one provider (Open-Meteo, hard-coded). There are no alerts or warnings, no AQI/UV/pollen/wind, and no multi-day forecast on the screen: only 2 days are fetched, and the next hours are shown.
- **Family-wide settings.** Ambient timing is family-wide, not per display. Ambient mode is idle-triggered only, with no time-of-day schedule.
- **No compositions to add.** `compositions.ts` falls back to Broadsheet for "a style without its own composition yet", but all three bundled styles have one; the fallback is defensive.
- **Clock granularity.** Today's clock text follows `TodayData.now`, which ticks every 15 s, so the minute can change up to 15 s late on Today. Ambient mode uses the page's 1 s clock.

### Beyond Home HQ (net-new ideas)

Home HQ already has Today, calendar views, chores, grocery, a slideshow, ambient/night modes and takeovers. These FamilyOS ideas are not in that list:

- **Device-code pairing for wall screens (RFC 8628 style).**
  - The screen shows a short unambiguous code and a QR code that deep-links the admin's phone to the confirm step.
  - Single-use tokens are kept only as hashes in an httpOnly cookie, and exactly one poll wins.
  - Starting a pairing is rate-limited per IP; a wrong code invalidates all open codes after 5 misses; `slow_down` is enforced.
- **Per-screen, capability-scoped, view-only grants.** The admin picks what each screen may read (calendar, chores, lists, meals, weather, photos, board, records, the waiting count). The server enforces it per request, for actions, routes and realtime alike, and revocation is instant.
- **"Hallway privacy" mode.** By default a screen shows no names or colours: one merged day, a family chore total, and a hero without "for whom". Names are an explicit opt-in per screen.
- **Never-scroll fit planner.**
  - It degrades in priority order: fewer items per person, then dropping widget-band items, then dropping blocks (photo → weather hours → extras → chores), then zooming down to a floor.
  - Ended items leave first.
  - Everything is measured offscreen in one frame, so there is no visible reflow.
  - A larger type step applies at ≥1024 px and at ≥1800 px (TV).
- **Weather that talks about your day.** Rain windows are linked to calendar items by time ("Rain from 15:00, during Football"). Other templates cover "rain until 14:00, then partly cloudy", cold mornings, and the evening low. These are deterministic templates, translatable, with no LLM.
- **Evening flip.** After max(20:00, the end of the last timed item), Today turns to tomorrow and adds a "done today" line.
- **Next-up countdown semantics.** Minutes under an hour, a compact duration under two hours, then the start time. Once started it reads "now, until…". A pill shows the place or the start, and each person can choose whether the hero shows the family's next thing or their own.
- **A per-person one-clause status row** ("Piano lesson in 25 min", "1 chore left").
- **Deterministic photo of the day.** It never repeats on consecutive days, shows every photo once per round, and is the same on every screen all day.
- **Per-screen style, theme and hidden blocks,** resolved display → family → theme suggestion.
- **Live-update model.** Realtime events carry only IDs, and a 2-second shared-read de-duplication sits on top. There is a 60 s self-check plus a 5-minute full re-read as a safety net, and service-worker auto-update for tabs that never close.
- **Animated weather sky** from the WMO code and day/night: CSS-only, at most 11 animated elements (Pi-friendly), still under reduced motion, and its text contrast is tested per theme.
- **A one-line status pill on the screen** for "N suggestions waiting" (an approval queue count only, with no content).
- **Client-side photo shrink and metadata strip** before upload, plus server-side magic-byte verification.

### Evidence

- **Display route:** `apps/web/src/routes/display/+page.svelte`, `apps/web/src/routes/display/FamilyToday.svelte`, `apps/web/src/routes/display/PairScreen.svelte`
- **Today:** `apps/web/src/routes/+page.svelte`, `apps/web/src/lib/today/today-data.svelte.ts`, `apps/web/src/lib/today/compositions.ts`, `apps/web/src/lib/today/display-fit.ts`, `apps/web/src/lib/today/blocks.ts`, `apps/web/src/lib/today/Broadsheet.svelte`, `apps/web/src/lib/today/Lanes.svelte`, `apps/web/src/lib/today/Wall.svelte`, `apps/web/src/lib/today/chart.ts`, `apps/web/src/lib/today.ts`
- **Shared Today model:** `packages/ui/src/famos/today-v3.ts` (nextUp, countdown, pill, memberDays, statusOf); `packages/ui/src/famos/Countdown.svelte`, `WeatherBlock.svelte`, `WeatherSky.svelte`, `sky.ts`, `PhotoTile.svelte`, `shared-read.ts`; `packages/ui/src/slots.ts`
- **Next-up preference:** `packages/core/src/next-up.ts`
- **Devices, pairing and access:** `packages/core/src/devices.ts`, `packages/core/src/access.ts`, `packages/core/src/core-capabilities.ts`, `packages/core/src/action-runner.ts`, `packages/core/src/tables.ts`, `packages/core/src/migrations/0001_init.ts`
- **Server:** `apps/server/src/http/devices.ts`, `apps/server/src/http/app.ts` (`resolveActor`, `requireActor`), `apps/server/src/http/realtime.ts`, `apps/server/src/http/static.ts`
- **Client runtime:** `apps/web/src/lib/realtime.ts`, `apps/web/src/lib/sw-update.ts`, `apps/web/src/service-worker/index.ts`, `apps/web/src/lib/theme.svelte.ts`, `apps/web/src/app.html`
- **Admin UI:** `apps/web/src/routes/settings/devices/+page.svelte`, `AddDevice.svelte`, `DisplayForm.svelte`; `apps/web/src/routes/settings/theme/+page.svelte`
- **Ambient:** `extensions/ambient/src/server/{index,capabilities,settings,store,caches,weather,photos,photo-of-the-day,image}.ts`, `extensions/ambient/src/contract/{index,sentence}.ts`, `extensions/ambient/src/web/{index.ts,Ambient.svelte,AtmosphereLine.svelte,TodayWeather.svelte,agenda.ts,sentence.ts,weather.ts,resize.ts,data.ts,DisplayContent.svelte,PhotosSection.svelte,WeatherSection.svelte,TimingSection.svelte,SetupPlaceStep.svelte}`, `extensions/ambient/src/messages/en.json`, `extensions/ambient/test/*.test.ts`
- **Display widgets:** `extensions/board/src/web/DisplayBoard.svelte`, `extensions/lists/src/web/DisplayTasks.svelte`, `extensions/records/src/web/DisplayComingUp.svelte`, `extensions/inbox/src/web/DisplayWaiting.svelte`
- **Docs:** `docs/guide/displays.md`, `docs/guide/appearance.md`, `docs/design/design-system.md` (§1.12, §3.1, §6.5, §9), `docs/architecture.md` (Family displays, Display content); e2e `apps/e2e/tests/display.spec.ts`

---

## 3. Calendar

The calendar is one extension (`extensions/calendar`, package `@famos/ext-calendar`). It holds the family's own events (with members, all-day and multi-day days, simple recurrence, one reminder per event, and tags), publishes secret read-only ICS feeds out, and pulls read-only ICS subscriptions in. It has no CalDAV, no Google/Microsoft OAuth and no two-way sync. `docs/architecture.md` ("Calendar feeds") says this is deliberate: "two-way CalDAV sync would need a sync engine with conflict handling and is not built." Date and recurrence math lives in `packages/time` (Temporal polyfill plus `rrule-temporal`). Feed parsing uses `ical.js` 2.2.1.

### What the user can do

**Screens** (`extensions/calendar/src/web/CalendarPage.svelte`)

- Tablet and up (768 px or wider): a board with **Day / Week / Month** buttons. Week needs 1024 px. On a narrower window, picking Week shows the Day view and the Week button is hidden.
  - Day and Week are an hour grid (`WeekBoard.svelte`):
    - an all-day lane above the grid, up to 3 rows;
    - a now-line;
    - today's next event outlined in the `accent` colour;
    - on open, the grid scrolls to 07:00, or to an hour before the first event (`firstVisibleHour` in `layout.ts`);
    - clicking empty space adds an event at that time, snapped to 30 minutes (`snapSlot`).
  - Month is a 5- or 6-week grid (`MonthBoard.svelte`; the sixth week is dropped when it is all next month). Each cell shows as many rows as fit, then "+N more". Clicking a day opens it in Day view.
  - Keyboard: left and right arrows move by a day, week or month; `t` goes back to today.
- Phone: a header with a **Week / Month** toggle.
  - Week is a 7-day strip with coloured dots per day. Month is a grid with dots. Tapping a day shows that day's agenda list below it.
  - The agenda uses the shared `Timeline` component. Groups run all-day, then timed, "Any time", "Evening". There is a now-line, and the next item says "in N min" or "in N h", or "first" on days other than today (`packages/ui/src/famos/timeline.ts`).
  - A floating "+" button adds an event.
  - The phone has no Day/Week/Month board and **no person filter**.
- Filters:
  - **Person** (tablet only): avatar toggles. A selected person keeps their own events plus whole-family events (`filterBoard` in `board.ts`).
  - **Subscription**: dashed chips in each feed's colour, one per subscribed calendar with events in view. Tapping one hides it.
  - **Tag**: chips. The selection is stored per device in `localStorage` key `famos.calendar.tagFilter` (`tags.ts`).
- Event detail: on tablet a family event opens read-only in the side panel first (`EventDetailView.svelte`: when, repeat, who, where, reminder, tags, notes), and **Edit** turns it into the form. On a phone, tapping opens the form directly. Subscription events open read-only (`ExternalDetail.svelte`, marked "From <calendar name>", with notes).
- Hour-grid layout (`layout.ts`, tested in `test/calendar-layout.test.ts`):
  - Overlapping events form clusters and get columns greedily.
  - A chip widens into free columns to its right.
  - At most 2 to 6 columns, depending on column width (56 px minimum chip width). Anything beyond goes into one "+N more" chip.
  - Events of 6 hours or more (`isLongBlock`, e.g. someone at work) are drawn hollow and dashed behind the other events and take no column.
  - A block is never shorter than one title line.
  - Events that cross midnight are clipped onto both days (`clipToDay`).
- Colour:
  - An event's stripe is its members' colours (bands when there are several). Whole-family events are neutral `--ink-soft`.
  - Subscription events use the subscription's colour.
  - Tags have their own colours from an 8-colour palette (`SUBSCRIPTION_COLORS` in `contract/index.ts`, the same as core's member palette). A new tag gets the next unused colour automatically (`ensureTags` in `store.ts`).
- Live updates: the page reloads on any realtime event whose name starts with `calendar.` (`app.onEvent('calendar.', …)`).
- Offline: the service worker keeps the last-seen `/api/v1/calendar/occurrences` responses (`apps/web/src/service-worker/index.ts`). Editing needs a connection.

**Event fields** (`EventInputSchema` in `contract/index.ts`, form in `EventForm.svelte` / `form.ts`)

- title: 1 to 120 characters.
- allDay. date: start date. endDate: last day, inclusive. **Only all-day events can span several days.**
- startTime and endTime as HH:MM:
  - The end defaults to 1 hour after the start.
  - An end earlier than the start means the next morning (the form says "(next day)").
  - An end equal to the start is rejected.
  - **A timed event can be at most just under 24 hours long.** Something like Fri 18:00 to Sun 14:00 cannot be entered as one timed event.
- memberIds: empty means "whole family".
- location: up to 200 characters. notes: up to 2000.
- recurrence: see the repeat options below.
- reminder: one preset or none.
- tags: up to 5, each up to 24 characters, compared case-insensitively. Turkish dotted and dotless I are folded too (`tagKey`).
- The new-event form defaults to the next full hour today, or 09:00 on other days, with a one-hour length (`emptyForm`). Moving the start moves the end by the same amount (`shiftEnd`).
- There is no per-event colour, no attachments, no URL field, no attendee or RSVP, and no travel time.

**Repeating events** (`packages/time/src/index.ts`, `RecurrenceSchema` / `toRRule` / `fromRRule`; picker in `packages/ui/src/famos/RepeatPicker.svelte`)

- UI options:
  - Does not repeat
  - Every day
  - Every week, on chosen weekdays
  - Every month (same day of month)
  - Each option takes an optional **Until** date.
- The schema also has `interval` (daily up to 365, weekly up to 52, monthly up to 12), but **the picker has no interval control**. "Every 2 weeks" can only come from the assistant. A form opened on such an event keeps the interval.
- **No yearly option.** Birthdays and anniversaries cannot be made to repeat yearly in the UI or through the AI. Yearly rules exist only on subscribed feeds.
- Rules are stored as RFC 5545 RRULE text:
  - Timed series: `UNTIL` is 23:59:59 local time of the until-day, written in UTC.
  - All-day series: `UNTIL` is a plain date.
  - A rule outside the presets shows as "Custom rule" and cannot be picked. `validate()` blocks saving until a preset is chosen.
- Exceptions:
  - "Delete only this one" adds the occurrence key to `exdates`. The key is epoch ms for timed events and `YYYY-MM-DD` for all-day ones.
  - "Delete whole series" soft-deletes the event (`deleted_at`).
- Editing from the UI always changes **the whole series** (the form says "Changes apply to the whole series"). The UI has no edit-this-one or this-and-future option.
- The assistant/AI path has an edit-this-one. `calendar.changeEvent` with an `occurrence` splits that date out as a new one-off event and adds the date to the series' `exdates`. Its undo is `calendar.putOccurrenceBack` (`server/index.ts`, test `test/occurrence-move.test.ts`). Neither path has this-and-future for the family's own events.

**Reminders** (`server/reminders.ts`, docs `docs/guide/calendars.md` "Reminders")

- One reminder per event.
- Timed presets: at the start, 5, 10, 15, 30 or 60 minutes, 2 hours, or 1 day before. The default is 30 minutes.
- All-day presets: 09:00 on the day (the default), or 21:00 the evening before. The 09:00 anchor is fixed in code (`ALL_DAY_ANCHOR`).
- `reminder_set` = 0 means "nobody chose", so the default applies, which is different from an explicit "none".
- Recipients:
  - the event's members, by member id;
  - else everyone holding `calendar.view` (`to: { capability: 'calendar.view' }`).
- Delivery uses core notifications (kind `calendar.reminder`): the in-app list, web push and Telegram, with per-person quiet hours and channel switches (`docs/guide/notifications.md`). Text is translated per recipient. The title is the event title. The body is the time, or "All day", plus " · location".
- Repeating events remind before every occurrence.
- Subscription events remind only when the subscription's "Remind about its events" switch is on, and then always with the defaults.

**Subscriptions in** (`server/subscriptions.ts`, `SubscriptionsSection.svelte`; Settings → Family → Calendars & tags; needs `calendar.manage`)

- The only way in is an ICS address (`https:` or `webcal:`/`webcals:`, which are treated as https).
- The docs tell users how to get one:
  - Google: "Secret address in iCal format".
  - iCloud: share the calendar as a Public Calendar.
  - School, club or holiday ICS links.
- Plain `http:` is refused by the contract, even for a NAS on the home network.
- Fields:
  - name (up to 80 characters)
  - colour (palette)
  - "Whose is it?" members (empty means everyone)
  - "Remind about its events"
  - "Calendar on my home network" (`allowLocal`)
- The address is never shown again. Editing shows only the host, and an empty address field keeps the old one.
- Status line: host, event count, last fetched time, and an error message per code: `unreachable`, `too_large`, `not_calendar`, `blocked_address`, `bad_url`, `http_<status>`.
- Buttons: **Refresh now** and **Remove**. Remove deletes the subscription's events too.
- Refresh runs every 6 hours (`REFRESH_EVERY_MS`).

**Feeds out** (`server/feeds.ts`, `FeedsSection.svelte`; Settings → You → Calendar on my phone)

- Creating an address asks for:
  - a scope: "My events" (your events plus whole-family events), or "The whole family", which needs `calendar.manage`;
  - "Include notes": off by default; titles, times, places and tags are always included;
  - an optional label (e.g. "Mum's phone").
- The address is shown **once**, with Copy and "Open in the calendar app" (a `webcal:` link). The page also shows phone setup steps.
- The list shows label, scope, created date and "last used". **Remove** revokes at once:
  - owners can revoke their own addresses;
  - `calendar.manage` holders can revoke any.

**Tags** (`server/tags.ts`, `TagsSection.svelte`)

- People with `calendar.edit` can rename, recolour or delete a tag. Rename and delete rewrite every event that has the tag.
- Renaming onto an existing name merges the two tags.
- The tag input suggests existing family tags (`suggestTags`).

**Birthdays, holidays, conflicts, privacy, search**

- Birthdays: no member birthday field and no yearly repeat. Only `extensions/pets` has birth dates.
- Holidays: only by subscribing to a public-holiday ICS feed (suggested in `docs/guide/calendars.md`).
- Conflicts: no double-booking warning. The only check is a duplicate hint for AI or draft proposals: the same title, compared loosely, on the same day (`eventExists` / `alreadyThere`). The approve button then asks for a second tap.
- Privacy: no private or hidden events. Anyone with `calendar.view` sees every event, including guests by default. Members on an event decide reminders, filters, feeds and Today lanes, never who can see it.
- Search: there is no search box on the calendar page. Family search (core, embeddings) indexes the title, location and notes of family events, but not subscription events (`server/search.ts`). The assistant's `calendar.findEvents` does a text match on title and location.

**Approval and roles** (`server/capabilities.ts`, `packages/sdk/src/capabilities.ts` `LEVEL_DEFAULTS`)

- `calendar.view` (view level: admin, member, supervised, guest).
- `calendar.edit` (edit level: admin, member).
- `calendar.manage` (manage level: admin only by default).
- createEvent, updateEvent, changeEvent, deleteEvent, restoreEvent and putOccurrenceBack are `approval: { unless: 'calendar.edit' }`:
  - a supervised member's save becomes a draft that someone must approve ("Sent for approval");
  - a guest gets `forbidden` (test `calendar.test.ts`);
  - AI actors always produce drafts.
- Displays (kiosk devices) hold only granted view capabilities and cannot run actions.

### Data model

All tables are created in one migration, `0001_init`, in `extensions/calendar/src/server/store.ts`. They are SQLite STRICT tables, with types hand-written in the same file as `CalendarTables`.

- `calendar_events`: the family's own events.
  - `id` (`cal_event_…` UUIDv7), `title`, `notes`, `location`.
  - `all_day` (0/1).
  - Timed events: `starts_at` / `ends_at` (epoch ms).
  - All-day events: `start_date` / `end_date` (inclusive `YYYY-MM-DD`).
  - `time_zone`: the family zone at save time.
  - `rrule`, plus `exdates` as a JSON array (epoch ms or dates).
  - `member_ids`: JSON array; empty means the whole family.
  - `reminder_minutes`, and `reminder_set` (0 means use the default).
  - `tags`: JSON array or null.
  - `created_by`, `source_kind`, `source_id`, `created_at`, `updated_at`, `deleted_at` (soft delete).
  - Indexes on `starts_at` and `start_date`.
- `calendar_tags`: `key` (case-folded name, PK), `name`, `color`, `created_at`.
- `calendar_feeds`: `id` (`cal_feed_…`), `token_hash` (SHA-256, unique), `scope` (`family` or a member id), `include_notes`, `label`, `created_by`, `created_at`, `last_used_at`, `revoked_at`.
- `calendar_subscriptions`:
  - `id` (`cal_sub_…`), `name`.
  - `sealed_url`: AES-GCM with `FAMOS_SECRET`.
  - `url_host`: for display only.
  - `color`, `member_ids`, `reminders`, `allow_local`.
  - `etag`, `last_modified`, `last_fetched_at`.
  - `last_error`: a code only.
  - `event_count`, `created_by`, `created_at`, `updated_at`.
- `calendar_external_events`: one row per VEVENT of a subscription.
  - PK (`subscription_id`, `uid`, `recurrence_key`); FK to subscriptions with cascade.
  - `recurrence_key` is:
    - `''` for a series or a plain event;
    - the original start for a RECURRENCE-ID override;
    - `>key` for the new tail of a THISANDFUTURE split;
    - `~key` for occurrences expanded by ical.js.
  - The same time columns as `calendar_events`, plus `fingerprint`: a 20-hex SHA-1 of the content, excluding tags.
  - `tags`: the feed's CATEGORIES.
- API shapes (`contract/index.ts`):
  - `EventDetail`: the editable form, with recurrence as a preset or a raw `rrule`.
  - `Occurrence`, one expanded instance:
    - `eventId` (for external ones, `<sub>:<uid>:<recurrenceKey>`);
    - `key`, `start`/`end` or `startDate`/`endDate`, `memberIds`, `location`, `recurring`;
    - `notes` (external only), `source` (subscription id, name, colour), `editable`, `tags`.
- Routes (`/api/v1/calendar/...`):
  - `GET /occurrences?from&to` (at most 62 days)
  - `GET /events/{id}`
  - `GET /tags`
  - `GET /feeds`
  - `GET /feed/{token}.ics` (public)
  - `GET /subscriptions` (manage)
- Actions are posted to `/api/v1/actions/calendar.*`:
  - `createEvent`, `updateEvent`, `changeEvent`, `deleteEvent`, `restoreEvent`, `putOccurrenceBack`
  - `findEvents`, `aiContext`
  - `updateTag`, `deleteTag`
  - `createFeed`, `revokeFeed`, `touchFeed`
  - `addSubscription`, `updateSubscription`, `removeSubscription`, `refreshSubscription`
- Jobs: `calendar.remind`, `calendar.refreshSubscription`.
- Realtime events: `calendar.event.changed`, `calendar.subscription.changed`, `calendar.tag.changed`.

### Rules and logic worth copying

- **Time representation** (`packages/time`, `store.ts` `toRow`):
  - Timed events are stored as UTC instants plus an IANA zone.
  - All-day events are date strings that are never converted to instants, so they never shift.
  - Local wall time is turned into an instant with `disambiguation: 'compatible'`.
  - A local day has DST-correct bounds (`dayBounds`: 23 or 25 hours).
  - Timed series are expanded in the event's own zone with `rrule-temporal` (DST-safe, "every Tuesday 17:30" stays 17:30). Date series are expanded on a UTC calendar.
- **One expansion path for everything** (`expandRow` in `store.ts`): family events and subscription events share the row shape, the expansion and the reminder code.
  - Multi-day all-day series are found by starting the search `spanDays` before the range.
  - Timed events are found by starting it `duration` before the range.
  - So an occurrence that began earlier but overlaps the range is included.
  - Zero-length external events still show at their start time.
- **Occurrence identity**: an occurrence key is the start in epoch ms (timed) or the date (all-day). Delete, change and reminders all use it.
  - Models tend to send a date instead of the key. `occurrenceKey` maps a date to the single occurrence on that day.
  - When the date has zero or several occurrences, it fails with an error message aimed at the model ("Use the `occurrence` that calendar.findEvents shows…").
- **Partial change with undo** (`changeEvent`):
  - Only the fields sent are changed.
  - The output carries a `before` object, so the undo is the same action with the old values.
  - A new start without a new end keeps the event's length, capped at 23:59. A new first day keeps a multi-day event's length.
  - A move clears the skipped dates; other changes keep them.
  - A rule outside the presets is kept untouched.
- **Reminder scheduling** (`reminders.ts`):
  - Only the **next** reminder of an event is queued (`runAt`). When it fires, it notifies and queues the one after.
  - Every change re-plans inside the same transaction.
  - The job payload carries `version`: the event's `updated_at`, or the external fingerprint.
  - At run time the job does nothing when any of these no longer match: the version, the occurrence still existing, or the planned instant.
  - Queue dedupe key: `calendar.remind:<id>:<key>:<version>:<at>`. Notification dedupe key: `reminder:<id>:<key>`.
  - A reminder may be up to 5 minutes late when planned. It is dropped if the event is already over (server was down). Look-ahead is 400 days.
- **Subscription sync** (`subscriptions.ts`):
  - Conditional GET with `If-None-Match` and `If-Modified-Since`. A 304 only updates `last_fetched_at`.
  - On 200, the feed is parsed and **all** of that subscription's rows are replaced in one transaction (inserted in batches of 100), so readers never see a mix.
  - On failure, the old events are kept and only an error code is recorded. The URL and feed content are never logged.
  - Scheduling uses fixed 6-hour slots plus a per-subscription offset of up to 30 minutes from a hash of the id. The dedupe key is per slot, so only one follow-up exists however many refreshes happen.
  - "Refresh now" has its own dedupe key. The chain stops by itself when the subscription is deleted.
  - Changing the URL clears the ETag and Last-Modified. Changing members, reminders or allowLocal triggers a refresh.
  - The job times out after 60 s and is tried at most 3 times.
- **ICS import robustness** (`ical-import.ts`, tests `subscriptions.test.ts`, `ical-future.test.ts`):
  - TZIDs:
    - Feed VTIMEZONEs are registered with ical.js.
    - Floating times are read in the family's zone.
    - A non-IANA TZID (e.g. "W. Europe Standard Time") keeps the right instant, but the series is expanded by ical.js into single events in a window from 31 days back to 400 days ahead, at most 800.
    - A rule that our expander cannot handle, or a non-Gregorian RSCALE, takes the same path.
  - Overrides and cancellations:
    - A RECURRENCE-ID override becomes its own row and excludes the original occurrence.
    - `STATUS:CANCELLED` on an override only excludes; on a master it drops the series. An orphan override is kept as a plain event.
  - `RANGE=THISANDFUTURE`:
    - The series is split: the master gets an UNTIL, and a new tail series takes over, with COUNT reduced and exdates shifted.
    - Cases that cannot be split cleanly fall back to the ical.js expansion: several such overrides, a weekday move on a BYDAY rule, a clock change on a BYHOUR rule.
  - Limits: titles are clipped to 300 characters, notes to 4000. A missing title becomes "—". At most 5000 events, otherwise the feed counts as `too_large`. CATEGORIES become read-only tags.
- **Duplicate and staleness guards on AI drafts** (`server/index.ts`):
  - `alreadyThere`: a same-title event on the same day triggers the "may already be there" double-tap.
  - `draftDate`: a draft for a one-off event expires after its last day. A repeating one never expires.
  - `targetTitle` with `ReviewLabelSchema`: the AI must give the person's own words for the target, and the runner rejects them when they do not name the actual event (`test/label-guard.test.ts`).
- **"When" classification** (`when.ts`): `past`, `now` or `upcoming` relative to now, with all-day events counted against the family date. Every AI answer about today carries it, so "coming up" never lists something already over.
- **findEvents ergonomics**:
  - `from` alone means that one day. `text` alone searches from today for 62 days.
  - Results are capped at 60, with a note telling the model to narrow the query.
  - An unknown tag returns a note suggesting a text search.
  - When nothing matches, it looks up to 366 days further and returns `nextMatch`, so the answer is "the next one is on …".
- **Feed security** (`feeds.ts`):
  - The token is 32 random bytes in base64url (43 characters), and only its SHA-256 is stored, so a copied database reveals no working address.
  - The route needs a `.ics` suffix and is public. Every miss is a 404.
  - `last_used_at` is updated at most hourly, through a system actor, because the caller is anonymous.

### Integrations

- **ICS fetch**: `ctx.http.get` from the job handler only (`packages/adapters/src/http-guard/index.ts`).
  - DNS is resolved and checked first, then the connection goes to that checked IP.
  - Private, loopback and link-local addresses are blocked unless `allowLocal`. Some ranges are never allowed.
  - The `calendar` contract still requires https, so a NAS calendar needs a valid TLS certificate. Plain http is allowed by the guard only on the LAN, and the calendar never uses it.
  - At most 3 redirects, same host only. 20 s timeout. 5 MB body cap (`DEFAULT_MAX_BYTES`).
  - gzip, deflate and br are accepted. Sends `User-Agent: FamilyOS`, `Accept: text/calendar…`, and conditional headers.
- **Secrets**: the subscription URL is sealed with `uow.secrets.seal` (AES-GCM, key from `FAMOS_SECRET`). Only the host goes back to the browser.
- **Feeds out** (`ics.ts` `buildCalendar`):
  - Calendar headers:
    - `X-WR-CALNAME` is "FamilyOS" or "FamilyOS · <name>".
    - `REFRESH-INTERVAL;VALUE=DURATION:PT1H` and `X-PUBLISHED-TTL:PT1H`.
    - Response headers: `Cache-Control: private, max-age=300`, and no ETag.
  - Event times:
    - One-off timed events are written in UTC.
    - Repeating timed events use `TZID`, with a VTIMEZONE built from Temporal transitions, from the series start to 10 years ahead.
    - All-day events use `VALUE=DATE` with an exclusive DTEND.
  - Event content:
    - RRULE and EXDATE are written.
    - Tags go out as CATEGORIES. Notes go out as DESCRIPTION only when the feed allows notes.
    - `UID:<id>@familyos`; DTSTAMP and LAST-MODIFIED are `updated_at`.
  - Text is escaped and lines are folded at 75 octets, never inside a UTF-8 character.
  - Window: events from 90 days ago onward. Every repeating series is always included.
  - **Subscription events are not re-published** in the feed, and members or attendees are not written.
- **AI, assistant and MCP**:
  - `findEvents`, `createEvent`, `changeEvent` and `deleteEvent` carry `ai` hints. They are offered to the built-in assistant and to MCP clients (renamed `calendar_findEvents` etc. in `apps/server/src/mcp/server.ts`).
  - `calendar.aiContext` gives Quick add (inbox) up to 50 existing tag names, so proposals reuse them. Quick add proposes `calendar.createEvent` drafts. The draft cards come from `web/presenter.ts`, and the editable draft form reuses `EventForm` (`DraftEditor.svelte`).
  - `updateEvent` has no AI hints and is used only by the UI.

### Today / display relevance

- The calendar contributes **no `dashboard.card` or `display.widget`**. It feeds Today through a `timeline` contribution, `calendarTimeline` (id `calendar.today`, in `web/timeline.ts`):
  - `load(day)` is called for today and tomorrow.
  - `loadAhead(from, to)` is one range call for the "Next days" look-ahead on a quiet day (Today asks for days +1 to +7). A multi-day event appears only on its first day.
  - Each item carries:
    - `start`/`end`; the start is clipped to the day start for events from the night before;
    - `slot: 'allday'` for all-day events;
    - a subtitle "who · time range · place". For subscription events, the subscription name stands in for "who";
    - `members`, used for the per-person columns, lanes and rows and the "Me" filter;
    - `location`, used for the hero's "At <place>";
    - `href: '/calendar'`.
- Today itself (`apps/web/src/lib/today/today-data.svelte.ts`, Next-up and countdown in `packages/ui/src/famos/Countdown.svelte` / `today-v3.ts`) is app code that consumes these items. Its "Next up" hero, the "Family | Me" choice (kept server-side) and the countdown come from calendar items:
  - under an hour: minutes;
  - under two hours: "X h Y min to go";
  - after that: "starts HH:MM";
  - once started: "now".
- Without `calendar.view`, a display has no day at all: no hero, no per-person day, no evening or quiet note (`DAY_CAPABILITY`). "See the calendar" is one of the ticks when a display is paired (`docs/guide/displays.md`). In family mode, items lose their links, and with names off they also lose members and subtitles.
- Ambient mode (`extensions/ambient/src/web/Ambient.svelte`) reads today's timed occurrences, refreshed every 60 s and on `calendar.` events. It shows "Next" plus the time and title of the upcoming events, only one when a photo is showing, or "nothing" when the day is empty.
- The weather sentence (`extensions/ambient/src/web/sentence.ts`) reads the same occurrences, through a shared read with Today, to say things like "Rain from 15:00, during Football practice."

### Docs vs code

- `docs/guide/calendars.md` says the feed scope choice is shown to "only admins". The code checks `calendar.manage` (`FeedsSection.svelte` `mayShareFamily`, `feeds.ts`). That is admin-only by default, but a custom role can hold it.
- `docs/guide/calendars.md` says tags are managed "under **Phone and other calendars**, then **Tags**". In code, tags live on Settings → Family → **Calendars & tags** (`/settings/calendars`).
  - The Settings nav entry for that page requires `calendar.manage`.
  - Members who have `calendar.edit` but not manage can reach it only through the calendar page's settings icon, which is labelled "Phone and other calendars". The page then shows them only the Tags section.
- `docs/guide/everyday.md` says "Pick **Day**, **Week** or **Month**" and "The people pills filter by person". Neither is quite true everywhere:
  - Week is hidden below 1024 px.
  - The phone has only Week/Month and an agenda.
  - The person filter exists only on tablet and desktop.
- `docs/guide/everyday.md` says "A change to a repeating event changes the whole series". That is true, but undocumented: **`updateEvent` (the UI save) sets `exdates: null`**, so editing a series in the UI brings back every date that was skipped with "Delete only this one".
  - If the assistant had split a date out, that date then appears twice: the restored series date and the one-off.
  - By contrast, `changeEvent` (the AI path) keeps skipped dates unless the event moves.
  - This looks like a bug, or at least a surprise.
- `docs/guide/calendars.md` says "Your phone refreshes the calendar by itself, usually every few hours." The feed asks for a 1-hour refresh (`REFRESH-INTERVAL PT1H`) and allows 5 minutes of HTTP caching. Actual timing depends on the client.
- The `sub_error_too_large` text says "over 5 MB". The same code is also used when a feed has more than 5000 events (`TooManyEventsError`), whatever its size.
- `docs/guide/assistant.md` describes moving one date of a repeating event. That path exists only in the assistant or AI, not in the calendar UI, and the guide's calendar pages do not mention it.

### Unfinished or stubbed

- `calendar_events.source_kind` / `source_id` are always written as null for family events. They are filled only on the in-memory shape of subscription events (`externalAsEvent`). They look reserved for provenance that is not built.
- There is no UI for `calendar.restoreEvent` or `calendar.putOccurrenceBack`. They exist as undo targets only.
- Recurrence UI gaps: no interval, no yearly, no "nth weekday of month", no COUNT, no edit-this-one, no this-and-future. A "Custom rule" can be shown but cannot be edited.
- `changeEvent` cannot change recurrence or the reminder; only `createEvent` and the full `updateEvent` can. Its tag field allows 3 tags of 40 characters, while event input allows 5 of 24. The AI is told "at most 2".
- No two-way sync, CalDAV or OAuth (stated as not built in `docs/architecture.md`). Subscriptions are read-only, with a fixed 6-hour interval that cannot be configured.
- The outbound feed has no ETag, even though `TextResponseSchema` supports one. It excludes subscription events and has no attendees.
- No TODO or FIXME markers in `extensions/calendar/src`. No feature flags beyond capabilities and the per-family extension on/off switch.

### Beyond Home HQ (net-new ideas)

- **Secret per-person ICS feeds out**, so each family member sees "my events plus family events" in their phone's own calendar app:
  - hashed tokens, shown once;
  - a label per device and "last used" tracking;
  - instant revocation;
  - optional notes.
- **Read-only ICS subscriptions in** for school, club, holidays and Google/iCloud secret addresses:
  - each with its own colour, member assignment and reminder switch;
  - a home-network (NAS) opt-in;
  - error codes shown on the subscription.
  Holiday calendars are the cheap fix for "holidays".
- **Tags/categories** on events with family-wide colours, rename/merge/delete, a tag filter remembered per device, and round-trip through ICS CATEGORIES.
- **Approval workflow**: supervised members' and the AI's changes become drafts with undo. The duplicate-on-same-day hint, draft expiry once the date has passed, and the "your words must name the target" guard all come with it.
- **Natural-language calendar queries and edits** through an assistant or MCP. That includes past/now/upcoming tagging and "next match" answers, and moving a single date of a repeating event (split plus undo).
- **Overnight events** (end earlier than start means the next day). **Long-block rendering**: events of 6 hours or more are drawn hollow behind others, which suits work shifts on a wall display.
- **Overlap layout** with column widening and "+N more", fitted to the column width. Copy this for the Home HQ week view.
- **Reminder engine**: a "next occurrence only" chain with version checks and a DST-safe all-day anchor at 09:00 or 21:00 the evening before. Reminders go to the event's members, or to everyone. Delivery goes through quiet hours and per-channel switches.
- **Weather sentence** that names the event the rain overlaps, and an **ambient "Next" line** on the screensaver.
- **Offline cache** of the calendar range that was last seen.

### Evidence

- `extensions/calendar/package.json`
- `extensions/calendar/src/contract/index.ts`
- `extensions/calendar/src/server/index.ts`
- `extensions/calendar/src/server/store.ts`
- `extensions/calendar/src/server/reminders.ts`
- `extensions/calendar/src/server/feeds.ts`
- `extensions/calendar/src/server/ics.ts`
- `extensions/calendar/src/server/subscriptions.ts`
- `extensions/calendar/src/server/ical-import.ts`
- `extensions/calendar/src/server/tags.ts`
- `extensions/calendar/src/server/search.ts`
- `extensions/calendar/src/server/when.ts`
- `extensions/calendar/src/server/capabilities.ts`
- `extensions/calendar/src/web/index.ts`
- `extensions/calendar/src/web/CalendarPage.svelte`
- `extensions/calendar/src/web/WeekBoard.svelte`
- `extensions/calendar/src/web/MonthBoard.svelte`
- `extensions/calendar/src/web/EventChip.svelte`
- `extensions/calendar/src/web/EventForm.svelte`
- `extensions/calendar/src/web/form.ts`
- `extensions/calendar/src/web/layout.ts`
- `extensions/calendar/src/web/board.ts`
- `extensions/calendar/src/web/timeline.ts`
- `extensions/calendar/src/web/presenter.ts`
- `extensions/calendar/src/web/data.ts`
- `extensions/calendar/src/web/tags.ts`
- `extensions/calendar/src/web/FeedsSection.svelte`
- `extensions/calendar/src/web/SubscriptionsSection.svelte`
- `extensions/calendar/src/web/TagsSection.svelte`
- `extensions/calendar/src/web/EventDetailView.svelte`
- `extensions/calendar/src/web/ExternalDetail.svelte`
- `extensions/calendar/src/messages/en.json`
- `extensions/calendar/test/calendar.test.ts`
- `extensions/calendar/test/occurrence-move.test.ts`
- `extensions/calendar/test/reminders.test.ts`
- `extensions/calendar/test/feeds.test.ts`
- `extensions/calendar/test/subscriptions.test.ts`
- `extensions/calendar/test/ical-future.test.ts`
- `extensions/calendar/test/label-guard.test.ts`
- `extensions/calendar/test/calendar-layout.test.ts`
- `packages/time/src/index.ts`
- `packages/ui/src/famos/RepeatPicker.svelte`
- `packages/ui/src/famos/timeline.ts`
- `packages/ui/src/slots.ts`
- `packages/ui/src/famos/Countdown.svelte`
- `packages/sdk/src/capabilities.ts`
- `packages/adapters/src/http-guard/index.ts`
- `apps/web/src/lib/today/today-data.svelte.ts`
- `apps/web/src/service-worker/index.ts`
- `extensions/ambient/src/web/Ambient.svelte`
- `extensions/ambient/src/web/sentence.ts`
- `docs/guide/calendars.md`
- `docs/guide/everyday.md`
- `docs/guide/displays.md`
- `docs/guide/notifications.md`
- `docs/guide/assistant.md`
- `docs/architecture.md` (section "Calendar feeds", "Time and recurrence")

---

## 4. Lists (tasks and shopping)

One extension (`extensions/lists`, id `lists`) holds two separate features: a family **task list** (to-dos with an optional assignee, due date/time and repeat rule) and any number of named **shopping lists**. They share one nav group ("Lists"), one capability for viewing, and the same offline and AI plumbing, but have separate tables and pages (`/lists/tasks`, `/lists/shopping`). On a phone the single Lists tab opens whichever page was used last on that device, remembered in `localStorage` key `famos.lists.page`, defaulting to Shopping (`src/web/lists-page.ts`, `src/web/ListsHome.svelte`).

### What the user can do

**Tasks page** (`src/web/TasksPage.svelte`)
- Quick add: a one-line composer at the top, Enter adds the task. If the person lacks `lists.tasks.edit` (supervised members by default) the task is assigned to themselves so it needs no approval; otherwise it is unassigned ("Anyone"). "Add details" opens the full form with the typed text as title.
- Full form (`src/web/TaskForm.svelte`, side panel / phone sheet): Title (max 160), Who (zero or one member chip; none = Anyone), Due date, Time (only with a date), Repeats (Never / Daily / Weekly with weekday chips / Monthly), Until, Notes (max 2000). A repeating task requires a due date (the first occurrence). Unsaved changes prompt a discard dialog.
- Grouping: Overdue, Today, This week (next 6 days), Later, No date (`src/web/format.ts` `urgency`, `groupTasks`). On a wide container (60rem) near groups and far groups split into two columns, but only when both sides have content.
- Filter: Everyone | Mine (Mine = tasks assigned to me only; unassigned tasks are excluded from Mine on this page).
- Tick: round check button. Toast "Done" with Undo; for a repeating task the toast says "Done. Next: <date>". Done one-off tasks show in a folded "Done" section for 7 days, then disappear from the list (`DONE_VISIBLE_MS`, `src/server/tasks.ts`). Tapping the check on a done task reopens it.
- Each row shows the assignee's avatar (or an "anyone" avatar), the due label ("Today", "Tomorrow", "3 days overdue", "Fri 9 Oct", plus time), a repeat icon, and a "Waiting to send" mark when queued offline (`src/web/TaskRow.svelte`).
- Edit / delete in the side panel; delete is a two-tap confirm.
- No manual reordering, no subtasks, no priorities, no tags, no attachments, no comments on tasks.

**Shopping page** (`src/web/ShoppingPage.svelte`), designed for one-handed phone use in a shop
- Lists appear as chips with their open-item count; "+" creates a list (name max 30, unique ignoring case and spacing); the list's "..." menu has Rename and Remove (Remove only when more than one list exists; the last list cannot be removed, server returns `conflict`). A list made on install has an empty name and shows the translated default "Groceries".
- Add field: fixed above the tab bar on phones (thumb reach), at the top on wide screens. One line can hold many items: split on commas, semicolons and newlines (a decimal comma such as "1,5 kg" is kept). See quick-add parsing below.
- Items are grouped under store-section headings in a fixed order (13 categories, below). Headings only show when there is more than one section.
- Tick: tapping a row moves it to "In the cart" with a 150 ms slide (none with reduced motion). On a wide container (>= 640 px and something in the cart) the cart is a side column; otherwise it is folded under the list, and ticking into a folded cart shows a toast "In the cart" with Undo (`movedToHiddenCart`, `src/web/format.ts`). Tapping a cart item puts it back.
- "Clear the cart" (two-tap confirm) soft-deletes all bought items of that list. Bought items not cleared drop out of view after 7 days anyway (`BOUGHT_VISIBLE_MS`).
- Row "..." menu (popover on wide, action sheet on phone): Edit item (title max 120, quantity max 40, section dropdown) and Delete item (immediate, toast with Undo that restores the same row).
- Items queued offline show at the top with "Waiting to send".
- No manual drag reorder of items or lists, no per-store aisle order, no item notes, no prices, no photos, no barcode scan, no "usuals"/favorites list, no per-item assignee.

**Sharing and visibility**: every list and every task is family-wide. There are no private lists or private tasks; anyone with `lists.view` (all built-in roles including Guest) sees everything. "Mine" is only a filter.

**AI / Quick add flows** (handled by the inbox/assistant extensions, contributed here as actions, draft presenters and AI context)
- Messages, photos or voice notes can produce draft cards: add task, change task, complete task, delete task, add shopping items, tick off item, remove item (`src/web/presenters.ts`). Draft cards for tasks and shopping items have inline editors (`TaskDraftEditor.svelte`, `ShoppingDraftEditor.svelte`).
- The add-shopping draft card names the target list when the family has more than one, and shows quantity changes that the add would make to items already on the list ("Rice: 1 kg -> a bag") by reading the list through the API (`src/web/shopping-draft.ts`).
- Open shopping items (newest 60) and open tasks (newest 40) are given to the model as context so a message like "got the light bulbs" can propose ticking an item off instead of adding it (`src/server/context.ts`).

### Data model

From `src/server/store.ts` (migration `0001_init`, STRICT tables, timestamps epoch ms, ids UUIDv7 with prefix):

- `lists_tasks`: `id` (`task_...`), `title`, `notes`, `assignee_id` (null = anyone; no FK), `due_date` (YYYY-MM-DD, next due for a repeating task), `due_time` (HH:MM), `rrule` (RRULE text), `rrule_start` (DTSTART of the series, so moving `due_date` never shifts the rule), `done_at`, `done_by`, `created_by`, `source_kind`, `source_id`, `created_at`, `updated_at`, `deleted_at` (soft delete). Index `(done_at, due_date)`.
- `lists_task_completions`: `id` (`task_done_...`), `task_id` (FK), `due_date` (the occurrence completed), `done_at`, `done_by`. One row per completion, used for Undo of a repeating task. Index `(task_id, done_at)`.
- `lists_shopping_lists`: `id` (`shop_list_...`), `name` (empty string for the install-time default), `sort`, `created_at`, `updated_at`, `archived_at`. A default list with fixed id `shop_list_00000000-0000-7000-8000-000000000001` is inserted by the migration.
- `lists_shopping_items`: `id` (`shop_item_...`), `list_id` (FK), `title`, `title_key` (normalized title for dedupe and category memory), `quantity` (free text), `category` (enum below), `done_at` (bought), `done_by`, `added_by`, `source_kind`, `source_id`, `created_at`, `updated_at`, `deleted_at`. Indexes `(list_id, done_at)` and `(title_key, updated_at)`.
- Categories (`CATEGORIES`, `src/contract/index.ts`), which double as the store-section sort order: produce, bakery, dairy, meat, pantry, frozen, drinks, snacks, household, personal, baby, pets, other.
- Contract `Task` exposes `recurrence` (parsed preset) and `rrule` (raw text only when the rule is outside the editable presets).

### Rules and logic worth copying

**Shopping dedupe and merge** (`lists.addShoppingItems`, `src/server/shopping.ts`)
- Match key: `titleKey(title, locale)` = trim, collapse whitespace, lower-case using the family locale (so Turkish dotted I folds correctly) (`src/contract/amount.ts`).
- For each incoming item, look for an existing non-deleted item on the same list with the same key, preferring an open one over a bought one, then most recently updated.
- `whenOnList: 'update'` (default): reuse the row, un-tick it if bought, replace quantity only when the add has one (an empty quantity never clears an existing one), replace category only when given. `whenOnList: 'keep'`: an open item is left exactly as it is; a bought item still comes back.
- Output returns `addedIds` and `merged` (each merged row's prior quantity, category, done state). Undo (`lists.undoAddShoppingItems`) soft-deletes the added rows and restores merged rows exactly.
- Category memory: an item added without a category gets the category last used for the same `title_key` on any list, ignoring `other`, else `other`. The query does not filter deleted rows, so memory survives deletion and clearing.
- `alreadyThere` hook (used to flag AI drafts as possible duplicates) returns true / `'some'` / false depending on how many of the proposed keys are open on any list.

**Quick-add parsing** (`src/web/quick-add.ts`, `src/contract/amount.ts`)
- Split on `\n`, `;`, and commas not followed by a digit.
- Leading amount with optional unit becomes the quantity: "2 kg tomatoes" -> title "tomatoes", quantity "2 kg". Numbers can be integers, decimals with `.` or `,`, simple fractions "1/2", mixed "1 1/2", Unicode vulgar fractions "½", "1½", and ranges "2-3".
- A trailing amount is read only when a unit is present ("Parmesan 150 g"), so "iPhone 15" stays whole.
- A part that is only an amount attaches to the previous item ("hooks, 4").
- Units come from the language pack (`lists:quantity_units`: kg, g, mg, l, ml, cl, pcs, pack(s), lb(s), oz, gal, qt, pt, doz, dozen, can(s), bottle(s), x), matched longest first. Max 50 items per add. Nothing is truncated: an over-long item blocks the add with a message.
- No unit conversion or quantity arithmetic anywhere: quantity is free text and a new one replaces the old.

**Repeating tasks** (`lists.completeTask`, `src/server/tasks.ts`)
- Recurrence presets: daily / weekly (by weekday set) / monthly, each with `interval` and optional `until` (`RecurrenceSchema`, `packages/time/src/index.ts`), stored as RRULE with DTSTART = `rrule_start`.
- Completing records a `lists_task_completions` row, then moves `due_date` to the next occurrence strictly after max(today, current due date). Missed occurrences are skipped, so an overdue weekly task completed late jumps to the next future date rather than the next missed one (test "rolls a repeating chore forward, skipping missed occurrences" in `test/lists.test.ts`). When no next occurrence exists (past `until`), the task is marked done.
- Stale guard: the client sends the `dueDate` it saw; if the task is already done or its due date has moved on, the call is a no-op. This makes double taps and offline replays safe. The action's `settled` hook closes AI drafts that would change nothing.
- `lists.reopenTask` (Undo) deletes the latest completion row and restores its `due_date`, clearing `done_at`.
- Editing the due date of a repeating task via `changeTask` restarts the rule from the new date (`rrule_start = dueDate`).
- Monthly repeats use plain RRULE semantics through `nextDate` (`packages/time`); unlike Chores there is no month-end clamping.

**Reminders** (`src/server/reminders.ts`)
- One `lists.remind` job per task, enqueued in the same transaction as each insert/edit/complete/reopen/restore, at the due time or 09:00 when there is no time (family time zone). Dedupe key includes task id, due date and `updated_at`; the job checks that the task still matches that version, is open, and that the due day has not ended (a server that was down does not send stale reminders).
- Recipient: the assignee, or every holder of `lists.view` when unassigned (by default that includes Guests). Notification kind `lists.taskDue`, dedupe `reminder:<task>:<date>`, link `/lists/tasks`.
- Repeating tasks are re-planned when completion moves the date.

**Permissions** (capability levels map to roles in `packages/sdk/src/capabilities.ts`: view = all, use = admin/member/supervised, edit = admin/member, manage = admin)
- `lists.view` (view), `lists.tasks.complete` (use), `lists.tasks.edit` (edit), `lists.shopping.edit` (use), `lists.shopping.manage` (edit).
- Supervised members: may add a task assigned to themselves directly (`addTask` approval exception), but a task for someone else, edits, deletes and restores become drafts for an adult to approve. They may add, edit and tick shopping items directly; deleting items, bulk removal, clearing the cart, undoing an add, and creating/renaming/removing lists become drafts (`test/lists.test.ts`).
- Any holder of `lists.tasks.complete` can complete any task, not only their own (no ownership check in `completeTask`), unlike Chores.
- AI actors: everything non-read-only is a draft; `completeTask` and `reopenTask` declare `approval: 'ai'`. Auto-approval (admin setting) can apply AI-proposed shopping adds and tasks automatically when the person they are for holds a chosen capability (`autoApprove` on `addShoppingItems` and `addTask`; `test/auto-approval.test.ts`).

**AI targeting safeguards**
- Actions that act on an existing row (`changeTask`, `completeTask`, `deleteTask`, `checkShoppingItem`, `deleteShoppingItem`) take a `label` that must be the person's own words naming the target; the runner checks it against the real title (`targetTitle`) and drops suggestions that point at something else ("Milk" cannot slide onto "Oat milk"; `test/ai-targeting.test.ts`).
- Lists can be named in words: `resolveNames` maps a name (or aliases "Shopping list", "Groceries", "Market" for the unnamed default list) to an id before the draft is stored, and refuses unknown names with the real list names.
- `lists.shoppingList` read tool returns open items in section order plus items bought in the last 7 days flagged `bought`, so the model can tell "already bought" from "not on the list"; a `text` filter finds one item in a long list; capped at 100 items with `truncated`.
- Task duplicate check (`addTask.alreadyThere`) matches by title words regardless of due date while open; a done task only on the same due date.
- Task drafts never expire even when the due date has passed (`test/lists.test.ts` "never expires").

**Offline** (see also `docs/guide/offline.md`): `addShoppingItems`, `checkShoppingItem`, `completeTask`, `reopenTask` are `offline: true`. The web client queues them with a one-time key; pages show queued state optimistically with "Waiting to send", re-read when the queue shrinks, and reload on reconnect. Replays are idempotent: same key applies once, a re-add of the same product merges, a late tick on a deleted item becomes a stored `not_found`, a repeating task completed offline twice advances only once (`test/offline.test.ts`).

### Integrations

- No external services. No `ctx.http` use.
- Cross-extension: Kitchen calls `lists.addShoppingItems` and `lists.shoppingList` as actions (never the tables) and reuses the contract's `titleKey`, `amountPattern`, `splitAmount`, `CATEGORIES`, size limits (`extensions/kitchen/src/server/shopping.ts`).
- AI: contributes two AI context providers (`lists.aiContext`, `lists.aiContextTasks`), two read tools (`lists.openTasks`, `lists.shoppingList`) and AI-proposable write actions; used by Quick add, the assistant and MCP agents.
- Realtime events `lists.task.changed` and `lists.shopping.changed` drive UI reloads.

### Today / display relevance

Slots declared in `src/web/index.ts`:
- `dashboard.card` `lists.overdue` in band `needs` (Needs you): late tasks that are mine or unassigned, up to 5, each with "Tomorrow" (postpone a one-off task by setting due date to tomorrow) and "Done" buttons (`src/web/OverdueCard.svelte`).
- `dashboard.card` `lists.tasks` in band `notes`, `todo: true`: open non-late tasks, today first, mine before others, 3 rows (5 when wide), tick in place (`src/web/TasksCard.svelte`).
- `dashboard.card` `lists.shopping` in band `notes`, `glance: true`: the first shopping list only. Rows form shows item chips (6, or 10 wide); glance form (Wall/Broadsheet day-at-a-glance and family displays) shows a big number "N items to get" with the first three named. On a display it renders without a link because displays have no app pages (`src/web/ShoppingCard.svelte`).
- `display.widget` `lists.tasks` (requires `lists.view`): read-only family-display tile of open tasks, due/overdue first (highlighted), then undated, then later; up to 7 rows with avatar (names only if the display holds `core.members.view`) (`src/web/DisplayTasks.svelte`).
- Timeline contribution `lists.timedTasks`: open tasks due today with a time appear on Today's day timeline at that time, tagged with the assignee (`src/web/timeline.ts`).
- All cards report counts (`count`, `urgent`, `quietKey`) that Today uses for its head sentence and ordering. Displays never queue offline actions.

### Docs vs code

- `docs/guide/everyday.md` says "A number at the start becomes the quantity". Code also reads a trailing amount with a unit ("parmesan 150 g"), attaches a bare amount part to the previous item ("hooks, 4"), and also splits on semicolons and newlines (`src/web/quick-add.ts`).
- `everyday.md` says FamilyOS "remembers the section you last gave a product". Code: the memory is family-wide across all lists, ignores `other`, and includes deleted/cleared rows.
- `everyday.md` calls a task "one person's to-do", yet tasks support "Anyone" and repeating schedules; the contract describes recurrence as "Repeat rule for chores" and the extension comment says "family tasks (with repeating chores)", overlapping with the separate Chores extension.
- `everyday.md` says reminders go to "everyone when it is for Anyone". Code addresses all holders of `lists.view`, which by default includes Guests.
- `docs/guide/offline.md` matches the code's offline set (add item, tick/untick item, complete/reopen task).
- The guide does not mention that done one-off tasks and bought items disappear from view after 7 days.

### Unfinished or stubbed

- `source_kind` / `source_id` columns on tasks and items are always written as null; provenance is reserved but unused.
- `lists_task_completions` comment says it "shows history later": there is no route or UI for task completion history.
- `interval` exists in the recurrence model and form state, but the task form has no control for it (`TaskForm.svelte`); "every 2 weeks" can only be set via AI/API. Rules outside the presets show as a disabled "Custom rule" and cannot be edited.
- `created_by`, `done_by` and `added_by` are stored but not shown in any UI.
- `lists_shopping_lists.sort` is set on creation only; there is no action to reorder lists.
- The Today shopping card and display glance only ever show the first list.
- OverdueCard's "Tomorrow" calls `lists.updateTask`, which drafts for people without `lists.tasks.edit`; the card ignores the outcome, so a supervised member gets no "Sent for approval" toast and the task appears unchanged (`src/web/OverdueCard.svelte`).

### Beyond Home HQ (net-new ideas)

Home HQ already has a shared grocery list with stores, categories and usuals. Things FamilyOS does that are worth considering:
- A general family task list separate from chores: one-off to-dos with assignee or "Anyone", due date and time, optional repeat, reminders at the due time, grouping by urgency, and an "Overdue" card in a "Needs you" band with one-tap "Tomorrow" postpone.
- Timed tasks placed on the Today timeline next to calendar events.
- Multiple named shopping lists (pharmacy, hardware) addressable by name from AI, with aliases for the default list.
- Merge-on-add semantics: no duplicates, bought items resurrect, quantity replace-vs-keep, and an undo that restores merged rows exactly.
- Free-text quick add that parses many items and amounts (fractions, ranges, decimal comma, trailing units) from one line.
- Category memory per product family-wide.
- "In the cart" model: ticked items stay visible (side column or folded) until cleared, auto-hide after 7 days.
- Offline queue for in-store use with idempotent replays and visible "Waiting to send" state.
- AI capture from messages/photos/voice with draft approval, duplicate hints, "already done" auto-closing, target-label guard, and optional auto-approval for routine adds.
- Role-aware permissions: children add and tick freely but deletions and list management wait for a parent.
- Read-only wall-display tasks widget.

### Evidence

- `extensions/lists/src/contract/index.ts`, `extensions/lists/src/contract/amount.ts`
- `extensions/lists/src/server/index.ts`, `store.ts`, `tasks.ts`, `shopping.ts`, `reminders.ts`, `context.ts`
- `extensions/lists/src/web/index.ts`, `TasksPage.svelte`, `TaskForm.svelte`, `TaskRow.svelte`, `ShoppingPage.svelte`, `OverdueCard.svelte`, `TasksCard.svelte`, `ShoppingCard.svelte`, `DisplayTasks.svelte`, `quick-add.ts`, `shopping-draft.ts`, `presenters.ts`, `format.ts`, `task-form.ts`, `timeline.ts`, `lists-page.ts`
- `extensions/lists/src/messages/en.json`
- `extensions/lists/test/lists.test.ts`, `offline.test.ts`, `reminders.test.ts`, `ai-targeting.test.ts`, `auto-approval.test.ts`, `list-names.test.ts`, `quick-add.test.ts`
- `packages/sdk/src/capabilities.ts` (role defaults per capability level), `packages/time/src/index.ts` (RecurrenceSchema, nextDate)
- `docs/guide/everyday.md`, `docs/guide/offline.md`, `docs/guide/auto-approval.md`

---

## 5. Chores

`extensions/chores` (id `chores`) handles shared household work that repeats and takes turns between people. It is separate from Lists tasks: a chore has a list of people in turn order and a rotation strategy, and the turn passes on automatically when it is done. There are no points, streaks, stars or rewards anywhere in the code.

### What the user can do

**Chores page** (`/chores`, also `/chores/:id` to open one in the side panel; `src/web/ChoresPage.svelte`, nav under Household, requires `chores.view`)
- Groups: Overdue, Today, Coming up (every future due date, no horizon), Done (one-off chores finished in the last 7 days, showing "Done by <name>") (`src/web/form.ts` `groupOf`; `listAllChores` in `src/server/chores.ts`).
- Filter Everyone | Mine. Mine = chores whose current turn is mine plus chores anyone may do. When nothing is open for me it says "Nothing is waiting for you."
- Each chore is a tile (`src/web/ChoreRow.svelte`): a round check whose ring takes the current assignee's member color, the title, a repeat icon, the due label ("Today", "Yesterday", "3 days overdue", "Fri 9 Oct"), whose turn it is, and the assignee's avatar. If it is someone else's turn and I may not tick it, the check is disabled with the hint "<name>'s turn".
- One tap marks done. Toast "<title> done" with Undo. A queued offline tap shows "Waiting to send".
- Add chore (editors) or Suggest a chore (people with `chores.use` but not `chores.edit`, i.e. supervised members by default; it becomes a draft for approval). A FAB on phones.
- Tapping a chore opens the edit form only for editors; others cannot open existing chores. The form has Archive (two-tap confirm) as its danger action.
- No completion history view, no stats per person, no skip or snooze, no photo proof, no parent approval of a completion, no due time of day.

**Chore form** (`src/web/ChoreForm.svelte`, `src/web/form.ts`)
- Chore title (max 120), First day it is due (date), Repeats via the shared `RepeatPicker` (Once / Every day / Every week with weekday chips / Every month, plus Until), Who takes turns (multi-select member chips), Turn order (when 2+ people: numbered list with move up / move down buttons), How the turn moves on (when 2+ people: Always the same person / Take turns / Whoever did it least, each with a hint), Notes (max 1000).
- New chores default to weekly on today's weekday with rotation "Take turns". With 0 or 1 person the rotation is forced to `fixed` on save.
- Saving an edit calls `chores.updateChore` and, if the people changed, `chores.setAssignees` as a second action.

**AI** (via Quick add, assistant, MCP; all writes are drafts a person approves)
- Propose a new chore with people and rotation (`createChore`), propose a fair split of an existing chore (`setAssignees`, shown as "Who does it: A -> B"), propose marking a chore done with an optional `doneBy` (`markDone`), and answer "whose turn is the dishes?" / "what do I have this week?" through the read tool `chores.listChores` (`src/server/chores.ts`, `src/web/presenters.ts`).
- Not offered to the AI: `undoDone`, `updateChore`, `archiveChore` (`test/chores.test.ts` "the AI").

### Data model

From `src/server/store.ts` (migration `0001_init`, STRICT):

- `chores_chores`: `id` (`chore_...`), `title`, `notes`, `recurrence` (JSON of the `Recurrence` preset, not RRULE; null = one-off), `start_date` (first due date, also the anchor for monthly counting), `next_due` (null once a one-off is done), `rotation` (`fixed` | `round_robin` | `least_done`, CHECK constraint, default `fixed`), `assignee_id` (whose turn now; null = anyone; no FK), `created_by`, `created_at`, `updated_at`, `archived_at`. Index `(archived_at, next_due)`.
- `chores_assignees`: `chore_id` (FK, cascade), `member_id`, `position`. PK `(chore_id, member_id)`. Turn order.
- `chores_completions`: `id` (`chore_done_...`), `chore_id` (FK, cascade), `member_id` (credited doer), `due_date` (the occurrence completed), `prev_assignee_id` (whose turn it was, for undo and for Today's rings), `done_at`, `undone_at` (soft undo). Partial unique index `chores_completions_live_idx` on `(chore_id, due_date) WHERE undone_at IS NULL` guarantees one live completion per occurrence. Index `(member_id, done_at)`.
- Contract `Chore` view adds `overdue`, `assigneeIds` (ordered), `lastDone {memberId, doneAt}`. `ChoreDetail` adds `history` (last 30 completions, including undone ones) (`src/contract/index.ts`).

### Rules and logic worth copying

**Rotation** (`src/server/rotation.ts`, pure function `nextAssignee`; run only when the chore has a next due date)
- `fixed`: whoever is up stays if still in the list, else the first person.
- `round_robin`: the person after whoever did it (or after whoever was up, if the doer is not in the list), wrapping to the first.
- `least_done`: the listed person with the fewest live completions of this chore in the last 90 days (`LEAST_DONE_DAYS`), counting the completion just made; ties go to list position.
- Empty list: anyone (null), and the chore stays open to everyone.
- Changing the people (`setAssignees`) keeps the current turn if that person is still listed, else the first person is up. Reordering alone never changes whose turn it is now.

**Scheduling** (`src/server/schedule.ts`)
- Daily and weekly use RRULE expansion from `@famos/time` (`nextDate`) with DTSTART = `start_date`; weekly only counts DTSTART if its weekday is in the rule.
- Monthly is custom: occurrences are `start_date + k * interval months`, always computed from the start (never from the previous occurrence), and Temporal's month arithmetic clamps a missing day to the month's last day (Jan 31 -> Feb 28 -> Mar 31). This differs from Lists tasks, which use plain RRULE.
- After a completion: next due = first occurrence strictly after max(today, completed due date). Missed occurrences are skipped, so a chore never piles up behind a holiday; completing early keeps the schedule.
- Editing a chore reschedules only when `start_date` or `recurrence` changed, to the first occurrence on or after max(start, today); other edits keep `next_due`. A repeat whose `until` is before the first due date is refused.
- A one-off chore has `next_due = start_date`; after completion `next_due` becomes null and it shows in Done for 7 days.

**Completion** (`chores.markDone`, `offline: true`)
- Input carries the `dueDate` the caller saw. If that occurrence already has a live completion the result is `reason: 'already_done'`; if the chore has moved to another date, `reason: 'stale'`. Nothing changes in either case. This makes double taps and offline replays harmless, and the `settled` hook closes AI drafts that would change nothing.
- Who gets credit: an explicit `doneBy`, else the current assignee, else the person tapping (for an anyone-chore). So when a parent ticks a child's chore, the child is credited and fairness for `round_robin` / `least_done` is preserved. The web client passes `doneBy = assigneeId` when the tapper can edit (`src/web/data.ts`).
- Who may tick: without `chores.edit`, a member may only tick a chore whose turn is theirs or that anyone may do, and only for themselves; ticking someone else's turn or naming another doer is `forbidden`. With `chores.edit`, anything (`test/chores.test.ts` "who may do what").
- Guests (no `chores.use`) cannot tick.

**Undo** (`chores.undoDone`, `offline: true`)
- Without `chores.edit`: only your own completion made on the same family day.
- Only the newest live completion of a chore can be undone (`conflict` otherwise).
- Restores `next_due` to the undone occurrence and the turn to `prev_assignee_id` if that person is still listed, else the first person. Sets `undone_at` rather than deleting.

**Reminders** (`src/server/reminders.ts`)
- One `chores.remind` job per chore at 09:00 family time on the due day, re-planned in the transaction of every change (create, edit, set people, done, undo). Only the person whose turn it is is notified; anyone-chores remind nobody. Payload carries `updated_at`; a job that no longer matches (done, handed over, moved, archived) does nothing, and a day that has already ended is skipped.
- There is no repeat reminder for an overdue chore: after its due day passes nothing more is sent until the chore changes.
- Notification kind `chores.choreDue` ("Your turn: <title>", "It is due today."), audience `chores.view`, opt-out per person in notification preferences.

**Today counting** (`src/server/today.ts`, `todayByMember`)
- Per person: total = chores due today or overdue (counted for whoever is up now) + chores completed today (counted for whoever's turn it was, `prev_assignee_id`); done = the latter. So a ring stays whole after a tap, even if the turn moved on.
- Chores held by someone no longer in the family count for "anyone". Only people with something today are listed, in family order, with anyone last. "Today" is the family time zone's day.
- A completion made today for a future occurrence also counts as done today (the query filters on `done_at`, not due date).

**Permissions** (levels from `packages/sdk/src/capabilities.ts`)
- `chores.view` (view: all roles incl. guest), `chores.use` (use: admin, member, supervised), `chores.edit` (edit: admin, member).
- Create, update, archive and set people need `chores.edit` or become drafts (`approval: { unless: chores.edit }`).
- Duplicate hint for drafts: `createChore.alreadyThere` matches an active chore with exactly the same title (case-sensitive).

### Integrations

- None external. No `ctx.http`.
- AI: write actions with `ai` hints (`createChore`, `setAssignees`, `markDone`) and read tool `listChores` (returns `today`, per chore `nextDue`, `overdue`, `assigneeId`, `assigneeIds`, `rotation`, `repeats`; max 60). AI guidance asks the model to spread chores fairly using members' descriptions. The family can switch Chores off for the AI under Settings -> AI.
- Realtime event `chores.changed` drives reloads on the page, the Today card and Today's rings.

### Today / display relevance

- `dashboard.card` `chores.today` (band `notes`, `glance: true`, requires `chores.use`) (`src/web/ChoresCard.svelte`). Rows form, titled "Chores today": my chores due today or late plus anyone-chores, mine first, 3 rows (5 wide), one-tap tick with Undo toast, overdue meta in attention color, assignee avatar. Glance form: the card draws nothing; the Today page itself draws per-person rings from `GET /api/v1/chores/today`.
- How core Today uses the rings (`apps/web/src/lib/today/today-data.svelte.ts`, `Wall.svelte`, `Lanes.svelte`, `Broadsheet.svelte`): Wall puts a ring on each person's "Who, next" row; Lanes folds chores into the lane heads; a person's status line falls back to "the chores they have left" when they have nothing timed; with names off (a display without member names) only the family's total shows as a tile. On a family display chores are the last block dropped to fit one screen (`display-fit.ts`).
- Displays read only routes, never actions: `GET /chores/today` exists separately from the `chores.todayByMember` action for that reason; pairing a display offers `chores.view` (`today-data.svelte.ts` `CHORES_CAPABILITY`).
- No `display.widget` contribution of its own.

### Docs vs code

- `docs/guide/chores.md` says the AI "never ticks chores off", and the server entry comment (`src/server/index.ts`) says the AI "never touches completions". The code offers `chores.markDone` to the AI as a draft-producing action with a presenter ("Mark done"), and tests assert it is in the AI catalog and that approving it records the completion (`test/chores.test.ts` "lets the AI propose marking a chore done").
- The guide calls the Today card "My chores today"; the code title is "Chores today" (`card_title` in `src/messages/en.json`), and it includes anyone-chores as well as mine.
- The guide's undo rule ("undo your own tick the same day; someone who can edit chores can undo any") omits that only the latest completion of a chore can be undone.
- The guide's role table says Supervised "Add, change, assign: suggestions wait for approval". In the UI a supervised member can only suggest a new chore; existing chores do not open for them, so change/assign/archive suggestions are possible only through the API or AI.
- The guide says "Take turns is chosen at first": true for the web form; the API/AI contract defaults `rotation` to `fixed`.
- The guide does not mention that overdue chores get no further reminders, or that monthly scheduling is counted from the start date (it does mention the month-end rule).

### Unfinished or stubbed

- `GET /chores/chores/{choreId}` returns a 30-entry completion history (with undone entries), but no web UI calls it; the page reads only the list route.
- No unarchive / restore action for an archived chore (archive is also the undo of an approved AI-created chore).
- `interval` exists in the recurrence model but `RepeatPicker` has no interval control, so "every other week" is API/AI only (`packages/ui/src/famos/RepeatPicker.svelte`).
- `created_by` is stored but never shown.
- No way in the UI to hand the current turn to a different listed person, or to credit a different doer; `doneBy` is settable only via AI/API.
- No skip-this-occurrence: an overdue chore stays overdue until someone completes it.

### Beyond Home HQ (net-new ideas)

Home HQ has chores per person with star rewards and time-of-day routines. FamilyOS has no rewards or routines, but it does have things Home HQ lacks:
- Shared chores with an ordered rota and three rotation strategies: fixed, take turns (round robin) and "whoever did it least in the last 90 days".
- An "anyone may do it" mode, where the tapper is credited.
- Credit attribution: an adult ticking a child's turn credits the child, so the rota stays fair.
- Turn-aware permissions: a child can tick only their own turn or an anyone-chore; the button explains "<name>'s turn".
- Skip-missed scheduling (no backlog after holidays), month-end clamping for "the 31st", and early completion keeping the schedule.
- Idempotent completion per occurrence (unique live completion per due date) with offline taps, plus a strict undo model (own and same day, latest only) that restores the previous turn.
- Per-person "done of total today" rings that stay correct after the turn passes on.
- Due-day reminder to whoever is up, sent once and only while the day lasts.
- AI suggestions for new chores and for fair redistribution among members, all as approvable drafts with undo.

### Evidence

- `extensions/chores/src/contract/index.ts`
- `extensions/chores/src/server/index.ts`, `chores.ts`, `rotation.ts`, `schedule.ts`, `today.ts`, `reminders.ts`, `shared.ts`, `store.ts`, `capabilities.ts`
- `extensions/chores/src/web/index.ts`, `ChoresPage.svelte`, `ChoreForm.svelte`, `ChoreRow.svelte`, `ChoresCard.svelte`, `data.ts`, `form.ts`, `presenters.ts`
- `extensions/chores/src/messages/en.json`
- `extensions/chores/test/chores.test.ts`, `rotation.test.ts`, `today.test.ts`, `offline.test.ts`, `migrations.test.ts`
- `apps/web/src/lib/today/today-data.svelte.ts`, `Wall.svelte`, `Lanes.svelte`, `display-fit.ts`
- `packages/sdk/src/capabilities.ts`, `packages/ui/src/famos/RepeatPicker.svelte`
- `docs/guide/chores.md`, `docs/guide/offline.md`, `docs/guide/everyday.md`

---

## 6. Kitchen

`extensions/kitchen` (id `kitchen`) has a recipe box, a weekly meal plan, and a one-way "ingredients to the shopping list" handoff into Lists. Ingredients are free-text lines on purpose: no structured quantities, no unit conversion, no servings scaling, no pantry or inventory, and no nutrition. Nav entry is under Household (`/kitchen`, requires `kitchen.view`) with two tabs, Recipes and Meal plan (`src/web/KitchenFrame.svelte`).

### What the user can do

**Recipes tab** (`/kitchen`; `src/web/RecipesPage.svelte`)
- A list of recipes sorted by title, with a photo thumbnail when there is one, servings and minutes. On wide windows the selected recipe opens on the right (list-detail); on a phone one or the other.
- Search box (substring of the title only, filtered in the browser over the full list) and label chips built from all recipes' tags.
- Each row has a calendar button that opens the "plan this" panel.
- Add recipe (form in a side panel / phone sheet; FAB on phones). People without `kitchen.edit` who may propose (supervised members) submit a draft for approval; guests cannot add.

**Recipe form** (`src/web/RecipeForm.svelte`, `src/web/form.ts`, limits in `src/contract/index.ts`)
- Name (required, max 160); Servings as free text ("4", "4-6", max 30); Total time in minutes (1 to 10080); Ingredients (one per line, max 100 lines of 300 chars); Steps (one per line, max 100 of 3000); Notes (max 4000); Source web address (http/https only, no embedded credentials; stored as a link, never fetched); Labels (comma-separated, max 20 of 40 chars, deduplicated).

**Recipe page** (`/kitchen/recipes/:id`; `src/web/RecipePage.svelte`)
- Header with the photo (or a pot icon), servings and minutes; buttons "Put it on the meal plan" and Edit (for editors and proposers); a "..." menu with Add/Change photo (editors only).
- Tags as pills and a "Source: <host>" external link.
- Ingredients as a checklist with "Select all" and "Add selected to shopping list". The button only appears when the person can read shopping lists (Lists on and visible). With more than one list, a list picker appears. After adding, a toast gives the headline and a short list under the ingredients says which items were already on the list (with both quantities when they differ) and which lines were not added (`src/web/shopping-summary.ts`).
- Steps numbered; Notes.
- Edit opens the side panel; Archive (two-tap confirm) sits in that panel's danger area with the hint "Archiving hides the recipe. Meals on the plan that use it stay."
- Photo upload: one JPEG, PNG or WebP under 8 MB.
- No cook mode, no step timers, no keep-awake, no scaling, no print view, no rating, no "cooked on" history.

**Meal plan tab** (`/kitchen/plan`, `?date=YYYY-MM-DD` opens that week; `src/web/PlanPage.svelte`, `src/web/plan.ts`)
- One week at a time, starting on the family's first weekday; seven columns from 1280 px, stacked days below. Today is highlighted with a "Today" pill and a lit card.
- Each day always shows Breakfast, Lunch and Dinner; Other appears only when it has a meal. Several meals can share a slot.
- Tap an empty slot, or the "+" on a day (defaults to Dinner), to open the plan panel. "x" removes a meal immediately (no confirm, no undo toast in this UI; the action itself declares an undo for AI drafts).
- Week arrows and "This week".
- Planned recipe meals link to the recipe page; written dishes are plain text.
- Supervised members see "Your changes to the plan wait for approval."; guests see "Breakfast · Nothing planned" rows read-only.
- No editing or moving of a planned meal in the UI, no copy-last-week, no drag and drop, no "shop for this week".

**Plan panel** (`src/web/PlanPanel.svelte`)
- From a recipe: pick Day and Meal (breakfast, lunch, dinner, other).
- From an empty slot: pick Meal, then a recipe from a radio list of all recipes (no search inside the panel) or "Something else (write it)" with a free-text dish (max 160).
- Duplicate guard: if the same recipe, or the same written dish ignoring case and spacing, is already planned for that day and meal, a warning appears and "Add it anyway" must be ticked (`src/web/plan-duplicate.ts`).
- After adding from a recipe: toast "Added to the meal plan" with "See the plan".

**AI** (via Quick add, assistant, MCP; drafts a person approves)
- A shared photo of a recipe, a page of text or a message becomes a proposed recipe: title, ingredients one per line in the original language, steps in order without numbering, servings/time only when stated, source URL only when visible (`createRecipe` hints, `src/server/recipes.ts`).
- Propose a meal for a day ("pasta on Friday") with either a recipe handle or a written dish (`addToMealPlan`), and propose taking a planned meal off (`removeMealPlanEntry`, guarded by a label naming the meal).
- Read tools: `kitchen.listRecipes` (up to 30, optional title query) and `kitchen.mealPlan` (from a day, 1 to 31 days, default 7 from today).
- Family search indexes each recipe's title, ingredients, steps, tags and notes (`src/server/search.ts`), linking to the recipe page.

### Data model

From `src/server/store.ts` (migration `0001_init`, STRICT):

- `kitchen_recipes`: `id` (`recipe_...`), `title`, `servings` (text), `total_minutes` (integer), `ingredients` (JSON array of strings, default `[]`), `steps` (JSON array, default `[]`), `notes`, `source_url`, `photo_key` (file store key `kitchen/recipes/<recipeId>/<photoId>`), `photo_type` (sniffed MIME), `tags` (JSON array), `created_by`, `created_at`, `updated_at`, `archived_at` (soft archive). Index `(archived_at, title)`.
- `kitchen_meal_plan`: `id` (`meal_...`), `date` (YYYY-MM-DD), `slot` (`breakfast` | `lunch` | `dinner` | `other`, default `dinner`), `recipe_id` (nullable FK to `kitchen_recipes.id`), `title` (written dish when no recipe, default ''), `note`, `created_by`, `created_at`, `updated_at`. Index `(date, slot)`. Hard delete on removal.
- A meal entry's displayed title is the recipe's current title when linked (even if the recipe is archived), else the written title (`toMeal`).
- Contract: a meal needs a recipe or a title (`AddToMealPlanSchema` refine). `GET /kitchen/meal-plan?from&to` is capped at about two months (62 days).

### Rules and logic worth copying

**Ingredient line to shopping item** (`src/server/ingredient.ts`, `ingredientToItem`; examples from `test/ingredient.test.ts`)
- Bracketed text containing a digit joins the quantity ("1 tin chopped tomatoes (400 g)" -> "Chopped tomatoes", "1 tin (400 g)"); other bracketed notes are dropped ("1 lemon (optional)" -> "Lemon", "1").
- The line is split at commas not followed by a digit. A part after a comma is dropped when it contains a preparation phrase from the language pack ("60 g feta, crumbled" -> "Feta", "60 g"; "Lemon, zest and juice" -> "Lemon"; "Butter, plus extra for the pan" -> "Butter"). A part that is only an amount becomes the quantity ("Lime, 1", "Flour, 2 cups"). Other parts stay in the name: "salt, pepper" becomes one item "Salt, pepper".
- A leading amount (integers, decimal comma or point, "1/2", "1 1/2", "½", "1½", ranges "2-3") with an optional unit becomes the quantity. Units are the Lists units plus kitchen units from the language pack (tbsp, tsp, cup(s), tin(s), can(s), jar(s), clove(s), pinch(es), handful(s), slice(s), bunch(es), sprig(s), stick(s), piece(s), head(s), sachet(s), packet(s), fl oz).
- A link word after the amount is stripped ("2 cups of rice" -> "Rice", "2 cups"; `kitchen:words_amount_link` = "of").
- The first letter is capitalized in the family locale; title and quantity are truncated to the Lists limits (120 / 40).
- All words (units, link words, around 70 preparation phrases) are language-pack vocabulary read with `uow.vocabulary`, not code, so other languages work (Turkish is tested).

**Ingredients to the shopping list** (`kitchen.addIngredientsToShopping`, `src/server/shopping.ts`)
- Only lines that match the recipe's own lines are used. Matching is loose: trim, collapse spaces, strip trailing punctuation, lower-case. Anything else comes back in `unknownLines`, so the client cannot inject arbitrary items.
- Parsed items are deduplicated within the request by the Lists `titleKey`.
- Open items on the target list (read through `lists.shoppingList`, so with the person's own visibility) are themselves cleaned with the same parser before comparison, so an old entry typed like a recipe line ("Lemon, zest and juice") matches the cleaned item "Lemon".
- Matched items are left exactly as they are and reported in `existing` with `listQuantity` and `recipeQuantity` ("Chicken breast is already on the list (500 g; the recipe needs 600 g)"). Quantities are never summed, raised or lowered.
- The rest goes to `lists.addShoppingItems` with `whenOnList: 'keep'`, run as the person, so Lists' permission and approval rules apply (a guest gets `forbidden`; with Lists disabled the call fails with `feature_off` and nothing is added). A bought item that matches comes back onto the list and is counted as added. No category is sent, so Lists' per-product category memory applies.
- Returns `added`, `alreadyThere`, `existing[]`, `unknownLines[]`; the UI turns these into a headline and detail lines.

**Meal plan**
- Duplicate checks at two levels: the UI's case/space-insensitive `plannedAlready` with an explicit "Add it anyway", and the server's `alreadyThere` hook (same recipe, or same exact written title, in the same date and slot) that flags AI drafts.
- AI meal suggestions carry `draftDate: input.date`, so a suggestion for a day that has passed is expired by the draft-expiry job and drops out of the pending count, while today's stays (`test/expiry.test.ts`).
- Removing an already-removed meal is a no-op; the removal returns the removed fields so undo re-adds it.
- Archiving a recipe keeps meal entries that reference it readable (title still resolved).

**Photos** (`uploadRecipePhoto` route + `setRecipePhoto` action)
- Multipart upload, max 8 MB; the type is sniffed from magic bytes (JPEG, PNG, WebP), never trusted from the client. The file is stored first, then the action records it; on failure the file is deleted again; the previous photo file is deleted after success. `setRecipePhoto` only accepts keys under that recipe's prefix. Only `kitchen.edit` may upload (supervised members cannot, even as a draft).
- Photo URLs are cache-busted with `?v=<updatedAt>`.

**Permissions**: `kitchen.view` (view: all roles including guest), `kitchen.edit` (edit: admin, member). Create, update and archive recipe, and add, update and remove meal become drafts without `kitchen.edit` (`approval: { unless: kitchen.edit }`); supervised members propose, guests only look (`test/kitchen.test.ts` "capabilities"). Turning Kitchen off for the AI closes both its read tools and its proposals for AI actors.

### Integrations

- No external fetching. `sourceUrl` is validated and shown as a link; nothing is fetched or scraped from it (no `ctx.http`).
- Recipe import from a photo or text happens through the AI pipeline of Quick add (inbox extension) using `createRecipe`'s AI hints; there is no dedicated importer, no URL import, no schema.org parsing.
- Lists integration only through actions (`lists.shoppingList`, `lists.addShoppingItems`) and the Lists contract helpers (`amountPattern`, `amountOnlyPattern`, `splitAmount`, `titleKey`, size limits), following the no-cross-table rule.
- File store for photos (`ctx.files`). AI search index for family search (`ai: { index: searchIndex }`).

### Today / display relevance

- No `dashboard.card` and no `display.widget`. Kitchen contributes a timeline source `kitchen.meals` (requires `kitchen.view`; `src/web/timeline.ts`): today's meals are placed at nominal hours (breakfast 08:00, lunch 12:30, dinner 18:30, other 16:00) with `nominal: true`, so Today shows no time for them and never treats one as "next". Title "Dinner: Lentil soup", subtitle "Meal plan", link to the recipe or the plan.
- Core Today collects timeline items of kind `meal` into its own "kitchen" block (`apps/web/src/lib/today/today-data.svelte.ts` `meals`; rendered as a list in `Broadsheet.svelte` and as one of three tiles in `Wall.svelte`, next to shopping as a number and what is ahead).
- On a family display the kitchen is part of the "extras" that are dropped, after the photo and atmosphere, to fit one screen (`display-fit.ts`). A display shows it only if it was granted `kitchen.view` at pairing.

### Docs vs code

- `docs/guide/kitchen.md` lists the AI's abilities as suggesting recipes and meals and answering questions; the code also lets the AI propose taking a planned meal off the plan (`removeMealPlanEntry` has AI hints and a presenter; `test/meal-ai.test.ts`).
- The guide says the recipe page has "Archive recipe"; in the code Archive is inside the Edit panel, not a top-level action.
- The guide's "Search recipes" matches titles only in the Recipes tab; ingredient and step text is searchable only through family search.
- The guide does not mention the meal `note` field, which exists in the model and AI drafts but has no input or display on the plan page.
- The guide says a preparation note after a comma is left out of the name. The code also keeps any other comma part in the name, so "salt, pepper" becomes one shopping item "Salt, pepper" rather than two.
- The guide's description of the shopping handoff (not added twice, left as is, quantity comparison message, list choice, needs Lists on) matches the code.

### Unfinished or stubbed

- `kitchen.updateMealPlanEntry` exists (change date, slot, recipe, title, note) but has no UI and is not offered to the AI; only the API/MCP can use it.
- Meal `note` has no UI.
- No unarchive for recipes; archive is also the undo of an approved AI-created recipe.
- `created_by` on recipes and meals is stored but never shown.
- The server recipe route accepts `q` and `tag`, but the Recipes tab loads everything and filters in the browser; `searchRecipes` reads at most 1000 rows.
- `servings` is free text, which rules out scaling as modeled.
- The plan panel's recipe picker is an unsearchable radio list of every recipe, which will not scale to a large collection.

### Beyond Home HQ (net-new ideas)

Home HQ has nothing for the kitchen, so all of this is new. The parts most worth considering:
- A recipe box with photo, servings, time, labels, source link, ingredient checklist and numbered steps.
- AI recipe capture from a photo or text (cookbook page, message) into a structured draft, kept in the original language.
- A weekly meal plan with breakfast/lunch/dinner/other slots, several meals per slot, saved recipes or free-text dishes, and a duplicate warning.
- Today's meals on the wall display's day view, placed at nominal hours and never shown as "next".
- Ingredients to the shopping list via a robust line parser (fractions, ranges, bracket amounts, preparation notes, link words, language-pack vocabularies), with dedupe against the existing list using cleaned titles, a rule never to overwrite what someone already put on the list, and a "you have 500 g, recipe needs 600 g" report.
- AI meal suggestions that expire once their day has passed.
- Gaps FamilyOS leaves open that Home HQ could fill: shop for the whole week's plan, servings scaling, a cook mode for a wall/tablet screen (large steps, timers, keep-awake), pantry staples, and URL import.

### Evidence

- `extensions/kitchen/src/contract/index.ts`
- `extensions/kitchen/src/server/index.ts`, `store.ts`, `recipes.ts`, `meal-plan.ts`, `shopping.ts`, `ingredient.ts`, `search.ts`, `capabilities.ts`
- `extensions/kitchen/src/web/index.ts`, `RecipesPage.svelte`, `RecipePage.svelte`, `RecipeForm.svelte`, `PlanPage.svelte`, `PlanPanel.svelte`, `KitchenFrame.svelte`, `timeline.ts`, `plan.ts`, `plan-duplicate.ts`, `shopping-summary.ts`, `presenters.ts`, `meal-draft.ts`, `data.ts`, `form.ts`
- `extensions/kitchen/src/messages/en.json` (`words_units`, `words_amount_link`, `words_preparation`)
- `extensions/kitchen/test/kitchen.test.ts`, `shopping.test.ts`, `ingredient.test.ts`, `expiry.test.ts`, `meal-ai.test.ts`, `plan-duplicate.test.ts`
- `extensions/lists/src/contract/amount.ts` (shared amount parsing)
- `apps/web/src/lib/today/today-data.svelte.ts`, `Broadsheet.svelte`, `Wall.svelte`, `display-fit.ts`
- `docs/guide/kitchen.md`, `docs/guide/everyday.md`

---

## 7. Household records: records, home, pets, vehicles

Four extensions work together. `records` is a shared engine with no page of its own. It owns "duties that come back" (obligations, with reminders and a completion history) and "papers" (files attached to a record). `home`, `pets` and `vehicles` own the records themselves (a home, a thing in a home, a utility, a pet, a vehicle). Each one renders the records UI on its detail pages through the `record.section` slot and calls records only through actions (`records.addObligation`, `records.renameRecord`, `records.archiveForRecord`, `records.completeObligation`). Records never reads the domain tables. It knows a record only by its typed id (`vehicle_<uuid>`, `pet_<uuid>`, `home_<uuid>`, `home_item_<uuid>`, `home_service_<uuid>`) plus a cached name and URL (`extensions/records/src/contract/index.ts`, `extensions/records/src/server/index.ts`).

`home` is household records: homes, appliances with warranties, utilities and subscriptions. It does no smart-home control.

All four are date-only and carry no money. There are no costs, amounts, "paid" flags, odometer readings or units. Code comments and AI guidance repeat this rule ("Do not put amounts of money anywhere").

### Shared records engine (`extensions/records`)

#### What the user can do

There is no Records page. Two sections appear on every vehicle, pet, home, thing and service page (`extensions/records/src/web/index.ts`, slot `record.section`):

- **What needs doing** (`RecordObligations.svelte`)
  - Lists the open duties soonest first. Each row shows a repeat or one-off icon, the title, a due label ("3 days overdue (12 Oct)", "Today", "Tomorrow", "In 12 days (…)", "Ended on …"), the repeat ("every 3 months"), the assignee and the notes.
  - **Add** opens a side panel (`ObligationForm.svelte`) with these fields: What, Due date, Repeats (none / every N months / every N years), "Remind me this many days before" (0–365, default 14), Who (one member or Anyone), Notes.
  - Picking a repeat starts the interval at 3 months or 2 years, not 1 (`repeat.ts`).
  - Tapping a row opens the same panel in edit mode. The panel shows **History** (completion dates and notes, newest first) and a two-tap **Remove**.
  - **Done** opens an inline note field (for example a new policy number), then **Save as done**. One-off "lapsing" dates (a warranty end) have no Done button.
  - A "What usually needs doing" / "Usually also:" chip row offers the domain's presets that are not already used by an open duty. A chip pre-fills the form.
  - A collapsible "N finished" list holds done one-offs and lapsed dates.
- **Papers** (`RecordPapers.svelte`)
  - Lists the attached files newest first, each with its label (Policy, Invoice, Card, Receipt, Other).
  - A file with a "Valid until" date gets an expiry pill. It is amber within 30 days of the date and red ("Expired on …") once the date has passed.
  - Tapping a row opens the file in a new tab. Images and PDFs open inline.
  - The ⋯ menu (an action sheet on phones) has **Edit**, which changes the label and expiry, and **Detach**, which asks for confirmation. The file stays with its Inbox item.
  - **Add a paper** links to `/add` (Quick add / Inbox). Papers cannot be uploaded directly on the record page.
- **Filing papers.** In Quick add, a person shares a file and chooses **Add to…**, or uses **File into…** on an existing item. The Inbox then calls `records.attachDocument` inside its own transaction (`extensions/inbox/src/server/items.ts` `fileOnRecord`). The item's status becomes `filed` and it is never queued for AI reading. This is the deliberate "keep this document away from the AI" path. The domains supply the choices through the `inbox.fileTarget` slot (for example `extensions/home/src/web/FileTargetHome.svelte`), which needs both the domain's view capability and `records.edit`.
- **Approvals.** Supervised members' changes become drafts that need approval (`approval: { unless: 'records.edit' }`). Guests are refused (`extensions/records/test/obligations.test.ts` "who may change what").

#### Data model (`extensions/records/src/server/store.ts`)

- `records_obligations`
  - `id`, `record_id` and `record_kind`. There is no foreign key across extensions.
  - `record_name` and `record_url`, cached from the owning domain and kept current by `records.renameRecord`.
  - `kind`: a preset id or `custom`.
  - `title` and `due_date` (YYYY-MM-DD, the next due date).
  - `rrule` (`FREQ=MONTHLY|YEARLY;INTERVAL=n` or null) and `rrule_start` (the series anchor).
  - `lead_days` (default 14) and `assignee_id`.
  - `audience`: the capability whose holders are reminded when nobody is assigned. The default is `records.view`; domains pass `vehicles.view`, `pets.view` or `home.view`.
  - `notes`, `lapses` (0/1, added in migration `0002_lapses`), `last_done_on` and `archived_at`.
  - `source_kind` and `source_id`. They are always null today (reserved).
  - `created_by`, `created_at` and `updated_at`.
  - Indexes on (`record_id`, `archived_at`) and (`archived_at`, `due_date`).
- `records_obligation_history`: `id`, `obligation_id` (FK, cascade), `due_date` (the occurrence that was completed), `done_on`, `done_by`, `note`, `created_at`.
- `records_documents`
  - `id`, `record_id`, `record_kind`.
  - `file_key`: the file stays where the Inbox stored it, at `inbox/<item id>/<n>`.
  - `name`, `type`, `size`, `label` (an enum of policy, invoice, card, receipt, other), `expires_on`, `source_item_id`.
  - `created_by`, `created_at`, `updated_at`, `archived_at`.
  - A unique index on (`record_id`, `file_key`).
- Capabilities (`capabilities.ts`): `records.view` (level `view`) and `records.edit` (level `edit`). Under the SDK level table (`packages/sdk/src/capabilities.ts` `LEVEL_DEFAULTS`), `view` is granted to guests by default. Guests can therefore list duties and open papers unless an admin changes the role.

#### Rules and logic worth copying

- **Recurrence is anchored, not chained** (`packages/time/src/index.ts` `nextEveryDue`, used by `extensions/records/src/server/recurrence.ts`).
  - Each occurrence is `anchor + k × interval` (`Temporal.PlainDate.add`). It is never "previous occurrence + interval".
  - So a duty on the 31st goes 31 Jan → 28 Feb → 31 Mar, and 29 Feb goes to 28 Feb in non-leap years and back to 29 Feb in leap years.
  - The rule is stored as an RFC 5545 RRULE for interoperability. The expansion is custom because plain RFC 5545 would skip months that have no 31st.
- **Next due on completion** (`obligations.ts` `completeObligation`).
  - The next due date is `nextDue(anchor, every, max(dueDate, today))`. Done early, the duty moves exactly one period. Done late, missed periods are skipped and the next date is the first occurrence after today.
  - The caller can pass `nextDueDate` (for example "the renewed policy runs until …"). It must be after `doneOn`, and for a repeating duty it becomes the new anchor.
  - A one-off is archived with `last_done_on` set.
  - Every completion writes a history row.
- **Idempotent and stale-safe completion.**
  - The client sends the `dueDate` it saw. If the duty has already rolled forward, or is archived, the call changes nothing (`before: null`).
  - Completing again with the date the duty already has is also a no-op.
  - The completion returns a `before` snapshot (entry id, previous due date, anchor, last done date). `records.undoCompletion` uses it to restore the state exactly and delete the history entry. Running the undo twice is harmless.
- **Editing preserves the series.** `updateObligation` keeps `rrule_start` only when both the rule and the due date are unchanged. Otherwise the new due date becomes the anchor. Fields left out of the input (kind, audience, lead time, assignee, notes, lapses) are kept, and `null` clears assignee or notes.
- **"Lapsing" dates** (`lapses = 1`) are used for things like a warranty ending that ask nothing of anyone.
  - They never repeat (the rrule is forced to null) and are never overdue (status `ended`, not `overdue`).
  - They have no Done button, and they drop out of every "upcoming" list once the day has passed (`openUntil` in `obligations.ts`, `dueStatus` in `web/status.ts`).
  - The reminder text says "Ends in N days" instead of "Due in N days".
- **Status buckets** (`web/status.ts`): `overdue` means due < today and not lapsing. `soon` means due ≤ today + leadDays. `ok` covers everything else, and `ended` is a lapsed date. "Soon" is driven by each duty's own lead time.
- **Reminders** (`reminders.ts`)
  - Two stages per due date: `lead` on `due − lead_days` and `due` on the due date, both at 09:00 in the family time zone. With `lead_days = 0` only the due-day reminder is sent.
  - A stage whose time has already passed is not enqueued (5-minute tolerance).
  - Only the current due date is planned. Every add, edit, completion or undo re-plans inside the same transaction.
  - Each job carries `{obligationId, dueDate, stage, version: updated_at}` and a dedupe key built from them. When the job runs it does nothing if the row is gone, archived, or has a different `updated_at` or `due_date`. A job that ran late (server down) for a day that has already passed is dropped.
  - `renameRecord` deliberately does not bump `updated_at`, so a rename keeps the reminders valid.
- **Notification routing.**
  - If an assignee is set, the reminder goes to that member only.
  - Otherwise it goes to every holder of the obligation's `audience` capability. If that capability is unknown (for example the domain was removed), it falls back to `records.view` and logs a warning.
  - Notification kind `records.obligationDue`. Title "{title}: {record}", the deep link is the record URL, and the dedupe key is `obligation:<id>:<due>:<stage>`. Delivery channels (push, Telegram, email) and per-person mute are handled by core notifications.
- **Overdue handling is visual only.** There is no repeat nagging after the due-day reminder. An overdue duty sits in Today's "Needs you" band with an urgent count until someone completes or edits it.
- **Duplicate guard.** `addObligation.alreadyThere` matches on record id + title + due date, which stops AI or Inbox suggestions from proposing the same duty twice.
- **Archive cascade.** A domain calls `records.archiveForRecord` inside its own archive action (`uow.run`), so the permission check is the domain's. Obligations and papers are archived (kept and hidden, with no reminders) and removed from the search index. "Archived with the record" is told apart from "done" by `last_done_on`.
- **File access control.**
  - `attachDocument` only links files that exist in the file store. Keys must match `^inbox/[A-Za-z0-9_-]+/\d{1,3}$`.
  - It needs `records.edit` outright (`approval: 'never'`, no drafts).
  - Re-filing the same file restores an archived link instead of duplicating it.
  - `getDocumentFile` serves only files linked to a non-archived record document, never an arbitrary key. It is streamed through `FileResponseSchema` (sandboxed, inline only for images and PDFs).
  - The file store is `packages/adapters/src/files-fs` (local filesystem under the data `files/` directory, with a path-escape guard). That directory is included in backups.
  - Detaching deletes only the link.
- **AI privacy by area.**
  - `aiContext` and `upcoming` filter rows with `uow.can(row.audience)` for the AI actor. When a family sets Vehicles, Pets or Home to Off for the AI, that area's duties ("Car insurance (Family car)") disappear from prompts.
  - The search index entry for an obligation carries the same `audience` capability.
  - Filed papers are never read by the AI. Only the file name, label and record name are indexed for family search.
  - The AI may propose only `records.addObligation` and `records.completeObligation`, both as drafts (`obligations.test.ts` "offers adding and completing to the AI, and nothing else"). The prompt guidance tells the model to complete a duty only on proof that it was done (a renewed policy, an invoice), never on a renewal notice.

#### Integrations

- Inbox / Quick add is the only way to bring in papers (`extensions/inbox/src/server/items.ts`).
- The AI uses records for Inbox suggestions and as the assistant's read tool (`records.upcoming`, "what is coming up").
- Family search embeds obligations and paper metadata (`extensions/records/src/server/search.ts`).
- There are no external services.

#### Today / display relevance

- `dashboard.card` `records.overdue`, band `needs` (OverdueCard.svelte): overdue duties across all records, 4 rows plus "and N more", reported as `urgent`.
- `dashboard.card` `records.comingUp`, band `ahead` ("Coming up", ComingUpCard.svelte): duties due within 30 days that are not late (status soon or ok), 4 rows, each linking to the record page. It has no "more" line.
- `display.widget` `records.comingUp` (DisplayComingUp.svelte): read-only "Coming up" list on wall displays. It includes overdue duties, which a display needs because it has no "Needs you" band (`docs/guide/displays.md`). It shows 2 rows plus "+N", is hidden when empty, and needs the display to be granted `records.view`.
- All three read `/api/v1/records/upcoming?days=30` through one shared read and refresh on `records.*` events and on a one-minute clock.

#### Docs vs code

- `docs/guide/records.md` says Today's Coming up lists duties "due in the next 30 days or overdue (five at a time, then 'and N more')". In code, overdue duties sit in a separate "Needs you" card (4 rows plus "and N more"). Coming up shows 4 rows with no "more" line, and the display widget shows 2.
- The docs mention a **History** button on a duty and an **Edit** action. The code has no separate buttons: tapping the row opens the edit panel, and history shows inside it.
- The docs say "Expired papers are marked". The code also flags papers that expire within 30 days, which the docs do not mention. Paper expiry dates create no reminder: they are a display pill only.

#### Unfinished or stubbed

- `source_kind` and `source_id` on obligations are never written (always null).
- Paper expiry (`expires_on`) has no notification and does not appear in Coming up.
- Papers can only be added through the Inbox. There is no direct upload on a record page.
- The search entry for a paper uses the `records.view` capability only. Unlike obligations, it does not carry the domain area's capability, and its link is resolved from any obligation of the same record (`/` if the record has none) (`search.ts`). Locking an area for the AI may therefore not hide paper names from search. This is inferred from the code, not covered by a test.

#### Beyond Home HQ (net-new ideas)

- A generic "duty that comes back" engine for adult household admin (renewals, inspections, vaccinations, filter changes), separate from kids' chores. It has anchored month and year recurrence, per-duty lead times, an assignee-or-audience reminder route, a completion history with notes and undo, and "lapsing" dates for warranties.
- Domain presets ("What usually needs doing") with suggested interval and lead time that the person can change when ticking them at record creation.
- Papers attached to records, with labels and validity dates, filed from a phone share sheet. A "file it privately" path keeps the AI from reading sensitive documents.
- AI suggestions that read a shared invoice or policy and propose "mark Car insurance done, next due 2027-09-01", for a person to approve and undo.

#### Evidence

- `extensions/records/src/contract/index.ts`, `src/server/obligations.ts`, `src/server/reminders.ts`, `src/server/documents.ts`, `src/server/store.ts`, `src/server/search.ts`, `src/server/recurrence.ts`
- `extensions/records/src/web/index.ts`, `RecordObligations.svelte`, `RecordPapers.svelte`, `ObligationForm.svelte`, `status.ts`, `repeat.ts`, `ComingUpCard.svelte`, `OverdueCard.svelte`, `DisplayComingUp.svelte`
- `extensions/records/test/obligations.test.ts`, `reminders.test.ts`, `documents.test.ts`, `recurrence.test.ts`
- `packages/time/src/index.ts` (`everyToRule`, `nextEveryDue`), `packages/sdk/src/capabilities.ts`, `packages/adapters/src/files-fs/index.ts`, `extensions/inbox/src/server/items.ts`
- `docs/guide/records.md`, `docs/guide/displays.md`

### Home (`extensions/home`)

#### What the user can do

The nav entry is Household → Home (`/home`). Each home has its own page at `/home/:id`, each thing at `/home/items/:id`, and each service at `/home/services/:id`.

- **Homes**
  - Add a home with a name, address (multi-line), tenure (owner, tenant, other) and notes.
  - While adding, tick presets and give each a first due date. The interval and lead time can be changed per pick (`PresetPicker.svelte`).
  - Edit, or archive from the ⋯ menu (a dialog confirms). Archiving cascades to the home's things and services and to all their duties and papers.
- **Things** (appliances and similar)
  - Fields: name, room, brand, model, serial, bought on, warranty ends, notes.
  - Things can get item presets when added.
  - A warranty date automatically maintains a "Warranty ends" duty on the thing's page.
- **Services**, labelled "Subscriptions" in the UI
  - Kind: electricity, gas, water, internet, phone, TV, insurance, building dues, other.
  - Fields: provider, account or subscriber number, usual billing day (1–31, informational only), notes.
  - Each service is a record with its own What needs doing and Papers, but no presets are offered (`ServicePage.svelte` passes no `presets`).
- Every page renders the `record.section` slot with audience `home.view`.

#### Data model (`extensions/home/src/server/store.ts`)

- `home_homes`: `id`, `name`, `address`, `tenure`, `notes`, `created_by`, `created_at`, `updated_at`, `archived_at`.
- `home_items`: `id`, `home_id` (FK, cascade), `name`, `room`, `brand`, `model`, `serial`, `purchased_on`, `warranty_until`, `warranty_obligation_id` (the linked records obligation, no FK), `notes`, plus timestamps and `archived_at`.
- `home_services`: `id`, `home_id` (FK), `kind`, `provider`, `account_number`, `due_day`, `notes`, plus timestamps and `archived_at`.
- Capabilities: `home.view` (view level) and `home.edit` (edit level).

#### Rules and logic worth copying

- **Presets** (`extensions/home/src/contract/index.ts`)

  | Record | Preset | Suggested repeat | Suggested lead time |
  |---|---|---|---|
  | Home | Home insurance | 1 year | 30 days |
  | Home | Heating or cooling service | 1 year | 21 days |
  | Home | Smoke and CO alarms check | 1 year | 14 days |
  | Home | Lease or contract review | 1 year | 30 days |
  | Thing | Filter change | 6 months | 14 days |
  | Thing | Service | 1 year | 21 days |

  The person's pick overrides the suggestion (`scheduleOf`).
- **Warranty sync** (`extensions/home/src/server/items.ts` `syncWarranty`)
  - A warranty date creates a lapsing obligation: kind `warranty_end`, 30-day lead, audience `home.view`. Its title is translated into the family language and stored.
  - Changing the date moves the obligation, and clearing the date removes it.
  - If the old duty was finished or removed, a new date creates a fresh one.
  - Re-saving with the same date restores `lapses = true` if someone had turned it into a normal duty.
- **Name sync.** Renaming a home or thing, or changing a service's provider, calls `records.renameRecord`, so Coming up and reminders show the new name.
- **Partial update for AI.** `home.updateItem` changes only the fields it is given. Its undo restores just those fields from a `before` map. This lets an AI suggestion such as "warranty extended" touch one field only.
- **AI exposure is minimised.**
  - `home.aiContext` lists home names and tenure only. `home.aiContextItems` lists thing name, brand and model, room, home and warranty end, with no serial, account number, address or notes (`context.ts`).
  - The AI may propose `home.addItem` (from a receipt or warranty card) and `home.updateItem` (for example a warranty extension) as drafts.
- **Duplicate guards.** Homes match on name. Things match on home + name + brand + model. Services match on home + kind + provider.

#### Integrations

There are none beyond Inbox filing and the AI suggestions. There is no smart-home or device integration. A "thing" is an inventory entry.

#### Today / display relevance

Home contributes nothing of its own to Today or displays. Its duties, including "Warranty ends (Washing machine)", reach Today and displays through the records cards and widget described above.

#### Docs vs code

- The header comment of `extensions/home/src/contract/index.ts` mentions "the people who fix things". The code has no contacts or tradespeople feature: no table, field or UI.
- `docs/guide/home.md` says "Utilities and subscriptions are never sent to the AI". They are indeed missing from `home.aiContext`. However, `extensions/home/src/server/search.ts` indexes services (kind, provider, notes) for family search, and `core.searchFamily` declares AI hints (`packages/core/src/search.ts`). Unless the Home area is off for the AI, service text is embedded and can be found by the assistant.
- The docs describe archiving as "tap again to confirm". The code uses a ⋯ menu item followed by a confirm dialog.

#### Unfinished or stubbed

- There are no contacts or service providers, as noted above.
- The service `dueDay` creates no reminder, and nothing tracks amounts or payments.
- Services have no presets (for example "contract ends").

#### Beyond Home HQ (net-new ideas)

- An appliance inventory by room, with brand, model and serial, and warranty-end tracking that maintains itself.
- A list of utilities and subscriptions per home with account numbers and papers (contracts, bills).
- Seasonal upkeep presets (alarm check, heating service, filter change).
- Receipt-to-inventory AI suggestions.

#### Evidence

- `extensions/home/src/contract/index.ts`, `src/server/items.ts`, `src/server/homes.ts`, `src/server/services.ts`, `src/server/context.ts`, `src/server/search.ts`, `src/server/store.ts`
- `extensions/home/src/web/index.ts`, `HomeContent.svelte`, `ItemPage.svelte`, `ServicePage.svelte`, `FileTargetHome.svelte`, `PresetPicker.svelte`
- `extensions/home/test/home.test.ts`, `docs/guide/home.md`

### Pets (`extensions/pets`)

#### What the user can do

The nav entry is Household → Pets (`/pets`, `/pets/:id`, with a list and detail in one component).

- **Add a pet**
  - Fields: name, species (dog, cat, bird, rabbit, fish, reptile, other), breed, sex, birth date, microchip number, vet (free text: clinic and phone), "Who looks after it" (members), notes.
  - Species-filtered presets can be ticked with first due dates (`presetsFor(species)`).
- **The pet's page**
  - A header with facts and caretakers.
  - What needs doing and Papers from records, with species-filtered presets as "Usually also" chips.
  - **Treatments and vaccinations** (`PetTreatments.svelte`), a history. Add a treatment with kind (vaccination, internal parasite, external parasite, check-up, medication, other), date, product, batch, vet and notes. The optional **Fulfils** picker lists the pet's open duties. The optional **Paper** picker links one of the pet's records documents.
  - Remove a treatment with two taps.
- Edit and archive the pet from the ⋯ menu.
- The code has no feeding schedule, weight log, medication schedule or pet photo ("A pet photo is not kept yet", `extensions/pets/src/contract/index.ts`).

#### Data model (`extensions/pets/src/server/store.ts`)

- `pets_pets`: `id`, `name`, `species`, `breed`, `sex`, `birth_date`, `chip_number`, `vet` (free text), `member_ids` (JSON array), `notes`, plus timestamps and `archived_at`.
- `pets_treatments`: `id`, `pet_id` (FK, cascade), `kind`, `given_on`, `product`, `batch`, `vet`, `notes`, `document_id` (a records doc id, no FK), `created_by`, `created_at`.
- Capabilities: `pets.view` and `pets.edit`. Duties use audience `pets.view`.

#### Rules and logic worth copying

- **Presets with species filters** (`PET_PRESETS`)

  | Preset | Suggested repeat | Suggested lead time | Suggested for |
  |---|---|---|---|
  | Rabies | 1 year | 30 days | dog, cat, rabbit, other |
  | Core vaccine | 1 year | 30 days | dog, cat, rabbit, other |
  | Internal parasites | 3 months | 7 days | all except fish |
  | External parasites | 1 month | 3 days | dog, cat, rabbit, other |
  | Check-up | 1 year | 21 days | all |

- **A treatment completes a duty atomically** (`pets.addTreatment` in `pets.ts`).
  - With an `obligationId`, the treatment calls `records.completeObligation` with `doneOn = givenOn` and an optional `nextDueDate` taken from the vet card, in the same transaction.
  - If the duty belongs to another record, the whole action rolls back.
- AI context lists name, species and breed only, with no chip number, vet or notes. The AI may propose `pets.addTreatment` as a draft from a vet card or receipt. The guidance asks it to include the matching duty handle and the next date when the document gives one.
- Duplicate guard on treatments: pet + date + kind + product.

#### Integrations

There are none beyond Inbox and AI.

#### Today / display relevance

Pets has no cards of its own. Vaccinations and parasite treatments appear through records' Coming up and overdue cards and the display widget.

#### Docs vs code

- `docs/guide/pets.md` says Archive pet is "at the bottom of its page, tap again to confirm". The code puts it in the ⋯ `RecordMenu` with a confirm dialog (`PetPage.svelte`).
- Otherwise the docs match the code, including the stated lack of a photo.

#### Unfinished or stubbed

- There is no pet photo (stated in the contract and the docs).
- Undoing a treatment does not undo the duty completion. `addTreatment`'s undo is `pets.removeTreatment`, which deletes only the treatment row. The duty stays rolled forward and its records history entry remains, even though `records.undoCompletion` exists.
- Pets has no family-search index (`ai.index` is not declared in `extensions/pets/src/server/index.ts`). Vehicles has none either. Home does.

#### Beyond Home HQ (net-new ideas)

- A pet health log (vaccinations with batch numbers and the vet) tied to recurring vaccination and parasite duties. Logging a treatment rolls the duty forward automatically.
- Caretaker assignment per pet.
- Vet cards filed as papers and linked to the treatment.

#### Evidence

- `extensions/pets/src/contract/index.ts`, `src/server/pets.ts`, `src/server/store.ts`, `src/web/PetPage.svelte`, `src/web/PetTreatments.svelte`, `src/web/PetsList.svelte`, `src/web/presenters.ts`
- `extensions/pets/test/pets.test.ts`, `docs/guide/pets.md`

### Vehicles (`extensions/vehicles`)

#### What the user can do

The nav entry is Household → Vehicles (`/vehicles`, `/vehicles/:id`).

- **Add a vehicle** with name, plate, make, model, year, fuel (petrol, diesel, hybrid, electric, LPG, other), VIN, "Who drives it" (members) and notes. Presets can be ticked with first due dates.
- **The vehicle's page**
  - What needs doing and Papers from records.
  - **Service and repairs** (`VehicleLogs.svelte`), a log. Each entry has kind (service, repair, other), date, place (garage), "what was done" notes and an optional linked paper (for example the invoice). Entries can be removed with two taps.
- Edit and archive from the ⋯ menu.
- Mileage is deliberately not modelled: "Dates only, no odometer or units: a mileage … goes in the notes of a log entry" (`extensions/vehicles/src/contract/index.ts`).

#### Data model (`extensions/vehicles/src/server/store.ts`)

- `vehicles_vehicles`: `id`, `name`, `plate`, `make`, `model`, `year`, `fuel`, `vin`, `member_ids` (JSON), `notes`, plus timestamps and `archived_at`.
- `vehicles_logs`: `id`, `vehicle_id` (FK), `kind`, `done_on`, `place`, `notes`, `document_id` (a records doc id), `created_by`, `created_at`.
- Capabilities: `vehicles.view` and `vehicles.edit`. Duties use audience `vehicles.view`.

#### Rules and logic worth copying

- **Presets** (`VEHICLE_PRESETS`)

  | Preset | Suggested repeat | Suggested lead time |
  |---|---|---|
  | Liability insurance | 1 year | 30 days |
  | Comprehensive insurance | 1 year | 30 days |
  | Inspection | 1 year | 30 days |
  | Service | 1 year | 21 days |
  | Tyres | 6 months | 14 days |
  | Vehicle tax or registration | 1 year | 14 days |

  Preset ids are region-neutral and the person sets the actual interval.
- AI context lists name, plate, make, model, year and fuel, never the VIN or notes. The AI can propose `vehicles.addLog` from a service invoice. It can also propose `records.completeObligation` with `nextDueDate`, for example when a renewed policy states its new end date.
- Duplicate guard on log entries: vehicle + date + kind.

#### Integrations

There are none beyond Inbox and AI.

#### Today / display relevance

Vehicles has no cards of its own. Insurance, inspection and tax dates appear through records' Coming up, overdue cards and the display widget.

#### Docs vs code

- `docs/guide/vehicles.md` places Archive vehicle "at the bottom of its page, tap again to confirm". In code it is a ⋯ menu item with a confirm dialog.
- Unlike pets treatments, a vehicle log entry has no "Fulfils" link. `vehicles.addLog` never completes a duty, so logging a service does not roll the "Service" duty forward and the person must also press Done. The docs do not claim otherwise, but the two domains behave differently.

#### Unfinished or stubbed

- There is no mileage or odometer field and no mileage-based service intervals. This is a deliberate choice.
- Log entries are not linked to duties, as noted above.
- Vehicles has no family-search index.

#### Beyond Home HQ (net-new ideas)

- Vehicle renewals (insurance, inspection, road tax) with reminders weeks ahead.
- A service and repair history with linked invoices.
- "Who drives it" for routing reminders through assignees.

#### Evidence

- `extensions/vehicles/src/contract/index.ts`, `src/server/vehicles.ts`, `src/server/store.ts`, `src/web/VehiclePage.svelte`, `src/web/VehicleLogs.svelte`, `src/web/VehiclesList.svelte`
- `extensions/vehicles/test/vehicles.test.ts`, `docs/guide/vehicles.md`

---

## 8. Family board

`extensions/board` is a fridge-door board of short notes for the whole family. Each note has text, one of five paper colours, an optional photo, an optional end day and an optional pin. Notes have no replies or threads. The only reaction is "seen", which is recorded automatically (`extensions/board/src/contract/index.ts`).

### What the user can do

- **Open the board** at `/board` (`BoardPage.svelte`).
  - The page looks like a cork wall with a decorative garland (`Garland.svelte`: paper flags by day, small lights by night, CSS-only, and no animation for people who ask for reduced motion).
  - Notes are papers laid out in columns.
  - The nav entry has `placement: 'primary'`, group `household`. It sits in the rail on wide screens and moves into More on phones (`web/index.ts`).
- **Write a note.** The first tile, "Write a note…", turns into an inline composer (`Composer.svelte`) with these fields:
  - Text, up to 500 characters. Line breaks are kept.
  - Paper: five swatches, `note-1` … `note-5`, coloured by the theme.
  - **Add a photo**: one per note.
  - **Stays on the board**: Until I take it down / Until today / Until tomorrow / Until the end of the week / Until a day I pick. "End of the week" respects the family's week start and is offered only when it falls later than tomorrow.
  - **Pin it to the top**: shown only to managers.
  - **Put it up** publishes the note.
- **Read notes.**
  - Each note shows the author's avatar, how long ago it was posted ("Just now", "5 min ago", "3 h ago", "Yesterday", or a date) and an "Until Fri" / "Until tomorrow" chip.
  - Long notes clamp to 3 lines with More/Less. Notes of 40 characters or fewer without a photo are written large.
  - Photo notes render polaroid-style, and tapping the photo opens it large.
  - A footer line says who has seen the note: "Seen by A, B and C" for up to three names, otherwise "Seen by N". The author is excluded.
- **Note menu (⋯)**
  - **Edit**: text, paper, end day, or a replacement photo.
  - **Move earlier** / **Move later**.
  - **Pin** / **Unpin**: managers only.
  - **Take down**: tap twice.
- **Arrange by drag.**
  - With a mouse, press and move. On touch, hold for 350 ms, then move. A finger that moves more than 8 px before the hold completes scrolls the page instead.
  - The page auto-scrolls within 64 px of the edges.
  - A note moves only within its own group, pinned or unpinned (`order.ts` `placeAt`, `gapAt`).
- **Who can do what** (`server/capabilities.ts` with `packages/sdk/src/capabilities.ts` `LEVEL_DEFAULTS`)
  - `board.view` (level view): everyone, including guests and family displays that were granted it.
  - `board.post` (level use): admin, member and supervised member. Supervised members post directly with no approval; only the AI's posts become drafts (`approval: 'ai'`). Guests cannot post.
  - `board.manage` (level manage): admin only by default. Managers can pin, and can edit, move or take down anyone's note.
  - The author may edit, move, re-photo or take down their own note, but only while they still hold `board.post` (`assertMayChange` in `server/notes.ts`).
  - Displays can never mark a note seen or change anything.

### Data model (`extensions/board/src/server/store.ts`)

- `board_notes`
  - `id`, `text`.
  - `photo`: a file-store key, `board/<noteId>/<photoId>`.
  - `photo_type`, `paper`, `pinned` (0/1).
  - `ends_at`: epoch ms, the end of the last day in family time, or null to keep the note.
  - `created_by`, `created_at`, `updated_at`.
  - `position`: REAL, fractional order within the group. It is null for notes nobody has arranged.
  - Indexes on (`pinned`, `created_at`) and on `ends_at`.
- `board_note_seen`: (`note_id`, `member_id`) primary key, `seen_at`. It cascades when a note is deleted.
- The API shape (`BoardNoteSchema`) adds `endsOn` (the last day, computed in the family time zone), `hasPhoto` and `seenBy[]`.

### Rules and logic worth copying

- **Ordering** (`onBoard` in `server/notes.ts`)
  - Pinned notes come first.
  - Within each group, arranged notes come first in `position` order, followed by never-arranged notes, newest first. Ties are broken by id.
  - A new note, or a newly pinned or unpinned one, joins the top of its group (`topOf` = smallest position − 1, or null while nobody has arranged the group).
- **Moves change one row.** The note gets the midpoint between its neighbours. The first time a group is arranged, or when a gap falls below 1e-6, the whole group is renumbered 1, 2, 3…
  - A move to where the note already is returns `moved: false` and sends no event.
  - Moving beside a note of the other group, or one that has gone, is refused with a plain message.
  - The client applies the same order optimistically (`moveLocal`).
- **Expiry**
  - `endsOn` becomes `dayBounds(date, tz).end`, the end of that day in the family time zone. Past dates are refused.
  - A note is hidden as soon as `ends_at <= now`. Clients reload on `board.*` events, and the display widget also reloads every 5 minutes.
  - A daily job (`board.cleanup`, next run 03:00 UTC, deduplicated per day) hard-deletes notes that ended more than 30 days ago (`KEEP_AFTER_END_DAYS`), together with their photo files and seen marks. It deletes the file before the row, so the job can safely run twice.
- **Seen marks.**
  - Loading the board page while the tab is visible marks each unseen note as seen, once per visit (`BoardPage.svelte` `load`).
  - The server ignores repeats (`onConflict doNothing`) and emits an event only on the first mark.
  - The author is marked as having seen their own note when posting.
- **Notifications.**
  - Posting notifies every holder of `board.view` except the author. Kind `board.posted`, title "{name} put up a note", the body is the first 120 characters of the text on one line (`excerpt`), the link is `/board` and the dedupe key is per note.
  - Edits, pins and seen marks notify nobody.
  - Each person can mute the kind.
- **Photo handling**
  - The client shrinks the image to a longest side of 1400 px and re-encodes it as JPEG at quality 0.85 on a white background, which strips EXIF data including location, because notes show on wall displays (`web/resize.ts`).
  - The server caps the upload at 6 MB, sniffs magic bytes (JPEG, PNG or WebP, ignoring the client's claimed type) and stores the file under a key scoped to the note.
  - `board.setNotePhoto` accepts only keys under `board/<thatNoteId>/`. If the action fails, the stored file is deleted. A replaced photo's old file is deleted. Taking a note down deletes the photo through a queued job, so the file goes only if the row really went.
  - Photos are served only through `GET /notes/{id}/photo`, which requires `board.view`.
- **Deterministic look.** Paper tilt (−2° to 2°) and the fastener (tape for one note in three, otherwise a push pin in ink, info or sun colour; pinned notes always get an accent push pin) are hashed from the note id with FNV-1a. A note looks the same on every screen (`web/paper.ts`). Under `prefers-contrast: more`, papers are straight and plain.
- **AI.** The AI can propose `board.postNote` as a draft, never pinned, and undo takes the note down. It can read the board through `board.listNotes` (30 notes, board order). A post whose last day has already passed is not offered as a draft (`draftDate`). Duplicate guard: identical text already on the board.

### Integrations

- The AI posts through Quick add or assistant suggestions and reads the board through the assistant tool.
- The file store is `files-fs`.
- There are no external services.

### Today / display relevance

- `dashboard.card` `board.today` (`BoardCard.svelte`): band `notes`, order 5, labelled "The board". It shows small papers in two columns in board order: 4, or 6 when the card is wide, plus "and N more". Each paper links to `/board`.
- `display.widget` `board.notes` (`DisplayBoard.svelte`): order 15. Read-only papers in larger type, at most 2 notes (a code comment says a third was cut off) plus "+N". It shows "Nothing on the board" when empty and never marks notes seen. It needs the display to be granted `board.view`. `docs/guide/displays.md` places it in the bottom band of the display.

### Docs vs code

- `docs/guide/board.md` says "Pinned notes sit first, held by a strip of tape". In code a pinned note always gets an accent push pin, and tape is used on a random third of unpinned notes (`web/paper.ts` `fastenerOf`, test `paper.test.ts`). A comment in `BoardCard.svelte` also refers to a pinned note's tape, so the code itself is inconsistent here.
- The docs say "A counter appears when you are close to the limit". `Composer.svelte` has only `maxlength={500}` and no counter.
- The docs say Edit can change "the photo". In code a photo can be replaced but not removed, because no action clears `photo`. The composer's remove button only discards a photo that has been chosen but not yet uploaded.
- The docs say the board is "under Household in the side bar on a computer". In code the nav entry is `placement: 'primary'` (rail on wide screens, More on phones), which is roughly consistent.

### Unfinished or stubbed

- There is no way to remove a photo from a posted note.
- There is no character counter.
- `getNotePhoto` does not check `ends_at`. The photo of an ended note can still be fetched by id until the 30-day cleanup. The risk is low, but it is a gap.
- The board has no family-search index (`server/index.ts` declares no `ai.index`).

### Beyond Home HQ (net-new ideas)

- A shared fridge-door board, separate from chat:
  - short notes with optional photo, colour and end day that expire automatically;
  - admin-only pinning;
  - family-arranged order with drag on touch screens;
  - passive "seen by" receipts instead of reactions.
- Notes appear on Today and on the wall display, and posting pings everyone except the author.
- Privacy on wall displays: photos are stripped of EXIF in the browser before upload.
- The AI can suggest a family note from something shared to Quick add, for a person to approve.

### Evidence

- `extensions/board/src/contract/index.ts`, `src/server/notes.ts`, `src/server/capabilities.ts`, `src/server/store.ts`, `src/server/index.ts`
- `extensions/board/src/web/index.ts`, `BoardPage.svelte`, `Composer.svelte`, `NoteCard.svelte`, `BoardCard.svelte`, `DisplayBoard.svelte`, `order.ts`, `paper.ts`, `resize.ts`, `Garland.svelte`
- `extensions/board/test/board.test.ts`, `move.test.ts`, `order.test.ts`, `paper.test.ts`
- `packages/sdk/src/capabilities.ts`, `docs/guide/board.md`, `docs/guide/displays.md`

---

## 9. Quick add (inbox), drafts and approval

"Quick add" is the user-facing name of the `inbox` extension (`extensions/inbox`, page `/add`). Anything a person gives it (typed text, a link, photos, PDFs, text files, voice notes) becomes an inbox item. A background job reads the item with the family's AI model and proposes changes in other areas. Each proposal is stored as a core **draft** (`core_drafts`) that a person approves, edits, rejects or later undoes. The draft machinery lives in `packages/core/src` and is shared by every AI path: Quick add, the assistant (section 10) and MCP agents (section 11). It is also used when a person without a capability proposes a change by hand, for example a supervised member adding an event.

### What the user can do

**Ways in**
- **Quick add page `/add`** (`src/web/QuickAddPage.svelte`, `CaptureBox.svelte`). One box, not a chat: type or paste text; attach up to 6 files (`image/*`, `audio/*`, `application/pdf`, `text/plain`, 25 MB total, `UPLOAD_LIMITS` in `src/contract/index.ts`); drag and drop or paste photos; record a voice note with the browser `MediaRecorder` (it becomes an audio attachment); press **Read it**. The box clears after each send.
- **Side panel from any page**: slot `nav.action` (`QuickAddPanel.svelte`), the "+" in the sidebar or tablet rail. After **Read it** the panel follows the capture: Reading…, then its suggestions, then Add selected.
- **Today capture box**: slot `today.capture` (`TodayCapture.svelte`), shown to new families.
- **Phone share sheet**: a PWA Web Share Target (`apps/web/static/manifest.webmanifest`, POST `/share-target`, multipart: title, text, url, files `image/*`, `application/pdf`, `text/*`). The service worker (`apps/web/src/service-worker/index.ts`) stores the share in Cache Storage (`famos-share-target`) and redirects to `/share`. `SharePage.svelte` shows a preview and sends only when the person taps **Read it** (or **Discard**). The comment gives the reason: a share-target POST can come from any site. HTTPS is required.
- **Telegram** and **email** (see Integrations).
- **"Add to…" / "File into…"**: files the papers on a vehicle, pet or home record through `records.attachDocument`. The AI never reads them; the item status is `filed` (`FileTargetPicker.svelte`, `inbox.fileItem` in `src/server/items.ts`).

**Watching a capture** (`CaptureStatus.svelte`). The capture shows its state in place: Reading…; "Trying again (attempt n of 4)"; "could not be read" with a reason and **Try again**; "AI is off. Kept here unread." with **Read now**; "Nothing to add was found." with **Remove**; filed. If the model classifies the item as a question for the organizer (`kind: 'question'`), the capture links to the assistant with the text prefilled (`askAssistantHref`, `src/web/item-view.ts`). Skipped proposals are counted ("1 suggestion was left out: it pointed at something that is not on your lists").

**Deciding suggestions** (`CaptureGroup.svelte`, `DraftView.svelte`)
- Under "Waiting for you", each capture is one group: a one-line label (the model's summary), **Show the original**, and one card per suggestion.
- The extension that owns the action draws each card through a registered presenter (`DraftPresenter` in `packages/ui/src/slots.ts`): kind label, title, field rows, approve label, optional inline editor. Presenters exist for 26 actions: calendar create/change/delete event; lists add task, change/complete/delete task, add shopping items, tick/delete shopping item; chores create, set assignees, mark done; kitchen add to meal plan, remove entry, create recipe; home add home/item, update item; pets add pet/treatment; vehicles add vehicle/log; records add/complete obligation; core.addMemory. Inline editors exist for `calendar.createEvent`, `lists.addTask` and `lists.addShoppingItems`.
- **Tick and add**: every suggestion starts ticked. **Add selected (n)** approves the ticked ones one by one through the normal approve endpoint, with edited inputs where the card was edited (`src/web/edits.ts`). One toast offers **Undo**, which takes all of them back. **Dismiss the rest** (two taps) rejects the unticked ones.
- Fields the AI was unsure of are marked **Check this**. A suggestion that may already exist says so and needs a second tap ("Tap again to add it anyway"). A suggestion made for someone else says who asked ("Asked by Noah").
- Decided rows stay with **Undo** until the whole group moves to Recent. Closed states: **Taken back** (undone), **Approved automatically**, **It no longer exists** (target gone), **Already done** (nothing to do, no undo).
- A person who may not approve (a supervised member) sees "Waiting for approval" instead of ticks, and can **Withdraw** their own suggestion.
- **Requests**: drafts that did not come from a capture (from the assistant, MCP agents, or hand-made proposals that need approval, which tell the person "Sent for approval").
- **Recent** (`RecentList.svelte`, `feed.svelte.ts`, last 20 captures): an outcome chip per capture (1 added, Nothing to add, Filed, Rejected, Could not read). Opening one shows the detail panel (`CaptureDetail.svelte`): the original, decided suggestions with Undo, File into…, Try again, Remove.
- **Remove** a capture (two taps): it is dismissed and its waiting suggestions are rejected (`inbox.dismissItem` → `uow.rejectDraftsOf`). Only the person who made a capture can remove it, retry it or file it.

**Elsewhere**
- **Today "Needs you" card** (slot `dashboard.card`, band `needs`, `InboxCard.svelte`, `NeedsDraftRow.svelte`): each waiting suggestion is one row with Reject (two taps), Edit and Approve, plus an Undo toast. A supervised member sees one quiet line instead.
- **Nav badge**: the count from `GET /api/v1/core/drafts/waiting`. It counts only drafts this member may decide, and leaves out expired ones.
- **"Other ways in"** tiles (`WaysIn.svelte`, `ways.ts`): share sheet, Telegram and email, each saying ready, link your account, Set up (for admins), or "Not set up yet".
- **Notifications**: "Suggestions waiting" goes to approvers. "Something you shared has been read" goes to the sharer, only for items from outside the app (`telegram`, `email`, `share`; `OUTSIDE_CHANNELS` in `src/server/process.ts`).

### Data model

`inbox_items` (`extensions/inbox/src/server/store.ts`, migrations 0001–0003)
- `id`, `channel` (`share|upload|text|telegram|email`), `title`, `text` (≤20,000 chars), `url`
- `files`: JSON `[{key,name,type,size}]`, stored under `inbox/<item id>/<n>` in the file store
- `status`: `received → normalized → extracted`, or `failed | dismissed | held | filed`
- `filed_to`: JSON `{recordId, recordKind, label, recordName}`; labels are `policy|invoice|card|receipt|other`
- `normalized_text`: the text prepared for the model (≤30,000 chars). `transcript`: speech to text of voice notes.
- `summary`: the model's short label (≤80 chars, tidied by `tidySummary`)
- `kind`: `actionable|nothing|question`
- `error_code`: one of `not_configured, limit_reached, auth, rejected, data_policy, ai_local_required, timeout, unavailable, invalid_output, output_too_long, unsupported, files_off, voice_not_read`
- `read_attempt`: the next attempt while a retry waits
- `proposals`, `skipped`, `skipped_unknown`: counts
- `model`: "provider / model"
- `extraction`: bumped on each manual retry, so a new reading gets new draft dedupe keys
- `created_by`, `created_at`, `updated_at`

`core_drafts` (`packages/core/src/migrations/0001_init.ts`)
- `id`, `action_id`, `input` (JSON, stored with `label` = the real title of the target), `edited_input`
- `status`: `pending|approved|rejected|applied|failed|undone|expired`
- `proposed_by`: the actor as JSON (`{kind:'ai', source:'inbox'|'assistant'|'mcp', onBehalfOf?}` or a member)
- `source_kind` / `source_id`: `inbox`/item id, `assistant`/turn message id, `mcp`/token id
- `summary`, `model`, `prompt_version` (e.g. `inbox.extract/18`), `confidence` (0–1), `uncertain_fields` (JSON list of input paths)
- `decided_by`, `decided_at`, `applied_at`, `result` (the action output as JSON, used for undo)
- `error`: free text, or `gone` / `settled`
- `dedupe_key`: unique where not null
- `auto_approved`: 0/1
- `for_member`: set when the draft is addressed to one member alone
- Indexes: `(status, created_at)`, `(source_kind, source_id)`

Other core tables
- `core_audit_log`: `at`, `actor` (JSON), `action_id`, `draft_id`, `outcome` (`applied|drafted|rejected|undone`)
- `core_idempotency`: PK `(member_id, key)`, `action_id`, `status`, `response`, `created_at`; pruned after 7 days (`packages/core/src/idempotency.ts`)
- `core_notifications`: `kind`, `title_key`/`body_key` plus params, `url`, `source_kind`/`source_id`, `dedupe_key`, `read_at`
- `core_settings`:
  - key `core.autoApproval`: JSON map `actionId → {proposerCapability, enabledBy, enabledAt}`
  - key `ai_areas`: per-area AI access

Telegram tables (`packages/adapters/src/telegram/tables.ts`)
- `telegram_settings`: single row; sealed `bot_token`, `bot_id`, `bot_username`, `update_offset`, `status_code` (`ok|conflict|invalid_token|offline`), `status_at`
- `telegram_links`: `member_id` PK, `telegram_user_id` unique, `chat_id`, `username`, `linked_at`, `thread_id` (the member's assistant thread), `waiting_reply_id`
- `telegram_link_codes`: `code` PK, `member_id`, `expires_at`

Email tables (`packages/adapters/src/email/tables.ts`)
- `email_settings`: single row; `host`, `port`, `tls_required` (CHECK = 1), `username`, sealed `password`, `mailbox` (default `INBOX`), `processed_folder` (default `FamilyOS`), `poll_minutes` (≥2, default 5), `enabled`, `status_code`/`status_text`, `last_checked_at`
- `email_senders`: `address` PK (lowercase), `member_id`, `label`
- `email_seen`: PK `(uidvalidity, uid)`, `message_id` (indexed), `outcome` (`processed|unknown_sender|refused`)

### Rules and logic worth copying

**Upload and storage** (`src/server/items.ts`)
- Per-member quota: 200 items and 300 MB per 24 h (`UPLOAD_QUOTA`), so a lost phone or a script cannot fill the disk.
- JPEG, PNG and WebP lose EXIF, XMP, IPTC and comments before they are stored. This is pure segment parsing with no re-encode; the orientation is kept in a minimal EXIF segment, and a malformed image is refused rather than passed through (`src/server/metadata.ts`).
- Files cannot join a transaction, so they are stored first, then the action runs; on failure the files are deleted again. An item may only reference file keys under its own id.
- Items are personal. Only the creator may dismiss, retry or file one. Another member sees an item only while it has a suggestion they may decide (`core.decidableSources`). Someone else's item answers `not_found`, never `forbidden`.

**The reading job** (`inbox.process`, `src/server/process.ts`)
- Timeout 150 s, 4 attempts, backoff 30 s doubling to at most 10 min.
- Step 1, normalize: title, text and URL, plus text files up to 200 KB and voice-note transcripts, joined and capped at 30,000 chars. The result is saved as `normalized`, so a retry skips this step.
  - Transcription uses the `transcribe` role with the family's bare language code and a 120 s timeout.
  - If a voice note cannot be read for a reason a person must fix (`not_configured`, `disabled`, `rejected`), it is skipped and the rest of the item is still read.
- Step 2, extract:
  - AI off: the item is `held`, which is not a failure.
  - Files go to the model only if "Send photos and documents" is on: JPEG, PNG, WebP, GIF or PDF within a 20 MB budget. With files the `vision` role is used, otherwise `smart`.
  - The catalog holds only actions with `ai` hints, in areas that are on. Local-only areas are included only if the chosen model's provider is local.
  - Prompt contents (`systemPrompt`):
    - today, the weekday and a 14-day date table, so the model never counts days
    - a note on the region's numeric date order and 12/24 h clock
    - the family language
    - members as short handles (`member:1`) with their `about` text, only if names may be sent
    - who shared the item
    - the family's tags, to reuse exactly
    - household records from each area's `ai.context` read action, invoked as the AI actor, at most 50 per area, as handles like `vehicle:1`
    - explicit rules for dates, times, quantities, notes, titles and the summary
  - The shared content sits in `<shared_item>` and is declared to be data.
  - Call settings: `generateObject` with a JSON Schema whose `proposals.items` is an `anyOf` of each action's input schema (constraint keywords such as pattern, min/max and default are stripped so providers can compile it), temperature 0, max 6,000 output tokens, 90 s timeout.
- **Code review, one retry**: `reviewProblems` checks every proposal against its action's validator. It also flags a date marked uncertain but left out, a date without `dateText`, and clock times stated in the text that no proposal uses. If anything is flagged, the model is asked once more with a `<review>` block. If that retry fails, the first answer stands.
- **Post-processing per proposal**:
  - map handles back to ids (an invented handle cannot resolve)
  - fill `recordKind`, `recordName` and `recordUrl` from the context
  - turn a dated event without a time into an all-day event and mark `startTime` and `allDay` uncertain
  - drop proposals that point at unlisted records (`skippedUnknown`)
  - drop change proposals whose target is not the one the text names (`namesOtherTarget`, a word-share score; skipped when files are attached, because the text cannot be checked)
  - add to the uncertain fields any date whose quoted `dateText` disagrees with it (`isUncertainDate`, using the date words of every installed language pack) and any member field whose name never appears in the text (`unnamedMemberFields`)
- Each surviving proposal is invoked as `{kind:'ai', source:'inbox'}` with draft metadata, including `forMember` = the sharer and `dedupeKey` `inbox:<item>:<extraction>:<index>`. An input the action rejects (`bad_request` or `forbidden`) is skipped and counted. Only field paths are logged, never values.
- Errors: an LLM error that cannot be retried (no model, bad key, limit reached, local model required…) fails the item at once with that code. Outages are retried. `disabled` means held. When the last attempt fails, the item is `unavailable`. A manual retry bumps `extraction`.

**Action runner and approval** (`packages/core/src/action-runner.ts`)
- **When a call becomes a draft** (`needsApproval`): every non-read-only call by the AI is a draft. Otherwise the action's policy decides: `never`, `always`, `ai`, `{unless: capability, except?}` or a function. Members without the capability may still propose when they hold `core.drafts.propose`.
- **Authorization** (`authorize`, `ai-tools.ts`, `access.ts`):
  - A display may not run any action.
  - The AI holds only `view`-level capabilities, and only in areas that are not off.
  - Acting for a member, the AI never exceeds that member: a read needs the member's capability; a draft needs the member to hold `requires` plus either `approval.unless` or `core.drafts.propose`.
  - An area set to Off refuses the AI whatever the action declares.
- **Invariants checked at registry build** (`packages/core/src/registry.ts`):
  - a data-changing action with `ai` hints cannot have `approval: 'never'`
  - an action marked `external` (sends data outside, such as calendar feed links) cannot have `ai` hints
  - auto-approval is refused on read-only, external, core or sensitive actions
- **Target check** for AI change proposals (`checkTarget`): the model must send `label`, the name in the person's own words, and every significant word of it must appear in the real title (`labelNamesTitle`). Otherwise it is refused with "That is 'X', not 'Y'. Tell the person this and do not suggest anything else for it." `refuseProposal` stops pointless proposals (already in that state). `resolveNames` maps list names to ids and lists the valid names on error. Every draft stores the real target title, so cards and notices always name what will change.
- **Addressing** (`invokeOnce`, `deciderBehind`):
  - Personal drafts (actions with `ai.addressToMember`, today only `core.addMemory`) are addressed to that member.
  - An AI draft made for a person who could approve it themselves (holds `core.drafts.approve` plus the action's capabilities, not archived) is addressed to that person. Nobody else sees it: other members, admins included, get `not_found`.
  - Otherwise, for example a supervised member's share, the draft goes to all approvers.
  - In practice, an adult's own captures and assistant requests are theirs alone to decide.
- **Who may decide** (`assertApprover`): the addressee of a draft addressed to one member, who needs no approve capability; otherwise a member with `core.drafts.approve` plus the action's `requires` and its `approval.unless` capability. The member behind a draft may always reject (withdraw) it. Built-in defaults: `core.drafts.approve` is `edit` level (admin, member); `core.drafts.propose` is `use` level (admin, member, supervised); guests have neither (`packages/sdk/src/capabilities.ts`, `core-capabilities.ts`). Custom roles override per capability.
- **Approve**:
  - Approving an already applied draft is a no-op.
  - An expiry check runs first; an edit can move the date forward.
  - A conditional UPDATE `pending|failed → approved` claims the draft, so two people cannot both apply it.
  - Then `settled()` runs: if the request is already true (item already ticked), the draft closes as rejected with `error='settled'` and no undo, so an undo can never reverse somebody else's real change.
  - The action then runs in the same transaction. A `not_found` from the action closes the draft as `gone`. Other domain errors leave the draft untouched. An unexpected error marks it `failed`, which can be retried.
- **Undo**: each AI-proposable action declares `undo(output) → {actionId, input}`, its inverse. Undo runs the inverse and marks the draft `undone` in one transaction, and is idempotent. There is no server-side time window. Undo becomes unavailable only if the stored output no longer parses with today's schema.
- **Duplicate hint**: `alreadyThere()` returns true or `'some'`. It is a hint, never a block. The UI asks for a second tap and the AI is told "It may already be there; tell the person".
- **Expiry** (`draft-expiry.ts`): actions declare `draftDate`. These are `calendar.createEvent` (non-recurring), `calendar.changeEvent`, `board.postNote` (endsOn) and `kitchen.addToMealPlan`. A waiting draft about a past day is shown and counted as expired at once (`passedCheck`). A daily job at 00:05 family time marks such drafts `expired`, deduplicated per day. Tasks and recurring events never expire.
- **Notices** (`auto-approval.ts` `waitingNotice`, `draft-notices.ts`): "waiting" notices go to holders of `core.drafts.approve`, except the proposer, after a 30 s delay. All drafts from one source collapse into one notice (dedupe key `drafts:<sourceKind>:<sourceId>`). The notice body names the thing ("Remove a shopping item: Cheddar"). When no draft from that source waits any more, the notices are marked read. No notice goes out for addressed drafts or auto-approved ones.
- **Audit**: one `core_audit_log` row for every drafted, applied, rejected and undone outcome.
- **Idempotency**:
  - Request level: members may send an `Idempotency-Key` header (`apps/server/src/http/app.ts`). The result is stored in the action's own transaction. Refusals (`bad_request`, `forbidden`, `not_found`) are stored too, so a replay gets the same answer; other errors stay retryable. Reusing a key for a different action is an error. Keys are kept 7 days because phones stay offline for days. The offline queue (actions marked `offline: true`: ticking shopping items, completing tasks, chores) replays with the same key.
  - Draft level: a unique `dedupe_key`, so a job that runs twice proposes once.

**Auto-approval** (`packages/core/src/auto-approval.ts`, `docs/guide/auto-approval.md`)
- Opt-in per action through `autoApprove` hints. Only two actions opt in:
  - `lists.addShoppingItems`: the proposer must hold shopping manage or shopping edit
  - `lists.addTask`: the proposer must hold tasks edit (`extensions/lists/src/server/shopping.ts`, `tasks.ts`)
- Admin-only: `core.autoApprove.manage` is a critical capability that custom roles can never get.
- It qualifies only AI drafts proposed for a member who holds the chosen capability. Never a person's own draft, never an addressed draft, never system or display drafts.
- When such a draft is created, a job (`core.autoApprove`, 3 attempts) re-checks everything and approves as the admin who turned the setting on. It can do nothing that admin could not do.
- It stops itself, with an "Auto-approval stopped" notice, when that admin is archived (`gone`), loses the manage capability (`lost_access`) or is refused (`cannot`). Suggestions then wait for a person again.
- The draft gets `auto_approved=1` and can be undone like any other.

**Prompt-injection posture**
- Shared content and tool results are framed as data.
- The AI can only create drafts, and an action that sends data outside can never be offered to it.
- Email HTML is converted to text on the server, and hidden text and invisible Unicode are removed (`packages/adapters/src/email/text.ts`).
- Auto-approval is off by default, and the docs say why.

### Integrations

**Telegram** (`packages/adapters/src/telegram/service.ts`, `apps/server/src/telegram.ts`, `docs/guide/telegram.md`)
- **Transport**: long polling, no webhook, so no public address is needed. A loop on grammY's `bot.api.getUpdates` runs in the worker process with a 25 s poll timeout.
  - It calls `deleteWebhook` first.
  - It re-reads settings every round, so a token saved by the API process is picked up.
  - It persists `update_offset` after each update, so restarts never replay.
  - It backs off on network errors and on 409 Conflict (another instance polling the same token), which is shown in settings.
- **Bot setup (admin)**: paste the BotFather token in Settings → System → Telegram. It is validated with `getMe` and sealed with `FAMOS_SECRET`. The status codes are `ok`, `conflict`, `invalid_token`, `offline`, plus an unreadable-secret state. A token for a different bot removes all links; **Disconnect bot** removes the bot and every link.
- **Linking (member)**: Settings → You → Notifications → Link Telegram creates a one-time 8-character code from a 31-symbol alphabet with no lookalikes. It is valid 10 minutes and comes with a `t.me/<bot>?start=<code>` deep link.
  - `/start <code>` claims it. Claiming deletes the code row, so whoever deletes it owns it.
  - A Telegram account gets 10 wrong guesses per 10 minutes; after that, codes from it are ignored.
  - One account per member and one member per account.
  - `/stop` unlinks. Archiving the member unlinks too, and the archived state is checked on every message.
  - Only private chats with linked users are handled. Strangers only get a hint after a bare `/start`.
- **Message routing**:
  - Typed text (not forwarded, not a bare URL) goes to `assistant.sendMessage` as the member, in one remembered thread (`thread_id`). The bot shows "typing" and sends the answer back when `assistant.done` fires, if the chat is still waiting for exactly that reply. One question at a time; otherwise "still answering". If the member lacks `assistant.use` or the assistant is off, the text goes to Quick add instead.
  - Forwarded messages, bare links, photos (largest size), documents (PDF, text, images, audio; the type is guessed from the extension if missing; ≤20 MB, the Bot API download cap), voice notes (Ogg Opus) and `/share <text>` go to Quick add. Language packs can add localized command names (`/paylas` in Turkish).
  - Video notes are refused. The bot replies "Sent to Quick add." on success.
  - Shares go through `shareToInbox` (`apps/server/src/inbox-share.ts`), which calls the Inbox upload route handler as the member. Capabilities, quotas, feature switches and EXIF stripping therefore apply exactly as in the app.
- **Approve buttons**: Telegram is a notification channel (`isAvailableFor`, `send`). For a notice whose source has pending drafts, up to 5 button pairs are added (`✓ <title>` / `✗ Reject`, callback data ≤64 bytes). Only drafts this recipient may decide get buttons, and expired ones get none. Pressing a button calls `approveDraft` or `rejectDraft` as the member and removes that pair from the keyboard; "already decided" is reported. An **Open** URL button is added only for a public https base URL, otherwise the address is written in the text. A 403 or "chat not found" response unlinks the member. Quiet hours and per-kind notification preferences apply.

**Email** (`packages/adapters/src/email/*`, `apps/server/src/email.ts`, `docs/guide/email.md`)
- **One dedicated IMAP mailbox**, polled. There is no inbound SMTP address and no webhook.
  - Library: `imapflow`, parsing with `postal-mime`.
  - TLS is mandatory: port 993 is implicit TLS; port 143 must upgrade with STARTTLS or is refused. Certificates are always verified (Proton Bridge works via `NODE_EXTRA_CA_CERTS`).
  - Gmail and Outlook.com need OAuth for IMAP, which is not supported; the docs say to forward instead.
- **Admin setup**: Settings → System → Email: host, port, user, app password (sealed), mailbox, processed folder. **Save and test** logs in, opens the mailbox and creates the folder; reading stays off until a test passes.
- **Schedule**: an in-process scheduler checks every 30 s whether a poll is due (default every 5 min, minimum 2). The poll is a job (`email.poll`, lane `long`, 10 min timeout, 1 attempt) that records failures in settings instead of throwing. **Check now** queues one immediately.
- **Per check**:
  - Only unseen UIDs not already in `email_seen` are looked at, at most 40 per check; a source larger than 40 MB is refused.
  - The From address must be on the allowed-senders list, which maps an address to the member whose Quick add receives it. Otherwise the message is recorded as `unknown_sender` and left in place; adding that sender later clears those rows, so the mail is looked at again.
  - If the topmost `Authentication-Results` header says `spf=fail` or `dkim=fail`, the message is `refused`. Mail without the header is accepted.
  - A Message-ID already seen is a duplicate, so re-delivery or a UIDVALIDITY change never adds a message twice.
  - The subject becomes the title. The body is plain text, or HTML converted to text on the server, with hidden elements and invisible characters removed.
  - Attachments are kept only if sniffed bytes say PDF, image, audio or text (`sniff.ts`). Inline layout images are skipped. At most 6 files and 25 MB.
  - The message is recorded `processed` before it is moved to the processed folder, so a failed move cannot add it twice. A message that cannot be added (quota, Quick add off) stays and is retried next check, without blocking the others.
- The settings page shows counts of unknown-sender and refused mail, the last check and totals. Logs carry counts only, never addresses, subjects or bodies.

**Web push and in-app notifications** carry the same notices (core notifier, not in this scope).

### Display relevance
- **Wall display pill**: slot `display.status` (`src/web/DisplayWaiting.svelte`) shows "N suggestions waiting" next to the clock. It is a count only, and only if the display was granted `core.drafts.waiting` (view level) at pairing. Drafts addressed to one member are never counted for displays (`core.countWaitingDrafts`). It refreshes on `core.draft.*` and `inbox.*` realtime events.
- **Displays can never decide**: the runner refuses every action from a display actor, read-only ones included (`authorize`). Approval needs a signed-in member device.
- On member devices, Today carries the "Needs you" card (decide in place) and, for new families, the capture box. The family-tablet pattern (PIN sign-in for supervised members) means a child can share and see "Waiting for approval", and an adult approves on their own phone, from the Telegram buttons or on Today.

### Docs vs code
- `telegram.md`: "If you may not use the assistant, or no AI model is set up, typed text goes to Quick add instead." In `apps/server/src/telegram.ts` (`onAsk`) the fallback to Quick add happens only when the member lacks `assistant.use`, or on `forbidden`, `feature_off` or `not_found`. With no model configured, `assistant.sendMessage` still succeeds. The turn job then fails (`disabled` or `not_configured`) and the chat gets "The assistant cannot answer right now: the AI is not set up." or "I could not answer that…". The text does not go to Quick add.
- `everyday.md` says a change suggestion is kept only when the message names the item "every word of its name, in any order". In code there are two checks. The runner's `labelNamesTitle` compares the model's `label` with the title. Quick add's `namesOtherTarget` scores word shares against all listed items and is skipped entirely when the capture has files. So for photo or PDF captures only the label check applies.
- The share-target manifest accepts `text/*`, but the Inbox upload accepts only `text/plain`, so other text types shared from a phone are refused at upload. Audio is not in the manifest, so voice notes cannot come in through the share sheet; they come from the in-app recorder, Telegram or email.
- UI copy says Undo "shows for a few seconds" (the toast). The server has no time limit: decided rows and Recent keep Undo for as long as the draft is `applied` and its stored output parses.
- `ai.md`: "FamilyOS keeps no copy of prompts or answers." Raw prompts and responses are indeed not stored. The derived results are: `inbox_items.normalized_text`, `transcript`, the model's `summary`, and each proposal's input in `core_drafts`.
- Code comment in `telegram/service.ts`: "32 symbols"; the alphabet has 31 characters.
- `apps/server/src/prompt.ts` is not an AI prompt. It is a terminal passphrase reader for the backup and restore CLI.

### Unfinished or stubbed
- Comments in `action-runner.ts`, `DraftView.svelte` and `assistant/src/server/tools.ts` mention drafts addressed to a member for "questions to a person". No such action exists; `core.addMemory` is the only `addressToMember` action.
- Only two actions opt in to auto-approval. Calendar events, chores and meals always wait for a person.
- Not supported:
  - Telegram group chats and video notes
  - email "Forward as attachment" (the inner message is not read)
  - multiple mailboxes
  - Gmail and Outlook OAuth IMAP
- Inline editors exist only for event, task and shopping drafts. Other drafts are approve or reject only.
- If "Send photos and documents" is off, a photo-only share fails with `files_off`. There is no local OCR fallback.
- The prompt evaluation set (`extensions/inbox/eval/cases.ts`, 31 cases incl. Turkish, US/UK/EU date formats, prompt injection, tick-off and complete-task cases) runs only by hand against a real model (`FAMOS_EVAL_API_KEY`), not in CI.
- Telegram link codes are stored in plain text (short-lived, single use), unlike app keys, which are hashed.

### Beyond Home HQ (net-new ideas)
- **A universal capture inbox**: one place that takes text, photos, PDFs and voice notes from the app, the phone share sheet, a Telegram bot and a forwarding mailbox, and turns each into reviewable suggestion cards across calendar, tasks, shopping, chores, meals, pets, vehicles and home records.
- **Tick-and-add review**: pre-ticked cards per capture, "Add selected (n)" with a single Undo for the batch, per-field "Check this" from model and code uncertainty, a "may already be there" second tap, and "Already done" and "It no longer exists" auto-closing.
- **Undo for every approval**, implemented as a declared inverse action stored with the result, so even auto-approved changes are reversible.
- **Draft expiry** for date-bound suggestions (yesterday's dinner plan disappears by itself).
- **Supervised-member flow**: kids can propose (share or add by hand), adults approve. A kid's capture is visible to approvers only until it is decided. An adult's own suggestions are private to them.
- **Scoped auto-approval** for routine kinds, approving "as" the admin who enabled it, and switching itself off with a notice when that admin loses rights.
- **Telegram approve and reject buttons** on notifications, and "your share was read" notices for outside channels.
- **Wall-display pill** with the count of suggestions waiting (count only, never content).
- **Privacy details worth copying**: EXIF/GPS stripping on upload, an SPF/DKIM-fail refusal on email, hidden-text stripping before the AI sees mail, filing papers on a record without AI, and per-member upload quotas.
- **A regression eval set** for the extraction prompt, with a prompt version stamped on every draft.

### Evidence
- `extensions/inbox/src/contract/index.ts`, `src/server/{index,items,process,store,metadata,dates,search}.ts`, `src/web/{QuickAddPage,QuickAddPanel,CaptureBox,CaptureGroup,DraftView,NeedsDraftRow,InboxCard,DisplayWaiting,SharePage,WaysIn,CaptureStatus,CaptureDetail,RecentList,FileTargetPicker}.svelte`, `src/web/{index,ways,edits,item-view,feed.svelte,data}.ts`, `extensions/inbox/eval/{cases,run}.ts`
- `packages/core/src/{action-runner,auto-approval,draft-expiry,draft-notices,ai-tools,ai-areas,idempotency,access,core-capabilities,registry,core-extension}.ts`, `packages/core/src/migrations/0001_init.ts`
- `packages/sdk/src/actions.ts` (approval policies, `undo`, `alreadyThere`, `targetTitle`, `settled`, `draftDate`, `autoApprove`, `ai` hints), `packages/sdk/src/capabilities.ts`
- `packages/ui/src/slots.ts` (DraftPresenter), presenters in `extensions/*/src/web/presenter(s).ts`
- `apps/server/src/{inbox-share,telegram,email,prompt}.ts`
- `packages/adapters/src/telegram/{service,tables}.ts`, `packages/adapters/src/email/{service,mailbox,text,sniff,tables,jobs}.ts`
- `apps/web/static/manifest.webmanifest`, `apps/web/src/service-worker/index.ts`
- `extensions/lists/src/server/{shopping,tasks}.ts` (autoApprove hints)
- Docs: `docs/guide/{everyday,auto-approval,telegram,email,notifications,displays,ai}.md`

---

## 10. Assistant and semantic search

The `assistant` extension (`extensions/assistant`) is a per-member chat. The model looks things up with read tools and turns requests into the same drafts that Quick add produces (section 9). Family search (`packages/core/src/search.ts`) is embedding-based semantic search over a few areas. People use it from a search box, and the assistant and MCP agents use it as a tool. Both run under the asking member's own permissions and the family's AI area switches.

### What the user can do

**Assistant** (`/assistant`, `src/web/AssistantPage.svelte`; `docs/guide/assistant.md`)
- **Where it opens**: the "Ask the assistant" card at the foot of Today (slot `dashboard.card`, band `foot`, `AskCard.svelte`), the side bar entry **Assistant** (on phones under More), or Telegram (typed text, section 9). It needs the `assistant.use` capability (`use` level: admin, member and supervised by default; guests no).
- **Conversations**:
  - A thread list in a column on wide screens, under "Conversations" on phones (`ThreadList.svelte`).
  - Actions: New conversation, rename (pencil), delete (two taps), and a "used x ago" label.
  - A new conversation shows a time-of-day greeting and four starter suggestions: This week, Add to shopping, Open tasks, The shopping list (`greeting.ts`).
- **Sending**: a message box (`AskField.svelte`) of at most 4,000 chars. Enter sends on a keyboard; on touch screens the send button sends. The answer streams in over the WebSocket to that member only (`assistant.delta` / `assistant.done`, `stream.ts`). **Stop waiting** only hides the indicator. On failure the answer says why (turned off, no model, limit reached, key refused, local model needed, too slow, no answer, not allowed), with **Ask again**. One question at a time per thread.
- **Answers** are plain text. Only in-app paths such as `/calendar` become links; web addresses stay text and are never opened (`render.ts`).
- **Suggestions under the answer**: drafts the turn created appear as the same cards as in Quick add (`AnswerDrafts.svelte`, `DraftView.svelte`), with Approve, Edit, Reject and later Undo. Someone who may not approve sees "Waiting for approval".
- **Fixed "Suggested" lines**: the app appends one line per suggestion, built from the draft itself and not from the model's words: "Suggested: Add shopping items: Milk, Oat milk", or "Added automatically: …" under auto-approval (`proposalLines` in `src/server/turn.ts`).
- **What it can do**:
  - look things up (calendar, open tasks, shopping lists, chores, meal plan, recipes, board notes, records due, vehicle, pet and home records, its own suggestions' status)
  - find things by meaning (when search is set up)
  - suggest changes, including moving one occurrence of a repeating event
  - remember notes
  - follow up on "it" ("yes, move it to 18:00"), because the handles of records seen earlier in the thread are kept
- **Memory** (`/settings/memory`, `MemoryPage.svelte`; `packages/core/src/memory.ts`):
  - "remember that I'm vegetarian" produces a **Remember this** card. Only that person can see and approve it; admins cannot.
  - Notes can also be added, edited and forgotten (two taps) by hand.
  - At most 280 chars per note and 100 notes. A note says whether the person wrote it or the assistant suggested it.
- **Kill switches**: an admin can turn off the "Assistant chat" AI feature (Settings → AI); the answer is then "The assistant is turned off." and nothing is sent to a model. Settings → Family → Features hides the assistant entirely.

**Search** (`apps/web/src/lib/SearchBox.svelte`; `docs/guide/search.md`)
- A search box at the top of Today on wide screens (Today styles Broadsheet and Wall) and at the top of More on phones. The "/" key focuses it. It is hidden when no embedding model is set, in family (shared-device) mode, and on displays (comment in `lib/today/Wall.svelte`: "a display has none").
- Each result shows a kind, a title and a snippet, and links to the entity's page.
- Settings → AI shows which areas search covers and the last indexing failure (`core.searchStatus`).

### Data model

Assistant tables (`extensions/assistant/src/server/store.ts`)
- `assistant_threads`: `id`, `member_id`, `title`, `summary` (a rolling summary of old messages), `summarized_count`, `handles` (JSON `{ "event:1": {id, label} }`, the handles shown in earlier turns, at most 150 kept), `created_at`, `updated_at`
- `assistant_messages`: `id`, `thread_id`, `role` (`user|assistant`), `content`, `tool_calls` (JSON of tool names, ok flag and draft ids; never inputs or results), `status` (`pending|streaming|done|failed`), `error_code`, `created_at`

Core tables
- `core_member_memory` (`packages/core/src/migrations/0001_init.ts`): `id`, `member_id`, `text`, `source` (`member|assistant`), `created_at`, `updated_at`
- `core_embeddings`: one row per text chunk
  - identity and access: `id`, `extension_id`, `source_kind`, `source_id`, `capability` (the audience capability), `owner_member_id` (set for private items, e.g. one person's Quick add capture)
  - display: `title`, `href`, `snippet`
  - vector: `model`, `dimensions`, `text_hash`, `chunk_index`, `vector` (BLOB of little-endian float32, normalized to unit length), `updated_at`
  - unique `(source_kind, source_id, chunk_index)`
  - The comment says the table "can always be rebuilt from the extensions' data".
- `core_settings` key `search_index`: per-extension model key of the last complete build, plus the last failure that needs a person (code and time only).
- Drafts the assistant creates are in `core_drafts` with `source_kind='assistant'`, `source_id` = the reply message id, `prompt_version='assistant.chat/1'` and `dedupe_key` `assistant:<turn>:<n>`.

### Rules and logic worth copying

**The turn** (`assistant.turn` job, `src/server/turn.ts`; `assistant.sendMessage` in `threads.ts`)
- `sendMessage` stores the question and an empty `pending` reply, then enqueues the job. The model is never called inside an HTTP request. If the thread already has a pending or streaming reply, it answers `conflict` ("still answering").
- **Job settings**: lane `long`, timeout 150 s, 2 attempts with 5 s backoff. It is idempotent: a finished reply is left alone, and a restarted turn resets the streamed text.
- **Before calling the model** it re-checks that the member still exists and holds `assistant.use`. Someone archived or demoted since asking gets `forbidden`.
- **Actor**: the AI runs as `{kind:'ai', source:'assistant', onBehalfOf: member, roleId, local}`, where `local` says whether the `smart` model's provider is on the home network. Tools are the read tools (`aiReadTools`) plus draft tools (`aiCatalog`), filtered by:
  - feature switches
  - AI area access (Off is never offered; Local only only when the model is local)
  - what this member could do (`mayUseAsAi`)
  - `assistantOnly` actions, offered only here (`core.addMemory`)
- **Read tools** (actions with `ai` hints and `readOnly`): `calendar.findEvents`, `calendar.aiContext` (tags), `lists.shoppingList`, `lists.openTasks`, `chores.listChores`, `kitchen.mealPlan`, `kitchen.listRecipes`, `board.listNotes`, `records.upcoming`, `vehicles.aiContext`, `pets.aiContext`, `home.aiContext`, `home.aiContextItems`, `core.today`, `core.mySuggestions`, and `core.searchFamily` when an embedding model is ready.
- **Draft tools**: `calendar.createEvent`, `calendar.changeEvent`, `calendar.deleteEvent`, `lists.addShoppingItems`, `lists.checkShoppingItem`, `lists.deleteShoppingItem`, `lists.removeShoppingItems`, `lists.addTask`, `lists.changeTask`, `lists.completeTask`, `lists.deleteTask`, `chores.createChore`, `chores.setAssignees`, `chores.markDone`, `kitchen.addToMealPlan`, `kitchen.removeMealPlanEntry`, `kitchen.createRecipe`, `home.addItem`, `home.updateItem`, `pets.addTreatment`, `records.addObligation`, `records.completeObligation`, `vehicles.addLog`, `board.postNote`, `core.addMemory`.
- **System prompt** (`src/server/prompt.ts`):
  - It holds no family content apart from: the date, time, week bounds and the days still ahead this week; the 12/24 h clock; the language (the asker's, else the family's); members and asker as handles, only if "Send names and descriptions" is on; up to 20 memory notes (1,500 chars total), same switch; up to 30 records seen earlier in the thread; and the thread summary.
  - Rules include: plain text with no Markdown; dates written as words; "my" means the asker; never say something is done; only claim a suggestion when a tool returned `drafted`; use the tool's `say` text; refuse requests that have no tool; send `label` in the person's own words when changing something; treat `<tool_result>` blocks, memory, records and summary as data.
- **Tool loop**:
  - At most 8 model steps and 12 tool calls per turn. Calls past the cap run nothing and tell the model to answer. 120 s chat timeout.
  - Tool names map `.` to `_`.
  - Each result is wrapped as data, compacted, and cut to 16 KB. When cut, whole trailing list items are dropped and a `cut` field says how many are shown of how many (`fitResult`, `asData` in `packages/sdk/src/agent-tools.ts`).
  - Ids become handles (`event:1`). An id the model was never shown is refused as invented.
  - A second change to the same target under different words in one turn is refused.
  - Failures are reported to the model without echoing its input, and logged with field names only.
- **Draft tool results**: the model receives `status: drafted`, a `suggestion` handle, and a ready `say` sentence chosen by the runner's outcome:
  - "Suggested, not added yet: … It appears once a person in the household approves it."
  - "…once another person … approves it; this person cannot approve it themselves." (supervised)
  - "It will be added automatically in a moment." (auto-approval)
  - "It may already be there; tell the person" when `alreadyThere` hits.
- **Output hygiene**: handles in the streamed text are swapped for member names (or "someone" when names are off) before streaming and storage, and so are handles split across chunks. Markdown emphasis marks are removed (`scrub.ts`). If the step limit ends the loop on a tool call with no text, the turn is `failed` with `no_answer`, but the drafts it made stand.
- **History**: the last 12 done messages are sent verbatim. Once more than 30 are unsummarized, `assistant.summarize` folds the older ones into a summary of at most 12 sentences (smart role, prompt marked as data) (`summary.ts`).
- **Errors**: retryable LLM errors re-queue once; others store the error code. Conversation content is never logged.
- **Privacy**: threads are per member. Another member's thread answers `not_found`, never `forbidden`. Memory notes are read only through `core.recallMemories` as the AI working for that member, inside that member's own threads; MCP clients never get them.

**Search** (`packages/core/src/search.ts`)
- **What is indexed**: each extension declares `ai.index` kinds with a `load(id)` that returns `{title, text, href, ownerMemberId?, capability/audience}`, and a paged `list`.
  - **Quick add items**: summary, normalized text and transcript; owner = the sharer; filed and dismissed items are not indexed.
  - **Kitchen recipes.**
  - **Home**: things and services; serial and account numbers are excluded.
  - **Calendar events**: own events only, not subscribed calendars.
  - **Records**: obligations, and paper names and labels, each obligation following its area's audience capability.
  - Tasks, shopping and chores are never indexed.
- **Indexing**:
  - Jobs: `core.embed` (one entity), `core.embedSync`, `core.embedRebuild`.
  - Text is cut into chunks of about 800 chars with 100 chars of overlap, breaking at a paragraph, sentence or space, at most 40 chunks per entity.
  - A chunk whose text hash (cyrb53) and model are unchanged is not embedded again. Embedding runs in batches of 32 with a 90 s timeout.
  - Reindexing is enqueued in the same transaction as the data change (`uow.ai.reindex`).
  - A model change rebuilds everything; an area switched Off, or Local only with a non-local model, has its vectors deleted.
  - Failures a person must fix (no model, bad key, limit) are recorded once, and old results are kept.
- **Query** (`core.searchFamily` action and `GET /api/v1/core/search?q=`):
  - The question (2–300 chars) is embedded in the action's `prepare` step, outside the transaction, with a 10 s timeout. This is the one sanctioned exception to "no LLM in a request" (AGENTS.md rule 6).
  - Rows are then filtered in SQL first: same model and dimensions, allowed kinds and capabilities, and `owner_member_id` null or the asker.
  - Scoring is a brute-force dot product (cosine) in TypeScript over pages of 2,000 rows. Results with a score ≤ 0 are dropped, top-k is kept, there is one result per entity (its best chunk), and the default is 5 results (maximum 10).
  - The AI gets no `href`. An AI with no member behind it (Quick add) may not search.
- **No sqlite-vec, no FTS5, no keyword fallback**: a grep for `sqlite-vec|vec0|fts5|virtual table` finds nothing.

### Integrations
- **Model calls**: the `smart` role for chat and summaries, and `embed` for search. Both go through the LLM adapter (section 11), so monthly limits, area rules and usage logging apply. Every call is listed in AI activity with the areas it drew on.
- **Telegram**: questions typed in a linked chat go to `assistant.sendMessage` as the member, in one remembered thread that is also visible in the app. The answer comes back to the chat with ✓/✗ buttons for the drafts the turn created (`apps/server/src/telegram.ts`, `sendAnswer`).
- **Realtime**: per-member WebSocket events `assistant.delta` and `assistant.done`.

### Display relevance
- Limited. The assistant needs a signed-in member; wall displays cannot run actions and hold only `view` capabilities, so they cannot chat. Search is explicitly hidden on displays and in family mode.
- On a shared family tablet, the Today "Ask" card is a one-tap entry to the assistant with the cursor already in the box.
- The pieces most relevant to a wall product are the fixed "Suggested: …" lines and the "Waiting for approval" cards. A wall could show that something waits, but deciding happens on a phone.

### Docs vs code
- `assistant.md` mostly matches. Code adds tools the guide does not list: chores, meal plan, recipes, board notes, records, pets, vehicles and home, and `core.mySuggestions`, which the model uses to answer "was it approved?".
- `assistant.md`: "the assistant says it will be added automatically in a moment". The tool's `say` text matches. The fixed line the app appends reads "Added automatically: …" (`proposed_automatic` in `src/messages/en.json`), written before the approval job has actually run.
- `search.md`: "Search also needs Use AI to be on". This matches: `policy.enabled` is checked. A code comment in `search.ts` says the query feature is "switched in Settings → AI", but no `aiFeatures` entry exists for search in any web extension. The only switches are the master **Use AI** and choosing the embedding model.
- `search.md` says vehicle and pet papers are searchable. This is consistent with the code: those papers live in Records (`extensions/records/src/server/search.ts`), and the vehicles and pets extensions declare no index of their own. Their obligations follow the owning area's audience capability.
- The guides do not promise voice for the assistant, and there is none: no microphone in `AskField.svelte`. Voice notes sent in Telegram go to Quick add, not to the assistant.

### Unfinished or stubbed
- No voice input or spoken answers in the assistant.
- No attachments in chat. Photos and documents go through Quick add only.
- Search is pure vector search, with no keyword or FTS fallback and no hybrid ranking. Brute-force scanning is fine at family scale but is O(rows).
- Search has no per-feature switch in the UI (see Docs vs code).
- The assistant cannot send messages, emails or links (by design: actions marked `external` can never carry `ai` hints).
- There is no proactive assistant (no scheduled digests or nudges written by the model).

### Beyond Home HQ (net-new ideas)
- **A chat that can only propose**: every change becomes an approvable card with Undo. The answer ends with app-written "Suggested: …" lines, so the model cannot misreport what it did.
- **Name-matching guard**: a change must name the item in the person's own words, or the assistant says it is not there instead of changing a similar item.
- **Per-person private memory** with a 280-char note limit, approved by the person as a card, editable on a settings page, never visible to admins or external agents.
- **Thread continuity**: handles for records seen earlier in the thread are kept, so "move it" works.
- **The same assistant from Telegram**, with approve buttons in the chat.
- **Semantic family search** across Quick add captures, recipes, home items and services, events and records ("the boiler warranty paper"), with per-person visibility and private captures searchable only by their owner.
- **Area-level AI sandbox**: each area Off, Local only or Any provider. Local only is enforced by resolving the provider's address to the home network.

### Evidence
- `extensions/assistant/src/server/{index,capabilities,store,threads,turn,tools,prompt,scrub,summary,format}.ts`, `extensions/assistant/src/contract/index.ts`, `extensions/assistant/src/messages/en.json`
- `extensions/assistant/src/web/{index,greeting,render,stream,presenters}.ts`, `src/web/{AssistantPage,ThreadList,AskField,AskCard,AnswerDrafts,DraftView,MemoryPage}.svelte`
- `packages/sdk/src/agent-tools.ts` (`runAgentTool`, `asData`, `fitResult`, `say` texts)
- `packages/core/src/{memory,search,kernel,ai-tools,core-extension}.ts` (`core.today`, `core.mySuggestions`, catalog filtering in `kernel.ts` `offered`)
- `packages/core/src/migrations/0001_init.ts` (`core_embeddings`, `core_member_memory`)
- Search kinds: `extensions/{inbox,kitchen,home,calendar,records}/src/server/search.ts`
- `apps/web/src/lib/SearchBox.svelte`, `apps/web/src/lib/today/{Wall,Broadsheet}.svelte`, `apps/web/src/routes/more/+page.svelte`
- `apps/server/src/telegram.ts`
- Docs: `docs/guide/{assistant,search,ai}.md`, `AGENTS.md` (rule 6)

---

## 11. MCP server, app keys and AI providers

FamilyOS serves an MCP endpoint at `/mcp`. Members connect their own agents to it (Claude Desktop through `mcp-remote`, Claude Code, Hermes Agent, any client that can send a bearer header) with personal **app keys**. The agent gets exactly the read and draft tools the built-in assistant would get for that person, minus local-only areas and memory. Model access for all of FamilyOS's own AI goes through one adapter, `packages/adapters/src/llm-aisdk`, built on the Vercel AI SDK's OpenAI-compatible client. The family adds providers, maps them to model roles, sets privacy switches, per-area access and monthly limits, and sees a metadata-only log of every call.

### What the user can do

**Connected apps** (Settings → You → Connected apps, `apps/web/src/routes/settings/connected-apps/{+page,ConnectApp}.svelte`; `docs/guide/mcp.md`)
- **Family switch**: an admin turns on **Allow AI apps to connect** once (`core_settings` key `mcp`, `{enabled}`; needs `core.ai.manage`). While it is off, `/mcp` answers 404 and no key works.
- **Who may create keys**: anyone with `core.mcp.use` (`use` level: admin, member and supervised by default; guests no).
- **"Connect an app" panel, three steps**:
  1. Turn on (admin).
  2. Create the key: a name; **What it may do**, `read` (Read only) or `read_write` (Read and suggest changes); **Valid for** 30, 90 (default), 180 or 365 days.
  3. Copy a ready config block, for **Claude Desktop** (`npx mcp-remote <url> --header Authorization:${AUTH_HEADER}`), **Claude Code** (`claude mcp add --transport http familyos …`) or **Other apps** (JSON `mcpServers` with `url` and `headers`). The key is shown once.
- **Managing keys**:
  - **My app keys** lists name, scope, expiry and last used (updated at most hourly). **Revoke**, then **Revoke now**.
  - Admins see and revoke everyone's keys under **Other people's app keys**.
  - Keys end when their member is archived.
- **What an agent can then do**: ask "what is on the calendar this week", "put milk on the list". Changes arrive in Quick add under Requests, marked as coming from an AI app, for approval. They can be auto-approved if the setting covers them (section 9).
- **For developers** at the foot of the page shows the URL and header form. The guide also covers Hermes Agent YAML and a Cloudflare Tunnel limited to `^/mcp`.

**AI settings** (admins, `core.ai.manage`; `apps/web/src/routes/settings/ai/*`; `docs/guide/ai.md`)
- **Providers** (`ProviderPanel.svelte`):
  - **Add provider**, with kind OpenRouter, Ollama (local) or OpenAI-compatible. Name, Address and API key.
  - **Test connection** calls `GET /models` and shows the model count; for OpenRouter the key is checked separately, because OpenRouter lists models without a key.
  - **Remove key**, **Remove** (roles that used it become unset).
  - A provider set by `FAMOS_AI_*` environment variables shows as read-only ("set by the server configuration").
- **Models**: a provider and model for each role (`smart`, `vision`, `fast`, `transcribe`, `embed`). Off by default, **Choose a different model for each task** lets the three main roles differ; otherwise one choice covers smart, vision and fast.
- **What the AI may read** (`/settings/ai/privacy`, `AreasSection.svelte`):
  - The **Use AI** master switch.
  - Per-feature switches: "Read shared items" (`inbox.extract`), "Turn voice notes into text" (`inbox.transcribe`), "Assistant chat" (`assistant.chat`).
  - Privacy switches: Send photos and documents, Send names and descriptions, Send voice notes, Only providers that keep no data (OpenRouter).
  - **Areas the AI may use**: each area is Off, Local only or Any provider. Opening a sensitive area to Any provider asks for confirmation.
  - Search coverage per area.
- **Usage & limits** (`/settings/ai/usage`):
  - **Monthly limits**: a **Budget (USD)** and/or a number of **Model calls**, by calendar month in the family time zone. Empty means no limit.
  - **This month**: calls, errors, input and output tokens and cost, in total, by feature and by model.
  - **See what the AI did** (`/settings/ai/activity`): newest first, paged with **Show older**. Each entry shows when, the feature, the role, which areas it drew on, provider and model, tokens (incl. reasoning), finish reason, cost, latency, error code, and who it was for. Never any content.
- **Auto-approval** (`/settings/auto-approval`, AI section; section 9).

### Data model

App keys (`packages/core/src/migrations/0001_init.ts`, `packages/core/src/api-tokens.ts`)
- `core_api_tokens`: `id`, `member_id` (FK), `name` (≤60), `token_hash` (SHA-256 hex, unique), `scope` (`read|read_write`), `expires_at`, `last_used_at`, `created_at`, `revoked_at`
- `core_api_token_handles`: PK `(token_id, handle)`, `ref_id`, `created_at`; cascade on token delete. It remembers which ids a token was shown under which stable handles, at most 5,000 per token, oldest dropped.
- `core_settings` key `mcp`: `{enabled: boolean}`
- MCP calls are recorded in `ai_usage` with `role='mcp'`, `feature='mcp'`, `model='MCP / <token name>'`, `provider_id` null, no tokens or cost, `areas` = `[area]`, `error_code` `rejected` on failure.

AI adapter (`packages/adapters/src/llm-aisdk/tables.ts`, migrations 0001–0002)
- `ai_providers`: `id`, `name`, `kind` (`openrouter|ollama|openai-compatible`), `base_url`, `api_key` (sealed with the SecretBox, AES-GCM from `FAMOS_SECRET`), timestamps. The provider managed by environment variables has the fixed id `ai_provider_env`.
- `ai_settings`: key/value rows for
  - `roles`: `{fast, vision, smart, transcribe, embed}`, each `{providerId, model}` or null
  - `limits`: `{monthlyCostLimit USD ≤10,000, monthlyCallLimit ≤1,000,000}`
  - `privacy`: `{sendFiles, sendPeople, sendVoice, zeroRetention}`, all true by default
  - `switches`: `{enabled, off: [feature ids]}`
- `ai_usage`: `id`, `at`, `role`, `provider_id`, `model` ("provider / model"), `feature`, `member_id`, `job_id`, `input_tokens`, `output_tokens`, `reasoning_tokens`, `finish_reason`, `cost_micros` (millionths of a USD), `latency_ms`, `error_code`, `areas` (JSON); index on `at`
- `core_settings` key `ai_areas`: map extension id → `off|local|any` (`packages/core/src/ai-areas.ts`)

### Rules and logic worth copying

**MCP endpoint** (`apps/server/src/mcp/http.ts`)
- Stateless Streamable HTTP via `@modelcontextprotocol/server` and `@modelcontextprotocol/hono`. A new `McpServer` is built per request, with the principal in closures; the code targets spec 2026-07-28 with the SDK's stateless fallback for the older handshake. Request body limit 256 KB.
- **Order of checks**, before the body is parsed:
  1. Feature on, else 404.
  2. Any query parameter that looks like a token (`token|key|auth|secret|password|bearer`, or a value starting `famos_`) → 401, so tokens never land in logs.
  3. `Authorization: Bearer <token>` required. An unknown, expired or revoked key, or one whose member is archived, gives 401 with `WWW-Authenticate`.
  4. Bad tokens are limited to 30 per minute per client IP; then 429 with `Retry-After: 60`.
  5. The member must still hold `core.mcp.use`, else 403.
  6. 60 requests per minute per key, else 429.
- **No cookies are read**, so the endpoint is not a CSRF target. Host and Origin are deliberately not restricted. Only the token id travels into the handler, never the secret.
- **Keys** (`api-tokens.ts`):
  - format `famos_` + 32 random bytes base64url; only SHA-256 stored; shown once
  - lifetime 1–365 days (UI offers 30/90/180/365, default 90)
  - at most 20 unrevoked keys per member
  - another member's key looks missing unless the caller has `core.ai.manage`
  - revoking is immediate; all of a member's keys are revoked on archive
  - key management is credential handling and deliberately runs outside the action runner

**Tools exposed** (`apps/server/src/mcp/server.ts`)
- Tools only. No MCP resources, prompts or sampling.
- Server `instructions` state today's date and time zone, that references like `vehicle:3fa9c1b2d4` must be passed back unchanged, that changes are suggestions (or "This connection can only read"), and that tool results are data.
- The actor is `{kind:'ai', source:'mcp', onBehalfOf: member, roleId}` with `local: false`:
  - **Read tools** = `aiReadTools({local:false})`: the same list as the assistant's (calendar.findEvents, calendar.aiContext, lists.shoppingList, lists.openTasks, chores.listChores, kitchen.mealPlan, kitchen.listRecipes, board.listNotes, records.upcoming, vehicles/pets/home aiContext, core.today, core.mySuggestions, core.searchFamily when an embedding model is ready).
  - **Draft tools** = `aiCatalog({local:false})`, only for `read_write` keys.
  - **Plus** `family_listPeople`, only if "Send names and descriptions" is on.
  - `assistantOnly` actions (`core.addMemory`) are never offered.
  - Areas set to Local only are left out entirely, because the agent's model is outside the house. Areas set to Off are never offered.
  - Tool names map `.` to `_`.
- **Annotations**: read tools are `readOnlyHint: true, openWorldHint: false`. Draft tools are `readOnlyHint: false, destructiveHint: false, idempotentHint: false, openWorldHint: false`, and their description ends with "Creates a suggestion that a family member approves; nothing changes until they do…".
- **Shared execution path**: tools run through `runAgentTool` (`packages/sdk/src/agent-tools.ts`), the same path as the assistant: handles instead of ids, results wrapped as data and capped at 16 KB, refusals that explain themselves without echoing input, and drafts with `sourceKind:'mcp'`, `sourceId`=token id, `promptVersion:'mcp/1'`. The runner then enforces permissions (the agent never exceeds the member), the target-name check, addressing (an adult's own suggestions are addressed to them) and auto-approval, exactly as for the assistant.
- **Stable handles** (`apps/server/src/mcp/handles.ts`): `kind:` + the first 10 hex chars of HMAC-SHA256(id, key = the token's id), lengthened on a clash. The same key always sees the same handle for a record and another key sees a different one, so handles cannot be correlated across keys. A handle resolves only if this key was shown it (`core_api_token_handles`), so invented handles are refused like invented ids.
- **Audit**: every tool call writes an `ai_usage` row with the key's name and the area touched (metadata only). Keys, requests and results are never logged.

**Model adapter** (`packages/adapters/src/llm-aisdk/llm.ts`, `settings.ts`, `schemas.ts`)
- **One client for every provider kind**: `@ai-sdk/openai-compatible` `createOpenAICompatible`. Kinds are presets of base URL and behaviour:
  - OpenRouter: `https://openrouter.ai/api/v1`, key required
  - Ollama: `http://localhost:11434/v1`; the bundled compose profile uses `http://ollama:11434/v1`
  - OpenAI-compatible: any address (OpenAI, Groq, LM Studio, vLLM, speaches/whisper.cpp for speech to text)
  - There is no native Anthropic or Google provider. Anthropic models are reachable through OpenRouter, or through an OpenAI-compatible endpoint.
- **Roles**: `smart` (Quick add text extraction, assistant chat and summaries), `vision` (Quick add with photos or PDFs), `transcribe` (OpenAI-style `/audio/transcriptions`, multipart), `embed` (`/embeddings`), `fast` (configurable but not called anywhere; see Unfinished).
- **Every call names a `feature`, a role, the `areas` it draws on and an explicit timeout.** Before the call:
  - the feature must be enabled (master switch and per-feature switch)
  - the role must be set and its provider exist; OpenRouter must have a readable key
  - **area rules**: an area that is Off means `disabled`. Local only means `local_required` unless the provider's address resolves only to local addresses (loopback, 10/8, 172.16/12, 192.168/16, 169.254/16, 100.64/10, ::1, fc00::/7, fe80::/10, fec0::/10, NAT64 of those). DNS results are cached for a minute. A refused call is recorded and never retried with another provider.
  - **monthly limits**: the count of `ai_usage` rows and the sum of `cost_micros` in the current family-time-zone month are compared with the limits, else `limit_reached`.
- **OpenRouter specifics**:
  - The request asks for `usage: {include: true}`, so cost is reported (also from the last stream chunk).
  - With "Only providers that keep no data" on, it adds `provider: {zdr: true, data_collection: 'deny'}`.
  - A ZDR or data-policy refusal maps to `data_policy`, shown as "No provider of this model keeps zero data".
  - Reasoning effort goes in the body for OpenRouter and as `reasoning_effort` elsewhere.
  - Models that refuse a thinking setting are asked again without it.
- **Structured output**: strict JSON-schema mode is not used (optional fields break it). Output is validated with Zod in the adapter.
- **Cost recording**: the provider-reported cost for chat, object and transcription calls (OpenRouter, or any provider whose transcription body includes `usage.cost`). Ollama counts as 0. Other providers record null.
- **Errors**: mapped to `LlmError` codes: `not_configured`, `auth`, `limit_reached` (HTTP 402 means out of credit), `rejected`, `data_policy`, `local_required`, `disabled`, `timeout`, `unavailable`, `invalid_output`, `output_too_long`. `retryable` decides between retrying and failing for a person to fix. Failed calls are recorded too, with any token usage reported.
- **Keys**:
  - sealed with the SecretBox; never returned (only `hasKey` and `keyReadable`, which is false if `FAMOS_SECRET` changed)
  - **changing a provider's address drops its stored key** unless a new key comes with the change, so a key can never be sent to another host
  - provider and key writes run outside the action runner, as credential management
- **Environment provider** (`FAMOS_AI_PROVIDER`, `FAMOS_AI_BASE_URL`, `FAMOS_AI_API_KEY`, `FAMOS_AI_MODEL[_FAST|_VISION|_SMART]`; `apps/server/src/config.ts`): reconciled on every start (created, updated, or removed when the variables are gone) and read-only in the UI. Roles that are unset or already use it take its models; roles pointed elsewhere stay. Speech-to-text and embedding models are chosen in the app only.
- **What a prompt may contain**: AGENTS.md rule 6 says data for a prompt comes only from the job's own item or from read-only actions invoked as the AI actor. A prompt builder never reads another extension's tables, so area locks apply by construction.
- **Sensitive areas**: an extension can declare `sensitive: true`. Its area then defaults to Off, it cannot be auto-approved, and opening it to Any provider needs confirmation (`ai-areas.ts`, `registry.ts`).

### Integrations
- **Outbound to model providers**: HTTPS, or HTTP on the local network, to OpenAI-compatible endpoints. Paths used: `/chat/completions`, `/embeddings`, `/audio/transcriptions`, `/models`. No other telemetry.
- **Inbound from agents**: MCP over HTTP at `/mcp` with a personal bearer token. There is no OAuth (clients that can only sign in with OAuth cannot connect) and no agent-to-agent protocol. For cloud-hosted agents the guide recommends a Cloudflare Tunnel limited to `^/mcp`; Tailscale Funnel publishes the whole port.
- **Tested clients**: Claude Desktop through `mcp-remote` (it needs Node.js on that computer), Claude Code, Hermes Agent.

### Display relevance
- None directly. These are admin and member settings, and displays cannot run actions.
- The usage data is relevant to Home HQ's AI-plan meters. FamilyOS has, per family time-zone month, total calls, errors, tokens and USD by feature and by model, against a family-wide budget and a call cap that pause all AI when reached. A per-call activity log names who each call was for. A wall or phone meter could read the `usage` summary; FamilyOS itself shows it only on the admin page.

### Docs vs code
- `mcp.md`: "Each person can have up to 20 working app keys." The code counts keys whose `revoked_at` is null, so expired but unrevoked keys still count toward the 20 (`MAX_TOKENS_PER_MEMBER`, `api-tokens.ts` `create`).
- `ai.md`: the **Model calls** limit "pauses AI until next month". The check counts every `ai_usage` row in the month, including failed and refused calls and MCP tool calls, which are logged as rows with `role='mcp'` and call no model. An active MCP agent therefore uses up the family's model-call budget, and can stop Quick add, the assistant and search.
- `ai.md`: the **Budget (USD)** counts "as reported by the provider". In code, only OpenRouter (and transcription bodies that include `usage.cost`) report cost. Ollama counts as 0. Direct OpenAI, Groq and other OpenAI-compatible providers record null, so a dollar budget never trips with them; only the call cap works.
- `ai.md` says "Fast is for quick, cheap steps." No code calls the `fast` role (grep `'fast'` matches only the config mapping in `apps/server/src/config.ts`). The summarizer uses `smart`.
- `ai.md` describes sensitive areas marked **Private records** that start Off, and an example **Finance** area. No bundled extension sets `sensitive: true` (grep across `extensions/`), and there is no finance extension. The mechanism exists but is unused by the bundled features.
- `ai.md` lists providers (OpenRouter, Ollama, OpenAI-compatible) and the README says "bring your own model". There is no first-class Anthropic or Google provider. Anthropic is reachable only via OpenRouter or an OpenAI-compatible endpoint. This is not stated as a limitation.
- `mcp.md` "What the app can do" lists calendar, open tasks, shopping, what is due, records of vehicles, pets and home, Today and search. The code also offers chores, meal plan, recipes, board notes and `core.mySuggestions` (the suggestion status). The guide is incomplete, not wrong.
- Local detection treats 100.64.0.0/10 (CGNAT, which includes Tailscale addresses) as local. A provider on a remote machine reached over Tailscale therefore counts as Local only-eligible. The docs say "your own machine or home network".

### Unfinished or stubbed
- The `fast` model role is configurable (UI and `FAMOS_AI_MODEL_FAST`) but unused.
- No OAuth for MCP; personal bearer tokens only (documented).
- MCP exposes tools only: no resources (for example a calendar resource), no prompts, no notifications or subscriptions.
- MCP draft calls carry no dedupe key, and MCP has no idempotency key. A client that retries a tool call creates a second suggestion. The assistant and Quick add use per-turn and per-item dedupe keys.
- All MCP suggestions from one key share `source_id` = token id, so their "waiting" notices collapse into one per key while any of them waits (`draft-notices.ts` dedupe `drafts:mcp:<tokenId>`). This may be intended, but it differs from per-capture grouping.
- Limits are family-wide only: no per-member or per-feature budgets. Usage is broken down by feature and model, not by member (only the activity log names members).
- No native provider SDKs (Anthropic, Google, Azure). No fallback between providers (by design a refused call is never retried elsewhere).
- No sensitive extension ships, so the "Private records" path is untested by the bundled features.

### Beyond Home HQ (net-new ideas)
- **Bring your own agent**: an MCP door with per-person, expiring, revocable, read-only or suggest app keys. The agent sees only what that person may see, and every write becomes an approvable, undoable suggestion. Home HQ could expose its Today, calendar, chores and grocery list the same way.
- **Stable per-key handles** (HMAC of the id with the key id) instead of raw ids. Records can be referenced across stateless requests, but nothing can be correlated between keys, and invented references are refused.
- **Area-level AI sandbox** (Off / Local only / Any provider). Local only is enforced at the model adapter by resolving the provider address, never by trusting the provider kind, and never falls back to the cloud.
- **Privacy switches** for files, names, voice and OpenRouter zero data retention. Keys are dropped when a provider's address changes.
- **A metadata-only AI activity log** with the areas touched per call, which feeds both auditing ("did the locked area stay out?") and the usage and budget meters.
- **Monthly USD budget and call cap** in the family time zone, enforced before every call, with per-feature and per-model usage. This lines up with Home HQ's AI usage meters and could add a hard stop.
- **Env-managed provider** for headless installs, reconciled on start and read-only in the UI.

### Evidence
- `apps/server/src/mcp/{http,server,handles}.ts`, `apps/server/test/mcp.test.ts`
- `packages/core/src/api-tokens.ts`, `packages/core/src/core-capabilities.ts` (`core.mcp.use`, `core.ai.manage`), `packages/core/src/migrations/0001_init.ts`
- `packages/sdk/src/agent-tools.ts`, `packages/sdk/src/llm.ts` (`MODEL_ROLES`)
- `packages/core/src/{kernel,ai-areas,ai-tools,registry,search}.ts`
- `packages/adapters/src/llm-aisdk/{llm,settings,schemas,tables,index}.ts`, `packages/adapters/src/http-guard/index.ts` (`isLocalAddress`)
- `apps/server/src/config.ts` (`FAMOS_AI_*`, `FAMOS_SECRET`, `FAMOS_ROLES`)
- `apps/web/src/routes/settings/{ai,connected-apps,auto-approval}/*`, `apps/web/src/lib/nav.ts`
- Docs: `docs/guide/{mcp,ai,search}.md`, `README.md` ("AI providers", "Connect your own AI agent (MCP)"), `AGENTS.md`

---

## 12. Members, roles, permissions and sign-in

FamilyOS runs one household per instance. Everyone in it is a row in `core_members` with one role. A role is a set of capabilities. Code never checks a role name; it only asks `can(actor, capability)` (`packages/sdk/src/capabilities.ts`, `packages/core/src/access.ts`). Sign-in is handled by better-auth behind an `Identity` port: adults use email and password, and roles with the PIN capability pick their profile and type a PIN, which only works on admin-marked family devices (`packages/adapters/src/identity-betterauth/index.ts`).

### What the user can do

- **First-run setup** (`/setup`).
  - The server logs `<baseURL>/setup?code=…`. The code is also written to `data/setup-code` (`apps/server/src/main.ts` `announceSetup`, `apps/server/src/config.ts`).
  - The founder enters a name, email, password (10 or more characters) and language. The browser suggests time zone and region formats.
  - The founder can tick "This device is shared at home" so the setup device becomes a family device (`docs/guide/install.md`).
  - `POST /api/v1/core/setup` only works while `core_members` is empty. It creates the first Admin and the family settings in one transaction (`packages/core/src/core-extension.ts` `setup`).
  - Optional extra setup steps come from the `setup.step` slot (`docs/architecture.md`).
- **Members** (Settings → Family → Members, admins only, `core.members.manage`).
  - **Add**: name (60 chars), role, and an optional "About" (300 chars, free text the AI reads; it has no security meaning).
  - **Set email & password** or **Set PIN**. The UI shows whichever one the role uses (`apps/web/src/routes/settings/members/+page.svelte` `usesPin`).
  - **Edit**: name, role, About.
  - **Remove from the family**: archives the member. It cannot be undone.
  - The last admin cannot be demoted or removed (`activeAdminCount`).
  - Members cannot change their own password or PIN. The profile page says "Ask an admin to change your password or PIN." (`apps/web/src/routes/settings/profile/+page.svelte`, `apps/web/src/messages/en.json`).
- **Roles** (Settings → Family → Roles, `core.roles.manage`).
  - A matrix shows every capability against every role. A lock marks the critical, admin-only ones.
  - **New role from Member / Supervised / Guest**: give it a name (40 chars) and tick or untick single capabilities. Rename or re-tick it later.
  - **Delete role** works only when no member, archived members included, has the role (`core.deleteRole`).
  - The four built-in roles cannot be edited. Admin cannot be used as a base.
- **Profile & sign-in** (Settings → You).
  - Choose your own language. Empty means the family language (`core.setMyLanguage`, allowed for every role).
  - **Sign out** ends the session on this device. **Sign out everywhere** calls `POST /core/sessions/end`.
  - Per-person look and Today choices: `core.setMyTheme`, `core.setMyStyle`, `core.setMyTodayBlocks`, `core.setMyNextUp` (`packages/core/src/themes.ts`, `packages/core/src/next-up.ts`).
- **Family devices** (Settings → Family → Devices, `core.devices.manage`).
  - "Make this a family device" names the current browser, e.g. "Kitchen tablet" (`POST /api/v1/core/devices/trust`).
  - List devices and remove one (`DELETE /core/devices/{id}`).
  - **Stop using** under "This device" (`POST /core/devices/forget`).
- **Sign in on a family device** (`/login`).
  - "Who's using FamilyOS?" lists only active members whose role holds `core.signIn.pin`, as name and color (`core.listProfiles`, access `trusted-device`).
  - Tap a name, type the PIN on a number pad, and sign in. Adults use "Sign in with email".
  - On any other device only the email form is shown and no names appear.
- **App keys** (personal API tokens for MCP clients; `docs/guide/mcp.md`).
  - Create a key with a name, a scope (Read only / Read and suggest changes) and a lifetime (30, 90, 180 or 365 days; 90 by default). The key is shown once.
  - The key list shows each key's scope, expiry and last use.
  - Revoke your own keys. Admins (`core.ai.manage`) can also see and revoke everyone's.
  - The whole MCP endpoint is off until an admin turns it on (`core_settings` key `mcp`).

### Data model

- `core_members` (`packages/core/src/tables.ts`, `packages/core/src/migrations/0001_init.ts`)
  - Columns: `id`, `display_name`, `role_id` (built-in id or `role_…`), `about` (nullable), `color` (`#rrggbb`), `locale` (nullable; null follows the family), `created_at`, `updated_at`, `archived_at`.
  - Colors are assigned in order from 8 hues kept away from red, so a member color never reads as an alert (`MEMBER_COLORS` in `core-extension.ts`). The API accepts `color` on create and update, but the members page has no color picker.
  - There is no birthday, photo avatar or per-person time zone. Avatars are initials on the member color (`Avatar` in `@famos/ui`).
- `core_roles`: `id`, `name`, `base` (`member` | `supervised` | `guest`), `overrides` (JSON map of capability id to boolean), timestamps.
- Identity tables (`packages/adapters/src/identity-betterauth/migrations.ts`, committed SQL; better-auth never migrates at runtime):
  - `auth_users`: email unique, `memberId` unique, one user per member.
  - `auth_sessions`: `token`, `expiresAt`, `ipAddress`, `userAgent`, plus a custom `deviceId` (set only for PIN sessions, indexed).
  - `auth_accounts`: one row per provider: `credential` for the password, `pin` for the PIN. Both are hashed with better-auth's scrypt.
  - `auth_verifications`.
- **PIN-only members without email.** `setPin` creates the auth user with the address `<memberId>@child.famos.invalid` and only a `pin` account, so that person has no working email sign-in.
- `core_devices`
  - Columns: `id`, `name`, `kind` (`family` | `kiosk`), `token_hash` (SHA-256 of a 32-byte random token; the token itself lives only in the `famos_device` httpOnly cookie), `created_by`, `created_at`, `last_seen_at` (updated at most hourly), `revoked_at`.
  - Display-only columns: `display_capabilities`, `display_show_names`.
- `core_display_pairings`: an RFC 8628-style pairing (device code hash, `XXXX-XXXX` user code, expiry, attempts, status).
- `core_api_tokens`: `member_id`, `name`, `token_hash` (SHA-256), `scope` (`read` | `read_write`), `expires_at`, `last_used_at`, `revoked_at`.
- `core_api_token_handles`: maps short handles shown to a stateless MCP client back to ids, up to 5000 per token.
- `core_audit_log`: `actor` (JSON), `action_id`, `draft_id`, `outcome`.
- `core_drafts` holds suggestions, including those made by Supervised members. It is covered in the approvals section.

### Rules and logic worth copying

**Capability model** (`packages/sdk/src/capabilities.ts`)

- Every capability has a level: `view`, `use`, `edit` or `manage`. One table maps levels to the built-in roles:

  | Level | Admin | Member | Supervised | Guest |
  |---|---|---|---|---|
  | view | yes | yes | yes | yes |
  | use | yes | yes | yes | no |
  | edit | yes | yes | no | no |
  | manage | yes | no | no | no |

- An extension may declare `defaults` that only remove grants its level gives. `assertRestrictiveDefaults` throws at registry build if a default would add one.
- Only core may set defaults that add grants. It does so for `core.signIn.pin`: Admin false, Member false, Supervised true, Guest true.
- **Critical** capabilities are core-only. They are held by Admin alone, are forced to false in custom roles even if an override says true, and are rejected by `checkOverrides`. The list: `core.members.manage`, `core.roles.manage`, `core.devices.manage`, `core.settings.manage`, `core.ai.manage`, `core.features.manage`, `core.autoApprove.manage` (`packages/core/src/core-capabilities.ts`).
- A PIN never unlocks administration. Admin never holds the PIN capability, and critical capabilities cannot be added to a custom role.
- Non-critical core capabilities and their levels:
  - `core.members.view` (view)
  - `core.notifications.receive` (view)
  - `core.drafts.waiting` (view)
  - `core.drafts.propose` (use)
  - `core.mcp.use` (use; Supervised members can create app keys by default)
  - `core.memory.use` (use)
  - `core.drafts.approve` (edit)
- **Custom role effective set** (`createAccess` in `access.ts`): for each active capability, take the override if there is one, else the base role's default; critical ones are always false. Capabilities added by extensions installed later follow the base role.
- Effective sets are cached in memory and cleared on `core.role.changed` or `core.features.changed`. A turned-off extension's capabilities are inactive for everyone.
- **Per-extension capabilities**, from each `extensions/*/src/server/capabilities.ts`, `tasks.ts`, `shopping.ts`, `items.ts`:
  - calendar: `view`, `edit`, `manage` (feeds and subscriptions)
  - lists: `view`, `tasks.complete` (use), `tasks.edit`, `shopping.edit` (use: add and tick off items), `shopping.manage` (edit)
  - chores: `view`, `use`, `edit`
  - board: `view`, `post` (use), `manage`
  - inbox: `share` (use)
  - assistant: `use`
  - records, home, pets, vehicles, kitchen: `view` and `edit`
  - ambient: `photos` and `weather` (view, grantable to displays), `manage`
- **Actor kinds** (`packages/sdk/src/actor.ts`):
  - `member` holds what the role grants.
  - `system` may do everything, except the `anonymous` source used for unauthenticated public routes, which holds nothing.
  - `ai` only ever reads at `view` level, and every change it makes becomes a draft. Acting for a member (`onBehalfOf`), it never exceeds that member's capabilities (`memberCan`).
  - `display` holds only the view capabilities granted at pairing. It sees `core.members.view` only with "show names".
- **Registry guardrails** (`packages/core/src/registry.ts`). Registration fails when any of these is broken:
  - Every data-changing action declares `requires` or `approval: { unless }`.
  - Actions with AI hints cannot use `approval: 'never'`.
  - External actions cannot have AI hints.
  - Auto-approve is refused for core, external or sensitive actions.
  - Every capability id referenced anywhere must exist.

**Supervised changes become approvals** (`packages/core/src/action-runner.ts` `needsApproval`, `invokeOnce`, `assertApprover`)

- An action declares `requires` (who may invoke at all) and `approval: { unless: <edit capability>, except?: fn }`.
- A member lacking the `unless` capability gets a draft instead of a direct change, provided the role holds `core.drafts.propose`. Guests do not, so they are refused.
- Examples: `calendar.edit`, `records.edit`, `home.edit`, `lists.shopping.manage`, `kitchen.edit`, `chores.edit`.
- `lists.addTask` has an `except` rule: anyone who may propose can add a task assigned to themselves directly (`extensions/lists/src/server/tasks.ts`).
- Deciding a draft needs `core.drafts.approve` plus whatever would let the approver make the change directly. A draft addressed to one member (`for_member`) is decided by that member only, and is invisible to everyone else.
- Approvers (holders of `core.drafts.approve`, minus the proposer) are notified after a 30 s delay, so several drafts from one shared item collapse into one notice (`DEFAULT_DRAFT_NOTIFY_DELAY_MS` in `packages/core/src/auto-approval.ts`).
- Admins can turn on auto-approval for opted-in actions (shopping items, tasks). It approves AI drafts as the admin who enabled it (`docs/guide/auto-approval.md`).

**Built-in roles in practice** (`docs/guide/members.md` matches the level table)

- **Member**: everything that is not `manage`. Can approve suggestions and use the assistant.
- **Supervised**: sees everything. Can directly:
  - add and tick shopping items
  - tick off tasks
  - do their own chores
  - post board notes
  - share to Quick add
  - use the assistant
  - create app keys

  Anything else becomes a suggestion.
- **Guest**: view-level only. No sharing, suggesting or assistant. Signs in with a PIN.
- Guests also receive broadcast reminders whose audience is a view capability (see the notifications section).

**PIN and trusted devices** (`identity-betterauth/index.ts`, `apps/server/src/http/app.ts` `resolveMember`, `apps/server/src/http/devices.ts`)

- PIN format: 4 to 6 digits (`/^\d{4,6}$/`).
- `POST /api/auth/sign-in/pin` is refused with `UNTRUSTED_DEVICE` (403) unless the request carries a non-revoked `famos_device` cookie of kind `family`.
- The handler always runs a hash verification, against a dummy hash when there is no PIN, so response time reveals nothing. A role without PIN sign-in gets the same "Invalid PIN" as a wrong PIN.
- A PIN session stores `deviceId`. **Every request** re-checks that the device is still active and that the member's current role still holds `core.signIn.pin`. Revoking a device or changing a role takes effect on the next request, even if deleting sessions failed.
- Setting a PIN is refused for roles without the PIN capability (`core.setMemberPin`).
- Setting a new PIN signs the member out everywhere. Setting a password does the same, except when admins change their own.
- **Lockout.** 5 failed sign-ins per account lock it for 15 min, on every device.
  - The account key is `pin:<memberId>` or `email:<email>`.
  - The 429 response carries `Retry-After`, and the UI shows the server's wait.
  - The lockout is in memory, so a restart clears it.
  - Any non-OK response counts as a failure, including a 403 from an untrusted device.
- **Rate limits.**
  - better-auth: 100 requests per 60 s per IP overall, and 5 per 60 s on `/sign-in/pin`.
  - The client IP comes from the socket. `X-Forwarded-For` is trusted only from `FAMOS_TRUSTED_PROXIES`, taking the right-most untrusted hop (`apps/server/src/http/client-ip.ts`).
  - Auth request bodies are capped at 16 KB.
- **Family device cookie**: httpOnly, SameSite=Lax, `Secure` on HTTPS, max-age 400 days, refreshed on every request.
- Mutual exclusion between family devices and displays:
  - A paired display's browser cannot be made a family device (409).
  - A browser used by a member, or one already a family device, cannot start display pairing (409).
- `POST /core/devices/forget` needs no sign-in. Whoever holds the cookie can revoke that family device, and doing so ends the PIN sessions on it.
- Removing a device ends only PIN sessions there (`endDeviceSessions` by `deviceId`). Password sessions opened on that tablet survive.

**Sessions**

- better-auth sessions last 30 days, sliding: refreshed when older than 1 day (`expiresIn`, `updateAge` in `authOptions`). This applies to PIN sessions too.
- No idle timeout or automatic sign-out on shared devices was found in the web app.
- CSRF uses Hono `csrf()` on `/api/v1/*` with trusted origins. API responses are `no-store`.
- Archiving a member (`memberChanged` with `endSessions`, `packages/core/src/kernel.ts`):
  - deletes the auth user, accounts and sessions, which frees the email
  - revokes all API tokens
  - unlinks Telegram (`apps/server/src/telegram.ts`)
  - Web Push subscriptions are not deleted, but delivery skips archived members (`core.deliver`).

**API tokens** (`packages/core/src/api-tokens.ts`, `apps/server/src/mcp/http.ts`)

- Format: `famos_` plus 32 random bytes in base64url. The prefix lets secret scanners and owners recognize a leaked key.
- Only the SHA-256 is stored.
- Lifetime: 90 days by default, from 1 to 365 days. At most 20 active tokens per member.
- `last_used_at` updates at most hourly.
- Authentication fails if the token is revoked, expired, or belongs to an archived member.
- Every MCP request re-checks `core.mcp.use` for the token holder.
- Tokens in the URL are refused; the client must send `Authorization: Bearer`.
- Rate limits: 60 requests per minute per token, and a separate limit of 30 bad tokens per minute.
- The MCP actor is `{ kind: 'ai', source: 'mcp', onBehalfOf: member }`. A `read` token gets read tools only; `read_write` adds draft tools. Changes are always suggestions.

**Setup code**

- `FAMOS_SETUP_CODE` (8 to 100 chars), or 9 random bytes persisted at mode 0600.
- Compared without early exit, so timing reveals nothing (`sameCode`).
- Setup re-checks that no members exist inside the transaction.
- If the identity write fails, setup deletes the member it just created.

**Secrets and outbound HTTP**

- `createSecretBox` (`packages/adapters/src/secrets/index.ts`): AES-256-GCM, key derived with HKDF-SHA256 from `FAMOS_SECRET` (salt `famos`, info = purpose), stored as `v1.<iv>.<tag>.<ciphertext>`.
  - Purposes: AI provider keys, which also seal the Web Push VAPID private key and the Telegram token (`apps/server/src/runtime.ts`); plus `email`, `backup` and `extension-secrets`.
  - A changed secret makes `open` return null, and the UI reports the value as "unreadable".
- `http-guard` (`packages/adapters/src/http-guard/index.ts`) is an SSRF guard. It:
  - resolves DNS first, and requires every answer to be allowed
  - connects to the checked IP while keeping the host name for TLS and the Host header
  - always blocks reserved, multicast and documentation ranges
  - blocks private ranges unless the caller passes `allowLocal`
  - allows plain `http:` only to local addresses
  - follows at most 3 redirects, same host only
  - caps responses at 5 MB and 20 s by default
  - rejects credentials embedded in URLs

### Integrations

- **better-auth 1.7.x** with a custom `famos-pin` plugin. Telemetry is off, and runtime schema validation is off.
- **MCP** over Streamable HTTP with bearer app keys (`apps/server/src/mcp/*`).
- **Telegram links** are credentials too. An archived member's link is removed. Details are in the notifications section.

### Display relevance

- A wall display is a `core_devices` row of kind `kiosk`, paired by code. It is not a member and not a family device. Its actor is `{ kind: 'display', capabilities, showNames }`.
- A display cannot:
  - sign anyone in
  - open the profile picker
  - run any action, even a read-only one (`authorize` in `action-runner.ts`)
  - call `signedIn` or `trusted-device` routes
- `core.listMembers` gives a display names and colors only, with the role and About blanked.
- A member session in the same browser outranks the display cookie.
- The device is verified on every request. A change or revoke closes its WebSocket with code 4401.
- The shared family tablet, the "family device", is a different concept. It is a normal app with a profile picker and PIN sign-in. Home HQ's phone-plus-wall split maps to member devices, family devices (shared, PIN) and kiosks (read-only).

### Docs vs code

- **Deleting a role.** `members.md` says it works "when nobody has the role". The code also counts archived members, and archiving cannot be undone. So a custom role ever held by a removed member can never be deleted (`core.deleteRole`).
- **"Only admins" list.** `members.md` says only admins manage calendar subscriptions and display content. Those are `calendar.manage` and `ambient.manage`, which are manage-level but not critical, so a custom role can be granted them. The same holds for `board.manage`. Only the critical capabilities are truly admin-only.
- **Member colors.** A code comment says "admins can change them", and `core.updateMember` accepts `color`, but the members page shows no color field and `members.md` lists only name, role and About as editable.
- **Password versus PIN roles.** The docs say PIN roles sign in with a PIN, and the UI only offers a PIN for them. `PUT /core/members/{id}/password` does not check the role, though. A member moved from a password role to a PIN role keeps a working email and password login, usable from any device, because nothing removes the `credential` account.
- **Sessions.** The docs do not state the 30-day session length or that removing a family device leaves password sessions on it alive.
- **Region permission.** `region.md` says changing it needs "the permission to change family settings, which admins have". That permission is critical, so only admins can ever hold it.
- **Setup code length.** The README says `FAMOS_SETUP_CODE` must be 8 to 100 characters, and `install.md` says "at least 8". The code enforces 8 to 100.

### Unfinished or stubbed

- No TODO, FIXME or stub markers were found in the in-scope files. `CHANGELOG.md` only says "First public version."
- Absent by design:
  - passkeys (`docs/architecture.md`: they need HTTPS and a stable address)
  - self-service password or PIN change
  - email verification and password reset by email (`disableSignUp: true`; admins set credentials)
  - unarchiving members
- Lockout and rate-limit state live in memory per process. The docs say a restart clears them.

### Beyond Home HQ (net-new ideas)

- **Capability levels plus one role-mapping table.** Features declare `view`, `use`, `edit` or `manage`, and roles are data. Adding a feature never means editing every role. Custom roles are a base plus overrides, so future capabilities follow the base automatically.
- **"Propose instead of refuse."** Lacking the edit right turns a change into a suggestion an adult approves. This gives kids real participation, such as suggesting an event, without the risk. Per-action `except` rules allow self-service, e.g. a task assigned to oneself.
- **PIN only on admin-marked devices.** Wrong-PIN lockouts are per account, not per IP. Every request re-checks device and role, so revoking a lost tablet takes effect instantly. The profile picker is never shown off a trusted device, so names never leak to the internet.
- **RFC 8628-style display pairing.** It uses an unambiguous 8-character code alphabet. Five wrong codes end all open pairings. The display gets a scoped, view-only grant set and a "show names" switch.
- **Personal app keys for AI agents** (MCP). They are scoped read or read-and-suggest, expire, can be revoked, and never exceed the member.
- **Gaps Home HQ could close:**
  - quick person switching and idle auto-sign-out on the shared tablet
  - member birthday and photo avatar
  - per-person time zone
  - a self-service PIN change gated by the old PIN

### Evidence

- `docs/guide/members.md`, `docs/guide/install.md`, `docs/guide/mcp.md`, `docs/architecture.md` (sections on roles, devices, displays, pairing), `README.md`
- `packages/sdk/src/capabilities.ts`, `packages/sdk/src/actor.ts`
- `packages/core/src/core-capabilities.ts`, `access.ts`, `registry.ts`, `tables.ts`, `family.ts`, `devices.ts`, `api-tokens.ts`, `kernel.ts`, `core-extension.ts` (members, roles, setup, profiles, PIN and password routes), `action-runner.ts`, `auto-approval.ts`, `migrations/0001_init.ts`
- `packages/adapters/src/identity-betterauth/index.ts`, `migrations.ts`
- `packages/adapters/src/secrets/index.ts`, `packages/adapters/src/http-guard/index.ts`
- `apps/server/src/http/app.ts` (`resolveActor`, `requireActor`), `devices.ts`, `client-ip.ts`, `rate-limit.ts`, `apps/server/src/config.ts`, `apps/server/src/main.ts`, `apps/server/src/mcp/http.ts`, `apps/server/src/mcp/server.ts`, `apps/server/src/telegram.ts`
- `apps/web/src/routes/settings/members/+page.svelte`, `settings/profile/+page.svelte`, `apps/web/src/lib/auth.ts`
- `extensions/*/src/server/capabilities.ts`, `extensions/lists/src/server/tasks.ts`

---

## 13. Notifications, quiet hours, region and settings

A FamilyOS notification is data first. The in-app row in `core_notifications` is the record. Web Push and Telegram are copies of it, sent by background jobs. Extensions declare notification kinds and call `notify`. Core resolves who receives each one, applies each person's per-kind and per-channel choices and quiet hours, and renders the text in each recipient's language at delivery (`packages/core/src/notifications.ts`, `docs/architecture.md`). Region formats, the family time zone, feature switches and other household settings are JSON values in `core_settings`.

### What the user can do

**Notifications** (`docs/guide/notifications.md`, `apps/web/src/routes/settings/notifications/+page.svelte`)

- **In-app list.**
  - On desktop and tablet, a bell shows the unread count. Its popover has the latest items and "Mark all read".
  - On a phone, the bell sits on Today and opens the full list. Settings → You → Notifications also shows it, and More carries an unread badge.
  - Tapping an item marks it read and opens its in-app `url`.
  - Each person has their own list, in their own language.
  - `GET /core/notifications` returns 50 items by default, at most 200, plus the unread count. No pagination beyond the limit was found.
- **Phone or computer notifications (Web Push).**
  - "Turn on notifications on this device" asks the browser for permission and subscribes it.
  - "Send a test notification" goes direct, bypassing the queue and quiet hours (`sendTest`).
  - "Turn off notifications on this device" works per device.
  - "Devices with notifications" lists the person's devices by label, created time and last success.
  - On iPhone and iPad this needs the Home Screen app and iOS 16.4 or later. HTTPS is required.
- **Quiet hours.** One window per person: a switch with From and Until, defaulting to 22:00 to 07:00. It is evaluated in the family time zone and may cross midnight.
- **What goes where.**
  - A switch for each notification kind and each available channel ("Phone notifications" once a device is subscribed, "Telegram" once linked).
  - Kinds the person can never receive are hidden (the `audience` capability).
  - Without any channel the page only says notifications appear in the app.
  - The in-app copy cannot be turned off.
- **Telegram** (`docs/guide/telegram.md`).
  - An admin pastes a BotFather token in Settings → System → Telegram. The page shows "Connected as @bot" and a state: Working, conflict, invalid token, offline, or unreadable.
  - Each person links under Notifications with "Link Telegram". This produces a one-time code (10 min) and a `t.me/<bot>?start=<code>` link.
  - In the bot:
    - typed text goes to the assistant
    - forwards, photos, PDFs, audio, voice notes, links and `/share <text>` go to Quick add
    - suggestions arrive with approve and reject buttons
    - `/stop` unlinks

**Region and formats** (`docs/guide/region.md`, `apps/web/src/routes/settings/region/+page.svelte`, admins only)

- Settings:
  - time zone (searchable list)
  - region format: a BCP 47 tag, typed or picked, with "Use this browser's region"
  - first day of the week
  - clock: 24 h or 12 h
  - temperature: °C or °F
- Changing the region format pre-fills week start, clock and temperature from `Intl`. Each can still be changed before Save.
- A preview shows a date, time, number and temperature.
- Language is separate. The family has a default, and each person may override it under Profile.

**Features** (Settings → Features, `core.features.manage`)

- Turn any extension off or on. Core cannot be turned off.

### Data model

- `core_notifications` (one row per recipient, `packages/core/src/tables.ts`)
  - Columns: `id`, `member_id`, `kind`, `title_key`, `title_params` (JSON), `body_key`, `body_params`, `url` (in-app path), `source_kind`, `source_id`, `dedupe_key`, `created_at`, `read_at`.
  - Text is stored as translation keys plus parameters, never rendered text.
  - A unique index on (`member_id`, `dedupe_key`) where `dedupe_key` is set (`migrations/0001_init.ts`). The same key never notifies a person twice, ever.
- `core_notification_prefs`: `member_id` (primary key), `prefs` JSON of the form `{ kinds: { [kindId]: { [channelId]: bool } }, quietHours: { start: 'HH:MM', end: 'HH:MM' } | null }`.
  - Kinds are merged on save. Quiet hours are replaced.
  - Missing entries follow the kind's `defaultChannels`. No kind declares any, so every channel is on by default.
- `webpush_keys`: a single row holding the VAPID P-256 key pair. The private key is sealed with `FAMOS_SECRET`. The pair is generated on first use and never rotated, because rotating would invalidate every subscription (`packages/adapters/src/webpush/tables.ts`).
- `webpush_subscriptions`: `id`, `member_id`, `endpoint` (unique), `p256dh`, `auth`, `label` (e.g. "iPhone · Safari"), `created_at`, `last_success_at`. A subscription belongs to one browser endpoint and one member at a time.
- Telegram tables (`packages/adapters/src/telegram/tables.ts`):
  - `telegram_settings` (single row): sealed `bot_token`, `bot_id`, `bot_username`, `update_offset`, `status_code`, `status_at`.
  - `telegram_links`: `member_id` (primary key), `telegram_user_id` (unique), `chat_id`, `username`, `linked_at`, `thread_id` (the member's assistant thread for the chat), `waiting_reply_id`.
  - `telegram_link_codes`: `code` (primary key), `member_id`, `expires_at`.
- `queue_jobs` (`packages/adapters/src/queue-sqlite/index.ts`)
  - Columns: `id`, `name`, `payload`, `status` (`pending` | `running` | `succeeded` | `dead`), `attempts`, `max_attempts`, `run_at`, `lease_until`, `last_error`, `dedupe_key` (unique among pending and running jobs), timestamps.
  - Jobs are enqueued in the caller's transaction, so they commit or roll back with the data.
- `core_settings` (key → JSON value). Keys found:
  - `family`: `timeZone`, `locale`, `formatLocale`, `weekStart` (1 = Monday … 7), `hourCycle` (`h12` | `h23`), `temperatureUnit`
  - `features`: `{ disabled: [...] }`
  - `mcp`: `{ enabled }`
  - `ai_areas`
  - `core.autoApproval`
  - `languages`, `languages.fetch`
  - `themes.installed`, `themes.choices`, `styles.choices`, `today.hidden`, `themes.fetch`
  - `today.nextUp`
  - `search_index`

  Accessed through `readFamilySettings`/`writeFamilySettings` (`family.ts`), `readDisabled`/`writeDisabled` (`features.ts`) and `readJson`/`writeJson` (`settings-json.ts`).
- Defaults before setup are English, UTC, Monday week start and Celsius (`DEFAULT_FAMILY_SETTINGS`). Setup always stores a complete set.

### Rules and logic worth copying

**Declaring and sending** (`packages/sdk/src/notifications.ts`)

- A kind has an `id` namespaced to its extension (enforced in `registry.ts`), a label key, optional `defaultChannels`, and an optional `audience` capability. The audience only decides who sees the kind in preferences. Delivery follows the notify call.
- A request looks like `{ kind, to: { members?, capability?, except? }, title: {key, params}, body?, url?, source?, dedupeKey?, delayMs? }`.
- `uow.notify` enqueues `core.notify` inside the action's transaction, so a rollback tells nobody.

**Fan-out** (`core.notify` in `notifications.ts`)

1. Does nothing if the kind's extension is turned off.
2. For a `core.draftWaiting` request, does nothing if the draft was decided during the delay.
3. Resolves recipients **when the job runs**: active, non-archived members in `to.members`, plus every member whose role holds `to.capability`, minus `except`. A role change in between therefore changes who is told.
4. For each recipient, works out the channels that are enabled for the kind and available (`isAvailableFor`).
5. Inserts one row per recipient with `on conflict do nothing`, so retries never duplicate.
6. Enqueues one `core.deliver` job per recipient and channel.
7. Emits `core.notification.created`, so open tabs refresh their badge over WebSocket.

**Delivery** (`core.deliver`, 8 attempts, exponential backoff `min(2^n s, 15 min)` from `packages/sdk/src/jobs.ts`)

- Skips the send if the notification was **already read in the app**, the member is archived, the feature is off, or the channel was switched off since queuing.
- **Quiet hours** (`quietUntil`):
  - If now falls inside the member's window, in the family time zone, the job re-enqueues itself at the window's end and sends nothing.
  - The re-scheduled time is at least now + 60 s, which guards against a skipped hour on a DST day.
  - A window whose start equals its end means no quiet hours.
  - Nothing is dropped. At the end of the window, only still-unread items are pushed.
  - Quiet hours are per person, not per family. No kind is exempt: there is no urgent override, and Web Push always sends `Urgency: normal`.
- Text is rendered at send time in `member.locale ?? family.locale`. The in-app list renders at read time, so a language change re-renders old notifications too.
- Titles and bodies are never logged.

**Collapsing**

- `dedupeKey` collapses repeats per recipient for good. Reminder keys follow the pattern `reminder:<itemId>:<occurrence>`.
- `delayMs` holds a request back so bursts merge. Approval notices wait 30 s (`DEFAULT_DRAFT_NOTIFY_DELAY_MS`), and every draft from one shared item shares the key `drafts:<sourceKind>:<sourceId>`.
- When a draft is decided, the matching unread "waiting" notices are marked read for everyone (`settleWaitingNotices` in `packages/core/src/draft-notices.ts`). Stale approval prompts disappear by themselves.

**Which events notify whom** (all kinds found in code)

| Kind | Recipients | When |
|---|---|---|
| `core.draftWaiting` | Holders of `core.drafts.approve`, except the proposer | 30 s after a suggestion that someone else must decide. AI drafts that the person behind them may decide are addressed to that person and send no notice. |
| `inbox.itemReady` | The person who shared | Once the AI has read something shared from outside the app (share sheet, Telegram, email) and has suggestions. Not sent for the Quick add page itself. |
| `calendar.reminder` | The event's members, else every holder of `calendar.view` | Timed events: 30 min before by default. All-day events: 09:00 local minus the lead. Configurable per event. Also covers external subscriptions when that subscription's reminders are on. |
| `lists.taskDue` | The assignee, else every holder of `lists.view` | At the due time, or 09:00 when the task has none. |
| `chores.choreDue` | The person whose turn it is | 09:00 on the due day. A chore with no assignee reminds nobody. |
| `records.obligationDue` | The assignee, else the domain's audience capability (`vehicles.view`, `pets.view`, `home.view`…), else `records.view` | `lead_days` before the due date and on the due date, at 09:00. |
| `board.posted` | Every holder of `board.view`, except the author | When a note is put up. |
| `core.backupFailed` | Holders of `core.settings.manage` (admins) | An off-site backup upload failed. |
| `core.autoApprovalStopped` | The admin who turned auto-approval on | The setting stopped applying on its own. |

- "Everyone who can see X" includes **Guests** by default, because the audience capabilities are view-level. A babysitter with the Guest role gets unassigned family reminders and board notices.

**Reminder scheduling** (`extensions/*/src/server/reminders.ts`)

- Only the **next** reminder of each item is planned, as a job with `runAt`. It is enqueued in the transaction of every create, update or completion, and carries the item's `updated_at` (plus occurrence and instant for events).
- When the job runs, it reloads the item and does nothing if the item changed, was done, deleted or skipped.
- A reminder more than 5 min late at planning time is dropped. A job that waited past its day or event (server down) is not sent.
- A recurring event's job schedules the following occurrence, looking ahead 400 days.
- The anchor time of 09:00 is hard-coded. It is not a family or personal setting.

**Worker** (`packages/core/src/worker.ts`)

- One loop per lane, `default` and `long`, each running one job at a time. Long uploads therefore never delay reminders.
- Idle poll every 1 s, woken immediately after commits that enqueue jobs.
- Expired leases are recovered every 30 s. A job's lease is its timeout plus 30 s.
- Jobs of turned-off extensions stay queued.
- Succeeded jobs are pruned after 7 days (`apps/server/src/runtime.ts`). Dead jobs are kept.
- The worker runs only in processes started with the `worker` role (`FAMOS_ROLES`).

**Web Push** (`packages/adapters/src/webpush/service.ts`, `crypto.ts`)

- Implements RFC 8030, with VAPID and aes128gcm payload encryption written in-house, with no web-push dependency.
- VAPID JWT lifetime is 12 h. The `sub` claim is the instance base URL.
- **SSRF guard.** Endpoints must be `https:`, carry no credentials, and be on an allow-list: `fcm.googleapis.com`, `*.push.apple.com`, Mozilla push hosts, `*.notify.windows.com`. Redirects are an error.
- Request settings: 10 s timeout, `TTL: 86400`.
- The payload is capped at 3000 bytes by halving the body, then trimming the title. It contains `{title, body, url, tag}`, where `tag` is the notification id, so a repeat replaces rather than stacks.
- At most 20 devices per member.
- Responses:
  - 404 or 410 deletes the subscription
  - 413 counts as skipped
  - other failures retry
- `send` reports `gone` when no device is left. It throws (and so retries) only when nothing succeeded and something is retryable.
- If a browser that is already subscribed is signed in as someone else and subscribes again, the subscription moves to the new person.
- Service worker (`apps/web/src/service-worker/index.ts`):
  - always shows a notification, as Safari requires
  - accepts only in-app paths
  - on click, focuses an open window and navigates it, or opens a new one

**Telegram channel and bot** (`packages/adapters/src/telegram/service.ts`, `apps/server/src/telegram.ts`)

- **Polling.** Telegram is reached by **long polling, not a webhook**. The adapter runs its own loop calling `getUpdates` with a 25 s timeout, and calls `deleteWebhook` first.
  - Each round re-reads settings, so a token saved by the API process is picked up.
  - The offset is persisted after each update, so a restart never replays.
  - On 409 (another poller) it waits 30 s and shows "conflict". On 401 it shows "invalid_token". It honors 429 `retry_after`. Network errors back off from 2 s up to 60 s with jitter.
  - With no token, it idles in 10 s rounds.
- **Linking.**
  - The code is 8 characters from an unambiguous 31-character alphabet and lives 10 min.
  - It is single-use: whoever deletes the row owns it. Creating a new code removes the member's old ones.
  - One Telegram account maps to one member. A new link replaces either side.
  - Only private chats are handled. Unlinked senders are ignored, except a bare `/start`, which gets a hint.
  - Ten wrong codes per Telegram user in 10 min, and the bot stops answering that user.
  - A token for a different bot clears all links and codes. A token for the same bot keeps them.
- **Sending.**
  - Messages use HTML with escaped text: the title in bold, then the body.
  - Each message carries up to 5 approve and reject button pairs: `a:<draftId>` / `r:<draftId>`, within Telegram's 64-byte callback limit.
  - Buttons appear only for pending, unexpired drafts behind the notification's `source` that **this recipient may decide** (`runner.mayDecide`).
  - The "Open" URL button is added only for `https` base URLs. Otherwise the link is written into the text.
  - A 403 or "chat not found" unlinks the member and returns `gone`.
  - `isAvailableFor` requires both a connected bot and a link.
- **Pressing a button.**
  - The decision goes through the same `approveDraft` / `rejectDraft` as the app, acting as the member.
  - "Already decided" is reported for conflict or not-found.
  - Afterwards only that draft's row of buttons is removed.
- **Assistant answers.** Questions asked in Telegram are answered through the member-addressed `assistant.done` bus event, sent only to a chat waiting for exactly that reply, one question at a time. This is transport, not a notification, so quiet hours do not apply to it.

**Region** (`packages/core/src/family.ts`, `packages/time/src/index.ts`)

- `FamilySettingsSchema` validates the values: IANA time zone, canonical BCP 47 tags, `weekStart` 1 to 7.
- `regionFormats(tag)` derives the defaults:
  - week start from `Intl.Locale#getWeekInfo`
  - hour cycle from `Intl.DateTimeFormat`
  - temperature from a short list of Fahrenheit regions (US, LR, MM, BS, BZ, KY, PW, FM, MH, GU, PR, VI, AS, MP)
- `setupFormatLocale` combines the chosen language with the browser's region, e.g. Turkish in an en-US browser gives `tr-US`.
- `numericDateOrder` (DMY, MDY or YMD) tells the AI how to read "03/04". All-day dates are stored as `YYYY-MM-DD` strings and never converted to instants. Recurrence is stored as RRULE text (`packages/time`).
- `core.updateFamilySettings` emits `core.family.changed`, and other open screens refresh without a reload.

**Feature toggles** (`packages/core/src/features.ts`) are family-wide. A turned-off extension:

- keeps its data
- answers `feature_off` on its routes and actions
- leaves the AI catalog
- pauses its jobs
- loses its capabilities
- creates no notifications
- disappears from the UI and from displays (`disabledExtensions`)

**Event bus** (`packages/core/src/event-bus.ts`, `core-events.ts`)

- In-process, published after commit. Handler failures are logged and never propagated.
- Payloads carry ids only, e.g. `core.notification.created {memberId}`. The realtime hub forwards every event to every socket, displays included, and clients refetch (`apps/server/src/http/realtime.ts`).
- `publishTo(memberId)` is a private path, used for assistant answers, that reaches only that member's sockets and is never stored.

### Integrations

- **Web Push**: browser Push API with VAPID; FCM, APNs web push, Mozilla and WNS endpoints.
- **Telegram Bot API** via grammY's typed `bot.api`, without `bot.start()`. It uses long polling, so no public address or webhook is needed and it works behind NAT or Tailscale. The token is sealed with `FAMOS_SECRET`.
- **Email** is an input channel only (IMAP into Quick add). It is not a notification channel; the only channels wired are `[webPush.channel, telegram.channel]` (`apps/server/src/runtime.ts`).

### Display relevance

- Displays receive no notifications: they hold no member and no `core.notifications.receive`.
- A display can be granted `core.drafts.waiting`, which shows only a count of waiting suggestions (the Quick add status pill).
- `GET /core/displays/me` hands a display the family time zone, language, region format, week start, hour cycle, temperature unit and disabled extensions.
- Displays live-update on `core.family.changed`, feature changes and device changes.
- Region settings drive every date, time, number and temperature on the wall display. The temperature setting exists mainly for its weather widget.
- On a shared family tablet, push subscriptions follow the browser, not the session. Sign-out (`apps/web/src/lib/auth.ts`) does not unsubscribe, so the tablet keeps receiving the last subscriber's pushes until someone else subscribes on it.

### Docs vs code

- `notifications.md` "What you are told about" leaves out **board notes** (`board.posted`, to everyone who can see the board except the author). It is documented only in `board.md`.
- `notifications.md` says event reminders go to "everyone who can see the calendar when it is for nobody in particular", and similar wording for tasks and duties. It does not point out that this includes Guests by default.
- The docs say quiet hours stop sending "to phones or Telegram" and that unread items are sent at the end. The code matches: per person, family time zone, and no exemptions. Two things go unstated:
  - the test notification bypasses quiet hours
  - Telegram assistant answers are not subject to them
- `telegram.md` says the code "works once for ten minutes", which matches. It does not mention that 10 wrong codes in 10 minutes silences the bot for that Telegram user.
- `architecture.md` calls Web Push "the first channel". The code has two channels, Web Push and Telegram.

### Unfinished or stubbed

- No TODO, FIXME or stub markers were found in `notifications.ts`, `worker.ts`, `queue-sqlite`, `webpush`, `telegram` or `packages/time`.
- Not present:
  - **retention**: no pruning of `core_notifications` was found, so rows grow without bound
  - a per-notification delete
  - a digest or summary notification
  - an email notification channel
  - per-kind priority or a quiet-hours bypass for urgent kinds
  - per-person reminder times (09:00 is fixed)
  - a per-family quiet-hours default
- Reminder jobs carry instants computed in the family time zone when they were planned. No code was found that re-plans existing chore, task or duty reminders when the family time zone changes; an item is re-planned only when it is next edited. Calendar events store their own `time_zone`.
- Clicking a push notification navigates but does not mark it read on the server.

### Beyond Home HQ (net-new ideas)

- **Notifications as records, channels as copies.** Store translation keys plus parameters, and render per recipient at send or read time. This gives multilingual households and a lossless in-app history for free.
- **Resolve recipients by capability at send time** ("whoever may approve", "everyone who sees chores"), not by a static list. Add `except` for the actor.
- **Read-in-app suppresses the push.** Combined with quiet hours that *defer* instead of drop, this means nobody is pinged at night or about something they already saw.
- **Self-cleaning approval prompts.** Once a suggestion is decided, its "waiting" notices are marked read for everyone, and stale Telegram buttons are removed or refused.
- **One-tap approvals from chat**, with buttons shown only to people allowed to decide.
- **Version-stamped "next reminder only" scheduling** (`updated_at` in the payload). Edits never leave ghost reminders, and a server that was down never sends stale ones.
- **Region separate from language**, with `Intl`-derived defaults and a live preview, plus numeric-date-order hints for AI parsing.
- **Gaps Home HQ could fill:**
  - an "urgent" kind that pierces quiet hours (e.g. a school pickup change)
  - quiet hours scoped by person and by device (a wall display must stay silent, a phone may not)
  - a morning digest
  - configurable reminder anchor times
  - notification retention
  - unsubscribing push on sign-out for shared tablets

### Evidence

- `docs/guide/notifications.md`, `docs/guide/telegram.md`, `docs/guide/region.md`, `docs/guide/board.md`, `docs/guide/auto-approval.md`, `docs/architecture.md` (notifications, Telegram, reminders, expiry)
- `packages/sdk/src/notifications.ts`, `packages/sdk/src/jobs.ts`
- `packages/core/src/notifications.ts`, `draft-notices.ts`, `auto-approval.ts` (`waitingNotice`), `action-runner.ts`, `worker.ts`, `features.ts`, `settings-json.ts`, `family.ts`, `core-events.ts`, `event-bus.ts`, `tables.ts`, `migrations/0001_init.ts`, `kernel.ts`
- `packages/adapters/src/webpush/service.ts`, `crypto.ts`, `tables.ts`, `schemas.ts`
- `packages/adapters/src/telegram/service.ts`, `tables.ts`, `schemas.ts`; `apps/server/src/telegram.ts`, `apps/server/src/http/telegram.ts`, `apps/server/src/http/push.ts`
- `packages/adapters/src/queue-sqlite/index.ts`, `apps/server/src/runtime.ts`
- `packages/time/src/index.ts`
- `apps/server/src/http/realtime.ts`, `apps/server/src/http/devices.ts` (`/displays/me`)
- `apps/web/src/routes/settings/notifications/+page.svelte`, `settings/region/+page.svelte`, `apps/web/src/service-worker/index.ts`, `apps/web/src/lib/push.svelte.ts`, `apps/web/src/lib/auth.ts`
- `extensions/calendar/src/server/reminders.ts`, `extensions/lists/src/server/reminders.ts`, `extensions/chores/src/server/reminders.ts`, `extensions/records/src/server/reminders.ts`, `extensions/board/src/server/notes.ts`, `extensions/inbox/src/server/process.ts`, `packages/adapters/src/backup/jobs.ts`

---

## 14. Appearance: styles, palettes, themes and design system

FamilyOS splits "how it looks" into three independent choices.

1. **Style.** Code shipped with the app. It sets the arrangement of Today and of family displays, the type ramp (sizes of the clock, countdown, big numbers, hero title, date and sentence) and the default font per slot. There are three styles: Broadsheet (the default), Lanes and Wall. Users cannot install one. Sources: `packages/ui/src/styles.ts`, `packages/core/src/themes.ts` `STYLE_IDS`.
2. **Theme, also called the palette.** Data, never code. A theme pack is a JSON manifest plus token values for a day palette and a night palette, a corner radius, up to three font slots and optional WOFF2 fonts. Six packs are bundled, and admins can install more (`themes/*`, `packages/themes`).
3. **Day or night.** Chosen per browser: follow the system, always day, or always night (`apps/web/src/lib/theme.svelte.ts`).

The design system (`docs/design/design-system.md`) defines page templates, components, the add and edit rules, settings mode and the token contract for every page. Tests enforce part of it.

### What the user can do

**Choose a style and a theme** in Settings → You → Appearance (`apps/web/src/routes/settings/appearance/+page.svelte`).

- **Style.** The page shows three live previews of Today. Each is a made-up sample day (`apps/web/src/lib/today/StylePreview.svelte`, `sample.ts`) drawn in the person's current colours.
- **Colours.**
  - A segmented control for "Day and night on this device": Like the system, Day or Night.
  - A sample of real components.
  - Every theme as a card showing its day and night swatches.
  - What is in use on this device is marked "In use".
- **Use for me** saves the choice to the person. It follows them to every device (`core.setMyStyle`, `core.setMyTheme`). **Use the family style** clears the person's own choice.
- **Try.** Previews a style and/or a theme across the whole app without saving it.
  - A bar at the foot of every page says what is being tried, for example "Trying Lanes · Forest", with Use for me, Keep looking and Stop (`apps/web/src/lib/PreviewBar.svelte`).
  - Trying a style opens Today.
  - The trial lives in memory only. It ends on reload, sign-out or when another person signs in.

**Set family defaults** in Settings → Family → Theme (`apps/web/src/routes/settings/theme/+page.svelte`). This needs `core.settings.manage`.

- Family style and the default theme (`core.setFamilyStyle`, `core.setFamilyTheme`). The sign-in page uses the family theme.
- Family "What Today shows" (`core.setFamilyTodayBlocks`).

**Set looks per display** on the same page, under Displays. This needs `core.devices.manage`.

- Each display gets a Style, a Theme and a "What Today shows", or "Same as the family".
- Actions: `core.setDisplayStyle`, `core.setDisplayTheme`, `core.setDisplayTodayBlocks`.

**Choose what Today shows.**

- Every built-in block and every `dashboard.card` contribution has a switch: clock, greeting, weather, photo, next up, the day, where everyone is, chores, Needs you, meals, next days, done today, and the extension cards.
- The person picks Follow the family or My own choice (`core.setMyTodayBlocks`).
- On Today, a block's **⋯** menu offers "Hide from Today", with an Undo toast.
- If every block is hidden, Today shows an empty state with one button, "Choose what Today shows".
- The choice is show or hide only. The order and arrangement belong to the style (`apps/web/src/lib/today/blocks.ts`, `TodayBlocksList.svelte`).

**Add a theme** from Settings → Family → Theme → Add a theme. This needs `core.settings.manage`.

- **From a file:** one `.json` file containing `manifest`, `tokens` and base64 `fonts`.
- **From an address:** an `https` URL. The server fetches it in the background through the job `core.fetchThemePack`, using `ctx.http` with a 3 MB cap.
- A refused pack lists its reasons, for example "Contrast: ink on wall (day) is 1.6:1, needs 4.5:1".
- Installing a pack with a bundled pack's id replaces that pack. Removing it brings the bundled one back.
- People and displays that used a removed theme fall back to the family default.
- Installed packs live in `<data>/themes/<id>/`, with at most 40 installed (`apps/server/src/themes.ts`), and are included in backups.

**Make a theme.** Create the folder `themes/<id>/` with `manifest.json` (id, name, version, `famos` semver range, authors, optional `fonts`, optional suggested `style`), `tokens.json` and `.woff2` files, then check it with `pnpm theme:check <dir>` (`scripts/theme.ts`).

### Data model

**Server: `core_settings` JSON keys** (`packages/core/src/themes.ts`)

| Key | Shape | Notes |
|---|---|---|
| `themes.installed` | `{ [id]: { installedAt, url } }` | |
| `themes.choices` | `{ family: themeId \| null, members: { memberId: themeId }, displays: { deviceId: themeId } }` | |
| `styles.choices` | Same shape, with style ids | |
| `today.hidden` | `{ family, members, displays }`, each a list of block ids, at most 100 | Matches `/^[a-z][a-zA-Z0-9._-]{0,79}$/`. Only hidden blocks are stored, so a new block shows by default. |
| `themes.fetch` | `{ status: fetching \| done \| failed, url, error }` | State of the last install from an address. |

Missing or unknown ids fall back, so a stored choice can never break the app.

**Resolution, first match wins** (`createThemes`)

- **Theme:** the display's own, then the member's own, then the family's, then `ev-isigi`.
- **Style:** the display's own, then the member's own, then the family's, then the style the resolved theme's manifest suggests (used only while the family has never chosen a style), then `broadsheet`.
- **Hidden blocks:** the display's own, then the member's own, then the family's.

**Routes and events**

- `GET /core/appearance` returns the caller's resolved theme, version, style and hidden blocks, plus the family and own values.
- `GET /core/themes`, `GET /core/themes/{id}/stylesheet.css` and the font route are `public`, because the sign-in page already wears the family theme.
  - The stylesheet's ETag is the pack version plus a content hash.
  - The font route serves `font/woff2` from the same origin.
- `GET /core/themes/displays`, `/core/styles/displays` and `/core/today/displays` list the per-display choices.
- The event `core.themes.changed` makes open screens reload their look.

**Device: `localStorage`**

| Key | Contents |
|---|---|
| `famos.mode` | `system`, `day` or `night` |
| `famos.theme` | `{ id, version, css }`, the last generated stylesheet |
| `famos.style` | The last style id |

All three are applied before first paint by the inline script in `apps/web/src/app.html`: the `.dark` class, the `<style id="famos-theme">` element and `data-style` on `<html>`.

**Pack contract** (`packages/themes/src/tokens.ts`, `pack.ts`)

- **30 per-palette tokens** (`MODE_TOKENS`), set separately for day and night:
  - surfaces and text: `wall`, `surface`, `idle`, `ink`, `ink-soft`, `line`;
  - state: `attention`, `now`, `sun`, `info`, `heat`, `on-color`, `destructive`;
  - the hero: `accent`, `on-accent`;
  - photos: `scrim`, `backdrop`;
  - `shadow-lit`;
  - the board: `board`, `board-grain`, `note-1` … `note-5`, `note-ink`, `note-ink-soft`, `note-tape`;
  - the weather sky: `sky-cloud`, `sky-flake`.
- **4 shared tokens** (`SHARED_TOKENS`): `radius` (0 to 2rem), `font`, `font-display`, `font-serif`.
- **Limits:**
  - at most 1.5 MB decoded per pack;
  - fonts must be WOFF2, at most 400 KB each and at most 8 per pack;
  - `famos` is a semver range checked against the app version.
- **Colour formats:** `#rgb`, `#rrggbb`, `rgb()` and `oklch()`, converted to sRGB. Unknown tokens produce a warning and are ignored. Tokens a pack leaves out are inherited from Ev Işığı.
- **Bundled fonts**, all SIL OFL, latin and latin-ext subsets only, served from the app's own origin:

  | Font | Role it plays |
  |---|---|
  | Figtree (variable) | Body |
  | Barlow Condensed | Broadsheet display face |
  | Instrument Serif | Broadsheet serif |
  | Archivo (variable, width 62–125 %) | Lanes display face |

**Bundled packs** (`themes/*/manifest.json`)

| Pack | Notes |
|---|---|
| `ev-isigi` | Default. Mirrored in `packages/ui/src/theme.css`; a test keeps them equal. |
| `coast` | |
| `forest` | |
| `plum` | |
| `paper` | Suggests the Wall style. |
| `high-contrast` | Suggests the Wall style. AAA text contrast; `shadow-lit: 0 0 0 1px` turns tiles into outlined boxes. |

**Styles** (`packages/ui/src/styles.ts`, type ramp in `packages/ui/src/styles.css` under `[data-style]`)

| Style | Display font | Serif font | Body font |
|---|---|---|---|
| Broadsheet | Barlow Condensed | Instrument Serif | Figtree |
| Lanes | Archivo (80 % width) | none (falls back to the body face) | Figtree |
| Wall | Figtree | Figtree | Figtree |

Other pages take only the style's type roles from `packages/ui/src/type.css`: `type-page`, `type-record`, `type-figure`, `type-day`, `type-greeting`, `type-dateline`.

### Rules and logic worth copying

**A style is code; a palette is data.**

- A pack can change colours, the corner radius and fonts.
- A pack can never change sizes, arrangement or controls. The type ramp is deliberately not a token, so a pack cannot make text unreadable or hide anything (`docs/architecture.md`, appearance section).
- New arrangements need reviewed code.

**Packs carry no CSS** (`renderStylesheet`, `packages/themes/src/pack.ts`)

- The server writes the stylesheet from validated, canonical values.
- Its selectors (`:root`/`.day`, `html.dark`) are one step more specific than the app's own, so the theme always wins.
- `@font-face` rules point at same-origin URLs.
- Because no pack text reaches the browser, a pack cannot load remote resources, track users, hide approval warnings or inject script.

**The contrast gate** (`CONTRAST_RULES`, WCAG 2.2, checked in both palettes). Every pair needs at least 4.5:1:

- `ink` and `ink-soft` on `wall`, `surface` and `idle`;
- `ink` and `ink-soft` on `board`;
- `note-ink` and `note-ink-soft` on each of the five note papers;
- `on-color` on `attention`, `now` and `info`;
- `on-accent` on `accent` (the hero pill uses the inverse pair);
- `attention`, `now`, `info` and `destructive` used as text on `wall` and `surface`.

`sun` and `heat` only colour icons next to a text label, so they have no rule.

- Only the pairs a pack sets itself are checked (`contrastProblems` with `explicit`). Pairs it inherits are the default theme's responsibility.
- Installed packs must pass. Bundled packs are loaded with contrast shortfalls reported as warnings only, because Ev Işığı predates the rule (`apps/server/src/themes.ts`).
- Extra tests cover:
  - text on the animated weather sky in every pack, day and night (`scripts/sky-contrast.test.ts`);
  - tinted state pills (`packages/themes/test/pill-contrast.test.ts`);
  - the member colour palette (`scripts/member-colors.test.ts`);
  - hard-coded colours anywhere in the app (`scripts/theme-colors.test.ts`).

**Colour semantics**

- State colours (`attention`, `now`, `sun`, `info`, `heat`) describe state only and are never assigned to people.
- Member colours (`MEMBER_COLORS` in `packages/core/src/core-extension.ts`) are assigned in order and kept away from coral and red, so a person never reads as a warning.
- Letters on a member colour are white or the day ink, whichever contrasts more (`onMemberColor`).
- Member colours are day colours, drawn inside `.day`.
- `accent` fills at most one surface per page (the next-up hero) and is never used as text.
- Tints use `color-mix(in oklab, state 15%, transparent)`. Text on a tint is the state colour mixed toward `ink`, never the pure state colour.

**No flash on load.**

- The last style, stylesheet and mode are applied before first paint from `localStorage`.
- The stylesheet is revalidated by ETag, so a 304 costs almost nothing.
- `meta[name=theme-color]` follows `--wall`, so the browser chrome matches the theme.

**Day and night**

- Night is the `.dark` class on `<html>`.
- "system" follows `prefers-color-scheme` live through a `matchMedia` change listener.
- The choice is per browser, not per person.
- Every theme has both palettes.
- `.day` pins the day palette inside night mode. Two uses: QR codes, and white text over photos with `--scrim`.

**Design-system rules** (summary of `docs/design/design-system.md`)

- **Principles**
  - Lit (a shadow) is rare, and a saturated surface appears once.
  - There are three surfaces and they never nest:
    - *tile*: `surface` with a shadow;
    - *quiet row*: `idle`;
    - *card*: `surface` with a 1 px line, holding rows.
  - Sections are uppercase labels, not boxes.
  - State is said once.
  - Width decides the layout, never the device; touch targets are 44 px.
  - Undo is preferred over confirm.
  - Updates are instant, then honest: optimistic, with rollback and a stated reason.
- **Breakpoints and shells**

  | Width | Shell |
  |---|---|
  | Under 768 px (phone) | Bottom tabs: Today · Calendar · Lists · Quick add · More |
  | 768–1023 px | 80 px rail |
  | 1024 px and up | 248 px sidebar |
  | 1280 px and up | Content max 1200 px |
  | `/display` | Today in family mode, no navigation, never scrolled, one type step larger |

- **Page templates.** Every page renders inside `<Page template=…>` and never sets its own width or centring; `scripts/page-frame.test.ts` enforces this. There are four width classes:

  | Class | Width | Used by |
  |---|---|---|
  | Narrow | 760 px | `chat`, `capture` |
  | Medium | 60 rem; 76 rem with paired settings sections | `settings` |
  | Standard | 1200 px | `list`, `list-detail`, `record` |
  | Full | Whole content area | `day` (Today and the display), `calendar`, `board` |

- **Today's style compositions at each width**

  | Width | Broadsheet | Wall | Lanes |
  |---|---|---|---|
  | 1280 px and up | Three columns, `280px 1fr 300px`: atmosphere / the day / what needs doing | Two columns, `400px 1fr`: atmosphere / the day | Head, departure-board hero with "Then", the day chart 07:00–22:00 (at most six person lanes, then "Others", plus a family lane, with a now-line and hourly weather on the axis), then a band |
  | 1024–1279 px | Atmosphere as a band on top | Clock and weather share a band | — |
  | Under 1024 px | One column | One column | One column; the chart turns upright (at most 5 person columns, then "Others") |

- **Adding and editing**
  - One primary (filled `ink`) button per view.
  - A form opens in a side panel of 420 px (360 px at 1024–1279) on wide windows, and in a full-height sheet below 1024 px.
  - Short picks use a menu, or a half sheet on phones.
  - Rows open on tap. Secondary actions go in a `⋯` menu, or an action sheet on phones. No pencil icons, no hover-only controls.
  - Destructive actions need two taps within 4 s. Completing an action is one tap plus an Undo toast that lasts 6 s.
  - Opening a panel adds `?edit=<id>` or `?new=1` to the URL.
  - Multi-step flows that may happen on another device use guided `Steps`. Device pairing uses the `large` variant on the wall.
- **Loading** ("a page opens once, whole")
  - `PageGate` keeps the page transparent until its first reads finish, then fades it in over 120 ms.
  - A quiet indicator appears after 800 ms.
  - After 6 s the page shows as it is.
  - An empty state is shown only once a load has proved emptiness.
- **Settings mode**
  - At 1024 px and up, the settings list replaces the app sidebar: "‹ Back to FamilyOS", a search field and four groups.
  - Below 1024 px, settings is a stack of pages under More.
  - The groups are You (follows the person), Family (managers), AI and System.
  - Setting rows put the label on the left and the control on the right, and save as you go.
- **Typography**
  - Weights are 400, 600, 700 and 800, with tabular digits everywhere.
  - Rows are 48 or 56 px; controls are 40 px with a mouse and 44 px on touch.
  - Base sizes: meta 13 px, body 15 px, labels 12 px uppercase.
  - Three voices: body, display (clock, countdowns, big numbers) and serif (date, hero title, weather sentence).
  - Ramp examples at 1440 px:

    | Role | Broadsheet | Lanes | Wall |
    |---|---|---|---|
    | Clock | 124 | 92 | 150 |
    | Countdown | 128 | 84 | 96 |

- **Motion**
  - Hover transitions 150 ms; panels slide in 200 ms; sheets 240 ms.
  - Everything is removed under `prefers-reduced-motion`.
  - The weather sky is the one continuous animation. It uses CSS keyframes on `transform` and `opacity` only, with at most 11 animated elements.
- **Accessibility**
  - Focus rings use `info`.
  - Every row with an action is a `button` or an `a`.
  - Colour is never the only signal.

### Integrations

- **Install from an address.** The `core.installThemePackFromUrl` action takes an https URL only and enqueues the job `core.fetchThemePack`. The job uses the guarded `ctx.http` port: no private addresses, a size cap and a timeout. A pack that fails validation fails for good, with no retry.
- **Fonts.** All fonts are served from the app's own origin. No Google Fonts, no CDN.
- **No other external services.**

### Docs vs code

- **"Family displays follow the sun" is not implemented.**
  - The claim appears in `docs/guide/appearance.md`, in `docs/design/design-system.md` principle 12 ("displays switch to night at sunset"), in the `devices_display_theme_hint` UI string and in a `theme.css` comment.
  - In code, `/display` uses the same per-browser mode as every page (`followDayNight` in `apps/web/src/lib/theme.svelte.ts`): `famos.mode`, defaulting to the OS colour scheme. Nothing on the display lets anyone change it.
- **The 52 px kiosk control size is not applied.** The design system (§3.1, §9) promises `--ctl: 52px` and 52 px targets on displays. `[data-ctl="kiosk"]` is defined in `packages/ui/src/theme.css` but never set, and the button's `kiosk` size is unused.
- **"The bundled packs pass every rule"** (appearance guide). The server registry treats bundled-pack contrast shortfalls as warnings only, because Ev Işığı predates the rule (`apps/server/src/themes.ts`). Whether any bundled pack actually falls short was not checked here.
- **Matches between docs and code:**
  - the token list (30 per palette plus 4 shared);
  - the size limits;
  - the resolution order;
  - the style fonts;
  - six bundled themes, of which Paper and High contrast suggest Wall.

### Unfinished or stubbed

- **No scheduled or automatic night switch** beyond the OS setting: no sunset-based switch and no time schedule.
- **Day/night is per browser only.** It cannot be set for a person, for the family or remotely for a display, and a display has no UI to change it.
- **No installable styles, by design.** `compositions.ts` falls back to Broadsheet for "a style without its own composition yet". All three styles have one today.
- **Font subsets are latin and latin-ext only.** Other scripts use the device's fonts.
- **Settings search filters list rows only.** Searching setting labels is noted as "later" (design-system §7.3).

### Beyond Home HQ (net-new ideas)

- **Style versus palette as separate axes.** Arrangement and type are reviewed code; colours are a data pack. Any style works with any palette.
- **Three-level choice with a clear fallback chain** (display, then person, then family, then the theme's suggestion, then the default), plus a per-display look and per-display hidden blocks for wall screens.
- **Installable theme packs as validated data:**
  - no CSS, a server-generated stylesheet;
  - a WCAG contrast gate in both palettes, with readable refusal reasons;
  - semver compatibility;
  - optional WOFF2 fonts served from the same origin;
  - install from file or URL as a background job;
  - included in backups.
- **Pre-paint application of mode, theme and style** from `localStorage`, plus an ETag-revalidated stylesheet, so there is no flash of the default look on wall tablets.
- **Try before you choose.** A whole-app preview with a persistent "Trying X · Y" bar (Use for me / Keep looking / Stop) that is never saved and ends on reload.
- **"What Today shows"** with show/hide per block, stored as a hidden list so new blocks appear by default, and hiding from the block itself with Undo.
- **Token rules worth adopting:**
  - one accent surface per page;
  - state colours never used for people;
  - member colours tuned for 4.5:1 initials;
  - tinted text mixed toward `ink`;
  - a test that fails on any hard-coded colour.
- **Bundled high-contrast pack** (AAA text, outlined cards) aimed at bright rooms and low vision.

### Evidence

- Styles: `packages/ui/src/styles.ts`, `packages/ui/src/styles.css`, `packages/ui/src/type.css`, `apps/web/src/lib/today/compositions.ts`
- Tokens and baseline: `packages/ui/src/theme.css`, `themes/ev-isigi/tokens.json`, `themes/*/manifest.json`
- Pack contract: `packages/themes/src/tokens.ts` (`MODE_TOKENS`, `SHARED_TOKENS`, `CONTRAST_RULES`, `BUNDLED_FONTS`), `packages/themes/src/pack.ts` (`validateThemePack`, `contrastProblems`, `renderStylesheet`, limits), `packages/themes/src/color.ts`
- Server registry: `apps/server/src/themes.ts` (`createThemeRegistry`, `MAX_INSTALLED = 40`)
- Core themes: `packages/core/src/themes.ts` (choices, resolution, routes, actions `core.set{Family,My,Display}{Theme,Style,TodayBlocks}`, `core.installThemePack`, `core.installThemePackFromUrl`, `core.removeThemePack`)
- Web: `apps/web/src/lib/appearance.svelte.ts`, `apps/web/src/lib/theme.svelte.ts`, `apps/web/src/app.html`, `apps/web/src/lib/PreviewBar.svelte`, `apps/web/src/routes/settings/appearance/+page.svelte`, `apps/web/src/routes/settings/theme/+page.svelte`, `apps/web/src/lib/today/blocks.ts`, `apps/web/src/lib/today/TodayBlocksList.svelte`, `apps/web/src/lib/today/StylePreview.svelte`
- Member colours: `packages/core/src/core-extension.ts` (`MEMBER_COLORS`)
- Tests named in docs: `scripts/theme-colors.test.ts`, `scripts/sky-contrast.test.ts`, `scripts/member-colors.test.ts`, `scripts/page-frame.test.ts`, `packages/themes/test/pill-contrast.test.ts`
- Docs: `docs/guide/appearance.md`, `docs/design/design-system.md`, `docs/architecture.md` (appearance: styles and palettes)

---

## 15. Platform: extensions and packs, real-time sync, offline, i18n, backups, jobs

This section covers the cross-cutting machinery of FamilyOS 0.1.0. Paths are relative to the repository root. Where `docs/architecture.md` and the code disagree, the "Docs vs code" notes say so.

### (a) Extension and pack system

FamilyOS has two kinds of add-ons and treats them very differently. Extensions are code, compiled into the app at build time. Language packs and theme packs are data, validated and installable at runtime by an admin. `docs/architecture.md` §1 states the reason: "There is no runtime installation of code and no marketplace, because there is no sandbox for third-party code."

#### What the user or admin can do

- Turn a whole extension off or on in Settings → Family → Features. The action is `core.setFeatureEnabled`, gated by `core.features.manage`, an admin-only critical capability. Data stays. Core cannot be turned off.
- Install a language pack or a theme pack from one `.json` file, or from an `https` address that the server fetches in the background. Remove an installed pack. Both need `core.settings.manage`.
- Choose the family theme, a personal theme and a per-display theme. The same three levels exist for the style (Broadsheet, Lanes, Wall) and for which Today blocks are hidden: `core.setFamilyTheme`, `core.setMyTheme`, `core.setDisplayTheme`, `core.setFamilyStyle`, `core.setMyStyle`, `core.setDisplayStyle`, `core.setFamilyTodayBlocks`, `core.setMyTodayBlocks` and `core.setDisplayTodayBlocks`, all in `packages/core/src/themes.ts`.
- Nobody can install an extension, a style or any code at runtime. Adding an extension means a pull request, a new entry in `apps/server/src/extensions.ts` and a rebuild of the image.

#### How an extension is declared on the server

An extension is one workspace package under `extensions/<id>/` with three subpath exports: `./contract` with Zod schemas and types anyone may import, `./server` imported only by `apps/server`, and `./web` imported only by `apps/web`. See `extensions/lists/package.json` and `extensions/README.md`.

`defineExtension` in `packages/sdk/src/extension.ts` takes:

| Field | Meaning |
|---|---|
| `id` | Lowercase letters and digits, regex `^[a-z][a-z0-9]*$`. Also the table prefix and the API path segment. |
| `dependsOn` | Extensions whose migrations must run first. |
| `migrations` | `Record<name, { up(db) }>`, forward-only, names sorted lexicographically (`0001_init`). |
| `actions`, `routes`, `jobs`, `events` | Built with `defineAction`, `defineRoute`, `defineJob`, `defineEvent`. |
| `capabilities` | Built with `defineCapability`: id, label key, level `view`/`use`/`edit`/`manage`, optional restrictive `defaults`. |
| `notifications` | Built with `defineNotificationKind`: id, label, `defaultChannels`, optional `audience` capability. |
| `sensitive` | AI access for this area starts Off instead of Any provider. |
| `ai.context` | Id(s) of read-only actions that list records for AI prompts as handles. |
| `ai.index` | Kinds of entity that family search embeds, with `load(ctx, id)` and a paged `list`. |
| `register(ctx)` | Runs once after migrations: event subscriptions, scheduling the first run of a recurring job. |

`defineExtension` refuses anything that is not namespaced as `<id>.…`: actions, route operation ids, jobs, events, notification kinds, capabilities, AI context actions and index kinds. A real example is `extensions/board/src/server/index.ts`, which declares 8 actions, 3 routes, 2 jobs, 1 event, 3 capabilities and 1 notification kind, and calls `scheduleCleanup(ctx)` in `register`.

`defineAction` in `packages/sdk/src/actions.ts` is the unit of change. One definition becomes an HTTP endpoint at `POST /api/v1/actions/<id>`, a draft type for AI proposals, and a tool for the assistant and MCP clients when it carries `ai` hints. Its fields:

- `id` must match `<extension>.<verbNoun>`. `input` and `output` are Zod schemas.
- `readOnly`. The default `approval` is `'never'` for read-only actions and `'ai'` otherwise.
- `requires` names a capability. `approval` is one of `'never'`, `'always'`, `'ai'`, `{ unless: capability, except? }` or a function.
- `ai` hints carry a model-facing description and guidance, plus `assistantOnly` and `addressToMember`.
- `external: true` marks an action that sends data out of the household.
- `autoApprove` opts the action into the family's auto-approval setting.
- `offline: true` marks an action the web app may queue. See (c).
- `undo(output)` returns the inverse action and its input.
- Review helpers, all read-only: `alreadyThere`, `targetTitle`, `refuseProposal`, `resolveNames`, `settled`, `draftDate`.
- `prepare(input, actor)` does slow work outside the transaction. `run(uow, input, prepared)` runs inside one SQLite transaction.

The action runner applies these steps in order: input validation, the access check, `prepare`, the approval policy (apply now or store a draft), `run()` in one transaction, the audit log, then events after commit (`docs/architecture.md` §7).

`defineRoute` in `packages/sdk/src/routes.ts` declares method, path, `operationId`, `requires` (`public`, `trusted-device`, `signedIn` or a capability id), Zod `params`/`query`/`body`, an optional `upload` spec, a `response` schema and `handle(ctx, request)`. Routes serve under `/api/v1/<extension>`. Two special response schemas exist. `TextResponseSchema` sends text with its own content type and ETag, which is how the ICS feed works. `FileResponseSchema` streams a stored file. Extensions never import Hono; `apps/server/src/http/app.ts` turns every route and action into `@hono/zod-openapi` routes and `openapi.json`.

`defineJob` in `packages/sdk/src/jobs.ts` declares `name` as `<extension>.<step>`, a Zod `payload`, `timeoutMs` (default 60 s), `maxAttempts` (default 5), `lane` (`default` or `long`) and `backoff` (default `min(2^n s, 15 min)`). Only job handlers receive `ctx.llm`.

`defineEvent(name, payload)` in `packages/sdk/src/events.ts` declares an in-process event. Payloads carry ids only.

What an extension gets at runtime is defined in `packages/sdk/src/context.ts`:

- `ExtensionContext`, for routes, jobs and `register`, has `db`, `clock`, `ids`, `logger`, `files`, `secrets`, `http` (guarded outbound GET), `events`, `actions` (invoke, `aiCatalog`, `aiReadTools`), `family` (settings and members), `translate`, `vocabulary`, `access`, `ai` (search reindex), `notify` and `enqueue`.
- `UnitOfWork`, inside an action's transaction, has the same minus network access, plus `actor`, `can()`, `emit()`, `run(actionId, input)` to call another extension's action in the same transaction, and `rejectDraftsOf(source)`.

#### Registry checks at startup

`buildRegistry` in `packages/core/src/registry.ts` orders extensions with a depth-first topological sort and rejects cycles and missing dependencies. It rejects duplicate actions, jobs, events, operation ids, capabilities and notification kinds across all extensions. It also refuses:

- a capability, `requires` or `approval.unless` that names an unknown capability, so a typo fails at startup instead of allowing everyone;
- a non-read-only action that declares neither `requires` nor `approval: { unless }`;
- an action with `ai` hints and `approval: 'never'`, which would let the AI change data directly;
- an action with both `ai` hints and `external: true`;
- `autoApprove` on read-only, external, core or sensitive-extension actions;
- an `ai.context` action that is not the extension's own or is not read-only;
- capability `defaults` that would add a grant beyond the level's defaults (`assertRestrictiveDefaults` in `packages/sdk/src/capabilities.ts`).

#### How the server wires extensions

`apps/server/src/extensions.ts` imports the 12 bundled extensions statically: calendar, chores, lists, inbox, records, home, pets, vehicles, kitchen, board, ambient and assistant. The kernel always adds core, which is itself an extension (`packages/core/src/core-extension.ts`). `apps/server/src/runtime.ts` is the composition root. It also registers infrastructure migration scopes that do not belong to any extension (`queue`, `identity`, `ai`, `webpush`, `telegram`, `email`, `backup`) and infrastructure jobs for email polling and backups. Migration order is core, then infrastructure, then extensions in dependency order (`packages/core/src/kernel.ts`).

When an extension is turned off, `packages/core/src/features.ts` stores `{ disabled: [...] }` in `core_settings` under key `features` and caches it in memory. Routes and actions then answer `feature_off`, the worker skips the extension's jobs and leaves them queued, and the `gatedEvents` wrapper in `kernel.ts` stops its event handlers. Its capabilities go inactive and the web app hides its nav, pages and cards using `me.disabledExtensions`. Ownership comes from the id prefix (`ownerOf('calendar.createEvent')` returns `calendar`), so core can stop an extension without its help.

#### How the web app takes an extension's UI

`defineWebExtension` in `packages/ui/src/slots.ts` takes `id`, `labelKey`, `descriptionKey`, `aiFeatures`, `slots`, `timeline`, `nav`, `drafts` and `pages`. All contribution ids must be namespaced with the extension id. Draft presenters must name the extension's own actions; the assistant is the one extension allowed to present `core.*` actions.

The typed slots are `dashboard.card`, `today.head`, `today.atmosphere`, `today.capture`, `nav.action`, `display.widget`, `display.status`, `display.ambient`, `record.section`, `inbox.fileTarget`, `setup.step` and `settings.display`. A slot contribution is plain data (`id`, `order`, `requires`, `band`, `size`, label keys) plus a lazy component loader. `timeline` contributions feed Today's day view with a `load(day)` function and an `invalidateOn` list of event-name prefixes.

`apps/web/src/lib/extensions.ts` collects every `extensions/*/src/web/index.ts` at build time with `import.meta.glob(..., { eager: true })`. The comment there says it plainly: "Extensions are first-party workspace packages, so a glob over the monorepo is enough; no runtime loading." Pages are served by the catch-all route `apps/web/src/routes/[...path]`, which matches exact paths first and then `:param` patterns such as `/vehicles/:id`.

Extension components never import app modules. They get services from `useApp()` in `packages/ui/src/app-context.ts`: `t`, `locale`, `formatLocale`, `weekStart`, `hourCycle`, `temperatureUnit`, `members`, `me`, `can`, `timeZone`, `onEvent`, `draftPresenter`, `slots`, `toast`, `runAction`, `waiting`, `offline`, `query` and `setQuery`.

#### Packs installed at runtime

Language packs are covered in (d). Theme packs work the same way:

- The format is a folder `themes/<id>/` with `manifest.json`, `tokens.json` (day and night token values, radius, fonts) and optional `.woff2` files. For install, the whole pack is one JSON document with fonts as base64.
- `validateThemePack` in `packages/themes/src/pack.ts` caps a pack at 1.5 MB, fonts at 400 KB each and 8 font files. Only WOFF2 is accepted. Colors are parsed (hex, `rgb()`, `oklch()`) and checked against WCAG contrast pairs. Installed packs are validated strictly, so a contrast failure refuses the pack and lists the failing pairs. Bundled packs only get warnings. Unknown tokens are ignored, and missing tokens fall back to the default pack `ev-isigi`.
- The server never passes pack text to the browser. It regenerates the stylesheet from parsed numbers. A pack may contain no CSS, selectors or scripts.
- `apps/server/src/themes.ts` allows at most 40 installed theme packs. `apps/server/src/languages.ts` has the same limit for language packs.
- Installed packs live in `<data>/themes/<id>/` and `<data>/languages/<code>/`. An installed pack overrides a bundled one with the same id. Backups include both folders.
- "Install from an address" never fetches inside the HTTP request. `core.installLanguagePackFromUrl` and `core.installThemePackFromUrl` only enqueue the jobs `core.fetchLanguagePack` and `core.fetchThemePack`, which use the guarded HTTP port (https only, no private addresses, size cap). Progress is stored in `core_settings` under `languages.fetch` and `themes.fetch`.
- The server raises the body limit for the two install actions in `apps/server/src/http/app.ts`, because packs exceed the default.

#### Data model

- `_migrations(scope, name, applied_at)`, primary key `(scope, name)`, in `packages/db/src/migrator.ts`. Each extension and infrastructure adapter is its own scope.
- `core_settings(key, value TEXT JSON, updated_at)` holds most platform state as small JSON values. Keys seen in `packages/core/src`: `family`, `features`, `ai_areas`, `languages`, `languages.fetch`, `themes.installed`, `themes.choices`, `themes.fetch`, `styles.choices`, `today.hidden`, `today.nextUp`, `core.autoApproval`, `mcp`, `search_index`.
- Extension tables are prefixed with the extension id and created by that extension's own migrations. Table types are written by hand next to the migration (`packages/core/src/tables.ts` for core). All tables are STRICT. There are no foreign keys across extensions; references are typed ids such as `vehicle_<uuidv7>`.
- Packs are files on disk, not rows. Only the list of installed packs and the family's choices sit in `core_settings`.

#### Rules and logic worth copying

- One action definition drives HTTP, drafts and AI tools. Permissions and approval live in the runner, not in route handlers.
- Everything an extension owns is namespaced by its id. That one rule gives turn-off, ownership lookup and collision checks for free.
- The registry fails fast at startup on unknown capabilities, unsafe AI hints and missing approval declarations.
- Extension UI goes through a fixed set of typed slots with lazy loaders, and the host filters contributions by `requires` and by turned-off extensions.
- Runtime-installable add-ons are data only, validated field by field, and re-rendered by the server. Code add-ons are build-time only.
- When an extension is removed from the build, `planMigrations` reports its scope as orphaned and keeps its data instead of refusing to start (`packages/db/src/migrator.ts`).

#### Display relevance

- Wall displays only render `display.widget`, `display.status` and `display.ambient` contributions from extensions that are on and whose `requires` the display was granted (`apps/web/src/routes/display/+page.svelte`, function `visible`).
- Each display can have its own theme, style and hidden Today blocks.
- Extensions themselves cannot be added to a display at runtime. New widget types need a build.

#### Docs vs code

- `docs/architecture.md` §6 shows `events: { 'shopping.item.added': ItemAdded }` as an object. The real `defineExtension` takes an array of `defineEvent` results.
- The architecture example omits `jobs` and `notifications`, which every real extension with reminders uses.

#### Evidence

`packages/sdk/src/{extension,actions,routes,jobs,events,capabilities,notifications,context,ports}.ts`, `packages/core/src/{registry,kernel,features,core-extension,themes,languages}.ts`, `apps/server/src/{extensions,runtime}.ts`, `apps/server/src/http/app.ts`, `packages/ui/src/{slots,app-context}.ts`, `apps/web/src/lib/extensions.ts`, `packages/themes/src/pack.ts`, `apps/server/src/themes.ts`, `extensions/board/src/server/index.ts`, `extensions/README.md`.

### (b) Real-time sync

#### How it works

- The server is the source of truth. Clients never receive data over the socket, only "something changed" notices. They refetch.
- The event bus is in-process (`packages/core/src/event-bus.ts`). Actions call `uow.emit`, and the runner publishes only after the transaction commits. Jobs call `ctx.events.emit` because their writes are already committed. Handler errors are logged and never reach the publisher. Payloads are validated against the event's Zod schema on emit.
- There is one WebSocket at `/api/v1/ws` (`apps/server/src/main.ts`), served by `@hono/node-server`'s `upgradeWebSocket` with the `ws` library. The upgrade runs the same `resolveActor` as the API, so archived members, stale PIN sessions and revoked displays get 401.
- The hub in `apps/server/src/http/realtime.ts` subscribes to `bus.onAny` and sends `{ type: 'event', name, payload }` to every open socket. On connect it sends `{ type: 'hello' }`.
- Private events use `bus.publishTo(memberId, ...)` (SDK `ctx.events.emitTo`). The hub delivers them only to that member's sockets, never to displays, and `onAny` never sees them. The assistant streams its answers this way with `assistant.delta` and `assistant.done`.
- On `core.device.changed` the hub closes that display's sockets with code 4401. The display reconnects and is authorized again, so a revoked display loses access at once and a changed one gets its new grants.

#### Core events

`packages/core/src/core-events.ts` declares `core.draft.created`, `core.draft.updated`, `core.member.changed`, `core.role.changed`, `core.device.changed`, `core.features.changed`, `core.aiAreas.changed`, `core.notification.created` and `core.notification.read`. Language and theme code adds `core.languages.changed`, `core.family.changed` and `core.themes.changed`. Each extension declares its own, such as `board.changed`. Telegram and email status changes are published as `telegram.status.changed`, `telegram.link.changed` and `email.status.changed` straight from `runtime.ts`.

#### How the web app refreshes

- `apps/web/src/lib/realtime.ts` opens one socket. It reconnects with exponential backoff from 1 s to 30 s. Subscribers filter by name prefix with `onEvent('calendar.', handler)`.
- Every incoming event calls `forgetSharedReads()`, so a read that started before the change cannot answer a component that reloads because of it.
- After a reconnect, though not on the first connect, `onReconnect` handlers run so pages catch up on events they missed while the socket was closed, for example a phone waking up.
- `apps/web/src/routes/+layout.svelte` refreshes session, family settings, languages, themes and features on their `core.*` prefixes. It also refreshes nav badges after any event, debounced by 400 ms.
- Today's timeline contributions declare `invalidateOn` prefixes (`apps/web/src/lib/today/today-data.svelte.ts`).

#### Rules and logic worth copying

- Emit after commit, ids only, then refetch. Clients need no merge logic and no versioning on the socket.
- Resync on reconnect, but skip it on the first connect, which already reads fresh data.
- A per-member private channel exists for streamed or private content, kept separate from the broadcast channel.
- Revoking a device closes its socket on the server side.

#### Display relevance

- The display page (`apps/web/src/routes/display/+page.svelte`) connects the same socket and listens to `core.device.`, `core.member.`, `core.features.`, `core.themes.`, `core.family.` and `core.languages.`.
- It does not trust the socket alone. Every 60 s it re-checks `GET /api/v1/core/displays/me`, and every 5 minutes it re-reads Today and the widgets "as a safety net for missed events".
- There is no heartbeat or ping on the WebSocket, neither server nor client side. A half-open connection after a Wi-Fi drop without a close frame would go silent until the 60 s and 5 minute timers catch up. For an always-on wall display that is the main real-time weak spot.
- The service worker checks for a new app version every 15 minutes and when the tab becomes visible. It activates a waiting worker with `SKIP_WAITING` and reloads once (`apps/web/src/lib/sw-update.ts`), because kiosk tabs are never closed.

#### Docs vs code

- The broadcast goes to every connected socket, displays included, whatever capabilities the socket holds. Payloads are ids only, so content does not leak, but a display does see event names and ids from areas it was never granted, such as `core.notification.created { memberId }`. The architecture says "invalidations" without spelling out that they are unfiltered.
- `FAMOS_ROLES` allows separate API and worker processes. The bus is in-process, and nothing bridges processes. In a split deployment, events emitted by jobs in the worker would not reach sockets held by the API process. That covers assistant streaming, drafts created by Quick add and notification badges. The language and theme registries also reload per process. This is my reading of the code; the docs do not mention it, and the default single process is not affected.

#### Evidence

`packages/core/src/{event-bus,core-events}.ts`, `packages/sdk/src/events.ts`, `apps/server/src/http/realtime.ts`, `apps/server/src/main.ts`, `apps/web/src/lib/realtime.ts`, `apps/web/src/routes/+layout.svelte`, `apps/web/src/routes/display/+page.svelte`, `apps/web/src/lib/sw-update.ts`.

### (c) Offline support

#### What the user can do

`docs/guide/offline.md` lists what works offline: adding shopping items, ticking them off or back on, marking a task done or reopening it, and marking a chore done or undoing that. Editing a task, creating a chore, the calendar, settings, the assistant and Quick add all need a connection. The app opens offline only after one online visit over HTTPS.

The user sees:

- a "Waiting to send" mark on each queued change, and a count at the top;
- "Offline — showing what was last seen" when reads come from the cache;
- a "could not be saved" list with the label, the time and the reason, dismissable one by one or all at once.

Wall displays queue nothing. The display's `useApp().runAction` rejects with "Displays cannot run actions".

#### How the queue works

- The server stores `(member_id, key) -> action_id, status, response` in `core_idempotency` inside the action's own transaction (`packages/core/src/idempotency.ts`). Any action call may send an `Idempotency-Key` header of 1 to 100 characters from `A-Za-z0-9_-`.
  - A repeated key returns the stored answer with `Idempotent-Replayed: true`.
  - The same key for a different action gets `422 idempotency_mismatch`.
  - `bad_request`, `forbidden` and `not_found` answers are stored too, so a replay fails the same way.
  - A failed transaction leaves no key, so the client can retry.
  - The daily job `core.pruneIdempotency` deletes keys older than 7 days.
- The client side lives in `apps/web/src/lib/offline-queue.ts`. It is pure TypeScript with injected storage, network and clock, and `offline-queue.test.ts` covers it.
  - `run(actionId, input, { offline, label })` sends at once. Without a network, or when earlier changes still wait, it queues the change and returns `{ queued: true }`. It also queues when the fetch throws with no answer at all.
  - `createdAt` values strictly increase, so tap order survives storage.
  - `flush` sends waiting changes oldest first. A 2xx removes the change. A 4xx other than 401, 408 and 429 is a final refusal: the change moves to the failed list and its input is dropped, keeping only the label. Anything else keeps the change, applies backoff (30 s doubling to 10 minutes) and stops the run so order holds.
  - Replay triggers are the `online` event, app start and a 30 s interval while changes wait (`apps/web/src/lib/offline.svelte.ts`).
- Storage is IndexedDB database `famos-offline`, version 1, with object stores `queue` and `failed` keyed by `key` (`apps/web/src/lib/offline-idb.ts`). It falls back to memory where IndexedDB is missing.
- The queue belongs to the signed-in member. `waiting()` filters by member id, and sign-out clears the queue, the failed list and the cached reads.

#### Conflict handling

There is no merge. The server applies each replayed action against current state with the current permissions. A tick on an item someone deleted gets `not_found` and goes to "could not be saved". A change that needs approval becomes a draft on arrival. The actions themselves are written to stay meaningful minutes later; for example, `settled` lets approval close a draft as "already done". The architecture says it deliberately avoids CRDTs, a sync engine and a local database (`docs/architecture.md` §14).

#### Service worker and PWA

- The service worker is hand-written in `apps/web/src/service-worker/index.ts`, with no PWA plugin.
  - It precaches the build output, static files and the SPA fallback `index.html`.
  - Navigations fall back to the cached shell when the network fails.
  - The cache name is versioned (`famos-<version>`), and old caches are deleted on activate.
- API reads are cached for an allow-list only: `core/me`, `core/setup`, `core/members`, `core/settings/family`, `calendar/occurrences`, `lists/tasks`, `lists/shopping/lists` and their items, and `chores/chores`. The strategy is network first. After 5 s without an answer, or on failure, the worker serves the cached copy with header `x-famos-offline: 1` and posts `FAMOS_SERVED_OFFLINE` to the page. Only 200 answers replace the copy. The API cache name is not versioned, so it survives app updates, and `CLEAR_API_CACHE` deletes it on sign-out. Everything else under `/api/` passes through untouched, and the server marks all API answers `no-store`.
- The Web Share Target is declared in `apps/web/static/manifest.webmanifest` and posts multipart to `/share-target`. The worker stores the title, text, URL and files in the `famos-share-target` cache, then redirects to `/share`, which uploads to Quick add. The share target accepts `image/*`, `application/pdf` and `text/*`.
- The worker also handles Web Push. It always shows a notification, because Safari requires one per push, and it opens only in-app paths on click.
- The language pack in use is cached in `localStorage` by version for offline starts. The last theme stylesheet and style are applied from `localStorage` by an inline script in `apps/web/src/app.html` before first paint.

#### Rules and logic worth copying

- Offline writes are a short list of declared, idempotent, replay-tested actions plus an idempotency key stored in the action's transaction. That is far cheaper than sync.
- Final versus retryable classification by HTTP status, with order preserved by stopping on the first retryable failure.
- A read cache limited to an explicit allow-list, marked as stale through a header and a message to the page.

#### Display relevance

- The display has no offline queue, which is correct for a read-only screen.
- When `GET /core/displays/me` fails, the display marks itself `unreachable` and keeps the last state it rendered. It does not drop back to pairing. 401 means revoked and does send it to pairing.
- The allow-list has no display-specific routes: not `core/displays/me`, not ambient photos or weather. A display that reloads while the server is down would get the shell and the cached calendar occurrences, but not its own identity. Wall-display offline behavior is therefore "keep showing the last render while the tab stays open", not "start cold offline".

#### Docs vs code

- `offline: true` on `defineAction` is a declaration only. Nothing on the server or in the web runtime reads it. Web code passes its own `{ offline: true }` to `app.runAction`, in `extensions/lists/src/web/data.ts` and `extensions/chores/src/web/data.ts`. The test `extensions/lists/test/offline.test.ts` checks the set of flagged actions. A new web caller could queue an action that is not flagged.
- Server-side flagged actions are in `extensions/lists/src/server/shopping.ts` (2), `extensions/lists/src/server/tasks.ts` (2) and `extensions/chores/src/server/chores.ts` (2). They match the guide.

#### Evidence

`docs/guide/offline.md`, `docs/architecture.md` §14, `apps/web/src/service-worker/index.ts`, `apps/web/static/manifest.webmanifest`, `apps/web/src/lib/{offline-queue,offline-idb,offline.svelte,sw-update}.ts`, `packages/core/src/idempotency.ts`, `apps/server/src/http/app.ts` (the `idempotency-key` header on action routes), `extensions/lists/test/offline.test.ts`, `extensions/chores/test/offline.test.ts`.

### (d) i18n and language packs

#### What the user can do

- Pick the language as the first screen of setup. It becomes the family language and the first admin's own.
- Set the family language in Settings → Family → Language & region (`core.settings.manage`). It applies to people who did not choose their own, to their notifications, to displays and to AI-written Quick add titles.
- Set a personal language in Settings → You → Profile & sign-in, through `core.setMyLanguage`, which any member may run. The choice covers the app, notifications, Telegram and the assistant. "Same as the family" clears it.
- Use a language switcher on the sign-in page, per device, when more than one language exists.
- Set region format separately: time zone, format locale such as `en-GB`, first day of the week (ISO 1 to 7), 12 or 24 hour clock, and °C or °F (`docs/guide/region.md`). Changing the language never changes these.
- Install, replace or remove packs as described in (a). The Settings page shows coverage per pack.

#### How it works

- English is the only language in code: `apps/web/src/messages/en.json` plus one `extensions/<id>/src/messages/en.json` per extension, with the namespace equal to the extension id. Turkish is bundled as `languages/tr/` with `manifest.json`, `app.json` and 12 extension catalogs.
- `packages/i18n/src/pack.ts` holds the pack contract:
  - `PackManifestSchema` has code (checked with `Intl.getCanonicalLocales`), name, nativeName, version, `famos` semver range, optional `dir`, and authors.
  - Limits are 2 MB per pack (`MAX_PACK_BYTES`) and 4 KB per string (`MAX_STRING_BYTES`).
  - Unknown namespaces and keys are reported and ignored. A string whose `{{placeholders}}` differ from English is dropped. Plural keys are checked against the language's own CLDR categories from `Intl.PluralRules`.
  - `coverage` counts a plural group once. `satisfiesRange` enforces the `famos` range against the app version.
- `apps/server/src/languages.ts` is the registry. At start and on every change it loads bundled packs, then installed packs. A broken pack is skipped and logged, never fatal. A pack whose folder name differs from its code is skipped. At most 40 packs may be installed. Each pack's version string is `<manifest version>-<fingerprint>`, which serves as the catalog ETag.
- Core sees languages only through `LanguagePort`. Codes are validated against the registry with `languages.assertKnown`, not against an enum.
  - Public routes: `core.listLanguages` and `core.getLanguageCatalogs`, with English never served.
  - Actions: `core.installLanguagePack`, `core.removeLanguagePack` and `core.installLanguagePackFromUrl`, which enqueues `core.fetchLanguagePack` with 2 attempts.
  - Removing an installed pack moves the family back to English, or to the bundled pack of that code, and clears members who used it.
- `ctx.vocabulary(key)` returns the comma-separated words for a key across English and every installed pack, lowercased in each language, deduplicated, and cached until the packs change (`createVocabulary` in `apps/server/src/i18n.ts`). Code uses it to recognize command names, weekdays, units and keywords in user input without holding words of any language. The Telegram share command is one example.
- Server-side text that is stored or sent uses `ctx.translate(locale, key, params)`, rebuilt when the registry changes. Notifications render in the member's locale, else the family's, falling back to English per string.
- Prompts are English only and name the answer language with `Intl.DisplayNames`.
- Formats always go through `formatLocale`, never the UI language. `scripts/region-formats.test.ts` fails if `app.locale()` reaches `Intl`. `scripts/language-branches.test.ts` fails on `=== 'tr'` style branches. `scripts/language-words.test.ts` fails on non-ASCII letters in code strings.

#### Data model

- `core_members.locale` is nullable. Null means "same as the family".
- `core_settings` key `family` holds `FamilySettings` from `packages/sdk/src/context.ts`: `timeZone`, `locale`, `formatLocale`, `weekStart`, `hourCycle`, `temperatureUnit`.
- `core_settings` key `languages` records installed packs; `languages.fetch` records URL-install progress.
- Pack files live in `<data>/languages/<code>/`, one JSON file per namespace.

#### Rules and logic worth copying

- Keep language and region format as separate settings: words versus dates, numbers, week start, clock and units.
- Per-person language with a family fallback. Displays always use the family's.
- Packs are data with placeholder and plural validation, so a half-finished pack still works with English fallback per string.
- "Vocabulary" keys solve input parsing in many languages without code changes.
- Tests that scan the source for language-specific branches and hard-coded locale use.

#### Display relevance

Displays speak the family language and follow `core.family.*` and `core.languages.*` events live, without a reload. The temperature unit applies to the weather on displays.

#### Evidence

`docs/guide/languages.md`, `docs/guide/region.md`, `docs/architecture.md` §6 ("Language and format locale", "Language packs"), `packages/i18n/src/{pack,index,node,catalog}.ts`, `apps/server/src/{languages,i18n}.ts`, `packages/core/src/languages.ts`, `packages/core/src/core-extension.ts` (`core.setMyLanguage`), `languages/tr/`, `apps/web/src/lib/i18n.svelte.ts`.

### (e) Backups

#### What the admin can do

All of this sits under Settings → System → Backups and needs `core.settings.manage`. The routes live in `apps/server/src/http/backups.ts`, outside the action runner, as credential management.

- See local backups with their reason (Nightly, Manual, Before an update), each remote target with its last result, and the encryption mode: `GET /api/v1/backups`.
- Back up now with `POST /api/v1/backups/run`. This enqueues the job `backup.run` in the worker. No backup is written inside the HTTP request.
- Choose encryption with `PUT /api/v1/backups/encryption`. The choices are a passphrase of 12 to 500 characters (Zod schema in `packages/adapters/src/backup/schemas.ts`), an age public key `age1…`, or an explicit "none".
- Add, edit, pause, delete and test remote targets: `POST /api/v1/backups/targets`, `PATCH` and `DELETE /api/v1/backups/targets/{id}`, and `POST .../{id}/test`, which writes and deletes a probe file. Credentials are never returned. A blank password field in an edit keeps the stored value.
- Run the CLI with `famos backup`, `famos restore <file> [--identity <file>] [--force]` and `famos migrate` (`apps/server/src/main.ts`).

#### Schedule and retention

- An in-process timer in the worker role ticks every 10 minutes (`startScheduler` in `apps/server/src/runtime.ts`). When `new Date().getHours()` equals `FAMOS_BACKUP_HOUR` (default 3) and no backup was written that day, it writes a `scheduled` backup and queues uploads. The hour is the server's local time, which is UTC in the container unless `TZ` is set. The guide states this.
- Local retention by reason (`pruneBackups` in `packages/db/src/backup.ts`): `scheduled` keeps `FAMOS_BACKUP_KEEP`, default 7, maximum 365. `pre-migration` keeps 5. `manual` backups are never deleted automatically.
- Remote retention is per target, `keep` with default 7 and maximum 365. After an upload, files matching `^famos-…\.tar(\.age)?$` beyond the newest `keep` are deleted, and other files in the folder are left alone.
- The same nightly tick prunes succeeded queue jobs older than 7 days.

#### Format and snapshot

- `createBackup` in `packages/db/src/backup.ts` writes `famos-<ISO timestamp>-<reason>.tar` to `<data>/backups`. The tar holds `manifest.json`, `db/famos.db` made with `VACUUM INTO` (a consistent snapshot of the live database), `files/`, `languages/` and `themes/`. It writes to `.partial` and renames at the end.
- The manifest records `format: 1`, `appVersion`, `createdAt`, `schema` (applied migrations per scope), `db.sha256`, `db.size` and `reason`.
- Local backups are not encrypted.
- `FAMOS_SECRET` and the data directory's `secret` file are never in a backup. They seal AI keys, the Telegram token, the mail password, backup credentials and passphrase, calendar subscription URLs and the VAPID key. A restore without the same secret keeps all data but loses those, signs everyone out and stops push notifications (`docs/guide/backups.md`).

#### Encryption and key handling

- `age` via the `age-encryption` 0.3.1 library (`packages/adapters/src/backup/encryption.ts`). Passphrase mode uses age's scrypt recipient. Recipient mode encrypts to an X25519 `age1…` public key. Encryption streams, and age's output size is computable, so WebDAV can send `Content-Length`.
- The passphrase is stored sealed in `backup_settings.sealed_passphrase` so nightly uploads run unattended. Sealing uses the shared `SecretBox`: AES-256-GCM with a key derived by HKDF-SHA256 from `FAMOS_SECRET`, with a per-purpose info string (`backup` here) and format `v1.<iv>.<tag>.<ciphertext>` (`packages/adapters/src/secrets/index.ts`). Recipient mode leaves no backup secret on the server.
- Encrypted remote copies are named `<local name>.age`.
- Nothing uploads until the admin picks a mode, "none" included. Without a mode the upload fails with the non-retryable code `encryption_unset` and admins are notified. A sealed passphrase that no longer opens, for example after `FAMOS_SECRET` changed, gives `unreadable`. A local file that is gone gives `missing_local` (`packages/adapters/src/backup/service.ts`).
- Retention runs after a successful upload, and a retention failure never fails the upload.

#### Remote targets (integrations)

- S3-compatible storage (`packages/adapters/src/backup/s3.ts`): AWS S3, Backblaze B2, Wasabi, MinIO and Cloudflare R2 per the guide. Signing is SigV4 through `aws4fetch` 1.0.20. Uploads are a single PUT or a hand-written multipart upload with 8 MiB parts, so memory stays near one part; a failed multipart upload is aborted. `pathStyle` covers MinIO and R2. Config: endpoint, region (default `us-east-1`), bucket, optional prefix, access key id, secret key.
- WebDAV (`packages/adapters/src/backup/webdav.ts`): plain `fetch`, `PROPFIND` to list, `MKCOL` to create the folder (405 on an existing folder is accepted). Config: folder URL, user name, password. Nextcloud is the documented example.
- Both refuse redirects (`redirect: 'manual'`), because they carry credentials. Both use `fetch` directly instead of the SSRF-guarded port, because an admin configures them and they may live on the LAN. Addresses must be http(s) with no user info, query or hash.
- Timeouts: 45 minutes per upload and 30 s for control calls.

#### Upload jobs

- `backup.upload` has payload `{ targetId, name }`, runs on the `long` lane with a 50 minute timeout and 5 attempts. Backoff is `min(5 min × 4^(n−1), 4 h)`, about 6 hours in total, which matches the guide. Uploads are deduplicated by `backup-upload:<target>:<name>`.
- HTTP 5xx, 408, 425 and 429, timeouts and network errors are retryable. Other 4xx answers and a missing encryption mode are not, and notify at once.
- On a non-retryable error or the last attempt, the job notifies everyone with `core.settings.manage` once, with kind `core.backupFailed` and dedupe key `backup-failed:<target>:<name>`. The target row stores `last_result`, which is `ok`, `retrying_<code>` or the code.
- A paused or removed target skips its queued uploads. Removing a target leaves the files it already received.
- `backup.run` has a 10 minute timeout and 2 attempts. It writes a manual backup, then enqueues one upload per enabled target.

#### Restore flow

- CLI only. Stop the server, then run `famos restore <file>` (`apps/server/src/restore.ts`).
- A `.age` file is decrypted by streaming into a temporary folder inside the data directory. The passphrase comes from `FAMOS_BACKUP_PASSPHRASE` or a terminal prompt, or `--identity <file>` supplies an `AGE-SECRET-KEY-…`. A wrong key, a damaged file or a truncated file fails before anything changes.
- The manifest is read and the format checked. The backup's `schema` is compared with the migrations this build knows, and a backup from a newer FamilyOS is refused unless `--force` is given.
- The database checksum is verified. Then `famos.db` with its `-wal` and `-shm` files, `files/`, and, if present in the backup, `languages/` and `themes/` are renamed to `*.pre-restore-<time>`, and the restored copies are put in place. Nothing is deleted.
- Migrations then run on the next start, as for any upgrade.
- `apps/e2e/tests/restore.spec.ts` covers an encrypted backup: upload to a fake WebDAV server, restore with the right passphrase, and failure with a wrong one or a damaged file.

#### Pre-upgrade backups

`migrate()` in `apps/server/src/runtime.ts` plans migrations first. If the database schema is newer than the code, `planMigrations` refuses to start. If there are pending migrations on an existing installation, it writes a `pre-migration` backup, prunes those to 5 and then migrates. Migrations are forward-only, so rolling back an upgrade means restoring that backup.

#### Data model

- `backup_settings`: a single row with `id = 1` enforced by a CHECK, `encryption_mode` in `passphrase | recipient | none`, `sealed_passphrase`, `recipient`, `updated_at`.
- `backup_targets`: `id`, `kind` in `s3 | webdav`, `name`, `config` (sealed JSON with credentials), `keep` (default 7), `enabled` (default 1), `last_run_at`, `last_result`, `last_size`, `created_at`.
- Both tables are STRICT, in migration scope `backup` (`packages/adapters/src/backup/tables.ts`).
- Local backups are files in `<data>/backups`, not rows.

#### Rules and logic worth copying

- `VACUUM INTO` for a consistent single-file snapshot, plus a manifest that records the schema per scope, so restore can refuse newer backups.
- An automatic backup before every migration. With forward-only migrations, that backup is the rollback plan.
- Restore moves current data aside instead of deleting it.
- Remote copies use a standard, tool-readable encryption format. `age -d` works without FamilyOS.
- The upload is a durable job on a separate lane, so a 50-minute upload never delays reminders.
- Retention deletes only files matching the app's own naming pattern.
- Errors are split into retryable and non-retryable, and the admin is notified once per backup and target.

#### Display relevance

None directly. A wall-display product needs the same pre-upgrade backup and restore story, because a kiosk is often the only screen the family looks at.

#### Docs vs code

- `docs/architecture.md` §12 says "restore via CLI or the first-run screen". The web app has no restore screen; restore is CLI only. The routes in `apps/server/src/http/backups.ts` have no restore endpoint.
- §12 says "Extensions may register pre/post-backup hooks". The SDK and core have no such hook.
- §12 says the manifest has "checksums". It has one, for the database. Files, language packs and themes are not checksummed.
- §17 lists "Docker smoke: `docker compose up` → `/health`; arm64 image builds; backup and restore inside the container". CI only builds the multi-arch image (`.github/workflows/ci.yml`, job `docker`). The Dockerfile has a `HEALTHCHECK` on `/health`, but no smoke run exists in CI.
- `docs/guide/install.md` lists the data folder contents and omits `themes/`, which the code creates and backs up.
- The guide says the hour of the nightly backup is server-local. README describes `FAMOS_BACKUP_HOUR` only as the "Hour of the nightly backup". Neither uses the family time zone.

#### Evidence

`docs/guide/backups.md`, `docs/architecture.md` §12, `packages/db/src/backup.ts`, `packages/adapters/src/backup/{encryption,jobs,s3,webdav,service,schemas,tables,target}.ts`, `packages/adapters/src/secrets/index.ts`, `apps/server/src/{restore,runtime,main}.ts`, `apps/server/src/http/backups.ts`, `apps/e2e/tests/restore.spec.ts`, `apps/e2e/support/fake-dav.ts`.

### (f) Jobs, the worker and files

#### Queue

- The `JobQueue` port lives in `packages/sdk/src/ports.ts`. The one implementation is a SQLite table in the same database (`packages/adapters/src/queue-sqlite/index.ts`, migration scope `queue`).
- `enqueue(db, name, payload, { runAt, dedupeKey, maxAttempts })` takes the caller's executor. A job written inside an action's transaction commits or rolls back with the data. The architecture calls this "Queue = outbox": there is no separate outbox table.
- `dedupeKey`: a pending or running job with the same key makes enqueue return the existing id. A partial unique index enforces this.
- `claim(names, leaseMs)` is one `UPDATE … WHERE id = (SELECT … status='pending' AND run_at <= now AND name IN (…) ORDER BY run_at, created_at LIMIT 1) RETURNING`. It increments `attempts` and sets `lease_until`.
- `fail(id, error, retryAt)` sets `dead` when `attempts >= max_attempts`, otherwise `pending` with the new `run_at`. Errors are cut to 4000 characters.
- `recoverExpired()` returns running jobs with an expired lease to pending, or to dead when out of attempts, with `last_error = 'lease expired (worker stopped or crashed)'`.
- `pruneSucceeded(olderThanMs)` deletes succeeded jobs. The nightly tick calls it with 7 days.

#### Worker

`packages/core/src/worker.ts`:

- One loop per lane, `default` and `long`, one job at a time each. Both share the single SQLite connection; Kysely's driver hands it out one holder at a time, so the lanes' transactions never interleave. A long job must keep its transactions short.
- Idle polling every 1 s. `ctx.enqueue` and `ctx.notify` call `worker.wake()` to start the next job at once.
- Lease length is `max(60 s, longest job timeout + 30 s)`. Expired leases are recovered every 30 s.
- Each attempt races the handler against `timeoutMs`. The payload is parsed with the job's Zod schema before the handler runs. An unknown job name is failed and retried after 60 s.
- Jobs of turned-off extensions are not claimed and stay queued until the extension is on again.
- `drain()` runs both lanes until empty, for tests and the CLI.
- Long-lane jobs today: `backup.upload`, `backup.run`, `email.poll`, `assistant.turn`, `assistant.summarize`, and `core.embedRebuild` in `packages/core/src/search.ts`.

#### How scheduled work runs

There is no cron table. Recurring work uses three patterns.

1. Self-rescheduling jobs. A job enqueues its next run with `runAt` and a dedupe key that names the day, so restarts and the job's own rescheduling never pile up. Examples: `core.expireDrafts` runs 5 minutes after midnight in the family time zone (`packages/core/src/draft-expiry.ts`). `core.pruneIdempotency` runs daily. `board.cleanup` is scheduled from `register`.
2. Next-occurrence reminders. `calendar.remind`, `lists.remind`, `chores.remind` and `records.remind` schedule only the next reminder, in the same transaction as the change. When the job runs it reloads the item and does nothing if the item changed since. `docs/architecture.md` §9 has the details.
3. In-process timers in the worker role. The nightly backup tick runs every 10 minutes. The email scheduler ticks every 30 s and enqueues `email.poll` when `poll_minutes` (default 5) have passed. The Telegram long-poll loop runs while a token is saved. All three start in `serveCommand` only when `FAMOS_ROLES` includes `worker`.

#### Files

- The `FileStore` port has `put`, `get` (a stream plus stat), `stat` and `delete`. The one implementation is `packages/adapters/src/files-fs/index.ts`, rooted at `<data>/files`.
- Keys must match `^[a-zA-Z0-9][a-zA-Z0-9._/-]*$`, must not contain `..`, must not end with `.meta.json`, and must resolve inside the root.
- Writes go to a temp file in the same folder, then a sidecar `<key>.meta.json` with `{ contentType }`, then an atomic rename.
- Extensions choose their own key layout, such as `inbox/<item>/<n>` for Quick add uploads.
- Deleting a file is a job (`board.deleteFile`, `ambient.deleteFile`), enqueued with the row deletion, so a crash never leaves a row pointing at a deleted file.
- A port contract suite in `packages/testing/src/contracts/index.ts` has `jobQueueContract`, `fileStoreContract` and `embeddingContract`. `packages/testing/test/adapter-contracts.test.ts` runs them against the real adapters.

#### Data model

`queue_jobs`: `id`, `name`, `payload` (JSON text), `status` in `pending | running | succeeded | dead`, `attempts`, `max_attempts`, `run_at`, `lease_until`, `last_error`, `dedupe_key`, `created_at`, `updated_at`, `finished_at`. The table is STRICT. Index `(status, run_at)`. Unique index on `dedupe_key` where status is pending or running.

#### Rules and logic worth copying

- The queue is a table in the same database and doubles as the outbox, so a side effect commits with the data.
- Day-named dedupe keys for recurring jobs.
- Only the next reminder is scheduled, and each job checks that its subject has not changed.
- A separate long lane, so uploads and AI turns never block reminders.
- Idempotent handlers are a stated rule, because delivery is at least once.

#### Display relevance

Reminders, notifications and calendar subscription refreshes all run in the worker. A wall display depends on a healthy worker only indirectly: it shows what the jobs wrote.

#### Docs vs code

- `docs/architecture.md` §9 says "dead-letter state visible in the UI". Nothing outside the queue adapter and tests reads `queue_jobs`. There is no route or screen for dead or failed jobs. Failures surface only per feature: a backup target's last result, an Inbox item's error code, email status, logs.
- `pruneSucceeded` deletes only succeeded jobs. Dead jobs stay in `queue_jobs` forever.
- The JobState type in the SDK includes `failed`, but the table has no `failed` status. `fail()` returns `'failed'` to mean "pending again".

#### Evidence

`packages/sdk/src/{jobs,ports}.ts`, `packages/core/src/{worker,kernel,draft-expiry,idempotency}.ts`, `packages/adapters/src/queue-sqlite/index.ts`, `packages/adapters/src/files-fs/index.ts`, `packages/adapters/src/email/service.ts` (`start`), `apps/server/src/runtime.ts` (`startScheduler`), `packages/testing/src/contracts/index.ts`.

### (h) Install, HTTPS and releasing

#### Install

`docs/guide/install.md`:

- `git clone`, then `docker compose up -d`. The log prints `/setup?code=…`. A Raspberry Pi 4 is enough.
- The one-time setup code is generated into `<data>/setup-code`, or set with `FAMOS_SETUP_CODE` (8 to 100 characters). It stops a stranger from claiming an instance that is reachable before the owner gets to it.
- Setup steps: language, then name, email and a password of at least 10 characters, then "This device is shared at home" to make it a trusted family device, then Create account (this person becomes admin), then an optional city for the weather. Time zone and region formats come from the browser.
- To update: `git pull && docker compose up -d --build`. A pre-migration backup runs automatically.
- `docker-compose.yml` has one service, `famos`, on port 3000 with volume `famos-data:/data`, environment `FAMOS_BASE_URL` and `TZ`, and an optional `ollama` profile.

#### HTTPS

`docs/guide/https.md`:

- Plain `http://<server>:3000` works on the LAN. Installing the PWA, the share target and Web Push need HTTPS.
- Option A is `tailscale serve --bg 3000`, with no open ports.
- Option B is Caddy with a domain: `caddy reverse-proxy --from <domain> --to famos:3000`, plus `FAMOS_TRUSTED_PROXIES=172.16.0.0/12` so `X-Forwarded-For` is believed for sign-in rate limits. Without that variable, the server ignores the header entirely.
- `FAMOS_BASE_URL` must be the address people use. The CSRF origin check on `/api/v1/*` trusts it, and extra addresses go in `FAMOS_TRUSTED_ORIGINS`. Cookies are `Secure` when the base URL starts with `https://` (`apps/server/src/main.ts`).
- The server keeps idle connections open 65 s, above the usual 60 s proxy idle timeout, to avoid `ECONNRESET` behind proxies.

#### Releasing

`docs/releasing.md`:

- SemVer for what a family sees. Minor releases are expand-only. Before 1.0, a minor may break, and its notes say so.
- `pnpm release x.y.z` sets the version, commits `chore(release): x.y.z`, tags and prints notes; a person pushes.
- The Release workflow runs lint, typecheck, tests and E2E, checks the tag against the version, and pushes `ghcr.io/onurceri/familyos` for amd64 and arm64 as `x.y.z`, `x.y` and `latest`.
- Release notes come from Conventional Commits through `scripts/release-notes.mjs`. Sections: Breaking changes, New, Fixed, Security (`fix(security)`), Faster (`perf`).
- The current version is 0.1.0, and `CHANGELOG.md` has only "Unreleased: First public version".

#### Display relevance

- A wall tablet on plain HTTP works for viewing. It gets no service worker offline shell, because browsers only register service workers on secure origins and localhost. It also gets no push.
- HTTPS through Tailscale or Caddy is a prerequisite for the offline behavior described in (c).

#### Evidence

`docs/guide/{install,https}.md`, `docs/releasing.md`, `docker-compose.yml`, `docker/Dockerfile`, `README.md` (configuration table), `apps/server/src/{config,main}.ts`, `.github/workflows/{ci,release}.yml`, `scripts/release.mjs`.

---

## 16. Tech stack

In one line: a TypeScript 6 pnpm/Turborepo monorepo with a Hono 4 server on Node 24, SQLite through better-sqlite3 and Kysely, a SvelteKit 3 / Svelte 5 static SPA and PWA served by the same process, i18next, Temporal, better-auth and the Vercel AI SDK, shipped as one multi-arch Docker image with a `/data` volume.

### Components and versions

Versions are pinned exactly (`.npmrc` save-exact plus the committed lockfile) and updated through Renovate (`renovate.json`).

| Concern | Choice | Version in the repo |
|---|---|---|
| Runtime | Node.js, `engines: >=24.12 <27`; Docker base `node:24-bookworm-slim` (Debian, not Alpine, for native modules on a Pi) | Node 24 |
| Language | TypeScript with `erasableSyntaxOnly` (no enums, namespaces or parameter properties) | 6.0.3 |
| Monorepo | pnpm workspaces (`apps/*`, `packages/*`, `extensions/*`) and Turborepo | pnpm 12.9.1, turbo 2.11.7 |
| HTTP server | Hono on `@hono/node-server`, `@hono/zod-openapi`, WebSocket through `ws` | hono 4.13.13, node-server 2.1.3, zod-openapi 1.6.3, ws 8.22.0 |
| Validation | Zod, the one source for request shapes, action inputs, LLM output and draft forms | 4.6.5 |
| Database | SQLite through better-sqlite3 (N-API) and Kysely `SqliteDialect`; WAL, `synchronous = NORMAL`, `foreign_keys = ON`, `busy_timeout = 5000`, STRICT tables (`packages/db/src/connection.ts`) | better-sqlite3 13.0.3, kysely 0.29.6 |
| Frontend | SvelteKit as an SPA with `adapter-static`, Svelte 5 runes, Vite, Tailwind v4, vendored shadcn-svelte components | kit 3.0.0, svelte 5.57.1, vite 8.3.2, tailwindcss 4.3.3, adapter-static 4.0.0 |
| PWA | Static `manifest.webmanifest` with `share_target`, hand-written service worker, no PWA plugin | n/a |
| i18n | i18next behind `@famos/i18n`, shared by web and server | 26.4.2 |
| Time | Temporal through `temporal-polyfill`, wrapped in `@famos/time`; RRULE through `rrule-temporal` | n/a |
| Auth | better-auth (email and password, profile plus PIN on trusted devices), behind an `Identity` port | 1.7.7 |
| AI | Vercel AI SDK with one OpenAI-compatible client for OpenRouter, Ollama and similar | ai 7.0.128 |
| MCP | `@modelcontextprotocol/server` and `@modelcontextprotocol/hono` | 2.3.1, 2.0.2 |
| Integrations | `age-encryption` (backups), `aws4fetch` (S3 SigV4), `imapflow` (email), grammY (Telegram), `ical.js` (calendar feeds), Open-Meteo (weather, no key) | 0.3.1, 1.0.20, 2.2.6, 1.46.0, 2.2.1 |
| API client | `openapi.json` generated by the server, then `openapi-typescript` types plus the `openapi-fetch` runtime in `packages/api-client` | 7.13.x, 0.17.x per architecture.md |
| Build | Server bundled by esbuild into one ESM file `apps/server/dist/main.mjs`, with better-sqlite3 external; dev runs `tsx watch` | esbuild 0.28.x |
| Quality | Biome (TS/JS/JSON/CSS), Prettier for `.svelte` only, svelte-check, Vitest, Playwright, dependency-cruiser | biome 2.5.15, vitest 5.0.3, dependency-cruiser 18.5.0 |
| License | AGPL-3.0-or-later, no CLA | n/a |

### Layers

The diagram in `docs/architecture.md` §3 matches the code:

- `apps/server` is the composition root and the only place that knows concrete libraries. `apps/server/src/runtime.ts` opens the database, builds every adapter (queue, identity, AI settings and LLM, web push, Telegram, email, backups, language and theme registries, file store, secret boxes, guarded HTTP) and passes them to `createKernel`. `apps/server/src/main.ts` is the CLI: `serve` (the default), `migrate`, `backup`, `restore <file>`, `openapi [file]`.
- Inbound adapters only validate, resolve the `Actor` and call core or an action. These are the HTTP routes in `apps/server/src/http/*`, the WebSocket, the CLI, MCP in `apps/server/src/mcp/*`, Telegram and email.
- `@famos/core` (`packages/core`) is the kernel: members, roles and RBAC, the action registry and runner, drafts, the job worker, the in-process event bus, notifications, devices and displays, languages, themes, search, auto-approval and member memory.
- Extensions under `extensions/*` own features. Each has its own tables, actions, routes, jobs, events, migrations and UI contributions.
- `@famos/sdk` is the only package extensions depend on: ports, `defineExtension`, `defineAction`, `defineRoute`, `defineJob`, `defineEvent`, capabilities, ids, handles.
- `@famos/adapters` implements the ports through subpath exports: `queue-sqlite`, `files-fs`, `llm-aisdk`, `identity-betterauth`, `http-guard`, `secrets`, `webpush`, `telegram`, `email`, `backup`, `runtime` (logger, UUIDv7, clock).
- `@famos/db` holds the SQLite connection, the migrator and the backup snapshot. `@famos/ui` holds Svelte components, slots and `useApp()`. `@famos/themes` holds the theme pack contract. `@famos/time` holds date math. `@famos/testing` holds the in-memory test kernel, fakes and port contract suites.

### Non-negotiable architecture rules

Condensed from `AGENTS.md` "Architecture rules" and `docs/architecture.md` §§1, 5, 7, 9, 12. Dependency-cruiser enforces the import rules with 13 rules in `.dependency-cruiser.cjs`, among them `core-only-sdk`, `extensions-only-sdk-ui-contracts`, `extensions-no-internals-of-others`, `only-server-wires-infrastructure` and `web-no-server-code`.

- Import boundaries. Extensions import only `@famos/sdk`, `@famos/ui`, `@famos/time` and other extensions' `./contract`. Core imports only the SDK. The web app talks to the server only through `@famos/api-client`. Only `apps/server` imports `db`, `adapters` and concrete libraries.
- Extensions never import each other's internals. They use actions (`uow.run`, `ctx.actions.invoke`), events and typed ids. No foreign keys cross extensions, and tables carry the extension id as prefix.
- Access is capabilities with a level, never role names. Routes declare `requires`. Data-changing actions declare `requires` and/or `approval: { unless }`.
- Every state change is an action, and the runner enforces permissions and approval. Credential management and device trust are the only exceptions; backup targets, push subscriptions, API tokens and Telegram linking live in dedicated routes for that reason.
- Zod is the single source of truth for shapes.
- LLMs are never called inside an HTTP request, apart from one exception for the family search question. Only job handlers get `ctx.llm`, and every call names a role, a timeout and the AI areas it reads.
- Durable side effects go through the job queue, enqueued in the same transaction. The event bus is only for UI invalidation and non-critical reactions.
- AI never changes data directly. Its changes become drafts a person approves, and proposable actions declare `undo`.
- Job handlers and action `run` functions must be idempotent.
- People are told through `uow.notify` / `ctx.notify` with declared notification kinds and translation keys, never by calling a channel.
- Migrations are hand-written, forward-only and expand-only within a major version. A released migration is never edited.
- Region-neutral and language-neutral. English is the only language in code, and words code must recognize come from catalogs through `ctx.vocabulary`. Formats follow `formatLocale`, never the UI language.
- One installation is one family, and SQLite is the only database. Extensions are build-time only. Smart-home control is out of scope.

### Configuration and runtime

Environment variables are parsed with Zod in `apps/server/src/config.ts`:

| Variable | Default | Note |
|---|---|---|
| `FAMOS_BASE_URL` | `http://localhost:<PORT>` | Public address. Used for origin checks, `Secure` cookies when https, links, and the VAPID contact |
| `FAMOS_SECRET` | generated into `<data>/secret`, mode 0600 | At least 32 characters. Seals credentials (AES-256-GCM, HKDF per purpose) and signs sessions. Never inside a backup |
| `FAMOS_SETUP_CODE` | generated into `<data>/setup-code` | 8 to 100 characters, needed for first-run setup |
| `FAMOS_TRUSTED_ORIGINS`, `FAMOS_TRUSTED_PROXIES` | none | Extra CSRF origins; proxies whose `X-Forwarded-For` is believed |
| `FAMOS_ROLES` | `api,worker` | Run the API, the worker, or both in one process |
| `FAMOS_DATA_DIR` | `./data`, `/data` in Docker | Holds `famos.db`, `files/`, `backups/`, `languages/`, `themes/`, `secret`, `setup-code` |
| `FAMOS_WEB_DIR`, `FAMOS_LANGUAGES_DIR`, `FAMOS_THEMES_DIR` | set in the image | The built SPA and the bundled packs |
| `FAMOS_BACKUP_HOUR`, `FAMOS_BACKUP_KEEP` | 3, 7 | Nightly backup hour in server-local time; scheduled backups kept |
| `FAMOS_BACKUP_PASSPHRASE` | none | Only for `restore` of `.age` files |
| `FAMOS_AI_PROVIDER`, `FAMOS_AI_BASE_URL`, `FAMOS_AI_API_KEY`, `FAMOS_AI_MODEL[_FAST/_VISION/_SMART]` | none | One provider with fixed id `ai_provider_env`, re-synced on every start |
| `PORT`, `HOST`, `LOG_LEVEL`, `TZ` | 3000, `0.0.0.0`, `info`, UTC in compose | |
| `FAMOS_TELEGRAM_API_ROOT`, `FAMOS_EMAIL_FAKE_DIR`, `FAMOS_NOTIFY_DELAY_MS` | none | Tests only |

`serve` runs migrations with an automatic pre-migration backup, syncs the AI provider from the environment, starts the kernel (calling every extension's `register`), and prints the setup URL while no member exists. With the `worker` role it then starts the worker loops, the nightly backup tick, the Telegram poller and the email scheduler. With the `api` role it starts Hono, the WebSocket and the static SPA. On SIGTERM or SIGINT it stops everything in reverse order.

### Build, packaging and deployment

- `turbo.json` makes `build` depend on `gen`. `@famos/server#gen` writes `apps/server/openapi.json` by running `main.ts openapi` against an in-memory database. `@famos/api-client#gen` turns it into `src/generated/schema.d.ts`. CI fails when either drifts (`AGENTS.md`, "API").
- `docker/Dockerfile` is multi-stage. The build stage runs on `$BUILDPLATFORM`, since all of its output is architecture-independent. The runtime stage installs only `better-sqlite3` (the exact version is read from the server's `package.json`), then copies `server/`, `web/`, `languages/` and `themes/`. It runs as user `node`, declares `VOLUME /data`, has a `HEALTHCHECK` that fetches `/health`, and uses `ENTRYPOINT node /app/server/main.mjs` with default command `serve`.
- `scripts/check-dockerfile.mjs`, part of `pnpm lint`, checks that the image copies the pack folders.
- `docker-compose.yml` has one service with port 3000 and the `famos-data` volume, plus an optional `ollama` profile.
- Releases publish `ghcr.io/onurceri/familyos` for amd64 and arm64 from `.github/workflows/release.yml`.
- `@famos/api-client` (`packages/api-client/src/index.ts`) exports `createApiClient` (openapi-fetch with `credentials: 'same-origin'`) and `unwrap`, which normalizes errors to `{ status, code, message, details }`. The architecture names it as the contract for future native clients.

### Testing

- Vitest across the repo (root `vitest.config.ts`). Extension and core tests run against in-memory SQLite with real migrations, through `createTestKernel` from `@famos/testing` and `packages/core/test/harness.ts`. A scripted fake model replaces real providers.
- Port contract suites in `packages/testing/src/contracts/index.ts` cover the queue, the file store and embeddings.
- Source-scan tests in `scripts/` fail on hard-coded colors, page frames, non-ASCII words, language branches, locale misuse and member-color misuse.
- E2E with Playwright in `apps/e2e`: 18 spec files with 25 tests. They run against the production bundle on a Pixel 7 device profile, with a fake OpenAI-compatible model and a fake WebDAV server in `apps/e2e/support/`. Each test gets its own copy of a prepared data directory. Specs cover first run, PIN sign-in, roles and members, displays, Quick add, calendar, lists, chores, kitchen, board, vehicles, Today, appearance, languages, email, MCP and restore.
- `pnpm sweep` takes screenshots of every page for review and is not a gate. `extensions/inbox/eval` runs an extraction evaluation against a real model and is not in CI.

### Display relevance

- One process serves the SPA, the API and the WebSocket on one port, which keeps a Pi deployment simple.
- There is no server-side rendering. Displays are SPA tabs that the service worker keeps updated, reloading when a new version deploys.
- HTTPS (Tailscale or Caddy) is needed for the service worker and push on tablets and phones.

### Docs vs code

- The exact-pin dependency policy has two exceptions in deployment files. `docker-compose.yml` uses `ollama/ollama:latest`, and the GitHub Actions use major-version tags (`actions/checkout@v5`, `docker/build-push-action@v6`).
- `docs/guide/install.md`'s table of the data volume omits `themes/`.

### Evidence

`docs/architecture.md` (all 552 lines), `AGENTS.md`, root `package.json`, `turbo.json`, `pnpm-workspace.yaml`, `.dependency-cruiser.cjs`, `apps/server/src/{main,runtime,config}.ts`, `apps/server/package.json`, `apps/web/package.json`, `packages/adapters/package.json`, `docker/Dockerfile`, `docker-compose.yml`, `packages/api-client/src/index.ts`, `apps/e2e/playwright.config.ts`, `apps/e2e/tests/*.spec.ts`, `.github/workflows/{ci,release}.yml`.

## 17. Unfinished, stubbed or TODO (repo-wide scan)

### Method

I grepped `apps/`, `packages/`, `extensions/` and `scripts/`, excluding `packages/api-client/src/generated`, `node_modules`, build output and message catalogs. The patterns were:

- `TODO|FIXME|XXX|HACK`;
- `not implemented|unimplemented|coming soon|stub|placeholder|wip|for now|temporary|not yet|future`;
- `throw new (Error|FamosError)(… not implemented/supported/available …)`;
- `it.skip|test.skip|describe.skip|test.todo|.only|skipIf|runIf|xit|xdescribe`;
- `deprecated|legacy|workaround|revisit`;
- `FAMOS_*` environment flags.

I also read the docs for "not built", "not yet" and "limitation", and checked architecture claims against the code.

### What the code scan found

The codebase is unusually clean of markers.

- Zero `TODO`, `FIXME`, `XXX` or `HACK` comments in source.
- No "not implemented", "coming soon" or stub functions. Every `placeholder` hit is an input placeholder attribute or i18n placeholder logic.
- The four "not available" or "not supported" throws are real runtime refusals, not stubs. Unknown language code in `packages/core/src/languages.ts:354`. Unknown theme in `packages/core/src/themes.ts:573`. Push endpoint not on a known push service in `packages/adapters/src/webpush/service.ts:125`. Filing a paper while `records` is off in `extensions/inbox/src/server/items.ts:90`.
- Skipped tests are all conditional, none disabled:
  - `apps/e2e/sweep/sweep.spec.ts:156,222,230` use `test.skip` to narrow the screenshot sweep by `FAMOS_SWEEP_STATE` and `FAMOS_SWEEP_ONLY`;
  - `packages/core/test/search-bench.test.ts:11,48` use `it.skipIf(!process.env.FAMOS_BENCH)`, an opt-in benchmark for 50,000 vectors;
  - `packages/adapters/test/http-guard.test.ts:252` uses `describe.skipIf(!hasOpenssl)` for TLS transport tests.
- There is no developer feature-flag system. Test-only variables are `FAMOS_TELEGRAM_API_ROOT`, `FAMOS_EMAIL_FAKE_DIR`, `FAMOS_NOTIFY_DELAY_MS`, `FAMOS_BENCH`, `FAMOS_EVAL_*`, `FAMOS_SWEEP_*` and `FAMOS_E2E_LOG`. Product switches are runtime settings: per-extension on/off (`core_settings.features`), the AI main and per-feature switches, per-area AI access, and the MCP switch (`core_settings.mcp`, default off).
- Code comments that name future work: `packages/time/src/index.ts:10` ("Temporal comes from a polyfill until Node ships it natively").

### Gaps between docs and code

These are the real unfinished items. The architecture describes them, but the code does not have them or has only part.

| Item | Where the doc says it | What the code has | Interpretation |
|---|---|---|---|
| Dead-letter jobs visible in the UI | `docs/architecture.md` §9 | Nothing outside `packages/adapters/src/queue-sqlite` and tests reads `queue_jobs`; no route or screen | Not built. Failed jobs show only in logs and in per-feature status |
| Dead jobs cleaned up | implied | `pruneSucceeded` removes only `succeeded` rows (`queue-sqlite/index.ts`) | `dead` rows accumulate without limit |
| Restore from the first-run screen | §12 "restore via CLI or the first-run screen" | CLI only (`apps/server/src/restore.ts`); `apps/server/src/http/backups.ts` has no restore route | Not built |
| Extension pre/post-backup hooks | §12 | No hook in the SDK, core or `packages/db/src/backup.ts` | Not built |
| Backup manifest "checksums" | §12 | Only `db.sha256`; files, languages and themes are not hashed (`packages/db/src/backup.ts`) | Partial |
| Docker smoke test (compose up, `/health`, backup and restore in the container) | §17 | CI job `docker` only builds the multi-arch image (`.github/workflows/ci.yml`) | Not built. The image has a `HEALTHCHECK` |
| Migration upgrade test from the previous release's fixture database | `AGENTS.md` "Testing", §17 | No fixture database in the repo; version 0.1.0 is the first release (`CHANGELOG.md` "Unreleased") | Not applicable yet. Starts with the second release |
| `offline: true` on actions | §14 "marks actions the web app may queue" | The flag is declared on 6 actions but no runtime code reads it. Web callers pass their own `{ offline: true }`, and only `extensions/lists/test/offline.test.ts` checks the set | Declarative only; drift is possible |
| Cross-process realtime with `FAMOS_ROLES=api` plus a separate worker | §16 says heavy installations may split | The event bus is in-process (`packages/core/src/event-bus.ts`) with no bridge, and language and theme registries reload per process | Likely broken realtime in split mode (my inference, undocumented) |
| WebSocket liveness | not documented | No ping or heartbeat on either side (`apps/server/src/http/realtime.ts`, `apps/web/src/lib/realtime.ts`) | Half-open sockets go unnoticed. The display page compensates with 60 s and 5 minute polling |
| Realtime broadcast scope | §14 "broadcasts `{ entity, id }` invalidations" | Every bus event goes to every socket, displays included, whatever their grants | Ids leak to displays and other members, but no content |
| Nightly backup in family time | not promised | `startScheduler` uses `new Date().getHours()`, which is server-local, UTC in the container (`apps/server/src/runtime.ts`) | Documented limitation in `docs/guide/backups.md` |
| Data volume contents | `docs/guide/install.md` table | Code also creates and backs up `<data>/themes/` | Docs gap |

### Documented "not built" or "not yet" items

These are deliberate, or stated as open in the docs.

- Two-way CalDAV sync is not built. Calendar feeds are ICS in and out only (`docs/architecture.md`, "Calendar feeds").
- There is no public directory of language packs "yet" (`docs/guide/languages.md:75`). The same applies to themes.
- MCP clients that support only OAuth "cannot connect yet". FamilyOS uses personal bearer tokens (`docs/guide/mcp.md:108`).
- Quick add's pipeline names "future steps (OCR, link fetching, transcription)". Transcription exists. OCR and link fetching do not; `extensions/inbox/src` has no OCR or URL fetch (`docs/architecture.md` §10).
- Family search keeps vectors in SQLite and scores them in TypeScript. Past about 200,000 chunks or 200 ms, the doc says to move to in-memory vectors or `sqlite-vec` (§11).
- Zero data retention cannot be guaranteed for speech to text through OpenRouter (§11 "Limitation").
- Telegram video messages are not supported (`docs/guide/telegram.md:39`).
- Passkeys are not offered, because they break when the instance's address changes (§2).
- No marketplace or runtime install of extensions or styles, by design (§1, §6).
- Watch items from §2: revisit TypeScript 7 when 7.1 ships its API, revisit `node:sqlite` at Stability 2, and pin rolldown if the Linux CI crash appears.
- Restoring a remote backup has no in-app path. The admin copies the `.tar` or `.tar.age` into the volume with `docker compose cp` and runs the CLI (`docs/guide/backups.md`).

### Evidence

`docs/architecture.md` §§9, 10, 11, 12, 14, 16, 17; `AGENTS.md`; `packages/adapters/src/queue-sqlite/index.ts`; `packages/db/src/backup.ts`; `apps/server/src/{restore,runtime,main}.ts`; `apps/server/src/http/{backups,realtime}.ts`; `apps/web/src/lib/realtime.ts`; `packages/core/src/event-bus.ts`; `extensions/lists/test/offline.test.ts`; `.github/workflows/ci.yml`; `CHANGELOG.md`; `apps/e2e/sweep/sweep.spec.ts`; `packages/core/test/search-bench.test.ts`; `packages/adapters/test/http-guard.test.ts`; `docs/guide/{languages,mcp,telegram,backups,install}.md`.

---

