# ADR 0004 — Image normalization happens on publish

**Status:** Accepted · **Date:** 2026-06-04 · **Decided by:** Emily

## Context

The tracker stores archival images as ingested — a raw mix of formats, color
spaces, and dimensions (today: ~97 jpg / 40 jpeg / 5 png / 1 gif, ~42%
grayscale, ~35 files under 800px). The app needs a single normalized form for
predictable rendering and bundle size.

## Decision

The tracker's publish step normalizes images to:

- **Format:** JPEG
- **Color space:** sRGB
- **Quality:** ~85
- **Dimensions:** max 1600 px on the long edge — **never upscale**
- **Grayscale source → grayscale JPEG** (smaller, truer)
- **PNG → JPEG** (convert)
- **GIF → JPEG** (after confirming not animated; if animated, flag in
  validation report)

**Originals stay in the tracker** as provenance. Only normalized derivatives
ship in the bundle.

## Consequences

- Tracker adds `sharp` (npm) for image processing.
- Publish skill produces `images/stop-NN-{hero,appendix}-NN.jpg` regardless
  of source format. The original extension is recorded in `manifest.csv` for
  traceability.
- The non-publish exports (e.g. `/api/export/scenes-zip`) keep shipping
  originals — different audience (designers, archivists) need the raw files.
- Validator on the app side checks output format/size/color space and
  hard-fails if a file violates the spec.

## Alternatives considered

- **Ship originals**: rejected — Emily's app would need to handle every format
  and the bundle would be bloated by oversized scans.
- **Multiple resolutions (@1x/@2x/@3x)**: rejected — single normalized size
  keeps the bundle small and the app loader simple. iOS scaling handles
  display.
- **Pre-resize on import**: rejected — losing fidelity at the wrong end of the
  pipeline. Publish-time is the right moment.
