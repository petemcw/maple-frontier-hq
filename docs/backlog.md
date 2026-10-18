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
