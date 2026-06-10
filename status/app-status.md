# App-side status

**Entry: 2026-06-10 — v0 contract review**
**Reviewed:** bundle-v0 contract @ commit `e62f117` (full schema set + 8 ADRs)
**By:** app side (Emily)

## Verdict

Strong v0. The archive/visual layer is well-aligned and ready to build against — filename conventions, the normalization spec, the locale mechanism, and the classification vocabulary all match our side. Two things to resolve before a clean first ingest: the scope/timing of ADR-0001, and three bibliography decode-breakers against our `Source` model.

## Accepted as-is

- **Filename pattern** (`stop-NN-hero-NN` / `stop-NN-appendix-NN`, two-digit padded) — matches our 143 real files. (Extension normalization noted under non-blocking.)
- **Normalization spec** (JPEG / sRGB / q≈85 / ≤1600px long edge / never upscale; grayscale stays grayscale) — exactly right.
- **Per-language files** (`en` / `ca` / `es` keyed by content ID, identical key sets, validator-enforced parity) — matches our `archives-{locale}.json` loader.
- **Classification values** (`cast` / `image`) — decode cleanly our side.
- **Versioned bundle** — concept accepted (one note under non-blocking).

## ADR-0001 (tracker owns stop structure) — direction accepted, timing deferred

We accept the **direction**: the tracker as the eventual source of truth for stop structure, coordinates included. The reorder/renumber drift argument is sound.

We do **not** accept the **launch timing**. ADR-0001 as written retires app-side stop ownership on first clean ingest and makes our tour JSON a derived artifact. That couples our end-of-June launch to the tracker emitting the *full* stop payload — including fields with no contract home yet (`subtitle`, `era`, `teaserText`, `heroImage`, `duration`, `chapters`, `sideQuests`) and audio that isn't built. The data it would relocate — 12 real, stable coordinates — already works app-side; moving it now puts our map and proximity behind a publish pipeline that has produced nothing yet.

**For launch:** app retains stop ownership; coordinates stay app-side; the tour JSON stays authoritative. Stop-structure + coordinate migration is **post-launch**, gated on the pipeline first proving itself on the archive/caption bundle (which originates with the tracker regardless).

**Request:** supersede ADR-0001 with an amending ADR that separates the accepted *direction* from the deferred *launch timing*, so the log reflects what both sides actually hold.

We'll share our 12 coordinates as reference to seed the tracker's eventual model — they remain app-authoritative until the post-launch migration.

## Coordinate reconciliation — what to change so v0 maps correctly

Concretely, so nothing in v0 tries to map against data that isn't in the bundle, and so the post-launch migration is clean:

- **v0 bundle — coordinates not authoritative.** Mark `coordinates` in `stops.schema.json` as optional / "app-owned through v0, not emitted in the launch bundle." The app reads coordinates from its own tour JSON for launch and will not consume `stops.json[].coordinates` until migration — so the tracker doesn't need to populate them for v0 and isn't blocked on them.
- **Shape at migration.** Our app stores flat `lat: Double` / `lng: Double` in decimal degrees (`Stop.swift:49-52`); the contract nests `coordinates: { lat, lng, arrival_radius_meters }` (`stops.schema.json:38-47`). At migration the publish step should emit flat `lat` / `lng`, or we add a transform — either's fine, pick one in the migration ADR.
- **`arrival_radius_meters` stays app-side.** Proximity tuning interacts with GPS accuracy, the map camera, and on-device testing — it's an app concern. Please drop it from the stop coordinate or mark it app-owned; the tracker shouldn't own the proximity radius.
- **Map by stable stop identity, not by number.** When coordinates migrate they attach to our stops by a stable `stop_id`, not by the filename `NN` or gallery number. Our tour has 12 stops; the archive gallery has 22 (1–22, 13 absent) — different spaces. Each tour stop needs one stable `stop_id` both sides agree on, with coordinates (and the fat-Stop fields, eventually) hanging off it, decoupled from the 22-gallery image numbering. (Same stop-identity question as the filename-`NN` ambiguity below.)
- **What we'll hand you now.** Our 12 coordinates keyed to our stop ids/slugs, as reference to seed your stop records — app-authoritative until migration, but in your hands so the tracker's stops aren't empty.

## Must fix before first ingest — bibliography

Three independent decode failures against our `Source` model:

1. **Categories.** Contract `Book / Article / Web / Interview` (`bibliography.schema.json:45-48`, ADR-0007) vs ours `book / archive / article_essay / digital` — different values *and* casing; every contract value falls to `.unknown` our side. We'd also **push back on dropping "Archive"**: ~42% of our images are archival scans and it's a meaningful source type for this project. Proposal: reconcile toward our (archival-aware) vocabulary; happy to align exact strings together.
2. **`ref_key` vs `id`.** Our `Source.id` is required; contract entries carry `ref_key`, no `id` (`bibliography.schema.json:19-22`) → decode fails. Fix: contract renames to `id`, or we add a CodingKeys map — either works, we just need one.
3. **`year` type.** Contract allows integer-or-string (`bibliography.schema.json:36-39`); ours is `String?` → a numeric year throws. Ask: contract commits to string (bibliography years need it anyway — "c. 1926", ranges).

Not urgent in the literal sense — `Tour.sources` is empty today, so nothing breaks now — but these should land before the tracker builds its bibliography export.

## Also noted — non-blocking

- **Filename `NN` index is ambiguous.** ADR-0005 ties the first `NN` to tour position (1-based `stop_number`), but our 143 files are numbered against the 22-stop *gallery* (1–22, 13 absent); the tour has 12 stops. Re-deriving filenames from tour position would renumber every file. Needs an explicit ruling — tangled with the open 22-vs-12 reconciliation.
- **Who emits `archives-{locale}.json`?** Contract keys captions by scene/appendix id; our loader keys by filename. CA/ES require a join (filename → scene/appendix id → locale files) the contract doesn't assign an owner to. Proposal: the publish step emits `archives-{locale}.json` in our loader's filename-keyed shape.
- **Normalization → `.jpg` will break current imageRefs** (ours embed real `.jpeg` / `.png` / `.gif`). Fine if adopted atomically with regenerated refs; flagging so it isn't done piecemeal.
- **Classification field is app-side wiring** — contract ships `cast` / `image` per-image correctly; our gallery model just needs the field added. Our backlog, not a contract gap.
- **Validator ownership** (open-questions): tracker side drafts the `validate-app-data` spec; we implement and own it.
- **Contract self-inconsistency:** ADR-0008 says `caption_source` is in `manifest.csv`, but it's absent from `manifest.schema.json`'s columns. Small, but worth fixing.
- **"Semver" is a `vN` ladder** (`bundle.schema.json:10` accepts `v0`, `v1.2`, `v1.2.3.4`). Either tighten to MAJOR.MINOR.PATCH or call it bundle-version, not semver.
