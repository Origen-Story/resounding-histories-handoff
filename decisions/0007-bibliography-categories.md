# ADR 0007 — Bibliography categories

**Status:** Accepted · **Date:** 2026-06-04 · **Decided by:** Matt

## Decision

The bibliography supports exactly four categories:

- **Book**
- **Article**
- **Web**
- **Interview**

These are the values permitted in `bibliography.json[].category`.

## Reason

These four cover the source types the tour actually draws on. "Archive" was
considered and rejected as too broad — archival items are typically referenced
indirectly via specific articles, books, or web pages.

## Consequences

- Tracker bibliography editor exposes a dropdown limited to these four.
- App's Sources display (when implemented) can group by these four; iconography
  / typography choices are Emily's call.
- Migration path: if a fifth category is ever needed (e.g. "Documentary"), it's
  an enum extension — schema bump to v0.x with a backward-compatible add.

## Notes

- The `ref_key` convention encourages first-letter alignment with category
  (`B1`, `A12`, `W3`, `I7`) but this isn't enforced by the schema. Authors are
  free to use any short stable key.
