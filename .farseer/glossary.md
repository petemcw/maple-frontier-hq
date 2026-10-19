# Glossary

The canonical terms for this project's domain. `/farseer:domain-modeling` keeps it current.

## Household

The home and the people Home HQ serves.

### Language

**Maple Frontier**:
The family's home, and the household Home HQ serves.
_Avoid_: the house (when the specific household is meant)

**Home HQ**:
The family and home hub this repository builds, used by every family member from their own devices.
_Avoid_: dashboard (alone), app, photo frame

**Family member**:
One of the six people who live at Maple Frontier and use Home HQ.
_Avoid_: user (when a specific person is meant), resident

**Region format**:
How Home HQ writes times, dates, numbers and units for the whole household: its time zone, unit system, clock, week start and date style, taken from HA's config. There is one, shared by every family member and kiosk.
_Avoid_: locale (alone), language, preferences

## Access

Who and what signs in, and how Home HQ reaches Home Assistant. See ADR-0004 to ADR-0006.

### Language

**Kiosk**:
A wall panel or shared tablet signed in as its own HA user with the display role. Its actions count as the household.
_Avoid_: display account, frame, wall display (in code)

**Wall panel**:
The 21.5" touch panel in the entry, Maple Frontier's one fixed kiosk.
_Avoid_: kitchen panel, kitchen screen (it isn't in the kitchen)

**Household**:
The actor recorded for an action taken on a kiosk, when no family member is named.
_Avoid_: family (as an actor), anonymous

**Role**:
What a signed-in member or kiosk may do in Home HQ: admin, member or display. Stored in Home HQ, not HA. There is no kid role.
_Avoid_: permission level, user type, kid, supervised

**Admin**:
A family member with the admin role. Both parents are admins.
_Avoid_: owner, parent (as a role)

**Owner**:
Whoever may delete an item outright: the person who added it, or the person it is for. The household owns what a kiosk added.
_Avoid_: author, assignee (when the delete right is meant)

**Delete request**:
A request from a member or a kiosk to delete an item they don't own. It takes effect only when an owner or an admin approves it.
_Avoid_: draft, proposal, suggestion

**Service user**:
The non-admin "Home HQ" user in Home Assistant whose token only the server holds. Every HA read and command goes through it.
_Avoid_: HA token (alone), bot user

**HA mirror**:
The server's live, in-memory copy of current entity states, rebuilt from HA on every connect and never persisted.
_Avoid_: cache, device store, state copy

## Shopping

What the family needs to buy, and where.

### Language

**Shopping list**:
A named list of things to buy, such as Groceries, Pharmacy or Hardware. Shared by the whole family.
_Avoid_: list (alone, where task lists are meant too), cart

**Store**:
Where an item will be bought, such as Costco. Any item on any shopping list may carry one, and an item remembers its last store.
_Avoid_: shop, vendor, list (for a store)

**Category**:
The aisle group an item is filed under, such as Produce or Pantry, shown in one household order. Remembered per item name for the whole family.
_Avoid_: section, aisle, department

**General list**:
The permanent shopping list named Shopping. Items with no list, and new items from the voice inbox, land here.
_Avoid_: inbox, default list, Shopping (in code)

**Voice inbox**:
HA's built-in shopping list, used only to receive adds from Assist, Siri and the HA app. Home HQ moves each item out and empties it.
_Avoid_: HA shopping list (as the family's list), HA list

**Usuals**:
The items most often bought lately from one shopping list, offered as one-tap adds on that list.
_Avoid_: favorites, staples

**Got it**:
The drawer of ticked-off items, from which an item can be added back. It empties itself after 7 days.
_Avoid_: cart, completed, done list

## Kitchen

The family's recipes and cooking from them. See ADR-0021.

### Language

**Recipe box**:
The family's one shared collection of recipes.
_Avoid_: cookbook, recipe list, library

**Recipe**:
A dish in the recipe box: title, ingredient lines, steps, servings, time, source, labels and a photo. Owned by whoever added it.
_Avoid_: meal (a meal plan is out of scope)

**Ingredient line**:
One line of a recipe's ingredients, read as an amount, a unit and the rest when it can be, so it can scale.
_Avoid_: ingredient (alone, for the line), item (a shopping item)

**Recipe draft**:
A recipe read from a link sent to the Telegram bot, seen only by its sender until they save it to the recipe box.
_Avoid_: proposal, pending recipe, import

**Cook mode**:
The recipe in large type, with ingredients and steps tickable and the screen kept awake, for cooking from a phone or tablet.
_Avoid_: kitchen mode, step mode

**Cook note**:
A dated note on a recipe, with who wrote it, such as "less salt, better".
_Avoid_: comment, review

## Timers

Countdowns the household runs in Home HQ. See ADR-0028.

### Language

**Timer**:
A labeled countdown that belongs to the household, started from a cook mode step or New timer. Every screen shows it, and anyone may cancel, pause or extend it.
_Avoid_: kitchen timer (it times laundry too), voice timer (the Alexa's own, which Home HQ doesn't see), alarm

**Time's up**:
The banner a timer becomes at zero. It chimes for a minute, then stays silent until dismissed or 30 minutes pass.
_Avoid_: ringing, timer done, alert

## Records

What the household keeps track of, and the jobs that come back. See ADR-0018.

### Language

**Record**:
Something the household keeps track of: the home, a thing, a service, a vehicle or a pet. A record can be archived when it leaves the household, keeping its history.
_Avoid_: asset, item, entry

**Home**:
The one record for Maple Frontier itself, which things and services belong to.
_Avoid_: house record, property

**Thing**:
An appliance or other object kept as a record, such as the fridge, the furnace or the mower, with its brand, model, serial and warranty.
_Avoid_: device (an HA device), appliance (alone), item

**Service**:
A utility, insurance or subscription kept as a record, such as electricity, internet or a streaming plan.
_Avoid_: subscription (alone), account, utility (alone)

**Duty**:
A dated job on a record that may repeat every so many weeks, months or years, such as renewing insurance or changing the furnace filter. Unassigned duties remind the admins.
_Avoid_: obligation, chore, task, reminder

**End date**:
A date on a record that asks nothing of anyone, such as a warranty end or a paper's valid-until. It reminds the admins ahead of time but is never done or overdue.
_Avoid_: lapsing duty, expiry, deadline

**Log**:
A record's dated list of what was done to it: services, treatments, repairs, filter changes. Completing a duty adds a log entry.
_Avoid_: history (alone), service log, treatment history

**Paper**:
A file attached to a record, with a label and an optional valid-until date. Only admins add or view papers.
_Avoid_: document, attachment, file (alone)

**Unfiled paper**:
A file found in a record's NAS folder that nobody has labeled yet.
_Avoid_: inbox item, orphan

**Preset**:
A suggested duty for a kind of record, such as a rabies shot for a pet, offered as a one-tap chip with a repeat and lead time. Admins edit presets.
_Avoid_: template, default duty

**Sensor prompt**:
The question "Was the furnace filter changed?" raised when a duty's linked HA entity reaches its trigger state. Answered once, it clears everywhere.
_Avoid_: done check, confirmation, alert

## Tasks

The work of running the house and the family's to-dos, from the weekly dishes to a one-off errand. Upkeep tied to a record is a duty, not a task.

### Language

**Task**:
Something a family member or Anyone needs to do: one-off, with an optional date, such as "book the dentist", or repeating daily or weekly on chosen days, such as the dishes or trash night. It has no stars or points.
_Avoid_: chore, to-do, job, routine, duty (upkeep on a record)

**One-off task**:
A task that comes due once, or never if it has no date. It stays open, and overdue once its day passes, until it's done.
_Avoid_: once chore, single task, errand

**Someday**:
The group of one-off tasks with no date.
_Avoid_: no date, backlog, later

**Who's up**:
How a task decides whose it is: a fixed person, Anyone, or, for a repeating task, take turns along a rota.
_Avoid_: assignment mode, strategy

**Rota**:
The ordered list of family members a take-turns task passes along.
_Avoid_: rotation list, roster, queue

**Turn**:
The family member on a task's rota whose go it is now.
_Avoid_: assignee (for a take-turns task), current owner

**Anyone**:
No one in particular: a task or duty any family member may do. Whoever does it gets the credit.
_Avoid_: unassigned, everyone, shared

**Occurrence**:
One coming-due of a repeating task, on a given day. Ticking the task completes its next open occurrence, even ahead of its day.
_Avoid_: instance, due date (for the thing itself)

**Task break**:
A stretch, until a set date, when a family member's turns are skipped and their fixed tasks fall to Anyone.
_Avoid_: chore break, away (HA presence), vacation, pause

**Evening check**:
The household's evening hour at which dated tasks still open without a due time get a reminder.
_Avoid_: chores check, deadline, nag, cutoff

**Snooze**:
Pushing an open, dated task later, by an hour, to the evening check or to tomorrow morning, but never past a repeating task's next occurrence. It isn't overdue until then.
_Avoid_: postpone, defer, skip

## Calendar

The family's events, read from Google Workspace and shown in Home HQ without editing. See ADR-0024.

### Language

**Family calendar**:
The one shared Google calendar, owned by a parent, that holds the events the family sees in full.
_Avoid_: shared calendar, household calendar, Family Sharing calendar

**Primary calendar**:
A family member's own default Google calendar. Home HQ sees it only as busy blocks.
_Avoid_: personal calendar, my calendar

**Busy block**:
A stretch of time when someone's primary calendar has something on, shown with no title or details.
_Avoid_: free/busy, private event, blocked time

**Everyone event**:
An event for the whole family: a Family calendar event with no family attendees or with all six, or an event from a calendar subscription.
_Avoid_: family event (in code), all-hands, shared event

**Calendar subscription**:
A read-only ICS feed, such as a school or US holidays, that an admin adds and Home HQ fetches itself.
_Avoid_: feed (alone), subscribed calendar, ICS calendar

**Drive time**:
The traffic-aware time to drive to an event's location, from the person's location or from home.
_Avoid_: travel time (in code), commute, ETA

**Leave-by**:
An event's start time minus its drive time.
_Avoid_: departure time, leave time

**Time to leave**:
The notification sent to an event's people 10 minutes before its leave-by.
_Avoid_: event reminder, alert, leave alert

## Photos

The family's photos, kept in Immich on the NAS and shown in Home HQ.

### Language

**Home HQ album**:
The one Immich album whose photos Home HQ shows. Family members add to it from their own Immich accounts.
_Avoid_: library, gallery, photo stream, feed

**Slideshow**:
Photos from the Home HQ album shown one slide at a time, laid out automatically.
_Avoid_: carousel, photo frame (the ambient screen), gallery

**Slide**:
One step of the slideshow: a landscape photo filling the screen, two portraits from the same day side by side, or a lone portrait over a blurred copy of itself.
_Avoid_: frame, card, page

**On this day**:
Photos from the Home HQ album taken on today's date in past years. They lead the slideshow and are captioned with how many years ago.
_Avoid_: memories (Immich's own, per owner), flashback, anniversary

**Favorite**:
A photo a family member has hearted in Home HQ, which comes up more often in their slideshow. Each member has their own, and a kiosk has none.
_Avoid_: star, like, Immich favorite (owner-only, not used)

**Hidden photo**:
A photo taken off every Home HQ screen while it stays in Immich and in the Home HQ album. Anyone may hide one, and only an admin may unhide it.
_Avoid_: deleted, removed, archived (Immich's own state)

**Photo of the day**:
The one photo the household sees for a day, the same on every screen, chosen by the server.
_Avoid_: daily photo, featured photo, hero (the Today block that may show it)

## Home control

The house's lights, garage, climate and cameras, run from Home HQ. See ADR-0011 and ADR-0026.

### Language

**Tile**:
The control for one labeled HA entity in Home control: a light, switch, fan, the garage or a camera. A tap acts on it, and a long press opens its sheet.
_Avoid_: card, widget, button, device (an HA device)

**Scene**:
An HA scene or script with the Home HQ label, run from a button in Home control and undoable. Home HQ never defines one.
_Avoid_: preset (a climate preset or a duty preset), routine, mode

**All off**:
Turning off every light in the house, or on one floor, at once, with Undo restoring exactly those lights.
_Avoid_: kill switch, Good Night (an HA scene)

**Room presence**:
The room HA places a family member in, from their phone heard in that room. It's missing when nothing hears the phone.
_Avoid_: location (GPS and zones), occupancy (a motion reading that doesn't say who)

## Alerts and house status

What Home HQ tells the family about the house without being asked. See ADR-0027.

### Language

**Banner**:
A notice that shows on every screen until someone acts on it, and clears everywhere at once. Home HQ has three: Garage left open, Delivered and Time's up.
_Avoid_: takeover (Home HQ has none), toast (a short confirmation of the viewer's own action), notification (a Companion push)

**Garage left open**:
The banner and admin push for a garage open 15 minutes, or 3 minutes at night. Leave open quiets it until the door closes, except that nightfall brings it back once.
_Avoid_: garage alarm, garage warning

**Parcel**:
A delivery the household knows about from the parcels mailbox. It belongs to the household, not to whoever ordered it, and Home HQ only knows about today's.
_Avoid_: package (the prototype's word), shipment, order

**Parcels mailbox**:
The dedicated Workspace mailbox that family members forward shipping emails to, read by Mail and Packages in HA, never by Home HQ.
_Avoid_: parcels inbox, voice inbox (the shopping one)

**House status**:
The Today line with the garage, the lights-on count and the inside temperature.
_Avoid_: security line, health pill, status bar

**House problem**:
Something in the house that needs fixing: a battery below 20%, a device fault, or a failed HA backup. An unavailable tile isn't one.
_Avoid_: alert, warning, issue
