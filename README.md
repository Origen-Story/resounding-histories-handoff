# Resounding Histories — Handoff Repo

Shared coordination space for the **Year of Gaudí** tour project. Two systems, two
codebases, one tour. This repo is how they talk to each other.

## Who uses this

| Side | Person | Repo | Role |
|---|---|---|---|
| **CMS / publisher** | Matt + Claude | `provenance-tracker` (private) | Authors content, signs assets, produces the publish bundle the app ingests |
| **Mobile app** | Emily + Claude | (her repo) | Consumes the publish bundle, renders the tour and archives |

## What's in here

```
schema/         JSON Schemas — the enforceable bundle contract
contracts/      Human-readable spec of what's in a publish bundle
status/         Living snapshots of "what we shipped this week" per side
decisions/      Architecture Decision Records (ADRs) — one file per call
open-questions.md   Active back-and-forth, dated, with answers when known
```

## How we use it

### When making a real architectural decision

1. Write a new ADR in `decisions/NNNN-short-name.md` using the template (see existing ones)
2. Bump the schema if applicable, append to `schema/CHANGELOG.md`
3. Push

### When you ship something material

Edit your side's `status/{tracker,app}-status.md`. Date the change, one or two
lines on what landed. Push.

### When you're stuck on a question for the other side

Add it to `open-questions.md` under the right section. The other side answers
inline (with the date). Once answered and the action's taken, move it to a
"Resolved" subsection or delete it.

### Schema changes are not silent

If `schema/*.json` changes:
- Append a one-line entry to `schema/CHANGELOG.md`
- Update `contracts/bundle-v*.md` if the human-readable spec drifts
- Mention it in the next status update

## Versioning

Schemas live at `vN` (currently `v0` — pre-launch, expect breakage). Once the
app launches we promote to `v1` and any breaking change becomes `v2`. Older
schemas stay in the repo so old bundles validate against the schema they
shipped against.

## For each side's Claude Code

Add this line to your project's `CLAUDE.md`:

> Before significant work, read `../resounding-histories-handoff/status/`
> and `decisions/` for context from the other side. After shipping something
> material, update your side's status file.

That's the whole protocol.

## Communication channels (out of band)

This repo is for **contracts, decisions, and status**. For active discussion
that hasn't crystallized yet, use whatever you'd normally use (Slack, email,
DM). Move it here when it's ready to be canonical.
