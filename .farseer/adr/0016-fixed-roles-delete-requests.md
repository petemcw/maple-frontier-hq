# ADR-0016: Three fixed roles, and delete requests for others' items

**Date**: 2026-10-18
**Status**: accepted (admin-only list extended by ADR-0018)
**Deciders**: Pete McWilliams

## Context

ADR-0005 listed four roles, including kid, taken from a prototype written for children of 10 and 6. Maple Frontier's children are 21, 19, 17 and 17. FamilyOS adds Supervised members whose changes become drafts, and custom roles built from per-feature capabilities (FOS-48). Settles the approval part of brainstorm section 18 question 8 and revises ADR-0005's role list. Whether AI changes need approval is question 16, decided separately.

## Decision

- Roles are fixed and checked in code: **admin**, **member** and **display**. There is no kid, Supervised or custom role, and no capability table. Whether guests need a role is decided with guest mode.
- Both parents are admins. The four children are members.
- Only admins manage members and roles (including letting in an unmapped HA user), kiosks, and integrations: iCloud, Immich, the AI key and spending limit, and the weather location. Every member may change layout overrides, quiet hours and the mode switches.
- Changes take effect at once, with one exception. A member who deletes a calendar event, household record, recipe, chore or task they don't own creates a **delete request**. The item stays until an owner or an admin approves. Unanswered requests expire after 7 days, or when the event ends. Shopping items aren't covered.
- An item's **owners** are the person who added it and the person it is for. An event with no known creator (added straight in iCloud) belongs to the person whose calendar it's on. On the shared Family calendar it belongs to the household.
- A kiosk deletes only what the household owns. Anything else goes through a delete request, as it would for a member.

## Alternatives Considered

### Keep the kid role for the twins

- **Pros**: matches the prototype's chore logic.
- **Cons**: built for two people who turn 18 within two years.
- **Why not**: a role built only to be retired.

### FamilyOS Supervised drafts and capabilities

- **Pros**: fine-grained, and you can propose a change instead of being refused.
- **Cons**: an approval queue for every change, plus a capability table per feature.
- **Why not**: too heavy for six adults and near-adults. Losing an item is the one real risk, and delete requests cover it.

### No approval at all

- **Pros**: simplest. Attribution and undo cover mistakes.
- **Cons**: iCloud deletes can't be undone, and a lost record may go unnoticed for months.
- **Why not**: the family wants protection against tidying away someone else's things.

### Only the creator owns an item

- **Why not**: a twin couldn't remove their own dentist visit that a parent added.

## Consequences

### Positive

- One small permission check: role, plus ownership on delete.
- No kid-specific logic. Chores and stars no longer hang off a role.

### Negative

- Home HQ records the creator and the subject of every covered item, including on iCloud events it writes.
- An approval queue exists for delete requests and needs a place in the app and in notifications.

### Risks

- Deleting directly in iCloud or another app bypasses the rule. Mitigation: accepted; the rule protects against accidents in Home HQ, not deliberate deletes elsewhere.
