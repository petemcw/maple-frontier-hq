# ADR-0028: Timers are Home HQ's own and shared by the household, with no voice timers or appliance cycles

**Date**: 2026-10-19
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

The kitchen-tablet thread shows voice timers from Google speakers as huge countdowns readable from the stove (RED-07), washer and dishwasher progress with a "Laundry is done" card (RED-09), and a tablet that jumps back to Home when a timer starts (RED-30). Cook mode already promises that tapping a time in a step starts a timer (ADR-0021). HA has no `timer` helpers, no Google speakers, and no voice satellites. It has no appliance integration, and no power metering on any appliance. The family has an Amazon Alexa in the kitchen, which isn't in HA. HA's core Alexa Devices integration exposes only the next timer's end time, with no label, polled every 5 minutes. The HACS Alexa Media Player exposes every timer, but issues report its timer sensors and push stream failing, and neither integration can cancel a timer. Both use Amazon's unofficial API. The 21.5" wall panel hangs in the entry, not the kitchen. Research: "Inventory Maple Frontier's Home Assistant" (`research/ha-inventory`).

## Decision

- **Timers belong to Home HQ, not to a speaker.** The server holds each timer's label, duration, end time and state, and pushes changes over `/ws`. A timer belongs to the household. Every screen shows it, kiosks and phones alike, and anyone may cancel it, add a minute, or pause and resume it.
- **Starting one.** Tapping a time in a cook mode step starts a timer labeled with the recipe and step ("Chili · Step 4"). New timer, from the timers sheet on any screen, takes a duration and an optional label. Without a label it reads "10 min timer".
- **Display.** Running timers show on every screen. Where they sit is the Today layout ticket's call. Starting a timer never moves the wall panel to Today, so RED-30's jump isn't adopted.
- **Time's up.** At zero the timer becomes a third banner, "Time's up · {label}". It clears everywhere at once (ADR-0027). Every screen with Home HQ open chimes for 1 minute, then the banner stays silent until someone dismisses it or 30 minutes pass. +1 minute on the banner snoozes it. The family member who started the timer gets one time-sensitive Companion push (ADR-0012). A timer started on a kiosk has no starter, so it pushes nobody.
- **No voice timers (RED-07).** Home HQ doesn't read the Alexa's timers. The Alexa keeps ringing its own, in the kitchen where the cook is.
- **No appliance cycles (RED-09).** Running appliances and "done" cards are out of scope, since nothing in the house reports a cycle. A metering plug or clamp bought later would start a fresh effort, as with the doorbell (ADR-0027).

## Alternatives Considered

### Alexa timers through Alexa Media Player

- **Pros**: "Alexa, set a pasta timer" would show up on every screen, which is what RED-07 shows.
- **Cons**: an unofficial API, reported failures in timer sensors and labels, and no cancel.
- **Why not**: the family wanted tap-only unless the Alexa could be used cleanly. The cook hears the Alexa anyway.

### An Assist voice satellite or Google speakers

- **Why not**: new hardware for a feature tap already covers. Google's route also depends on an unofficial API and a master token.

### A timer only on the starter's devices

- **Pros**: simpler, with no shared state.
- **Why not**: it's what the iPhone Clock app already does. A shared timer lets someone else take over the stove.

### Jump the wall panel to Today when a timer starts (RED-30)

- **Why not**: the panel is in the entry, where nobody cooks, and running timers already show on every screen.

### Chime until dismissed

- **Why not**: a timer forgotten with nobody home would chime in the entry all evening.

### A manual "laundry is done" timer as a stand-in for RED-09

- **Why not**: New timer covers it with a label, and nothing about an appliance would be known.

## Consequences

### Positive

- Timers need no hardware, no HA change and no outside account.
- The banner model from ADR-0027 covers Time's up, including dismissal everywhere at once.
- Timers are Home HQ data, so they keep running while the HA connection is down.

### Negative

- Voice timers on the Alexa and timers in Home HQ are separate, so neither shows the other's.
- A cook whose phone is locked and who didn't start the timer hears it only from an open screen.
- No laundry or dishwasher progress.

### Risks

- The 1-minute chime may be missed when no screen is open nearby. Mitigation: the starter's push, and +1 minute on the banner.
- Five of the six phones have no Companion app yet (ticket "HA accounts and the Companion app for every family member"). Until they do, only on-screen chimes reach those members.
