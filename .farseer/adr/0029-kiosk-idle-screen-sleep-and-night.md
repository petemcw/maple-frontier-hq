# ADR-0029: Kiosks go idle to ambient by day and a night clock at night, and the wall panel sleeps its screen by camera motion

**Date**: 2026-10-19
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

The prototype's frame uses touch-only idle timers (SHELL-04..08): back to Today after 60 s, then ambient from 06:00 to 22:00 or a night clock from 22:00 to 06:00, never dark. The Reddit panel wakes and dims by a kitchen mmWave sensor and shows an art screensaver against burn-in (RED-24, RED-25, RED-46), and FamilyOS gives each display its own grants and look (FOS-11, RED-44). Question 15 asks whether a presence sensor can wake the screen. The wall panel hangs in the entry (ADR-0028), an Android all-in-one running Fully Kiosk (ADR-0013). Nothing in the entry senses motion, and Fully Kiosk isn't in HA yet. HA already defines night with `binary_sensor.household_nighttime` (22:45 to 08:00), which the garage alert uses (ADR-0027). The main-floor lamps come on at sunset −45 min when someone is home. Research: "Inventory Maple Frontier's Home Assistant" (`research/ha-inventory`). Settles brainstorm section 18 question 15.

## Decision

- **Day, evening and night.** Night is when `binary_sensor.household_nighttime` is on, the same night the garage alert uses. Evening runs from sunset −45 min, when the lamps come on, until night. Kiosks dim in the evening, with lower brightness and a darker gradient behind ambient's text. RED-24's time-of-day brightness is adopted this way. SHELL-06's fixed 06:00 and 22:00 are dropped.
- **Idle timers.** A touch anywhere resets the timer (SHELL-04). After 60 s idle, sheets close and the kiosk goes to Today (SHELL-05). After 120 s it shows ambient by day and the night clock at night (SHELL-06). SHELL-07 is moot, since Home HQ has no takeovers (ADR-0027).
- **Ambient (AMB-01..03).** The shared slideshow at its 15 s pace, weighted on kiosks by anyone's favorites (ADR-0025), with each slide's place-and-date caption. Over it: the clock and date, the weather, and the next Family calendar event, as Next up counts it (ADR-0024). Presence avatars wait for the Presence and ETA decision. A tap goes to Today.
- **Night clock (NIGHT-01).** On black in dim red: a big clock; the weather with "First up tomorrow: {title} {time} · {person}" ("today" after midnight, or "Nothing early tomorrow"); and the house status line (ADR-0027), such as "Garage closed · 3 lights on · 68°". The hardcoded "House locked · alarm armed" is gone (DEF-05).
- **Night dashboard (NIGHT-02).** A tap on the night clock opens the full interface under a dim red-amber filter. It works normally and goes back to the night clock after 20 s idle (SHELL-08).
- **Screen sleep (question 15).** Wake comes from the panel's own front camera, through Fully Kiosk's motion detection, not from a new sensor. Home HQ's page drives the screen through Fully's JavaScript interface: it listens for motion, and turns the screen on, off and dim itself. By day the screen goes off after 10 min with no motion, and motion wakes it to ambient. At night it stays off, and motion or a touch brings up the night clock, which goes dark 1 min after the last motion. The HA Fully Kiosk integration isn't needed for any of this.
- **Banners on an idle kiosk.** Garage left open, Delivered and Time's up wake a sleeping screen and show over ambient or the night clock without leaving it. A tap on the banner acts on it, and a tap elsewhere does what that screen's tap does. The screen then sleeps by the usual motion rules, and the banner is still there when it wakes. Time's up chimes at night too.
- **Burn-in (NFR-12, AMB-02).** Ambient and the night clock shift their text by up to 4 px every minute. Screen sleep does most of the work, so RED-25's and RED-46's screensavers aren't adopted.
- **Kiosk settings.** An admin sets each kiosk's theme (system, light or dark), and turns its ambient, night clock and Time's up chime on or off. With ambient off, the kiosk stays on Today when idle by day. With the night clock off, its nights look like its days. A new kiosk starts with everything on and the system theme. Screen sleep isn't a setting: it runs wherever Fully Kiosk does, and a kiosk in a plain browser never sleeps its screen. FOS-11's per-display grants, palettes and hidden blocks aren't adopted, and RED-44's per-display pop-up switches shrink to the chime.

## Alternatives Considered

### Night from sunset −45 min

- **Pros**: matches the lamps automation, which the family already lives by.
- **Cons**: the red clock and a dark screen from 5 PM in winter, and a night that differs from the garage alert's.
- **Why not**: the family chose evening dimming at the lamps' time, and kept night as HA's.

### A presence sensor or ESPresense receiver in the entry

- **Pros**: mmWave senses someone standing still, which a camera may miss.
- **Cons**: hardware to buy and agree on. ESPresense hears phones, not people, so guests and phoneless family members wouldn't wake it.
- **Why not**: the panel's camera is already there, and ADR-0013 counted on Fully's motion wake.

### Drive the screen from the server through HA's Fully Kiosk integration

- **Pros**: the panel shows up in HA as a device.
- **Cons**: screen wake depends on the HA connection and on a service-user permission, and adds a round trip to every wake.
- **Why not**: the page knows the idle state already, and Fully's JavaScript interface works locally.

### Leave it to Fully Kiosk's own screen-off timer and motion wake

- **Why not**: Fully can't know the night window, the 10-minute rule, or that a banner just fired.

### Ambient all day, or the night clock always on

- **Why not**: an empty entry doesn't need a slideshow, and a dark entry at night is better than a red glow nobody is looking at.

### FOS-11's per-display grants and look

- **Why not**: much to build for one fixed kiosk. Four switches cover the cases the family named.

## Consequences

### Positive

- No new hardware: the panel's own camera wakes it.
- One night across Home HQ, set in HA.
- The panel is dark when nobody's around, which saves power and burn-in.
- Screen sleep keeps working while the HA connection is down.

### Negative

- Camera motion misses someone standing still, so a person reading ambient may see it sleep after 10 min.
- The panel needs a Fully Kiosk Plus license, with motion detection, camera access and the JavaScript interface turned on.
- A kiosk in a plain browser can't sleep, so its night clock stays lit all night.
- Kiosk settings are another admin surface to build.

### Risks

- Fully's motion detection can trigger on light changes, such as headlights or moving sunlight. Mitigation: tune its sensitivity on the device, and the 1-minute night timeout limits the effect.
- Fully's JavaScript interface is vendor-specific. Mitigation: keep it behind one small screen-control module that does nothing outside Fully.
