# Schema Changelog

Reverse chronological. One line per change. Bump the affected schema's
inline `$id` version too.

## v0 — initial pre-launch contracts

- **2026-06-04** — `stops.schema.json`: dropped `"meta"` from the `stop_type`
  enum. Bundle never carries meta-type stops; internal cross-stop buckets
  (`general` etc.) exist in the tracker only. See first inventory at
  `inventory/stops-2026-06-04.md`.
- **2026-06-04** — Initial drafts of all v0 schemas (bundle, locale, manifest,
  bibliography, stops, characters). Status: NOT YET PRODUCED OR CONSUMED by
  either side. Expect breakage during the iteration loop.
