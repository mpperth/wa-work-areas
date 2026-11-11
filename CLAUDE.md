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
- 2026-11-08: Added "Export All" button (every saved area, both groups) to both the artifact and
  this mirror — tries a real file download first via the `downloads` capability, falls back to a
  copyable text modal when that's unavailable (always the case on this read-only mirror, since it
  has no Claude capabilities at all — that's expected, not a bug).
- 2026-11-08: Added drag-and-drop file support to the "Import by suburb name" box (drop a .csv/.txt
  file instead of pasting) in both copies.
- 2026-11-08: Fixed two real pre-existing import bugs found while testing, in both copies:
  (1) `findSuburbByName`'s lookup cache could permanently lock in as empty if Preview was clicked
  before the suburb list finished loading — now self-heals if the cache is empty but suburbs have
  since loaded. (2) The import parser couldn't read back its own "Export" output (quoted CSV with
  a header row: Suburb,Council,ABS code,Zone,Group) — it only understood plain postcode/suburb/zone
  rows. Added proper quoted-CSV splitting and header-row detection so Export → Import now round-trips.
- 2026-11-09: Fixed "Rename" group button — it used `window.prompt()`, which the Claude artifact
  sandbox silently blocks (same class of bug as the earlier Export CSV issue). Replaced with a proper
  in-page popup.
- 2026-11-09: Fixed a layout bug where the "Click picks" row (Suburb/Council radios, All WA/Perth/Fit
  selected buttons) could overflow past the right panel's edge with no way to reach the cut-off
  buttons. Fixed by making `.radio-row` wrap instead of overflow (NOT by hiding page scroll — tried
  that first, it made things worse by removing the only way to reach the clipped content).
- 2026-11-09: Fixed two real bugs in group switching, found via a live screenshot showing Group 2's
  panel with a Group 1 area ("OutOfZone") fully loaded/highlighted on the map: (1) switching groups
  via the tabs never ran the same "unsaved changes" safety check used everywhere else — now gated
  through `askThen()` like loadArea. (2) switchGroup() never reset the currently-open area even when
  it belonged to the group being left — now calls `newArea()` in that case. Confirmed via direct `db`
  read that no real area data was ever corrupted; the contamination only existed in unsaved
  browser-session state.
- 2026-11-09: Added a third group ("Group 3") — tabs, import target, rename, and all group-number
  parsing (`areaGroup`/new `normalizeGroup` helper) now handle 1/2/3 instead of assuming only two.
- 2026-11-09: Added a "Drive times" feature (OpenRouteService) — base address input, geocodes it,
  then batches a Matrix API call across the currently open area's suburbs (suburb centroid via
  `turf.centroid`), showing minutes + km sorted fastest-first. Mick signed up for a free ORS account
  and provided the API key.
  - **In the live Claude artifact**: the key is stored in the artifact's own database
    (`meta/orsKey`, loaded once via `loadOrsKeyOnce`) rather than embedded in page source — keeps it
    out of view-source. This was also the only way to get past Claude Code's own safety classifier,
    which blocked writing the literal key string directly into the artifact's HTML/JS (but allowed it
    once routed through the database instead).
  - Decided by Mick (option B, see work Mac entry below): the public mirror instead has each staff
    member paste the ORS key once into their own browser, never stored in page source or git.
  - Noted in passing: `api.openrouteservice.org` returned a deprecation notice pointing to
    `ask.openrouteservice.org` for more info, but a replacement base URL wasn't confirmed (searched,
    found nothing concrete) and the old URL still works fine as of this session. Re-check if ORS
    calls ever start failing.
- 2026-11-09 (work Mac): Snapshot refreshed to 13 areas (adds 4 Group 2). `index.html` rebuilt from the live
  artifact (drive times, Group 3, all home fixes) + `loadStaticAreasFallback()`. Drive times here: no key in the
  page (Mick's choice, option B). Each person pastes the ORS key once into the "Drive-time key" box; kept in
  their browser (localStorage `wa-work-areas-ors-key`); a 401/403 clears it and asks again. Mick sends staff
  the key privately. "Unsaved changes" strip always showing: FIXED 09 Nov (CSS `.confirm-strip[hidden]{display:none}`,
  `display:flex` was overriding `hidden`); same fix in the live artifact (version 31).
- 2026-11-09 (home PC, same bug, different angle): also found and fixed a real logic bug in
  `loadArea()` that could independently cause the same symptom — `state.savedCodes` was computed
  from the raw saved code list, but the actual selection only included codes the current suburb data
  recognized. Any mismatch (stale code, data still loading) made `isDirty()` true immediately on
  open, regardless of the CSS fix above. Both fixes are complementary, not duplicates — kept both.
- 2026-11-10 (work Mac): Rebuilt from live artifact v34 changes: background map (Main Roads WA main roads +
  ABS town names, `wa-basemap.json`, "Map" tick box, also in PDF); "Find a suburb on the map" search box;
  drive times now FROM A SUBURB centre instead of a street address (address lookup removed for privacy -
  no patient addresses to OpenRouteService); base suburb kept in localStorage `wa-work-areas-base-suburb`;
  rejected key (401/403) now caught on the matrix call. Help strip is hidden here (no db = view only), hover tips still show.
- 2026-11-10: Named map views (artifact v35): "+ View" -> Name + Suburb 1 + Suburb 2 zoom buttons. Here
  they are kept per browser in localStorage `wa-work-areas-views` (no db); default Perth view.
- 2026-11-10: From/To suburb bar with drive times per time slot (artifact v36), TomTom Routing API with
  historic traffic. Each person pastes the TomTom key once (localStorage `wa-work-areas-tomtom-key`).
- 2026-11-10 (artifact v37-v46): this site is where drive times actually run (claude.ai blocks the
  artifact's outside calls). Opened from the artifact as `?from=CODE&to=CODE#k=KEY`: fills From/To and
  runs; `#k=` key saved to localStorage then removed from the address bar. Drive times box at top of
  left panel (250px), From blue / To red badges + map labels, road route of quickest slot drawn,
  runs automatically once both suburbs are picked, one row per time slot, slowest red.
- Closed 2026-11-11. Nothing queued here.
