# ADR-0023: Tasks replace chores, in one list for routine work and one-off to-dos

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams
**Supersedes**: ADR-0022

## Context

ADR-0022 settled chores: routine household work that repeats once, daily or weekly, with a fixed person, a rota or Anyone, and no stars or routines. FamilyOS also keeps a family task list beside its chores (FOS-19): one-off or repeating to-dos with an assignee or Anyone, a due date and time, and a reminder. ADR-0015 put tasks in scope. A chore that comes due once, with a fixed person or Anyone and an optional due time, is already a task under another name, so the line between them couldn't be drawn. "Chore" also reads as kids' work in a household of adults with no kid role (ADR-0016).

## Decision

- A **task** is anything a family member or Anyone needs to do: the dishes, trash night, mowing, booking the dentist, fixing the gate latch. There are no chores. Upkeep tied to a record stays a duty (ADR-0018), and monthly work goes on the home record as a duty.
- No stars, points or rewards. Credit is who did it and when. No routines. A dated task may have a due time ("by 7 AM").
- A task is either **one-off** or **repeating**. A one-off has an optional date, and a time only with a date. A repeating task repeats daily, or weekly on chosen days, from a first date, with an optional end date.
- **Who's up**: a fixed person or **Anyone**. A repeating task may instead take turns along an ordered **rota**.
- Anyone may tick any task. A tick on a phone credits the signed-in member, who can pick someone else with "Done by…". A kiosk always asks "Who did it?".
- A tick passes the turn to whoever follows the turn holder on the rota. Favors aren't tracked. A missed turn stays with the person who missed it.
- Ticking a repeating task completes its next open **occurrence**, even ahead of its day, and the schedule doesn't move. It turns overdue after its due time, or at the end of its day if it has no time. When the next occurrence comes due, the missed one is dropped as missed, so only one is ever open.
- A one-off turns overdue the same way and stays open and overdue until someone ticks it, changes its date or deletes it. A one-off with no date is never due or overdue.
- Anyone may undo a completion, without confirming: a repeating task until its next occurrence comes due (undo restores the turn), a one-off until the end of the day it was ticked. After that a done one-off can be reopened from its history.
- Anyone may **snooze** an open, dated task from its row or attention card: an hour, to the **evening check**, or to tomorrow's morning hour, and never past a repeating task's next occurrence. It isn't overdue until then, and its reminder fires again. There's no limit, and each snooze is logged. Undated one-offs can't be snoozed.
- Reminders go through the Companion app (ADR-0012) to whoever's up: one at the due time (the morning hour, 8 AM, for untimed tasks), and one more if it's still open, at the due time or at the evening check (7 PM) for untimed tasks. Admins set both hours. Anyone tasks and undated one-offs never send a notification.
- A member may put themselves on a **task break** until a date, and an admin may do it for anyone. Their turns are skipped and their fixed tasks fall to Anyone. If everyone on a rota is on a break, the task becomes Anyone.
- The **Tasks** screen groups tasks into Overdue, Today, Coming up, **Someday** (one-offs with no date) and Done (ticked today), filtered Everyone or Mine. Mine shows my fixed tasks, my turns and Anyone tasks.
- A one-line add at the top of the Tasks screen makes a one-off for Anyone with no date. "Add details" opens the full form.
- Every task is family-wide. There are no private tasks.
- Every member may add, edit and archive tasks. The owners are the creator and, for a fixed-person task, that person. Deleting someone else's task is a delete request (ADR-0016).
- Tasks don't link to HA entities.

## Alternatives Considered

### A task list beside chores (FamilyOS)

- **Pros**: covers FOS-19 as written, and keeps routine housework apart from errands.
- **Cons**: a once chore and a one-off task are the same row on two screens, with two reminder paths and two attention cards.
- **Why not**: no line between them held up.

### One-off tasks beside repeating chores

- **Pros**: each kind of work has one home, and chores lose "once".
- **Cons**: still two screens and two models for nearly identical rows.
- **Why not**: the family preferred one list.

### Keeping the name "chore"

- **Pros**: no rename of ADR-0022's terms.
- **Why not**: it reads as kids' work, and "call the plumber" isn't a chore.

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

### Dropping a missed one-off at the end of its day

- **Pros**: the list stays tidy, as it does for repeating tasks.
- **Why not**: "renew the passport" would vanish without being done.

### Private tasks

- **Why not**: Home HQ is a family hub. Personal to-dos belong in each person's own phone apps, and kiosks and Today would have to hide them.

### Following HA presence for breaks

- **Why not**: only Peter has GPS presence today, and being out for the day isn't the same as being off tasks.

## Consequences

### Positive

- One model, one screen and one reminder path for all of the family's work, with no role checks in the task logic.
- FOS-19 is covered without a second list.
- Missed repeating tasks never pile up, so a weekend away doesn't leave a backlog.
- The work still gets done when someone is away, because their fixed tasks fall to Anyone.

### Negative

- ADR-0022's terms change in the glossary and backlog: chore break becomes task break, and the chores check becomes the evening check.
- FOS-19 is narrowed: no monthly repeats, and Done shows only today's ticks instead of the last 7 days.
- The Tasks screen mixes routine housework with errands.
- Two more admin-only settings (the reminder hours), extending ADR-0016's list.
- Every-other-week and monthly work can't be a task. It becomes a duty on the home record.

### Risks

- Someday fills with tasks nobody is reminded about. Mitigation: it is always on the Tasks screen, and the What needs attention work decides whether old Someday tasks surface there.
- Anyone undoing anyone's tick could be misused. Mitigation: the completion history shows who did what, and this is a household of adults.
- Two notifications per task per person can feel like nagging. Mitigation: Anyone and undated tasks never notify, and quiet hours (the notifications work) still apply.
