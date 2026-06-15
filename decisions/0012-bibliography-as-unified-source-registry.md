# ADR 0012 — Bibliography as the unified source registry

**Status:** Accepted (tracker-internal) · **Date:** 2026-06-14 · **Decided by:** Matt

## Context

Originally the tracker had a polymorphic `scene_sources` table where each row
carried either an internal-asset reference (archival/produced/ai_asset) OR an
inline external citation (`externalUrl`, `externalTitle`, `citation`). This
worked but created friction in two ways:

1. **No way to deduplicate** — citing the same book from three scenes meant
   typing the citation three times, with drift risk.
2. **No way to track unattached sources** — background reading or "informs
   the project generally" references had no home; you'd either invent a
   throwaway scene to attach them to or skip recording them entirely.
3. **One-off vs multi-use was a forced upfront choice** that often turned
   out wrong. A "one-off" external citation often became multi-use later,
   requiring re-typing.

## Decision

The **bibliography** table becomes the project-wide source registry. Every
source the project uses lives there, regardless of scene linkage.

`scene_sources` becomes a thin **M:N link table** between scenes and
bibliography entries, carrying only the per-scene `influenceNote` ("this is
where the 'cantarets de barro' detail comes from"). All citation data —
title, author, year, link, category, notes — lives on the bibliography row.

Bibliography entries can optionally **point to an internal asset** (archival/
produced/ai_asset) for sources that ARE a file in the tracker — e.g. citing a
Wikimedia image you've already imported. For pure-citation sources (books,
web articles), all three internal-asset FKs are null.

### Model

```
bibliography
├─ id (serial)
├─ project_id
├─ ref_key (text, optional — author-chosen short key like "B1", "VANH09")
├─ title, author, publisher, year (string), link, category (book/article/archive/web)
├─ notes
└─ archival_id / produced_id / ai_asset_id (one-of, optional)

scene_sources
├─ scene_id
├─ bibliography_id  ← the canonical reference
├─ influence_note   ← per-scene gloss
└─ sort_order
```

### UX consequence

A single **+ Source** button on the scene editor opens a bibliography picker:
- Search/filter the existing list, click to attach
- OR click "+ Create new" inline to add a brand-new entry AND attach to this
  scene in one action

One workflow regardless of whether the source is a one-off or already in the
library. Promoting a one-off to multi-use needs no migration — you just
attach the existing entry to a second scene.

Background-reading sources that don't apply to a specific scene live in the
bibliography with zero scene links — they exist in the project but don't
clutter any scene's source list.

## Consequences (tracker-internal)

- New `bibliography` table (see schema migration `0008_clumsy_wild_child.sql`)
- `scene_sources` gains a `bibliography_id` FK column
- Legacy polymorphic columns on `scene_sources` (`sourceType`, `archivalId`,
  `producedId`, `aiAssetId`, `externalUrl`, `externalTitle`, `citation`) stay
  nullable during the transition window. A follow-up schema migration drops
  them after the data-migration script runs cleanly.
- New `npm run bibliography:migrate` script — idempotent, dry-run-able,
  walks every unmigrated `scene_sources` row, finds-or-creates a bibliography
  entry for it (deduplicating identical citations), and sets the
  `bibliography_id` link.
- New `/bibliography` route in the tracker with list + edit pages.

## Consequences (handoff bundle)

The v0 bundle's `bibliography.json` already exists in the schema set; this
ADR just confirms the tracker's data model can produce it. ADR 0010
(bibliography vocabulary) and `bibliography.schema.json` are unchanged.
`scene_sources` is tracker-internal — only the *resolved* bibliography
entries flow to the app, deduplicated, with optional `linked_stops` and
`linked_scenes` arrays derived at publish time from the join table.

No impact on Emily's side or the v0 contract.

## References

- ADR 0010 (bibliography vocabulary): `decisions/0010-bibliography-vocabulary-locked.md`
- Schema: `schema/bibliography.schema.json`
- Tracker migration `0008_clumsy_wild_child.sql`
- Tracker migration script: `npm run bibliography:migrate`
