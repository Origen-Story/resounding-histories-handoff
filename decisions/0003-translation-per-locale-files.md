# ADR 0003 — Translation: separate per-locale JSON files

**Status:** Accepted · **Date:** 2026-06-04 · **Decided by:** Emily

## Context

The tracker needs to ship trilingual content (EN / CA / ES) at launch. Two
candidate bundle shapes:

A. **Per-locale files** — `en.json`, `ca.json`, `es.json`, each a flat
   key-value map of content ID → string.

B. **Merged-per-record** — every content record carries an inline
   `{ "en": "...", "ca": "...", "es": "..." }` object.

## Decision

**Per-locale files** (option A). Files: `locales/en.json`, `locales/ca.json`,
`locales/es.json`. All three carry an identical key set; the app's inbound
validator enforces parity and flags any missing key.

## Reason

Emily's iOS loader already builds the filename from the active locale
(`archives-{locale}.json` pattern in her existing code). Per-locale files drop
straight into that mechanism and keep Swift models as plain `String` fields.
The merged shape would force a localized-string wrapper type into every model
for content that's all on-device anyway.

## Consequences

- Tracker's publish skill emits three files instead of one.
- Tracker schema adds per-language columns for each translatable field (e.g.
  `title_en`, `title_ca`, `title_es`) rather than a JSONB language map.
- Translation roundtrip (CSV export → translator → CSV import) generates one
  CSV with columns per language but writes back into the per-language columns.
- Missing translation policy: empty string permitted, missing key forbidden.
  Tracker generates all three files together so keys are always in sync.

## Alternatives considered

- **Merged** (option B): rejected for Swift ergonomics.
- **Single file with locale prefix in keys** (`en.scene.42.title`): rejected —
  inflates file size, defeats locale-based lazy loading.
