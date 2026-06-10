# Tracker — Status

Maintained by **Matt + Claude**. Updated when a meaningful chunk lands.
Most recent at the top.

---

## 2026-06-04 — Contracts drafted; schema migrations pending

**What's live in the tracker today:**
- Stop management (search groups) with multilingual keyword + Wikimedia category lists
- Wikimedia archival search worker (queued, retried, deduped against existing assets)
- Triage mode for archival images with multi-select, move/add/remove stop tags, mark selects, bulk operations
- Scene editor: hero picker (defaults to selects), beat, caption (single-field, to be replaced — see below), sources
- Stop appendix curation (inline in Scenes view)
- Character entity with FDX-driven scene-appearance sync (34 characters, 130 appearances synced from the current FDX)
- Asset import: URL, file upload, folder scan, file path; pre-tag stop at import time
- Per-scene, per-stop, and full-project ZIP downloads (originals — pre-normalization)
- Print-friendly PDF companion

**What the tracker does NOT yet produce** (needed for v0 bundle):
- `caption_source` / `title` / `short_caption` / `long_description` × EN/CA/ES (current single
  `captionOverride` model needs migrating — see ADR 0008)
- Per-asset `classification` flag (cast/image) — see ADR 0002
- Stop coordinates + `stop_type` field — see ADR 0001
- Bibliography table — see ADR 0007
- Image normalization on export (sharp pipeline) — see ADR 0004
- Per-locale JSON file generation — see ADR 0003
- `bundle.json` / `manifest.csv` / `stops.json` / `bibliography.json` / `characters.json`
  publish artifacts conforming to `schema/`

**On hold until app side validates the contracts:** all of the above. Per Plan A, no
schema migrations on tracker side until Emily's loader has read a sample bundle
that conforms to v0 schemas and reports back what works / what breaks.

## 2026-05-XX — Pre-handoff baseline

Tracker was built to support content authoring with the current single-caption
model, no character entity, originals-only exports. See provenance-tracker repo
for full feature list and conventions.
