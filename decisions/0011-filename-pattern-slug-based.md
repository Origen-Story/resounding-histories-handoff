# ADR 0011 — Filename pattern: slug-based, stable forever

**Status:** Proposed (awaiting Emily's sign-off) · **Date:** 2026-06-10 · **Decided by:** Matt
**Supersedes:** ADR 0005 (Filename conventions) — pending acceptance

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

## Status: Proposed

This ADR sits at **Proposed** until Emily's side signs off. Specifically
seeking confirmation on:

1. Slug-based filenames vs an alternative (numbered-against-gallery-with-explicit-clarification, or some other scheme)
2. Atomic cutover (option 1) vs transitional aliases (option 2)
3. Any Swift-side constraints on filename length / character set we should
   know about (we believe lowercase + underscore + digit is safe everywhere
   on iOS file systems)

Once Emily accepts (or proposes a counter), this ADR moves to **Accepted**
and ADR 0005's status updates to **Superseded**.

## References

- ADR 0005 (pending supersession): `decisions/0005-filename-conventions.md`
- ADR 0009 (related — stable stop_id as cross-system identity):
  `decisions/0009-defer-stop-structure-migration-post-launch.md`
- App-side review: `status/app-status.md` (entry 2026-06-10)
