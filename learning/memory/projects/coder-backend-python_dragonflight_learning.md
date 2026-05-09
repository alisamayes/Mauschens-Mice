# Memory: coder-backend-python / dragonflight

Lessons captured by `coder-backend-python` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: coder-backend-python-dragonflight-20260508-85501c
created: 2026-05-08
updated: 2026-05-08
agent_type: coder-backend-python
scope: project
project_slug: dragonflight
task_type: feature
tags: [pydantic, validation, security, loader]
trigger: Slice 1 round 0 shipped Pydantic v2 boundary models with bare `int`/`float` fields and `extra='allow'`; round-1 security review (L1) flagged the loader as a parse-time DoS surface (unbounded width/height/hexSize/q/r/schemaVersion).
evidence: Round 1 added `Field(ge=…, le=…)` to every numeric field on `_SettingsModel`/`_HexModel`/`_RawMapModel`, captured caps as named constants (`_MAX_MAP_DIMENSION`, `_MAX_HEX_SIZE`, `_MAX_SCHEMA_VERSION`, `_MAX_TILE_INDEX`, `_MAX_CUSTOM_HEX_TYPES`, `_MAX_HEXES`), and `tests/test_map_loader.py::TestPydanticBounds` now pins five rejection paths.
observed_count: 1
lesson: Bound every numeric field on a Dragonflight Pydantic v2 boundary model with an explicit `Field(ge=…, le=…)` from the first commit, especially when the model uses `extra='allow'`.
do: For each `int`/`float` field on any Pydantic v2 model that ingests untrusted JSON, attach `Field(ge=N_min, le=N_max)` (or `gt`/`lt`), record the cap as a named module constant, and add at least one rejection test per field.
dont: Ship a Pydantic v2 boundary model with bare `int` / `float` fields, even for "trusted" data files; the loader is the security boundary.
rationale: `extra='allow'` already absorbs unknown editor fields, so pairing it with unbounded numerics turns the loader into a parse-time DoS surface; bounds also surface authoring typos with the field name in the error.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-backend-python-dragonflight-20260508-3caac0
created: 2026-05-08
updated: 2026-05-08
agent_type: coder-backend-python
scope: project
project_slug: dragonflight
task_type: feature
tags: [hex-coord, axial, offset, map-schema]
trigger: Slice 1 round 0 keyed `Tile.coord` and `GameMap.tiles` by `AxialCoord` because the JSON field labels were `q`/`r` and the spec uses axial vocabulary. The user's round-1 feedback ("the rendered map looks like a rhombus") revealed the JSON `q`/`r` were really odd-q flat-top offset col/row authored by a map editor.
evidence: Round 1 amended the contract to `Tile.coord: OffsetCoord`, added `OffsetCoord` plus `offset_to_axial`/`axial_to_offset`/`offset_to_pixel` in `hex_coord.py`, and `tests/test_hex_coord.py::TestOffsetAxialRoundTrip` pins `OffsetCoord(16, 16) → AxialCoord(16, 8)` for the citadel — exactly the conversion that was missing in round 0.
observed_count: 1
lesson: Before keying Dragonflight simulation state by a coordinate type, verify which coordinate space the data file actually uses by checking that a known-position landmark (citadel at `(16, 16)`) projects to the geometrically expected pixel position, not just by reading the JSON field labels.
do: When the schema labels keys `q`/`r`, run both `axial_to_pixel` and `offset_to_pixel` on a landmark coord (e.g. the citadel) and pick whichever projection matches the authored visual layout (square for `width × height`, not rhombus); encode the resolved interpretation as a unit test (e.g. round-trip + landmark assertion).
dont: Assume `q`/`r` JSON labels mean axial just because the spec talks about axial; map editors routinely label odd-q offset col/row with the same letters and a silent mismatch distorts both rendering and any future axial pathfinding.
rationale: Coordinate-space mis-identification compiles cleanly, types fine, and only manifests as a wrong-shape map or wrong distance/aggression-radius math much later — pinning the convention up-front prevents a downstream refactor that touches every consumer of `Tile.coord`.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-backend-python-dragonflight-20260508-49c04c
created: 2026-05-08
updated: 2026-05-08
agent_type: coder-backend-python
scope: project
project_slug: dragonflight
task_type: feature
tags: [pydantic, validation, dict, dos-hardening]
trigger: Round 1 needed to bound the size of the top-level `hexes` mapping in `map_loader.py`. Even when a `Field(max_length=…)` is enforced on a Pydantic v2 `dict[str, X]` field, model validation still walks every entry first, defeating the security goal of rejecting hostile-sized dicts cheaply.
evidence: `load_map` now performs an explicit pre-flight `if isinstance(raw_hexes, dict) and len(raw_hexes) > _MAX_HEXES` check before calling `_RawMapModel.model_validate`, with `_MAX_HEXES = _MAX_MAP_DIMENSION ** 2`; the comment in `map_loader.py` documents why the pre-flight is needed in addition to (or instead of) `Field(max_length=...)`.
observed_count: 1
lesson: For any Pydantic v2 `dict[str, X]` boundary field whose entry count could be hostile in Dragonflight, add an explicit pre-flight `len(raw) > MAX` check before `model_validate`, in addition to any `Field(max_length=...)`.
do: Parse the JSON to a raw `dict`, range-check the suspicious mapping's `len()` against a named cap, raise `MapLoadError` with the offending size, and only then pass to Pydantic.
dont: Rely solely on `Field(max_length=N)` to defend a Pydantic v2 dict-typed boundary field against a multi-million-entry hostile mapping — Pydantic still has to materialise the dict before checking.
rationale: A pre-flight cap rejects the worst-case input in O(1) without paying the per-entry validation cost, making the loader's resource ceiling predictable.
confidence: medium
status: active
superseded_by: null
review_after: 2027-05-08
```
