# Publish Bundle — v0

The bundle is what flows from the tracker to the app. v0 is **pre-launch** —
expect the shape to evolve until both sides have validated it end-to-end. Once
the app is consuming v0 cleanly we promote to v1 and lock the shape.

## Delivery

Today: ZIP file dropped in a shared location (TBD — see open-questions.md).
Later: direct push to whatever delivery channel the app needs.

## Top-level layout

```
<bundle>.zip
├─ bundle.json              ← self-describing manifest (schema: bundle.schema.json)
├─ manifest.csv             ← flat per-image table (schema: manifest.schema.json)
├─ stops.json               ← tour structure (schema: stops.schema.json)
├─ bibliography.json        ← bibliography (schema: bibliography.schema.json)
├─ characters.json          ← characters (schema: characters.schema.json) — app may ignore at v0 launch
├─ locales/
│  ├─ en.json               ← English strings (schema: locale.schema.json)
│  ├─ ca.json               ← Catalan strings, same keys as en.json
│  └─ es.json               ← Spanish strings, same keys
└─ images/
   ├─ stop-NN-hero-NN.jpg
   ├─ stop-NN-appendix-NN.jpg
   └─ character-<id>.jpg    ← optional portraits
```

All filenames are lowercase. Stop and hero/appendix indices are two-digit
zero-padded (`stop-01-hero-01.jpg`, not `stop-1-hero-1.jpg`).

## Image normalization rules

- **Format**: JPEG. PNGs converted, GIFs converted (after confirming not animated)
- **Color space**: sRGB. Source non-sRGB and grayscale are converted
- **Quality**: q ≈ 85
- **Dimensions**: max 1600 px on the long edge. **Never upscale** — small archival
  scans stay at native size
- Grayscale sources stay grayscale JPEG (smaller, truer)

See `decisions/0004-image-normalization-on-publish.md`.

## Locale files

Three files: `en.json`, `ca.json`, `es.json`. Same key set in all three. Keys
are dot-delimited:

```json
{
  "stop.hospital_santa_creu.name": "Hospital de la Santa Creu",
  "scene.42.title": "EXT. FRONT ENTRANCE OF HOSPITAL - NIGHT",
  "scene.42.hero.title": "Hospital entrance",
  "scene.42.hero.short_caption": "Where Gaudí spent his final hours",
  "scene.42.hero.long_description": "...",
  "scene.42.beat": "...",
  "appendix.17.title": "...",
  "character.5.bio": "..."
}
```

Missing translations: empty string is permitted (`""`), missing keys break
validator parity check. Tracker generates all three files together.

## Stop structure ownership

**Tracker owns it.** The app reads `stops.json` as authoritative for the
canonical stop list, order, slugs, and coordinates. The app may continue to
own runtime concerns (arrival-detection algorithm, walking directions, map
rendering) but the *data* lives with the tracker.

See `decisions/0001-tracker-owns-stop-structure.md`.

Stop types:
- `listening` — physical stops the user stands at. Have coordinates.
- `walking` — transition narration between two listening stops. Coordinates optional.
- `meta` — utility buckets (e.g. `general` cross-stop) not rendered as tour stops.

## Cast classification

Per-asset intrinsic flag on every image: `cast` or `image`.

- **Cast** = portrait of a person who appears in the audio
- **Image** = everything else (buildings, documents, maps, scenes)

App's Cast filter shows all `cast`-classified images, flat (no per-character
grouping at launch). See `decisions/0002-cast-classification-per-asset.md`.

## Characters

Published in `characters.json` even though the app's v0 launch doesn't use them.
This lets the tracker stay the source of truth for who-appears-where (the data
is auto-sync'd from the FDX) and lets the app pick character data up post-launch
without a schema bump. See `decisions/0006-character-entity-maintained-tracker-side.md`.

## Bibliography

Categories: **Book**, **Article**, **Web**, **Interview**. See
`decisions/0007-bibliography-categories.md`. Entries can link to stops and/or
scenes; the app surfaces a per-stop "Sources" list (display details TBD —
see open-questions.md).

## Caption model (non-destructive)

The tracker keeps the *ingested* caption from the source archive as immutable
reference (`caption_source` — never sent to the app, never edited). The
app-facing fields are authored separately:

- `title` (~25 chars, one line under thumbnail)
- `short_caption` (~50 chars, two lines under thumbnail)
- `long_description` (1–3 sentences, full-screen view)

These live **per-use-site** — a scene's hero has its own caption set; the same
asset appearing in another stop's appendix gets its own. All three are
trilingual. See `decisions/0008-caption-model-non-destructive.md`.

## Bundle metadata

`bundle.json` at the root is self-describing — it carries the bundle version,
tour ID, generation timestamp, tracker commit SHA, and file counts. The app's
inbound validator reads this first to know what it's getting.

## What's NOT in this bundle (yet)

- **Audio**. Deferred until the audio workstream starts. Format/bitrate/segmentation
  spec'd later.
- **C2PA signatures / sidecar manifests**. Tracker has the framework but isn't
  yet auto-signing on export. Will appear as `<image>.c2pa.json` sidecars when
  wired up.
- **Tour-level metadata** (featured stop, splash, tour hero image). All app-side
  per Emily's section 8.

## Validation

Each side runs validation:

- **Tracker** (publish skill): produces a bundle that conforms to every schema
  in `schema/`. Hard-fails on any miss.
- **App** (inbound validator): reads `bundle.json`, then validates each file
  against its referenced schema. Checks locale key parity. Hard-fails on any
  miss with a specific error.

The two validators use the same schemas in this repo — when a schema changes,
both sides see it on next pull.
