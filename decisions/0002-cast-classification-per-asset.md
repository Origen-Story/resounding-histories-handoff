# ADR 0002 — Cast classification is a per-asset intrinsic flag

**Status:** Accepted · **Date:** 2026-06-04 · **Decided by:** Emily

## Context

The app has a Cast vs Image filter in the Archives section. The question was
whether classification is *per-asset* (intrinsic — a portrait of Gaudí is Cast
wherever it appears) or *per-use-site* (the same image could be Cast in one
scene and Image in another).

## Decision

**Per-asset, intrinsic.** An image is Cast if and only if it's a portrait of
a person who appears in the audio. Each archival/produced asset carries a
single `classification` field with one of:

- `cast` — portrait of a person who appears in the audio
- `image` — everything else (buildings, documents, maps, scenes, etc.)
- `''` (empty) — not yet classified

## Consequences

- Tracker adds a `classification` text column to `archival_assets` and
  `produced_assets`.
- Tracker UI: a small Cast/Image toggle on every asset card in the archival
  list and detail view; bulk-classify action in triage mode.
- Publish bundle: `classification` flows through to `manifest.csv`.
- App's Cast filter shows all `cast`-classified images **flat** at v0 launch —
  no per-character grouping (see ADR 0006 for why character data is published
  but not consumed at launch).

## Alternatives considered

- **Per-use-site**: rejected — would force every scene-hero / appendix pairing
  to re-decide a question that has a stable intrinsic answer.
