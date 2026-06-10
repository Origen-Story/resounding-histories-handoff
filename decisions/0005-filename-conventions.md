# ADR 0005 — Filename conventions

**Status:** Accepted · **Date:** 2026-06-04 · **Decided by:** Emily

## Decision

Images in the publish bundle use these filenames:

- **Scene heroes:**     `stop-NN-hero-NN.jpg`
- **Stop appendix:**    `stop-NN-appendix-NN.jpg`
- **Character portraits:** `character-<character_id>.jpg`

Rules:

- Both `NN` positions are **two-digit zero-padded** (`stop-01-hero-01.jpg`, not
  `stop-1-hero-1.jpg`).
- The first `NN` is the stop's tour position (1-based, matches
  `stops.json[].stop_number`).
- The second `NN` is the asset's order within its role at that stop (1-based,
  separate counters for hero vs appendix).
- Extension is always `.jpg` after normalization regardless of source format.
- `character-<id>.jpg` uses the tracker's `characters.id` integer, no padding,
  since IDs aren't ordinal.

## Consequences

- Tracker's publish skill renames on export (originals keep their natural names
  in the tracker; normalized derivatives match this pattern).
- Validator on app side checks every file in `/images/` matches one of these
  patterns and references a known stop/character.

## Notes

- Original source filenames (often messy — `Pati_de_l_27Hospital_de_l_C3_27...jpeg`)
  are preserved in `manifest.csv` under a separate column for round-trip
  traceability.
