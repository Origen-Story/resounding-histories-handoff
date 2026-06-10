# ADR 0009 — Defer stop-structure migration to post-launch

**Status:** Accepted · **Date:** 2026-06-10 · **Decided by:** Matt, Emily
**Amends:** ADR 0001

## Context

ADR 0001 declared the tracker the source of truth for tour stop structure and
required the app's tour JSON to become a derived artifact "once the first
bundle is consumed cleanly." Emily's v0 contract review (see `status/app-status.md`,
2026-06-10) accepted the direction but flagged the launch timing as a real
risk.

Her argument, accepted:

1. The 12 stops + coordinates already work on the app side
2. Migrating them now puts the map and proximity behind a publish pipeline
   that has produced nothing yet
3. The full stop payload includes fields with no contract home yet —
   `subtitle`, `era`, `teaserText`, `heroImage`, `duration`, `chapters`,
   `sideQuests` — plus audio that isn't built tracker-side
4. End-of-June launch can't absorb that risk for zero launch value

## Decision

**Accept the direction of ADR 0001. Defer the launch timing.**

- The tracker remains the **eventual** source of truth for stop structure
  and coordinates. ADR 0001's reasoning on reorder/renumber drift stands.
- For **v0 launch**: the app retains stop ownership. Coordinates stay
  app-side. The tour JSON stays authoritative. The bundle's `stops.json`
  does NOT carry coordinates in v0.
- **Post-launch**: a follow-up ADR will specify the actual migration —
  shape (flat `lat`/`lng`, see ADR 0009 consequences below), trigger
  (probably "first complete tracker-built bundle covering all fat-Stop
  fields"), and rollout (probably a soft-cutover with both systems live
  during validation).

## Consequences (v0)

- **`stops.schema.json` updated:** `coordinates` becomes optional and
  documented as "app-owned through v0, not emitted in the launch bundle."
- **`arrival_radius_meters` dropped from the contract permanently.**
  Proximity tuning interacts with GPS accuracy, the map camera, and on-device
  testing — that's an app concern, not a tracker concern. Even after the
  post-launch migration, the tracker will not own the radius.
- **Coordinate shape at migration: flat `lat`/`lng` decimal degrees** at the
  stop record's top level, matching Emily's `Stop.swift` model. The earlier
  nested `coordinates: {...}` shape from ADR 0001 is dropped.
- **Stable `stop_id` becomes the cross-system identity.** Coordinates,
  filenames, locale keys, and all other cross-side references attach to a
  stable `stop_id` — not to tour position, not to filename index, not to
  display number. This is also the basis for ADR 0011 (slug-based filename
  pattern).
- **Coordinate handoff:** Emily shares the 12 coordinates her app currently
  uses, keyed to stop IDs both sides agree on. They seed tracker stop
  records as reference data, but remain app-authoritative through launch.

## What this does NOT change

- The tracker still ships the archive/visual bundle (captions, classification,
  manifest, locale files, bibliography, characters) per the v0 contracts.
- The post-launch migration ADR is not blocked on v0 — it can be drafted
  whenever both sides are ready to plan it.

## References

- ADR 0001 (amended): `decisions/0001-tracker-owns-stop-structure.md`
- App-side review: `status/app-status.md` (entry 2026-06-10)
- Schema update: `schema/stops.schema.json`
