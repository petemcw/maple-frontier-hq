# ADR-0018: Household records keep duties in Home HQ, linked to HA sensors, with papers on the NAS

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Household records (FOS-37 to FOS-39, RED-17) are in scope (ADR-0015). Maple Frontier has a home, things, services, vehicles and pets to keep track of. HA tracks the HVAC filter and the fridge water and air filters today, with `input_datetime` and `input_number` helpers, days-remaining sensors, reset scripts and an automation on `binary_sensor.hvac_filter_tray_contact`. That setup lives in HA only because nothing else could hold it. Papers (policies, registrations, manuals) need a home that the family can also browse outside Home HQ. ADR-0016 leaves every action open to members except a short admin-only list.

## Decision

- **Duties live in Home HQ** (ADR-0008), on records: the home (one fixed record), things, services, vehicles and pets. They use FamilyOS's model: repeats every N weeks, months or years, anchored to the first due date. Each has a lead time, an assignee or Anyone (Anyone reminds the admins), a completion note and undo. Records carry dates only, no money.
- **HA sensors can complete duties.** A duty may link one HA entity and the state that triggers it. Reaching that state either raises a sensor prompt ("Was the furnace filter changed?") or completes the duty automatically, set per duty. Prompts go to a Companion notification with Yes/No (ADR-0012), an attention card and every kiosk. One answer clears them all. Duty state is household data, not device state, so it is persisted, unlike the HA mirror (ADR-0004).
- **HA's filter helpers, scripts and automation are retired** once Home HQ has taken over filter tracking. That is an HA config change to agree with the family at the time.
- **Papers live on a NAS share** mounted into the container, as a readable tree `Records/<kind>/<record>/`. It is two-way. A scan files hand-dropped files as unfiled papers and follows renames and moves by content. A paper whose file was deleted by hand is flagged as missing.
- **Admin-only additions to ADR-0016**: adding and viewing papers, viewing account numbers, and editing duty presets. The NAS share stays open to the whole family, so the paper limit holds inside Home HQ only.

## Alternatives Considered

### Keep filters in HA, show them in Home HQ

- **Pros**: no migration; HA automations keep working.
- **Cons**: two places hold recurring household jobs, and HA's helpers have no history, assignee or reminder lead.
- **Why not**: HA holds them only for lack of anything better.

### Mirror duty dates back into HA helpers

- **Pros**: HA dashboards and automations keep a replace-by date.
- **Cons**: two copies to keep consistent, and writes through the service user.
- **Why not**: nothing in HA needs the date once Home HQ reminds.

### Papers on Home HQ's own data volume

- **Pros**: one backup (ADR-0008), served only through the API, no network mount.
- **Cons**: invisible outside Home HQ.
- **Why not**: the family wants papers browsable as ordinary files on the NAS.

### Papers as links only

- **Why not**: papers have no other home today.

## Consequences

### Positive

- One place for recurring household jobs, with history and reminders, while HA sensors still feed them.
- Papers are ordinary files on the NAS, backed up with it, and survive Home HQ.

### Negative

- Home HQ depends on a NAS mount from the iot host, with its own NAS account and credentials kept out of the repo.
- The folder scan and content matching add work that a Home HQ-owned store wouldn't need.
- Members browsing the NAS can read papers that Home HQ hides from them.

### Risks

- NAS unreachable: papers can't be opened or added. Mitigation: records and duties still work. Papers show as unavailable, never as missing, until a scan has run against a reachable share.
- A flapping sensor raises repeated prompts. Mitigation: one open prompt per duty, and repeat triggers don't add another.
- Auto-complete on a sensor misfire. Mitigation: it's opt-in per duty, and undo is on the duty.
