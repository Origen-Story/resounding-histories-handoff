# Open Questions

Active threads. Add new questions under the right section, mark with a date.
Mark answered ones with their resolution date and either move them to
`## Resolved` at the bottom or delete after acting.

---

## Awaiting Emily's review (next response)

These are queued for Emily's next answer batch. None block her right now —
she said she'd come back with the rest of section 1, 2, 6, 7, 9 and the
non-blocking pieces of 3 and 4. Listed here for visibility.

### App stack specifics (Section 1)
- Swift + SwiftUI / Swift + UIKit / mix? Target iOS version?
- Android plan: Kotlin native, Flutter, React Native, KMP?
- Local storage: SQLite, Core Data, JSON files on disk, Realm?
- Audio engine when audio joins (AVPlayer / AVAudioEngine native, or wrapper)?

### Content delivery (Section 2)
- At install: ship bundle inside the IPA/APK, or fetch on first launch?
- Updates after install: in-app refresh, forced app-store update, OTA?
- Offline mode required once content is cached?
- Per-bundle version number + "what changed" diff?

### Data shape — non-blocking pieces of Section 3
- 3.1 — Confirm spreadsheet columns are still the final image schema
- 3.2 — Hard-enforce title (~25) and short-caption (~50) character limits,
  or just warn?
- 3.3 — Long description in full-screen view: replaces short caption or
  appears below it?
- 3.5 — Cast filter in app: shows all `cast`-flagged images flat (per ADR 0002)
  — confirmed. Anything else?
- 3.6 — Sources display: per stop, per scene, full project bibliography, all?
- 3.7 — Sources categories confirmed as Book/Article/Web/Interview (per
  ADR 0007).
- 3.8 — Credit line column verbatim display? Any formatting expectations?

### Translation (rest of Section 4)
- 4.2 — Final list of what gets translated (and what stays English-only)
- 4.3 — Confirm: stop slugs, character canonical names, filenames stay English-only
- 4.4 — Fallback behavior when a translation is missing (show English /
  empty / placeholder)
- 4.5 — Catalan diacritics + l·l rendering: any special handling needed?

### C2PA / provenance (Section 6)
- Does the app verify C2PA signatures or just display attribution from a sidecar?
- Where does attribution surface in the UI (info button, sheet, credit line)?
- For assets with no C2PA manifest: show source + license from our metadata?

### Publish flow (Section 7)
- Where the bundle lands: shared Dropbox/Drive, Git, S3, direct push?
- Frequency: weekly drops, daily during active dev, on-demand?
- Notification when a new bundle is ready (Slack, email, manifest poll)?
- Should the tracker side draft a starter `validate-app-data` skill spec, or
  does Emily's Claude design it independently and we just match the contract?

### Communication (Section 9)
- Where the conversation continues (Slack channel, GitHub issues here, DMs)?
- Should this Claude read Emily's repo too (read-only), or strict separation?
- Sync cadence (weekly demo / per milestone / async only)?

---

## Awaiting Emily's sign-off (highest priority)

### ADR-0011 — slug-based filename pattern
**Status:** Proposed by Matt 2026-06-10 — awaiting Emily.

Filename pattern proposed: `<stop_id>-hero-NN.jpg` /
`<stop_id>-appendix-NN.jpg` (slug-based, no numeric stop position embedded
in the filename). Resolves the gallery-vs-tour numbering ambiguity Emily
flagged.

See ADR-0011 for full reasoning + open sub-questions:

1. Slug-based filenames vs an alternative (numbered-against-gallery with
   explicit clarification, or some other scheme)?
2. Atomic cutover (regenerate her loader's imageRefs in one change) vs
   transitional `filename-aliases.json` mapping (one bundle of dual naming
   then drop)?
3. Any Swift-side constraints on filename length / character set we should
   know about?

When Emily accepts: ADR-0011 moves to **Accepted**; ADR-0005 status updates
to **Superseded**; manifest + bundle schemas drop the "Proposed pending" notes.

---

## Matt's reciprocal questions

### Stop inventory reconciliation (ADR 0001 follow-up)
**Status:** Inventory published — Emily to reconcile against her 12.

See `inventory/stops-2026-06-04.md`. Summary: **18 listening stops + 4 walking
segments = 22 tour stops**, plus 1 internal-only cross-stop bucket
(`general`) held back from the bundle. The 22 ≠ 12 difference is not the
walking segments — it's that Emily's tour data was built against an older
script revision. Latest tracker structure is now authoritative.

Emily to:
- Cross-check the 18 listening stops against her 12 — note any from her side
  that don't appear in our 18 (we may have renamed) and any of our 18 that
  weren't in her 12 (these are additions she'll need to bring in).
- Confirm walking-segment triggering preference (on departure / arrival /
  time-based / manual).
- Share the coordinates she has for her 12 so we don't re-source them.

### Coordinates for existing listening stops
**Status:** Pending Emily — needs to share what she has.

Emily had lat/lng for the 12 stops she had. To save Matt re-sourcing them,
those can be shared (CSV / JSON / inline in a question reply works). Matt
will fill in any new listening stops the tracker has that weren't in Emily's
12.

### Bibliography surfacing in app (ADR 0007 follow-up)
**Status:** Pending Emily.

Bibliography categories are locked (Book/Article/Web/Interview). Open: where
in the app are entries shown? Per-stop list, per-scene list, full project
library, all of the above? Affects whether each bibliography entry needs
`linked_stops` / `linked_scenes` populated or whether app derives the
relationship some other way.

---

## Resolved

(Move questions here when answered + actioned. Include date and the resolving
ADR if one exists.)

- **2026-06-10** — Stop-structure migration timing: deferred to post-launch.
  ADR-0001 amended by ADR-0009. App keeps stop ownership through launch.
- **2026-06-10** — `arrival_radius_meters` permanently app-owned. Dropped
  from contract. Per ADR-0009.
- **2026-06-10** — Coordinate shape at migration: flat `lat`/`lng` (not
  nested object). Per ADR-0009.
- **2026-06-10** — Bibliography vocabulary: lowercase
  `book / article / archive / web`. ADR-0010 supersedes ADR-0007. Archive
  brought back per Emily's pushback.
- **2026-06-10** — Bibliography `ref_key` → `id`. Per ADR-0010.
- **2026-06-10** — Bibliography `year` → string-only. Per ADR-0010.
- **2026-06-10** — `caption_source` column added to `manifest.csv`.
  Resolves ADR-0008 vs schema inconsistency Emily flagged.
- **2026-06-10** — `bundle_version` pattern tightened to `^v\d+$`; dropped
  "semver" terminology — it's a major-version ladder, not semver.
- **2026-06-10** — `archives-{locale}.json` companion files added to the
  bundle, keyed by filename to match Emily's loader. Tracker performs the
  join at publish.
- **2026-06-10** — Validator ownership split: tracker drafts the
  `validate-app-data` skill spec; Emily implements + owns it.
- **2026-06-04** — Cast classification scope (per-asset, intrinsic) — ADR 0002
- **2026-06-04** — Translation bundle shape (per-locale files) — ADR 0003
- **2026-06-04** — Image format/size/color space on publish — ADR 0004
- **2026-06-04** — Filename pattern — ADR 0005 (now pending supersession by ADR 0011)
- **2026-06-04** — Tracker owns stop structure — ADR 0001 (now amended by ADR 0009)
- **2026-06-04** — Character entity stays tracker-side, ships in bundle — ADR 0006
- **2026-06-04** — Bibliography categories — ADR 0007 (now superseded by ADR 0010)
- **2026-06-04** — Caption model non-destructive — ADR 0008
