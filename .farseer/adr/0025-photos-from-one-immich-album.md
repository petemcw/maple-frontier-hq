# ADR-0025: Photos come from one shared Immich album, with favorites and hiding kept in Home HQ

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Questions 5 (photo storage) and 9 ask where photos come from, who curates them, and where favorites live. The family's photos are in Immich on the NAS, one account per family member. In Immich only a photo's owner can favorite it, and its "on this day" memories cover only the owner's own photos. An API key can be made read-only but can't be limited to certain albums. The prototype mocked five layouts, an unsaved favorite and no source at all (PHOTO-01..10). The panel is 1920x1080 (ADR-0013), so Immich's default 1440 px preview is sharp on it. Research: "Immich API for the photo slideshow" (`research/immich-photos`).

## Decision

- Home HQ shows the **Home HQ album**: one Immich album, owned by a parent, with all six family members as editors. Members add photos from their own Immich app. Home HQ has no upload.
- Home HQ reads Immich as **its own Immich user**, which owns nothing. The album is shared to that user as viewer, and its API key is read-only (`album.read`, `asset.read`, `asset.view`). The key lives in a Docker secret (ADR-0009). Home HQ never writes to Immich.
- The server calls Immich through **`@immich/sdk`**, behind one adapter, and checks `GET /server/version` on connect. It needs Immich v3.2 or later and refuses an untested major version. Because the SDK is AGPL-3.0, **Home HQ is licensed AGPL-3.0** instead of MIT.
- The server re-reads the album every 15 minutes, and when someone opens Photos. It skips the re-read when the album's change timestamp hasn't moved. Only stills are shown. Videos and Live Photo clips are skipped.
- Browsers get images only through Home HQ's proxy, and only for photos in the album that aren't hidden. They never get the key or an Immich URL. A disk cache of recent previews (about 150 MB, keyed by photo and thumbhash) keeps slideshows playing while Immich is down. Photos aren't device state, so ADR-0004's rule doesn't apply. With an empty cache, Photos says "Photos are unavailable".
- The **slideshow** lays out each slide on its own: a landscape fills the screen with a slow pan, two portraits from the same day sit side by side, and a lone portrait sits over a blurred copy of itself. No layout picker. Collage and Polaroid are dropped.
- The caption is the place and date ("3 years ago · Duluth"), never who added the photo. **On this day** photos lead the queue, found by Home HQ's own date search over the album, since Immich's memories skip shared-album photos.
- On the Photos screen, the edges or a swipe step back and forward through history. A tap in the middle brings up the caption at full strength and a bar (back, heart, Hide, next) that fades after a few seconds. Slides change every 15 s, and the timer restarts on a manual step. The slideshow pauses while Photos is hidden.
- **Favorites** are per family member and live in Home HQ (ADR-0008). A favorite comes up more often in that member's slideshow, and an All / Mine filter plays only their favorites. Kiosks have no heart and no filter. On kiosks, and for the photo of the day, a photo anyone has hearted comes up more often.
- Anyone, a kiosk included (ADR-0006), can **hide** a photo. It leaves every Home HQ screen at once, with undo, and stays in Immich and in the album. Only admins see the Hidden list and unhide. Hiding isn't a delete, so ADR-0016's delete requests don't cover it.
- The **photo of the day** is one photo per day, the same on every screen, chosen by the server. It doesn't repeat on consecutive days, and every photo comes up once per round. An "on this day" photo wins, favorites weight the pick, and hidden photos never qualify. Where it appears is decided elsewhere.
- Hearts and hides are kept by Immich photo ID, so a photo removed from the album and added back keeps them.

## Alternatives Considered

### Several chosen albums, or everyone's whole library

- **Pros**: no curating step, or albums the family already keeps.
- **Why not**: a whole library brings screenshots and private photos to the wall. Several albums add an admin screen for little gain over one album everyone adds to.

### Upload from Home HQ or the Telegram bot

- **Pros**: one less app for adding a photo.
- **Why not**: the key would need upload rights. Everyone already backs up their phone to Immich.

### Favorites written back to Immich

- **Why not**: only a photo's owner can favorite it in Immich, and with one account per person, Home HQ's user owns nothing.

### One household set of favorites

- **Pros**: kiosks could heart, and there's one simple weighting.
- **Why not**: the family wanted their own hearts and a "Mine" filter. Weighting the shared screens by anyone's hearts keeps a household effect.

### Our own small client generated from the spec

- **Pros**: about six read calls, and the MIT license would stay.
- **Why not**: the family preferred the official SDK, which matches each Immich release exactly. AGPL costs nothing for a private family app whose only users can already see the source.

### Keep the five layouts and the picker

- **Why not**: Auto with Cover and Pair looks right on any screen, from phone to panel, and Collage and Polaroid are more to build for little use.

### Show unavailable while Immich is down

- **Why not**: a NAS reboot would blank the wall. Cached photos aren't misleading the way stale device state is.

## Consequences

### Positive

- Nothing private reaches a shared screen unless someone added it to the album, and a leaked key reads only that album.
- No write path to Immich, and no Immich account changes beyond one user and one album.
- Slideshows survive Immich outages.

### Negative

- Home HQ keeps its own favorites, hides and photo-of-the-day history in SQLite, and runs its own "on this day" search.
- Home HQ's license changes from MIT to AGPL-3.0. Anyone who later receives it, or uses a modified copy over a network, is owed its source.
- PHOTO-01, PHOTO-05 and PHOTO-06 are narrowed, and video (PHOTO-10) waits.
- A heart on the panel isn't possible, since a kiosk has no member to own it.

### Risks

- Immich ships breaking changes in majors, and its deprecated PUT routes point to undocumented PATCH ones. Mitigation: pin the Immich container tag, keep calls in one adapter, check the version on connect, and test against a fake Immich.
- Immich's capture-time filter works on timestamps while its memories use the local date, so "on this day" may be a day off for travel photos. Mitigation: match on `localDateTime`, and check against real travel photos.
- A member removes their photos from the album, or deletes them in Immich. Mitigation: they drop out at the next sync, and the cache never serves a photo that's no longer in the album.
