# ADR 0001 — Tracker owns the tour stop structure

**Status:** Accepted · **Date:** 2026-06-04 · **Decided by:** Matt, Emily

## Context

The tracker has 22 search groups (≈21 narrative stops + a `general` cross-stop
bucket). Emily's tour data has 12 stops with coordinates. The mismatch is from
an older tour design Matt iterated past — not a real architectural split. We
need a single source of truth for "what stops exist, in what order, with what
names and coordinates."

## Decision

The **tracker** is the source of truth for the tour stop structure:
- Which stops exist
- Their order (tour position)
- Slugs (stable IDs)
- Display names (per locale, via locale files)
- Coordinates (lat, lng, arrival_radius_meters)

The **app** remains responsible for runtime concerns built on top of that data:
- Arrival-detection algorithm
- Map rendering
- Walking directions
- Any UX layered on the coordinates

A new `stops.json` ships in the bundle (see stops.schema.json). The app's
existing tour JSON becomes a derived artifact rebuilt from each publish.

## Consequences

- Tracker schema gains `latitude`, `longitude`, `arrival_radius_meters`, and
  `stop_type` (`listening` | `walking` | `meta`) on `search_groups`.
- Tracker UI adds a coordinate editor on the Stop edit page.
- Matt produces a reconciled list of which of the 22 are real `listening` stops
  vs `walking` transitions vs `meta` so Emily can sanity-check against her 12.
- Emily's previous app-side stop data is retired once the first bundle is
  consumed cleanly.
- One-time data migration: Matt fills in coordinates for the existing stops
  (Emily can share the lat/lng she has so we don't re-source them).

## Alternatives considered

- **App owns it, tracker treats stops as scene-context only**: rejected — every
  script revision would require out-of-band reconciliation between two systems.
  The tracker has the richer view (scenes, assets, sources) and stops live
  naturally with that.
- **Both own it (mirror)**: rejected — drift is guaranteed.
