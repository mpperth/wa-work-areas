# WA Work Areas (standalone mirror)

This folder is a **read-only, no-login copy** of the real tool, which is a Claude Artifact:
`https://claude.ai/code/artifact/a20e4182-5799-4b4f-aaf0-93f2db0f8517`

The artifact is where Mick actually works (saves/edits/deletes areas, imports suburb lists).
This repo exists only so colleagues without a Claude account can open a plain link and pick
suburbs / view saved areas — Claude Team plan links force outside people to "join the org,"
which colleagues couldn't handle, so this static copy is the workaround.

Public repo `mpperth/wa-work-areas`, hosted on GitHub Pages (must stay **public** - private repos
need paid GitHub for free Pages). Live at: `https://mpperth.github.io/wa-work-areas/`

## Rules for this project

- **This folder is not the source of truth.** The Claude artifact is. Never edit `index.html`
  here first - make changes in the artifact's scratchpad copy, test/publish there, then copy
  the same change here so the two don't drift.
- `areas-data.json` is a **manual snapshot**, not live. It only updates when someone re-runs
  `read_db` on the artifact's `areas` collection and regenerates this file by hand. Tell Mick
  it's a snapshot (the page already says so) whenever he's confused about stale data.
- No `db`/`user`/`downloads` capability works here (no Claude account = no `window.claude`).
  Saved areas are view-only; Save/Delete/Import buttons are naturally disabled via the existing
  `state.db` checks. Don't try to make writes work here - that defeats the whole point.
- `git push` to this repo intermittently hangs for 1-2 minutes with no error before succeeding.
  Not a real failure - just retry the same `git push`.
- Ask Mick before committing/pushing, same as every project.

## How it works (key parts)

- `wa-suburbs.json` / `wa-councils.json` (10MB / 5.5MB): ABS suburb/council boundary data.
  Can't be fetched by tools directly (same-origin only inside the live artifact) - Mick pulled
  these once via browser DevTools (Network tab -> row -> Response tab -> copy -> save as).
  Static files, never need to change.
- `index.html` = the artifact's inner content wrapped in a real `<!doctype html><html><head>...
  <body>` shell (the artifact platform injects that wrapper itself; a standalone page needs its
  own). Same map/suburb-picker/draw-shape code as the artifact.
- `loadStaticAreasFallback()` in the script: when `window.claude` is missing, fetches
  `areas-data.json` and populates `state.areas` read-only instead of showing "not available."
- Suburb name matching (`normName()`) strips a trailing `" (WA)"` ABS disambiguation suffix
  (e.g. "Butler (WA)") - without this, ~1/3 of real suburb names fail to match. Fixed in the
  artifact's script; this copy's import is inert anyway (no db) so it doesn't matter here, but
  keep it in sync if `index.html` is ever re-copied from the artifact.
- Groups: the artifact has a Group 1/Group 2 tab concept (per-area `group` field, 1 or 2).
  As of 2026-11-07 everything lives in Group 1 - Mick tried splitting by zone data quality
  (Group 2 = new zone import) then merged back to one group once the import was done.

## Resume

- 2026-11-07: Created this repo from the artifact's scratchpad HTML (after groups/rename/layout
  features were built in the artifact). Pushed `index.html` + suburb/council JSON, enabled
  GitHub Pages (branch main, root). Live at the URL above.
- 2026-11-07: Added `areas-data.json` read-only snapshot + `loadStaticAreasFallback()` so the
  standalone copy shows saved areas (view-only) instead of "not available."
- 2026-11-07: Fixed snapshot data for zones N3/OutOfZone/S1O/SE_S (suburb-name matching had been
  dropping `"(WA)"`-suffixed names like "Butler (WA)"; root cause fixed in the artifact's
  `normName()`, snapshot regenerated with correct codes).
- 2026-11-07: Removed the 4 original test areas (Mandurah to Yanchep, Outer North, Teresa inner,
  Teresa Outer south) from both the artifact DB and this snapshot - Mick confirmed those were
  just throwaway test data ("group 2 is important other one is play").
- 2026-11-07: Moved all 9 real zone areas (E1/N1/N2/N3/OutOfZone/S1/S1O/S2/SE_S) from Group 2
  back into Group 1 in both the artifact DB and this snapshot - Mick decided to drop the
  two-group split and keep one group.
- 2026-11-07: Added Show all / Hide all buttons for the saved-areas map overlay checkboxes.
- **Still outstanding**: E1, N1, N2, S1, S2 have incomplete suburb lists (the paste Mick had
  only captured ~24 of 44-82 suburbs per zone from a scrollable grid widget - see the artifact
  conversation for the full truncation analysis). Needs a complete source list for those 5
  zones before their codes are trustworthy. N3/OutOfZone/S1O/SE_S are confirmed complete.
- "Two Rocks to Bouvard" was mentioned by Mick as the main working area but no action was taken
  yet - unclear if he wants a new saved area created for it or it's just context.
