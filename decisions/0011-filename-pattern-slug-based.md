# ADR 0011 — Filename pattern: slug-based, stable forever

**Status:** Accepted · **Date:** 2026-06-10 (Accepted by Emily 2026-06-23) · **Decided by:** Matt, Emily
**Supersedes:** ADR 0005 (Filename conventions)

## Context

ADR 0005 defined image filenames as `stop-NN-{hero,appendix}-NN.ext`, with
the first `NN` tied to the **tour position** (1-based `stop_number`). Emily's
v0 contract review surfaced a real ambiguity:

- Her 143 files today are numbered against the **22-stop gallery space**
  (1–22, with #13 absent — which happens to be `palau_moja` in our current
  numbering, a stop she hadn't yet)
- The **tour has 12 stops**, the gallery has 22 — different spaces
- Re-deriving filenames from tour position would renumber every file
  whenever the tour reorders, or whenever a stop is added/removed
- Stop identity (which stop a file belongs to) and tour position (when it's
  encountered) are two distinct concerns; ADR 0005 conflated them

Plus a related point from ADR 0009: stable `stop_id` should be the
cross-system identity, not any numbered space.

## Decision (proposed)

**Filenames use the stable `stop_id` slug as the primary identifier, with a
small per-role ordinal suffix.** No numeric "stop position" in the filename.

```
<stop_id>-hero-NN.<ext>
<stop_id>-appendix-NN.<ext>
character-<character_id>.<ext>
```

Examples:

```
hospital_santa_creu-hero-01.jpg
hospital_santa_creu-hero-02.jpg
palau_guell-appendix-03.jpg
palau_guell-appendix-04.jpg
character-12.jpg
```

Rules:

- **`<stop_id>`** is the stable lowercase slug from `stops.json[].stop_id`
  (e.g. `hospital_santa_creu`). Never changes once assigned.
- **`NN`** is the asset's per-role ordinal within its stop, two-digit
  zero-padded, 1-based. Separate counters for hero vs appendix (a stop can
  have `<id>-hero-01.jpg` and `<id>-appendix-01.jpg` simultaneously).
- **Extension** is `.jpg` post-normalization (ADR 0004) regardless of source
  format.
- **Character portraits** keep the simpler `character-<id>.jpg` pattern from
  ADR 0005 (no per-character ordinal; one portrait per character).

## Why slug-based instead of numeric

- **Stable forever.** Stop slugs are immutable once assigned (renaming a
  stop's display name doesn't change its slug). Tour reorders don't rename
  files. Stops added/removed don't shift other files' numbers.
- **Self-documenting.** A file's path tells you exactly which stop it
  belongs to without consulting a manifest.
- **Resolves the gallery vs tour ambiguity.** No numeric position appears in
  the filename, so there's nothing to conflate.
- **Matches Emily's "stable stop identity" architectural ask** from her
  v0 review.
- **Consistent with locale-file key conventions** which already use slug as
  the stop identifier (`stop.<stop_id>.name`).

Trade-off accepted: filenames are longer than `stop-NN-hero-NN.jpg`. We
believe stability + clarity outweigh compactness for this use case.

## Migration impact on Emily's existing files

Emily's current 143 files use the numeric `stop-NN-...` pattern from the
spreadsheet she shared earlier. Adopting this ADR means a rename pass on her
side. Two options:

1. **Atomic cutover at first ingest.** Tracker emits the new naming; her
   loader's `imageRefs` are regenerated in the same change. Cleaner.
2. **Dual naming during transition.** Tracker also emits a
   `filename-aliases.json` mapping old → new for one bundle, then drops it.
   Lower-risk but two-bundle deal.

If Emily prefers (2), we can build it. Default is (1) since v0 hasn't shipped
anything yet — no production cost to cutover.

## Consequences

- ADR 0005 marked **Superseded by ADR 0011** (pending acceptance).
- `schema/manifest.schema.json` `filename` column documentation updated to
  reflect the new pattern.
- Tracker's publish skill (when built) generates filenames using the slug
  pattern.
- `inventory/stops-2026-06-04.md` already uses stop slugs in its tables,
  so it's consistent with this ADR without revision.

## Acceptance (2026-06-23)

Emily confirmed on PR #2: slug-based filenames accepted, **hyphens
required** (not underscores — her app's loader matches filename strings
exactly; underscores would silently fail to load). Atomic cutover —
tracker renames, she re-points `imageRefs` against the renamed set in
one change. After cutover she'll eyeball all 22 galleries to spot any
mismatches (the app shows a placeholder rather than erroring on a missing
name, so the check is visual).

Hyphenated slug convention is now canonical. The tracker-side groupId
column will be migrated from underscored slugs (`hospital_santa_creu`) to
hyphenated (`hospital-santa-creu`) via the existing cascade-rename
tooling. ADR 0005 moves to Superseded by this ADR.

## References

- ADR 0005 (pending supersession): `decisions/0005-filename-conventions.md`
- ADR 0009 (related — stable stop_id as cross-system identity):
  `decisions/0009-defer-stop-structure-migration-post-launch.md`
- App-side review: `status/app-status.md` (entry 2026-06-10)
