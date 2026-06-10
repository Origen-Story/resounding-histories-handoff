# ADR 0008 — Caption model: non-destructive, per-use-site, trilingual

**Status:** Accepted · **Date:** 2026-06-04 · **Decided by:** Matt

## Context

Initial tracker design overwrote the asset's ingested caption with authored
text. This is destructive — the original archive metadata (often useful as a
writing aid or for source attribution) is lost. The Cast/Image filter spec
from Emily's metadata template also calls for three distinct caption fields
per image with hard character limits, not a single free-form caption.

## Decision

Captions split into four fields, conceptually:

1. **`caption_source`** — the caption text as ingested from the source archive
   (Wikimedia, etc.). **Preserved as reference, never edited, never shipped to
   the app.** Used in the tracker UI as a writing aid.
2. **`title`** — short heading (~25 chars, fits one line under thumbnail).
   Authored.
3. **`short_caption`** — ~50 chars (two lines under thumbnail). Authored.
4. **`long_description`** — 1–3 sentences for full-screen view. Authored,
   optional.

The three authored fields:
- Exist **per-language** (EN / CA / ES)
- Exist **per-use-site** (a scene's hero may want a different short caption
  than the same asset's appearance in another stop's appendix)

The asset's `caption_source` stays at the asset level (one per archival/produced
asset) and is the only caption-like field that's NOT per-use-site.

## Consequences

- Tracker schema: replace `captionOverride` on scenes' hero relationship
  (currently a single string) with three trilingual fields tied to the
  scene-hero use-site: 9 fields per scene hero.
- Same shape on `stop_appendix_assets` for each appendix slot.
- Tracker editor exposes the three fields × three languages stacked; English
  is required, CA/ES optional and filled in via the translator roundtrip.
- Publish bundle: per-locale files carry these strings keyed by
  `scene.<id>.hero.{title,short_caption,long_description}` and
  `appendix.<id>.{title,short_caption,long_description}`.
- Asset `caption_source` is included in `manifest.csv` for provenance, but
  doesn't appear in locale files.

## Alternatives considered

- **Overwrite ingested caption** (current state): rejected — destructive,
  loses provenance.
- **Single authored caption per asset** (not per-use-site): rejected — same
  photo can serve different narrative contexts and deserves context-specific
  framing.
- **Single authored caption per use-site** (not split into 3 sizes): rejected —
  Emily's app needs three sizes for three display contexts.
