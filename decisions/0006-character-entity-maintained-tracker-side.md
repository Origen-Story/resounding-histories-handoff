# ADR 0006 — Character entity maintained tracker-side; published for forward use

**Status:** Accepted · **Date:** 2026-06-04 · **Decided by:** Matt, Emily

## Context

Emily's app at v0 launch does not need a per-character data model. The Cast
filter is flat over all `cast`-classified images (ADR 0002). However, the
tracker side needs character data to:

- Track who appears where in the script (for editorial completeness)
- Spot gaps ("we have 34 characters, only 6 have portraits assigned")
- Sync appearances from the FDX automatically when the script changes
- Hold per-character bios for the eventual stop/scene detail section of the app

## Decision

The tracker maintains a full **Character** entity:
- Canonical name (ALL CAPS with diacritics, matches FDX cues)
- Polymorphic portrait (archival or produced)
- Trilingual bios (`bio_en`, `bio_ca`, `bio_es`)
- Life dates (birth/death year)
- Scene appearances (M:N, auto-sync'd from FDX)

The publish bundle includes `characters.json` from day one (see
`schema/characters.schema.json`). The app's v0 launch may ignore it without
issue; post-launch, the app can pick it up without a schema bump or out-of-band
data migration.

## Consequences

- Tracker: characters table already built; FDX bootstrap extracts character
  cues per scene and writes `scene_characters` links.
- Tracker UI: Characters list page + character editor with portrait picker
  and bio fields.
- Publish skill writes `characters.json` and any `character-<id>.jpg` portraits
  to `/images/`.
- App: free to consume now (post-v0) or later. Validator accepts the file as
  optional in v0 mode.

## Alternatives considered

- **Don't track characters at all in tracker**: rejected — lose editorial
  oversight of who's spoken for / portrait-assigned, and lose the FDX
  appearance sync.
- **Track but don't publish**: rejected — adds friction when the app side is
  ready to use them. Better to ship the data and let consumers opt in.
