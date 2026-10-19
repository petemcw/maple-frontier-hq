# ADR-0020: One household region format, taken from Home Assistant

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

The prototype used Celsius, a 12-hour clock and Canadian spelling, and NFR-05 asked for locale settings. FamilyOS keeps a family language and region format with per-member overrides and language packs (FOS-53). Maple Frontier is in the US: HA is set to US customary units (°F, mi, lb, gal), America/Chicago, country US, language en, and the ecobee reports °F. All six family members read English. HA's core config has no clock, week start or date format.

## Decision

- There is one **region format** for the household. Family members can't override it.
- The server reads it from HA's config (`unit_system`, `time_zone`, `country`, `language`) on every connect. Clock, week start and date styles come from `Intl` for HA's language and country, which is en-US today: 12-hour, Sunday, "Sat, Oct 18".
- The server keeps the last region format it read in SQLite and serves it to clients. This is HA config, not device state (ADR-0004). Before the first connect it falls back to en-US, America/Chicago and °F.
- Every device shows times in the household time zone, even a phone in another zone.
- Values display in the unit HA reports them in. Home HQ doesn't convert. Open-Meteo is asked for the household's units.
- The app is English only, with no i18n layer. Dates, times, numbers and units always go through `Intl` with the region format and are never built by hand.
- App copy, code, the glossary and docs use American spelling.
- Shopping adds read US and metric units as typed and never convert them. The decimal point is the only decimal separator, so a comma always splits items ("1,5 l" is two items).

## Alternatives Considered

### Home HQ's own region settings, seeded from HA

- **Pros**: an admin can change them in the app without touching HA.
- **Cons**: two places to keep in step, and a unit other than HA's means converting every sensor value and every thermostat setpoint.
- **Why not**: HA is already set correctly, and following it means converting nothing.

### Per-member overrides (FamilyOS)

- **Pros**: someone who prefers a 24-hour clock or °C gets it on their own phone.
- **Cons**: the phone and the kiosk show different formats, and every formatter needs to know who is looking.
- **Why not**: nobody in the household asked for one.

### Device time zone

- **Why not**: a twin away at school would see dinner at a time that doesn't match the kiosk. Home HQ is about life at home.

### Language packs or an i18n-ready catalog

- **Why not**: one language. Routing strings through a catalog costs effort on every screen for a second language nobody needs.

### Accepting "1,5" as a decimal

- **Why not**: the comma also splits items, so "1,5 l" is ambiguous. The US writes decimals with a point.

## Consequences

### Positive

- No locale settings screen, and the region format changes in one place.
- Sensor values and setpoints pass through unconverted.

### Negative

- Changing the region format means changing HA's config, which needs an HA admin.
- A family member who wants °C or a 24-hour clock can't have it.
- FOS-53 is ruled out, and NFR-05's "locale settings" become "follows HA".

### Risks

- If someone changes HA's unit system, every value Home HQ shows changes with it. Mitigation: that's the point of following HA. Values are always shown with their unit.
- A value from outside HA, like weather or an AI suggestion, could arrive in the wrong units. Mitigation: request the household's units, and format through `Intl`.
