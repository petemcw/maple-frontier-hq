# ADR-0022: Chores rotate between family members, with no stars and no routines

**Date**: 2026-10-18
**Status**: superseded by ADR-0023
**Deciders**: Pete McWilliams

## Context

The prototype's chores sit in time-of-day routines (Morning, After school, Evening) and pay kids stars. It has no schedules, rotation or daily reset (CHORE-10). FamilyOS has rotating chores with schedules and turn rules, and no stars or routines. Maple Frontier is two adults and four children aged 21, 19, 17 and 17, and ADR-0016 removed the kid role that stars and turn rules hung on. The two oldest may be away for weeks at a time. Duties (ADR-0018) already cover dated upkeep on records that repeats in weeks, months or years. Question 8 asked whether chores need stars, rotation, recurrence and a daily reset.

## Decision

- A **chore** is routine household work shared among family members: dishes, trash, laundry, mowing. Upkeep tied to a record stays a duty. Monthly work goes on the home record as a duty.
- No stars, points or rewards. Credit is who did it and when.
- No routines. A chore may have a due time ("by 7 AM").
- Repeats: once, daily, or weekly on chosen days, with an optional end date.
- **Who's up**: a fixed person, take turns along an ordered **rota**, or **Anyone**.
- Anyone may tick any chore. A tick on a phone credits the signed-in member, who can pick someone else with "Done by…". A kiosk always asks "Who did it?".
- A tick passes the turn to whoever follows the turn holder on the rota. Favors aren't tracked. A missed turn stays with the person who missed it.
- Ticking completes the next open **occurrence**, even ahead of its day, and the schedule doesn't move.
- A chore is overdue after its due time, or at the end of its day if it has no time. When the next occurrence comes due, the missed one is dropped as missed. Only one occurrence is ever open.
- Anyone may undo a completion, without confirming, until the next occurrence comes due. Undo restores the turn.
- Anyone may **snooze** an open occurrence from its row or attention card: an hour, to the chores check, or to tomorrow's morning hour, never past the next occurrence. It isn't overdue until then, and its reminder fires again. There's no limit, and each snooze is logged.
- Reminders go through the Companion app (ADR-0012) to whoever's up: one at the due time (8 AM for untimed chores), and one more if it's still open at the due time, or at the **chores check** (7 PM) for untimed chores. Admins set both hours. Anyone chores never send a notification.
- A member may put themselves on a **chore break** until a date, and an admin may do it for anyone. Their turns are skipped and their fixed chores fall to Anyone. If everyone on a rota is on a break, the chore becomes Anyone.
- Every member may add, edit and archive chores. The owners are the creator and, for a fixed-person chore, that person. Deleting someone else's chore is a delete request (ADR-0016).
- Chores don't link to HA entities.

## Alternatives Considered

### Stars with rewards (prototype)

- **Pros**: motivation for younger children, already designed.
- **Cons**: everyone is 17 or older, and the balance was what made someone a kid (DATA-10).
- **Why not**: no kid role, and nobody wanted a currency to manage.

### Time-of-day routines

- **Why not**: adults on different schedules don't share a morning, and "After school" no longer fits. A due time covers "trash out by 7 AM".

### Least done in the last 90 days (FamilyOS)

- **Why not**: harder to predict than a rota, and the family didn't want it.

### Swap turns when someone covers

- **Pros**: fairer over time.
- **Why not**: harder to follow. Credit already shows who did the work.

### Only the turn holder ticks (FamilyOS turn rules)

- **Why not**: blocks logging a favor, and FOS-29's rules existed to keep kids honest.

### Following HA presence for breaks

- **Why not**: only Peter has GPS presence today, and being out for the day isn't the same as being off chores.

## Consequences

### Positive

- One model for every adult in the house, with no role checks in the chore logic.
- Missed chores never pile up, so a weekend away doesn't leave a backlog.
- The work still gets done when someone is away, because their fixed chores fall to Anyone.

### Negative

- Two more admin-only settings (the reminder hours), extending ADR-0016's list.
- Every-other-week and monthly chores can't be chores. They become duties on the home record.

### Risks

- Anyone undoing anyone's tick could be misused. Mitigation: the completion history shows who did what, and this is a household of adults.
- Two notifications per chore per person can feel like nagging. Mitigation: Anyone chores never notify, and quiet hours (the notifications work) still apply.
