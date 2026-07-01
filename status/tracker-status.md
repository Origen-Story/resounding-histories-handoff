# Tracker — Status

Maintained by **Matt + Claude**. Updated when a meaningful chunk lands.
Most recent at the top.

---

## 2026-07-01 — park_guell moves to Stop 16 as a listening stop; casa_calvet + cafe_torino shift up

Narrative rework: Park Güell moves out of its walking slot (Stop 14) and
becomes a listening stop at the foot of the **Joan Güell monument** (Gran Via
× Rambla de Catalunya, across the roundabout from where Cafè Torino stood).
Editorial anchors on the 1888 Universal Exhibition monument inauguration
with time-jumps to the Park Güell dream (1900-1914). Slug stays `park_guell`.

Knock-on renumbering:

| Position | groupId | Display name | Type |
|---|---|---|---|
| 14 | `casa_calvet` | Casa Calvet (1900) | listening (was Stop 15) |
| 15 | `cafe_torino` | Cafe Torino (1898-1902) | listening (was Stop 16; year expanded to cover Cercle de Sant Lluc scene) |
| 16 | `park_guell` | Joan Güell Monument / Park Güell (1888-1914) | listening ← was walking, was Stop 14 |

New totals: **20 listening + 2 walking + 1 meta** (was 19/3/1). Stops 17–22
unchanged. `inventory/stops-2026-06-04.md` updated.

**Emily — needed from your side:** a coordinate for `park_guell` at the Joan
Güell monument, Gran Via × Rambla de Catalunya. The existing coordinate seed
doesn't have one (park_guell was walking previously). Everything else on this
change is tracker-internal.

FDX re-bootstrap for the new Stop 16 scene structure (5 scenes now: 1888
monument inauguration → Muntanya Pelada time-jump → 1906 Catalan Language
Congress garden party → 1914 WWI abandonment → 1916 unity festival) is
deferred a few days until the outline draft settles. No contract change.

## 2026-06-23 — ADR-0011 accepted, ADR-0005 superseded; reus/bcn_arrival reclassified

Emily signed off on ADR-0011 (slug-based, hyphenated filenames, atomic
cutover) on PR #2. Status flipped on both ADRs to reflect.

Inventory revised: **bcn_arrival** moves from walking → listening (Stop 3
is a stationary "stand on La Rambla" stop). Reus stays walking. New totals:
19 listening + 3 walking + 1 meta. See `inventory/stops-2026-06-04.md`
Revisions section.

Image normalization (ADR-0004) — Emily proposed dropping format conversion
to keep original extensions. **Pushing back** — uniform JPEG/sRGB/q85/1600px
is standard one-time work and reduces validator surface, runtime variance,
and bundle size. Will sit with her on this; reply queued for PR #2.

Slug rename not yet executed — pending Emily's reasoning on a couple of
proposed slug name choices (`la-pedrera` vs `casa-mila`, etc.) before we
finalize the canonical mapping.

Still no tracker code changes for the contract-affecting work.

## 2026-06-14 — Bibliography as unified source registry (tracker-internal, complete)

Built the bibliography table + UI, refactored `scene_sources` to be a thin
M:N link table, ran the data migration, and dropped the legacy polymorphic
columns. The refactor is complete; tracker is now wholly bibliography-backed
on the source side.

What landed:
- New `bibliography` table (schema migration `0008_clumsy_wild_child.sql`)
- `scene_sources.bibliography_id` link column added; data migration
  (`npm run bibliography:migrate`) converted existing rows
- Follow-up schema migration `0009_real_grey_gargoyle.sql` dropped the
  legacy polymorphic columns (`sourceType`, `archivalId`, `producedId`,
  `aiAssetId`, `externalUrl`, `externalTitle`, `citation`) and tightened
  `bibliographyId` to `NOT NULL`. lib + UI cleaned up to remove the
  transition-window branches.
- `lib/bibliography.js`, `routes/bibliography.js`
- `/bibliography` list + edit pages with internal-asset linking
- SceneEdit refactored — single + Source button, BibliographyPickerModal
  with inline create

Decision documented in **ADR 0012**. **No impact on the v0 publish
contract** — the app sees the same `bibliography.json` shape; this only
changed how the tracker stores sources internally.

Still Plan A — no contract-affecting work yet. Pending Emily's sign-off on
ADR 0011 (slug-based filenames) before the schema migrations for the
publish bundle start.

## 2026-06-10 — v0 contract revision pass (response to Emily's review)

Responded to Emily's app-side PR #1 (merged). Summary:

**Accepted in full:**
- ADR-0001 amended via ADR-0009 — defer stop-structure migration to
  post-launch. App keeps stop ownership through launch; coordinates stay
  app-side; tour JSON remains authoritative. Tracker still ships the
  archive/visual bundle.
- ADR-0007 superseded by ADR-0010 — bibliography vocabulary changed to
  lowercase `book / article / archive / web` (Archive restored per Emily's
  ask; ~42% of source material is archival).
- Bibliography `ref_key` → `id`; `year` → string-only.
- `arrival_radius_meters` permanently dropped from the contract.
- `caption_source` column added to manifest (resolves ADR-0008 vs schema
  inconsistency).
- `bundle_version` pattern tightened; "semver" terminology dropped.
- `archives-{locale}.json` companion files added to the bundle layout,
  matching her loader's filename-keyed shape.

**Pending Emily's sign-off:**
- ADR-0011 (Proposed) — slug-based filename pattern
  (`<stop_id>-hero-NN.jpg`) superseding ADR-0005's tour-position-numbered
  pattern. Awaiting confirmation; tracker-side build doesn't proceed on
  filenames until accepted.

**Still no tracker code changes.** Plan A holds — Emily's loader needs to
read a sample v0 bundle and report back before the tracker side starts
schema migrations.

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
