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
