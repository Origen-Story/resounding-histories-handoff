# Schema Changelog

Reverse chronological. One line per change. Bump the affected schema's
inline `$id` version too.

## v0 — initial pre-launch contracts

- **2026-06-10** — Response to Emily's v0 contract review (PR #1). Multiple
  schemas updated:
  - `bibliography.schema.json`: `ref_key` → `id`; `year` → string-only;
    `category` enum reset to lowercase `book / article / archive / web`
    (locked by ADR 0010, superseding ADR 0007). `archive` added back per
    Emily's pushback — archival scans are ~42% of source material.
  - `stops.schema.json`: `coordinates` flattened (`lat`/`lng` at top level,
    nullable, documented as app-owned through v0 per ADR 0009);
    `arrival_radius_meters` permanently removed (app concern, not a tracker
    field).
  - `manifest.schema.json`: added `caption_source` column. Resolves the
    ADR-0008-vs-schema inconsistency Emily flagged.
  - `bundle.schema.json`: tightened `bundle_version` pattern to `^v\d+$`
    (was loose pseudo-semver). Renamed concept from "semver" since it's a
    monotonic major-version ladder, not actual semver.
  - `locale.schema.json`: documented the `archives-{locale}.json` companion
    files (filename-keyed, matching Emily's loader). Bundle layout in
    `bundle.schema.json` updated to include them.
  - Filename pattern (ADR 0005) marked pending supersession by ADR 0011
    (slug-based). Bundle layout uses `<stop_id>-hero-NN.jpg` style.
- **2026-06-04** — `stops.schema.json`: dropped `"meta"` from the `stop_type`
  enum. Bundle never carries meta-type stops; internal cross-stop buckets
  (`general` etc.) exist in the tracker only. See first inventory at
  `inventory/stops-2026-06-04.md`.
- **2026-06-04** — Initial drafts of all v0 schemas (bundle, locale, manifest,
  bibliography, stops, characters). Status: NOT YET PRODUCED OR CONSUMED by
  either side. Expect breakage during the iteration loop.
