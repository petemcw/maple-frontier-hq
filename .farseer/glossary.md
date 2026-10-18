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

## Access

Who and what signs in, and how Home HQ reaches Home Assistant. See ADR-0004 to ADR-0006.

### Language

**Kiosk**:
A wall panel or shared tablet signed in as its own HA user with the display role. Its actions count as the household.
_Avoid_: display account, frame, wall display (in code)

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
Whoever may delete an item outright: the person who added it, or the person it is for. The household owns what a kiosk added, and Family calendar events with no known creator.
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
_Avoid_: favourites, staples

**Got it**:
The drawer of ticked-off items, from which an item can be added back. It empties itself after 7 days.
_Avoid_: cart, completed, done list

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
