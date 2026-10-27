# ADR-0031: The Today hero is a backdrop of Maple Frontier picked by rules, with weather from Open-Meteo

**Date**: 2026-10-27
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Question 13 asks whether the Today hero is a family photo (TODAY-14) or an illustrated scene that reacts to the weather and the family's day (RED-01, RED-02), and how scene rules are tested (RED-03). Ambient already plays the slideshow full screen on the wall panel (ADR-0029), and the server already picks a photo of the day (ADR-0025). The Reddit art is Google's weather frog, with no license, and scenes made in its style would still be derivatives. RED-02's rules are built around a kindergartener's day, and nobody here keeps that schedule. HA's weather is Met.no, which has no hourly rain chance outside the Nordic area, so it can't drive TODAY-03's rain advice. wttr.in has a moon phase but forecasts in 3-hour steps from an unnamed WorldWeatherOnline model, with no stated terms. Research: "Weather data and illustrated scene art" (`research/weather-and-scene-art`).

## Decision

- **A backdrop, not a photo.** The Today hero is a backdrop: a soft storybook painting of Maple Frontier itself, with no people, under the current weather, light phase and season. It shows on the panel and on phones, where a short banner is cropped around each image's focus point. The clock and weather sit over it at upper left.
- **The art is our own.** The Google frog panoramas aren't used. Pete generates and curates the images with an image model, from photos of the house, with one style preamble for every prompt (`docs/backdrops.md`). Each image is 21:9, with the house right of center and quiet sky at upper left.
- **Cells.** One image per cell, with no rotating variants:
  - **Weather**: 10 groups from Open-Meteo's hourly WMO code. Windy means clear or partly cloudy with wind over 19 mph.
  - **Light phase**: daylight, golden hour, twilight or dark, from the sun's elevation at sunrise and sunset alike. Golden hour exists only for clear, partly cloudy and windy.
  - **Season**: by month, except that snow and freezing rain always use winter art.
  - **Count**: 111 base images.
- **Holidays win all day.** New Year's Day, Valentine's Day, St. Patrick's Day, Easter, Mother's Day, Father's Day, the 4th of July, Halloween, Veterans Day and Christmas (Dec 24 and 25) each have a day image and a dark image for each weather group likely in their month, 182 in all. Thanksgiving and birthdays aren't included.
- **The moon.** On clear, partly cloudy and windy dark backdrops, Home HQ lays one of 7 painted moon sprites at a spot the manifest records, between moonrise and moonset. The phase and times come from wttr.in once a day. If wttr.in is down, no moon shows.
- **Stale weather.** When the last Open-Meteo fetch is out of date, the hero shows a weatherless backdrop for the season and light phase (16 images). The weather text says it's out of date. The art never claims weather Home HQ doesn't know.
- **The server picks.** One rules module on the server picks the cell and pushes it over `/ws`, so every screen shows the same image. Browsers only load the image.
- **Images live outside the repo.** They sit in a mounted backdrops folder with a manifest Pete edits. Each manifest entry gives the image's cell, its text tone (light or dark) and its focus point. A soft scrim sits under the text. A missing image falls back from holiday to base, then to weatherless, then to a plain background, so the backdrop can ship before the set is done. Admins see which cells have no image.
- **Weather comes from Open-Meteo.** The server calls Open-Meteo directly every 15 minutes, with no key, for the location an admin sets, in the household's units (ADR-0020). That covers the hourly WMO code, temperature, rain chance and wind, plus the local day's high and low. The same data feeds Today's weather, ambient, the night clock and dinner ideas. A "Weather data by Open-Meteo.com" link sits next to the weather, as its CC BY 4.0 license asks.
- **Exhaustive rule tests (RED-03).** Rule-driven content is tested by running every combination, not just examples. That covers the backdrop rules (weather code × half hour × month × holiday), the weather advice line (TODAY-03), and the kiosk idle and night timers.
- **The photo of the day** doesn't fill the hero. It's a candidate tile for the Today layout.

## Alternatives Considered

### Photo of the day or the slideshow as the hero (TODAY-14)

- **Pros**: no art to make, and it already exists (ADR-0025).
- **Why not**: the family wanted the illustrated backdrop. On the panel, a slideshow hero would also repeat what ambient shows 2 minutes later.

### The Google frog panoramas

- **Why not**: Google's artwork, with no license. New art in its style would still be a derivative.

### A mascot or plain landscapes as the subject

- **Why not**: keeping a character consistent across 300 images is the hardest part of curating. Plain landscapes aren't personal. The house is both personal and easy to hold steady.

### Several variants per cell, rotating every 15 minutes (RED-02)

- **Why not**: 2 to 3 times the art for variety the family didn't ask for.

### Weather and light first, seasons and holidays later

- **Why not**: the family wanted the whole set at once. Fallbacks let the feature ship while art is still being made.

### Holidays ignoring the weather

- **Why not**: a snowy Christmas under a rain forecast is wrong. Holidays come per weather, but only by day and dark, which saves about 100 images.

### Images in the repo and the container image

- **Why not**: over 100 MB in git, and every art change would need a release.

### wttr.in for all weather

- **Pros**: moon phase and many chance-of fields built in, with no attribution.
- **Why not**: 3-hour steps break the hourly strip (TODAY-04). It names no model and states no terms or limits. Open-Meteo blends NOAA's GFS and the 3 km HRRR for Minnesota. The two disagreed on the afternoon this was decided (0 to 5% against 31 to 43% rain).

### Met.no through HA, or adding NWS to HA

- **Why not**: Met.no has no US rain chance. NWS would mean an HA config change for twice-daily forecasts and HA's coarser conditions.

## Consequences

### Positive

- Today has its own look, distinct from ambient's photos.
- One weather source with hourly rain chance feeds every screen and dinner ideas.
- The rules are data plus one tested module, and every screen agrees.

### Negative

- About 316 images to generate and curate by hand, which takes far more time than the roughly $25 to $50 in generation.
- Two outside weather calls, Open-Meteo every 15 minutes and wttr.in daily, and an attribution link on screen.
- A cell without art looks plainer until its image exists.

### Risks

- The set may drift in style over 300 edits. Mitigation: one base per season edited into each cell, and locked style references (`docs/backdrops.md`).
- Photos of the house go to an image model's provider. They show no people or house number. Mitigation: Pete chooses the photos.
- wttr.in has no stated terms and could go away. Mitigation: the moon is optional, and Home HQ could compute it locally instead.
