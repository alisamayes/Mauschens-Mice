# Memory: game-map-designer / dragonflight

Lessons captured by `game-map-designer` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: game-map-designer-dragonflight-20260508-fafe93
created: 2026-05-08
updated: 2026-05-08
agent_type: game-map-designer
scope: project
project_slug: dragonflight
task_type: docs
tags: [hex-coords, offset-vs-axial, map-tooling, schema-doc]
trigger: Slice 1 round 0 silently inherited the bundled hex editor's q,r field names as axial; the rendered map looked like a rhombus instead of a square and the user diagnosed it visually in round 2 as odd-q flat-top offset coords.
evidence: Documentation/map-schema.md round-0 top bullet read "Coordinate system: axial (q, r)"; round-2 user correction rewrote it to "odd-q flat-top offset (q = column, r = row); axial is derived via offset_to_axial". The Coder (Python Backend) refactored OffsetCoord(col, row) in parallel.
observed_count: 1
lesson: When importing a hex map from an external authoring tool for Dragonflight, verify and document the coordinate convention (offset vs axial vs cube, plus parity for offset variants) explicitly before treating field names like q,r at face value.
do: On first contact with any hex map data file, render or visualise a known-shape sanity check (e.g. is a square-bounded grid actually rendering as a square?) and inspect the editor's documentation or two same-row tiles' pixel positions to confirm the convention before writing it into Documentation/map-schema.md.
dont: Assume that letters q,r in a JSON field name imply axial coordinates just because Dragonflight's spec uses axial elsewhere; the bundled editor exports odd-q flat-top offset coords using the same letters.
rationale: Misidentifying the coordinate convention cascades into the loader, the renderer, and every section of the schema doc, and the rhombus-vs-square symptom only surfaces visually long after the data and docs are committed.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: game-map-designer-dragonflight-20260508-8d74c3
created: 2026-05-08
updated: 2026-05-08
agent_type: game-map-designer
scope: project
project_slug: dragonflight
task_type: docs
tags: [schema-doc, loader-contract, role-collaboration, drift]
trigger: Slice 1 round 0 schema doc listed `woodland` as an accepted alias for the built-in `forest` hexType; the Coder (Python Backend) implemented the loader without that alias, and Documentation later removed `woodland` from the schema doc to match the code.
evidence: The slice-level Revision History recorded the `woodland` alias drift between Documentation/map-schema.md and the loader; round-2 read of map-schema.md showed line "The string `woodland` itself is not accepted as a `hexType` value" replacing the previous alias claim.
observed_count: 1
lesson: When authoring a Dragonflight map schema document that the Coder (Python Backend) will implement as a loader, only document values that have been agreed with the implementing role; mark anything else as "proposed, not yet accepted" or omit.
do: For each entry in the schema doc's built-in vocabulary table and customHexTypes name table, either confirm with the implementing coder that the loader accepts that exact string, or label the entry "proposed, pending loader support" so the doc cannot silently overstate the contract.
dont: Add aliases or "convenience names" to the schema doc unilaterally because they look reasonable; the doc is the loader's contract, and unilateral additions become silent loader-vs-doc drift that another role has to clean up.
rationale: The Game Map Designer's schema doc is consumed both by the Coder (Python Backend) implementing the loader and by future map authors; a doc that lists vocabulary the loader doesn't accept is worse than a doc that omits it.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: game-map-designer-dragonflight-20260508-333f46
created: 2026-05-08
updated: 2026-05-08
agent_type: game-map-designer
scope: project
project_slug: dragonflight
task_type: docs
tags: [map-loader, validation, fail-loud, vocabulary]
trigger: Slice 1 round 0 schema doc said the loader should "warn and fall back to GRASSLAND" on unknown built-in or custom-type names; the round-1 user correction tightened this to "fail map load with a clear error".
evidence: Documentation/map-schema.md §6 step-2 sub-bullet and §7 final paragraph both changed from a warn-and-coerce stance to fail-loud in round 1, after the user's response to the round-0 follow-ups about the 3 `sea` tiles and the unrecognised-vocab policy.
observed_count: 1
lesson: Document the Dragonflight map loader as fail-loud on unknown vocabulary (unknown built-in name, unknown custom-type id, unknown reserved customHexTypes name); never silently coerce to a default like GRASSLAND.
do: In Documentation/map-schema.md, write the unknown-vocab branch of every resolution rule as "fail map load with a clear error" pointing at the offending key, and surface this fail-loud contract once at the top of the resolution-rule section so it cannot be missed by the loader implementer.
dont: Specify a "warn and fall back to GRASSLAND" or any silent-coercion behaviour for unknown hexType vocabulary in Dragonflight; silent coercion masks authoring mistakes (`ocean`, `sea`, typos) and lets a malformed map render as a meaningful one.
rationale: Silent coercion turns map-authoring bugs into invisible balance bugs (a river silently rendered as grassland is reachable to armies); fail-loud surfaces the bug at load, when it is cheap to fix.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: game-map-designer-dragonflight-20260508-da6064
created: 2026-05-08
updated: 2026-05-08
agent_type: game-map-designer
scope: project
project_slug: dragonflight
task_type: docs
tags: [handoff, clarification, follow-ups, map-vocab]
trigger: In Slice 1 round 0 the brief asked only about `Village` and `ocean` but the file also contained 180 `forest` and 3 `sea` tiles; surfacing each as a distinct labelled follow-up with count and sample coords let the user resolve both in a single round-1 reply.
evidence: Round-0 handoff "Known follow-ups" listed (1) `forest` 180 tiles → mapping unstated and (2) `sea` 3 tiles at (4,26)/(8,27)/(23,27) → not addressed; round-1 user reply resolved both in one message ("forest_to_woodland" + "sea_to_river").
observed_count: 1
lesson: When a Dragonflight map JSON contains tiles using vocabulary the brief did not address, surface every distinct unexpected hex type in the handoff "Known follow-ups" section with at least its tile count and one sample (q, r), even when the brief's explicit asks are themselves unambiguous.
do: After completing the explicit asks of a map-edit task, list each unexpected built-in or custom-type id encountered as its own follow-up bullet with count and a representative offset coordinate, so the user can resolve all of them in one reply rather than one per round.
dont: Mention only the unexpected categories that block the explicit asks, or bundle several unexpected categories under a single vague "minor follow-ups" line; that forces a round per category instead of one round for all.
rationale: Map data files are denser than briefs; the brief almost never enumerates every category present, and structured per-category follow-ups collapse N revision rounds into one.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: game-map-designer-dragonflight-20260508-9fa3f8
created: 2026-05-08
updated: 2026-05-08
agent_type: game-map-designer
scope: project
project_slug: dragonflight
task_type: docs
tags: [validation, tooling, hex-vocab, before-after]
trigger: Slice 1 round 0 the brief named only `ocean` for water-tile retyping; a Counter audit of `hexType` values revealed both `ocean` (103 tiles) and a separate `sea` (3 tiles), plus 180 `forest` tiles the brief had also implied did not exist.
evidence: Counter output {grassland: 539, forest: 180, ocean: 103, mountain: 61, custom-5c5b120e: 9, custom-fe6e3b0a: 4, sea: 3, custom-0d9e6285: 1} from the round-0 audit; same Counter pattern was then used as before/after assertions in every subsequent revision round and caught zero drift defects.
observed_count: 1
lesson: Before editing any Dragonflight map JSON, run a Counter audit on every tile's hexType to enumerate the full vocabulary and spot near-synonyms (e.g. ocean vs sea) or unrelated categories the brief didn't mention.
do: Open every map-edit task with a Python `Counter(t.get('hexType') for t in data['hexes'].values())` print, sort by frequency, and treat any category not explicitly addressed by the brief as a follow-up candidate; then re-run the same Counter after each edit and assert expected post-edit counts in a validation script.
dont: Trust the brief's enumeration of categories present in the file; the bundled editor produces near-synonyms (notably both `ocean` and `sea`) that briefs routinely conflate or omit.
rationale: A 30-second Counter audit cheaply replaces the assumption that the brief enumerates every category present, prevents missed near-synonyms, and gives every revision round a tile-count regression guard for free.
confidence: medium
status: active
superseded_by: null
review_after: null
```
