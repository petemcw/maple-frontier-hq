# ADR-0033: Attention is a Today section of per-item cards, shown to whoever can act

**Date**: 2026-10-28
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Four earlier decisions left "where it appears" to the "What needs attention" ticket: delete requests (ADR-0016), sensor prompts (ADR-0018), overdue tasks and their snooze (ADR-0023), and house problems (ADR-0027). FamilyOS's Needs you (FOS-05) is person-only and never shown on a display. The Reddit dashboards use a header health pill (RED-06) and a notification sheet with Alerts and Active tabs (RED-51). The wall panel in the entry is the only kiosk (ADR-0032), and the family treats it as trusted (ADR-0006).

## Decision

- **Where.** Attention is one section on Today, labeled "Needs attention" on every device. The Today layout places it and sets how many cards fit. When nothing is waiting, the section leaves Today. No other screen shows attention.
- **What counts, and who sees it.**
  - Sensor prompts: the duty's assignee, or the admins for an Anyone duty, and every kiosk.
  - Delete requests: every owner of the item and both admins. The first answer clears it everywhere. Never on a kiosk.
  - Overdue tasks: on a member's device, their fixed tasks, their turns and Anyone tasks. On a kiosk, every overdue task with who's up.
  - Overdue duties: the assignee, or both admins for an Anyone duty, and every kiosk.
  - House problems: both admins and every kiosk.
- **Not attention.** Banners (already on every screen), RED-51's "on or open now" list (the house status line counts lights, and Home control has All off), waiting quick add suggestions (the bot's reply and the quick add tray hold them), and Someday tasks, however old.
- **Cards.**
  - One attention card per item, up to the layout's cap, then "+{n} more" opens the full list.
  - Order: sensor prompts and delete requests first, since they wait on an answer. Then overdue tasks and duties, most overdue first. House problems last.
  - Actions:
    - Sensor prompt: Yes or No.
    - Delete request: Delete or Keep.
    - Overdue task: Done or Snooze (ADR-0023).
    - Overdue duty: Done or Move date.
    - House problem: Snooze.
  - On a kiosk, Done asks "Who did it?".
- **Move date** moves only this due date of a duty, never past the next period's date, and a repeating duty stays anchored to its first due date. Anyone who sees the card may move it, a kiosk included. The move goes in the duty's log with who made it.
- **Snoozing a house problem** sets it aside until it gets worse:
  - A battery comes back after another 5% drop.
  - A fault sensor or a failed backup comes back the next time it fires.
  - A problem that clears and returns is a new one.
  - Admins and kiosks may snooze. It clears on every screen at once, and Home HQ records who or what snoozed it.
- **Live.** The server works out each viewer's attention and pushes changes over `/ws`. An item appears or leaves on every device as its cause changes. House problems come from the HA mirror, so they're withdrawn while HA is down (ADR-0004). Prompts, tasks, duties and delete requests are Home HQ's own, so they stay.

## Alternatives Considered

### A header chip and sheet on every screen (RED-06, RED-51)

- **Pros**: visible from any screen, and a small footprint.
- **Why not**: phones already get pushes for most items, so a second place adds noise. Today is where the family looks.

### Person-only, never on a kiosk (FamilyOS)

- **Pros**: nothing about anyone sits on a shared screen.
- **Why not**: whoever walks past the panel can tick a task or change a filter. The panel can already open the Tasks screen.

### One card per kind ("3 overdue tasks")

- **Pros**: a fixed size that always fits on the panel.
- **Why not**: every action costs an extra tap.

### Everyone's overdue tasks on every member's device

- **Why not**: it nags people about other people's jobs. The Tasks screen and the panel show them all.

### Waiting suggestions as attention (FamilyOS)

- **Why not**: the bot's reply and the quick add tray already hold them, and attention stays about overdue and broken things.

### Old Someday tasks resurfacing

- **Why not**: an undated task has nobody waiting on it. Anything that needs a nudge can be given a date.

### House problems that only clear themselves, or a fixed week's snooze

- **Why not**: a battery at 19% would sit on the entry panel for weeks. A week's snooze would bring it back unchanged, or hide a battery that kept draining.

### Move date that moves the whole schedule

- **Why not**: one late filter change shouldn't shift every later due date. It matches how a task's snooze leaves its schedule alone.

### "Needs you" as the label

- **Why not**: on the panel nobody is "you". One label everywhere matches the glossary.

## Consequences

### Positive

- The attention cards that earlier ADRs mentioned now have one home and one set of rules.
- The panel shows household jobs a passer-by can act on, and the things only a parent can answer stay on their phones.
- Snooze is one word for "not now, ask again later", on a task or a house problem.

### Negative

- Guests at the entry panel see overdue tasks with names, and house problems. The guest mode ticket decides whether guest mode hides them.
- Someone on Shopping or Recipes sees no attention until they open Today.
- A kiosk can push a duty later as the household.

### Risks

- The panel crowds when a lot is overdue. Mitigation: the layout's cap and "+{n} more".
- A backup that keeps failing comes back the day after every snooze. That's intended, since each failure is new.
- A battery snoozed at 19% isn't seen again until 14%. Mitigation: the 5% step is small, and a dead battery's device goes unavailable on its tile.
