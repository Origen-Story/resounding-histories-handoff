# ADR 0010 — Bibliography vocabulary locked

**Status:** Accepted · **Date:** 2026-06-10 · **Decided by:** Matt, Emily
**Supersedes:** ADR 0007 (Bibliography categories)

## Context

ADR 0007 defined `Book / Article / Web / Interview` as the bibliography
categories. Emily's v0 contract review flagged three decode failures against
her `Source` Swift model:

1. **Category vocabulary mismatch.** Her model used lowercase + a different
   set (`book / archive / article_essay / digital`). Every ADR-0007 value
   fell to `.unknown` on her side. She specifically asked to bring back
   `archive` — ~42% of the tour's images are archival scans and "Archive"
   is a meaningful source type for this project.
2. **`ref_key` vs `id`.** Her `Source.id` is a required field; the contract
   shipped `ref_key`, no `id` — decode fails.
3. **`year` type.** Contract allowed integer-or-string; hers is `String?`.
   Bibliography years often need text ("c. 1926", ranges like "1878–1883"),
   so committing to string is the right call regardless.

## Decision

### Categories — locked lowercase set

```
book
article
archive
web
```

Lowercase to match Swift enum conventions and her existing decoder. Includes
`archive` (per Emily's pushback — meaningful for an archival-heavy project).
`web` replaces her earlier `digital` (clearer name; digitized books are also
"digital" so the term was ambiguous). `article` covers her earlier
`article_essay` (we don't gain anything by distinguishing essays in a Swift
enum).

### `ref_key` → `id`

The bibliography entry's stable identifier field is renamed from `ref_key`
to `id`. Same semantics — author-chosen short stable string (`B1`, `A12`,
`W3`, `I7`) — just the field name matches Swift conventions.

### `year` → string-only

The `year` field is `string` (no integer-or-string union). Producers SHOULD
emit a plain four-digit year when known ("1926"); MAY use ranges
("1878-1883"), approximations ("c.1900"), or empty string when unknown.
Consumers parse as needed.

## Consequences

- `schema/bibliography.schema.json` updated to reflect all three changes.
- ADR 0007 marked **Superseded by ADR 0010**.
- Tracker's bibliography editor (when built) constrains the category dropdown
  to these four lowercase values.
- Locale-file keys that referenced bibliography entries (`bibliography.<ref_key>.title`)
  become `bibliography.<id>.title` — semantic shift only, no structural change.

## Notes

- The `id` convention encourages first-letter alignment with category
  (`B1`, `A12`, `W3`, `I7`, `R3` for arc**h**ive — though `R` is ambiguous;
  authors may prefer `H1`/`H2` for archive) but this is not enforced by
  the schema. Authors are free to use any short stable identifier.
- If a fifth category is needed post-launch (e.g. `documentary`, `audio`),
  it's a backward-compatible enum extension — schema bump.

## References

- ADR 0007 (superseded): `decisions/0007-bibliography-categories.md`
- App-side review: `status/app-status.md` (entry 2026-06-10)
- Schema update: `schema/bibliography.schema.json`
