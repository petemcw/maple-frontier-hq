# ADR-0034: The speaker and TVs are tiles in Home control, with Sonos favorites and LRCLIB synced lyrics

**Date**: 2026-10-29
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

The prototype has one now-playing card with play/pause (HOME-13). The Reddit dashboards add skip, volume and one-tap stations (RED-08), Sonos room grouping (RED-38), and a mini player on every page with synced lyrics (RED-50). ADR-0015 keeps synced lyrics in scope. The media research found that only the Living Room Sonos Play:1 reports a full now playing. The two Apple TVs report what plays and take remote commands. The Family Room's Samsung TV reports only on or off and can't be turned on from HA, and its Denon receiver handles volume and inputs. LRCLIB is the one workable synced-lyrics source. It's free and keyless, but its lyrics are community-submitted with no license. Musixmatch is licensed and starts at $199 a month. iPhones already carry the Apple TV remote in Control Center.

## Decision

- **The speaker.** The Living Room Sonos shows art, title, artist, play/pause, previous, next and volume. Home HQ doesn't offer shuffle, repeat, seek or a sleep timer.
- **Sonos favorites.** The speaker's sheet lists the household's "My Sonos" list as it stands, with artwork, and a tap plays one. The list is edited in the Sonos app. There's no Radio Browser and no admin-picked subset.
- **TVs, one per room.** The Family Room TV is the Samsung, the Denon and the Apple TV together. Double Down's is its Apple TV. A TV is playing when its Apple TV plays, and on when any part is on. It shows the app and the title, for everyone and on the panel. Its one action is Turn off, which turns every part off through `media_player.turn_off`, never `remote.turn_off`. Home HQ has no remote, volume or input control for a TV.
- **Placement.** The speaker and each TV are tiles in their HA room (ADR-0026). A fourth summary card, Media, shows "Quiet" or what's playing and opens a sheet of every player. A tap on the speaker tile plays or pauses it, and a long press opens its sheet. A tap on a TV tile opens its sheet, since a TV can't be turned back on.
- **Who and where.** Every member and kiosk may control media, at home or away, the same as climate.
- **Synced lyrics** come from LRCLIB, for the speaker only. They show in the speaker's sheet on every device, the panel included, with the current line highlighted. Lyrics appear only when LRCLIB has a match, and radio or a track with no duration gets none. A lyrics offset someone sets is saved with that song's cached lyrics for the household.
- **Not adopted.** A mini player on every screen (RED-50), now playing on ambient, and grouping (RED-38), which is moot with one Sonos. Whether Today shows now playing is the Today layout's call.

## Alternatives Considered

### A full TV remote

- **Pros**: RED-08's "TV & remote". The Apple TVs take about 26 commands and keyboard text.
- **Why not**: every phone already has the Apple TV remote, and the Samsung only takes blind keys while it's on. Turning off a TV someone left on was the real need.

### One tile per media device

- **Pros**: each device's own state and control.
- **Why not**: the Family Room's three boxes are one TV to the family. Four tiles for one screen would add noise, and turning off only one part leaves the rest on.

### Musixmatch, or no synced lyrics

- **Pros**: licensed lyrics, or no rights question at all.
- **Why not**: Musixmatch costs $2,388 a year or more, fires a tracking pixel on every display, and forbids caching below Enterprise. The family accepted LRCLIB on its own private screens.

### A mini player on every screen

- **Pros**: RED-50's always-there controls.
- **Why not**: the Media card covers it, and the panel hangs in the entry, away from the speaker.

### Media on its own screen

- **Pros**: RED-30's separate Media section.
- **Why not**: one speaker and two TVs fit as tiles. Home control already shows rooms, and a Media card gives the overview.

## Consequences

### Positive

- Media follows Home control's rules for tiles, sheets and summary cards, with no new pattern.
- "Someone left the TV on" is one tap from anywhere.
- Sonos favorites stay in one place, the Sonos app.

### Negative

- Anyone away can pause or skip the speaker while someone at home is listening.
- Titles on the TVs show on the entry panel, where guests can see them. The guest mode ticket decides whether guest mode hides them.
- Home HQ can't turn a TV on.

### Risks

- LRCLIB lyrics are community-submitted with no license, and its matches can be the wrong edit. Mitigation: they're shown only on the family's own screens, never republished, and only for a match within ±2 s of duration.
- The non-admin Home HQ user's access to media actions and browsing is unverified, since the research used an admin token. Mitigation: check it before building, along with what the Apple TVs report while playing.
- LRCLIB may rate-limit or go away. Mitigation: the server identifies itself, caches each song's lyrics and honors `Retry-After`. Without lyrics the speaker still works.
