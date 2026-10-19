# ADR-0027: Alerts are banners for the garage and parcels, with no takeovers

**Date**: 2026-10-19
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Brainstorm question 10 asks which devices back each alert and status line. ADR-0026 settled Home control and left alerts open. The prototype has two full-screen takeovers (doorbell, water leak), two banners (garage left open, package delivered), a "House locked · alarm armed" line, and a status line built on locks (ALERT-01..04, TODAY-17, DEF-05). The house has no doorbell, no leak sensors, no valve, no locks and no alarm in use. The porch camera gives HA no motion. The garage is a relay plus an open/closed sensor. Parcels have no buyer API, and email is the only free source that covers Amazon. Research: "Inventory Maple Frontier's Home Assistant" (`research/ha-inventory`) and "Parcel tracking sources" (`research/parcel-tracking`).

## Decision

- **No takeovers.** The doorbell takeover and its rules (ALERT-01, RED-40, DEF-06), the water leak takeover and valve (ALERT-02, RED-16), and the alarm with its PIN keypad (RED-57) are out of scope. Nothing in the house can trigger them. A doorbell, leak sensors or an alarm bought later would start a fresh effort.
- **Banners.** A banner shows on every screen, kiosks and phones alike. The server holds each banner's state, so dismissing it on one device clears it on all of them.
- **Garage left open (ALERT-03).** It fires when `binary_sensor.garage_door_status` has been open for 15 minutes, or for 3 minutes while `binary_sensor.household_nighttime` is on. A door already open past 3 minutes at nightfall fires at once. The banner reads "Garage open for {n} min · Opened at {time}" and offers Close garage, which opens the slide sheet, and Leave open. Leave open quiets it everywhere until the door closes, except that nightfall brings it back once. Closing the door clears it.
- **Garage push.** When the banner fires, both admins get a Companion push (ADR-0012). The push carries a Close garage action that iOS runs only after Face ID or the passcode (`authenticationRequired`). The action can close the door but never open it. Unlocking the phone counts as the deliberate gesture ADR-0006 asks for.
- **Parcels (ALERT-04, RED-19).** Mail and Packages (HACS, pinned to a release tag) runs in HA and reads a dedicated parcels mailbox in the family's Workspace. Each member's Gmail filter forwards carrier and shop emails to it. Home HQ reads only the integration's sensors and camera entities, so it holds no mailbox credentials (ADR-0004). Parcels are the household's, with no owner. While any are out for delivery, Today shows "{n} parcels today". Each delivery raises a "Delivered · {carrier}" banner with View photo, the carrier's delivery photo. OK on any device clears it. Deliveries send no push. Amazon locker and delivery codes show on every screen, kiosks included.
- **House status line (TODAY-17).** Garage open or closed, in the warning style when open, then the lights-on count and the inside temperature from the ecobee. Tapping it opens Home control. There's no lock or alarm wording anywhere.
- **House problems (RED-06).** A battery below 20%, named with its device, room and battery-type label. A Shelly or TP-Link fault sensor turning on. HA's daily backup failing. An unavailable tile is not a house problem, since its tile already says so. Where house problems appear belongs to the "What needs attention" ticket.

## Alternatives Considered

### A porch-camera pop-up in place of the doorbell

- **Pros**: something happens when a person comes to the door.
- **Why not**: wyzeapi gives HA no motion entity, its cloud motion polling is untested and slow, and holding the battery camera's stream open drains it.

### Leak sensors as a household purchase

- **Pros**: the one alert that protects against real damage.
- **Why not**: HA can push a critical alert itself if sensors are ever added. Home HQ doesn't need a takeover for hardware the house doesn't have.

### Enable the Aqara E1 alarm

- **Why not**: the family doesn't use an alarm, and an armed state nobody watches would mislead.

### Garage push to everyone at home

- **Pros**: whoever is nearest can close it.
- **Why not**: the parents wanted the garage to be their call, and five of the six phones have no Companion app yet (ticket "HA accounts and the Companion app for every family member").

### A garage alert the moment it opens after dark

- **Why not**: a 30-second drive-in at 11 pm would push to both parents every time. Three minutes clears a normal arrival.

### A one-tap Close garage action on the lock screen

- **Why not**: it skips ADR-0006's deliberate gesture. Requiring the phone's unlock keeps the speed and the safeguard.

### Parcels read by an LLM, or the Parcel app API

- **Pros**: multi-day ETAs and item names (LLM), or expected dates and Amazon sign-in (Parcel).
- **Why not**: the LLM route sends email bodies to a model provider and is more to build. Parcel is Apple-only and tracks one account's parcels. Mail and Packages keeps everything in HA.

### Hide Amazon codes on kiosks

- **Why not**: whoever's in the kitchen is often the one who picks up. The family accepts the panel as trusted (ADR-0006).

## Consequences

### Positive

- Every alert reads a real HA entity, and the fake defects (DEF-05, DEF-06) go with the features they belonged to.
- One banner model covers both alerts, and dismissal syncs through the server.
- Home HQ never holds mail credentials.

### Negative

- Parcels cover today only, with no "arriving Thursday".
- Household changes come before building: install Mail and Packages, create the parcels mailbox, and set up a Gmail forwarding filter for each member. The garage alert also needs the garage template cover from ADR-0026.
- Only the admins hear about an open garage away from a screen.
- Amazon codes are readable by anyone at the kitchen panel, guests included.

### Risks

- The parcels mailbox needs full Gmail scope, and an OAuth project left in testing logs out every 7 days. Mitigation: a dedicated mailbox, a project set to In production, and a house problem if its sensors go unavailable.
- Members forget to forward, or a shop changes its email format. Mitigation: the count is "parcels Home HQ knows about", nothing more.
- An unlocked phone can close the garage on someone under the door. Mitigation: close only, and the opener's obstruction reverse.
