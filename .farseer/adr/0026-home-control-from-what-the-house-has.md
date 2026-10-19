# ADR-0026: Home control covers the devices Maple Frontier has, with scenes from HA and automations left in HA

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams
**Note (2026-10-19)**: the wall panel hangs in the entry, not the kitchen. Wording that called it the kitchen panel is corrected; the decision is unchanged (ADR-0028).

## Context

Brainstorm question 10 asks which devices the house has for each Home control tile, and question 17 asks whether automation settings belong on the wall. The prototype's Home control has locks, blinds, a garage with Opening and Closing, five fixed scenes, Celsius climate and two fake camera tiles (HOME-01..12). The house has no locks, covers or valve. Its garage is a Shelly relay plus an open/closed sensor. It has one ecobee in °F, 16 room lights on 4 floors, a few plugs and a fan, and three Wyze cameras whose HA integration gives no still image, only cloud WebRTC. Room presence today reaches about a floor, from Wi-Fi. Research: "Inventory Maple Frontier's Home Assistant" (`research/ha-inventory`) and "Finding which room a family member is in" (`research/room-location`). This session also checked the wyzeapi integration source and docker-wyze-bridge.

## Decision

- **Only what the house has.** Lock and blind tiles, "Lock all doors" and locking in scenes are dropped (HOME-07, HOME-08). So are per-room picture scenes (RED-54), climate by room (RED-15, RED-56), and camera events: the package chip, "No recent activity", clip playback, two-way talk and the siren (RED-39, RED-58).
- **Layout.** Rooms are HA areas grouped under HA floors, and areas with no floor go last under "Outside". Stairs and Double Down get a floor in HA first. Tiles are entities with the Home HQ label (ADR-0011): lights, switches, fans, the garage, cameras, scenes and scripts. Room headings show "{n} on" and a temperature where the room has a labeled temperature sensor. Three summary cards top the screen: Lights, Garage and Climate.
- **Tiles.** Tap toggles and a horizontal drag dims (HOME-06). A long press opens the tile's sheet. There's no visible cog and no keyboard or screen-reader path to the sheet, because no family member uses either. A light's sheet holds what that light supports: brightness, warm to cool, color, and how long it's been on. A fan tile adds speed.
- **All off.** "Turn everything off" sits in the Lights summary sheet, and each floor heading has its own All off. Both offer Undo, which restores exactly the lights they turned off, at their levels.
- **Pending changes.** A change shows at once with a pending dot until HA reports the new state (HOME-10). With no change after 5 s, the tile reverts and toasts "Couldn't reach {name} · it may be offline". The garage waits about 30 s, since the door takes that long.
- **Scenes are HA scenes and scripts.** Any scene or script with the Home HQ label is a button. One with an area shows in its room, and the rest share a whole-house row. A script can pause an automation, which covers RED-21. The "{scene} on" toast offers Undo, which restores the entities the scene touched from the HA mirror as they were just before. No scene stays highlighted (HOME-04 narrowed). Home HQ stores no scene definitions.
- **Climate.** Every member and kiosk may change it. The panel shows the setpoint ± in 0.5 °F steps within the ecobee's own limits, and the mode. In Auto (heat_cool) it shows both setpoints. ± is disabled while the mode is Off. The climate sheet adds presets (home, away, sleep), fan mode and the humidifier (RED-37).
- **Garage.** The household adds an HA template cover built from the relay and the sensor, and Home HQ drives that cover. The slide confirm (HOME-09) works from anywhere, away from home included.
- **Cameras.** All three show, picked by label, on every device and kiosk. The School Room and Double Down cameras go through docker-wyze-bridge (the IDisposable fork), which publishes them to HA by MQTT discovery with a snapshot and a stream. Their tiles refresh every 20 s while on screen and the screen is awake, and the sheet plays live video. The battery porch camera stays on wyzeapi: no snapshot, live on tap through Wyze's cloud when it works. Home HQ's server relays the WebRTC signaling over its own HA connection, so no browser holds an HA token (ADR-0004).
- **Automations stay in the HA editor** (question 17). Home HQ shows no automation toggles, thresholds or freeze button (RED-22, RED-55's automation sheet).
- **Opening at your room.** ESPresense receivers go in the bedrooms and main rooms first, with more added later. Every family member's phone is enrolled. On a phone, Home control opens at the room HA places that member in. If that room has no tiles, it opens at the room's floor, and with no reading at all, at the top. Kiosks always open at the top.

## Alternatives Considered

### Home HQ drives the garage relay itself

- **Pros**: no HA change.
- **Why not**: garage-specific code in Home HQ, and Assist would still see a bare switch. A template cover is one standard entity for every client.

### Home HQ's own scenes (DATA-19)

- **Pros**: admins edit scenes in the app.
- **Why not**: a second place to define scenes, invisible to Assist and Siri, and unable to pause an automation.

### Highlight the active scene

- **Why not**: HA scenes report no active state, so Home HQ would have to watch every light a scene set, and a script's effects can't be tracked at all.

### Cameras as live-only tiles through wyzeapi

- **Pros**: no new service.
- **Why not**: tiles would be blank names with no snapshot. The bridge gives real refreshing tiles for the two wired cameras.

### The porch camera on the bridge too

- **Why not**: the fork connects every camera at startup, and users report Wyze Cam Outdoor batteries dying in about a day while a stream is held open.

### Labeled automation toggles, or full RED-22 settings

- **Pros**: the family could switch off a motion light without opening HA.
- **Why not**: the family keeps automations in HA. A script used as a scene covers the common "pause it for a movie" case.

### Guessing the room from Wi-Fi

- **Pros**: no hardware.
- **Why not**: five access points give a floor at best, and an iPad never matches its owner. The family chose receivers.

### A visible cog, or a hidden assistive button

- **Pros**: discoverable, and reachable by keyboard and screen readers.
- **Why not**: the family wanted clean tiles and has no keyboard-only or screen-reader users.

## Consequences

### Positive

- Every tile maps to a real entity, and adding one is an HA label.
- Scenes and automations live in one place, HA.
- Camera tiles finally show real images (HOME-12), and live view needs no HA token in the browser.

### Negative

- Before building, the household has HA and hardware changes to agree on: the garage template cover, floors for Stairs and Double Down, docker-wyze-bridge with a Wyze API key, ESPresense receivers with all six phones enrolled, and the Home HQ labels.
- Sheets can't be reached without a long press. This departs from the project's accessibility principle, and anyone who later needs a keyboard or screen reader would need a new path in.
- Indoor cameras show on the wall panel, where guests can see them. Removing a camera's label is the only way to hide it.
- A room with no receiver, or a phone out of range, falls back to its floor or the top.

### Risks

- docker-wyze-bridge lists both camera models as "should work" but untested, and it depends on Wyze's cloud login. Mitigation: a camera tile with no image says "Camera unavailable", and live view still works through wyzeapi where Wyze's cloud allows it.
- Wyze cloud lamps can take longer than 5 s. Mitigation: tune the timeout per integration if false reverts show up.
- Opening the garage from away, where nobody can see the driveway. Mitigation: the slide confirm, and the opener's obstruction reverse.
