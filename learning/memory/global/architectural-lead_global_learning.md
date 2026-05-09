# Memory: architectural-lead / global

Lessons captured by `architectural-lead` while working across projects. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: architectural-lead-global-20260508-0ab312
created: 2026-05-08
updated: 2026-05-08
agent_type: architectural-lead
scope: global
project_slug: global
task_type: feature
tags: [architecture, contracts, third-party-data, coordinates, conventions]
trigger: Slice 1 of Dragonflight locked `Tile.coord: AxialCoord` because the JSON's keys were named `q`/`r` and the project's spec used axial vocabulary; the user's "rhombus instead of rectangle" feedback in revision round 1 revealed the bundled hex editor actually exports odd-q flat-top offset coordinates under the same field names, forcing a downstream refactor of the data type, the loader, and the renderer.
evidence: Architectural memo round 0 specified `Tile.coord: AxialCoord` without consulting the actual visual layout of the example map; round-1 revision amended the contract to `Tile.coord: OffsetCoord` with `offset_to_axial`/`axial_to_offset` derivations after the user's correction was recorded in the slice-level Revision History (game-feature playbook step 10).
observed_count: 1
lesson: Verify that any external authoring tool's data uses the coordinate or vocabulary semantics implied by its field names before locking architectural contracts that bind a domain type to those fields.
do: For any architectural decision that binds a domain type to a third-party data file, list the assumed semantics for every field whose name matches an internal convention (e.g. `q`/`r` → axial, `(col, row)` → offset, `width` → tiles vs pixels) and require at least one landmark sanity check (a known position projects to the geometrically expected output) before approving the contract.
dont: Treat matching field names as proof of matching semantics; map editors, schema generators, and import/export tools routinely reuse names like `q`/`r` for whichever convention they prefer, and a silent mismatch only manifests as a wrong-shape rendering or wrong distance math much later.
rationale: Coordinate-system and vocabulary mis-identifications compile cleanly, type-check cleanly, and pass headless smoke tests; pinning the convention up-front via a landmark check prevents a downstream refactor that touches every consumer of the bound type.
confidence: high
status: active
superseded_by: null
review_after: null
```
