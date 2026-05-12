# Memory: coder-game-systems / dragonflight

Lessons captured by `coder-game-systems` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

<!-- Append new YAML blocks below this line. Separate blocks with --- on its own line. -->

---

```yaml
id: coder-game-systems-dragonflight-20260511-a4f1e2
created: 2026-05-11
updated: 2026-05-11
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: research
tags: [map-loader, serde, single-source-of-truth, architecture]
trigger: Recommending a minimal save/load API for user-created maps required deciding whether `save_map` belongs next to `load_map` in `src/dragonflight/map_loader.py` or as a method on `GameMap`.
evidence: `src/dragonflight/map_state.py` declares `GameMap` and `Tile` with `@dataclass(frozen=True, slots=True)` and the `map_loader.py` module docstring states it is "the only place where untrusted map JSON is parsed and turned into the simulation's source-of-truth `GameMap`" and "the rest of the codebase stays Pydantic-agnostic."
observed_count: 1
lesson: Put any new map serialisation function (e.g. `save_map`) inside `map_loader.py` alongside `load_map`; never add a `.save()` method to `GameMap` or `Tile`.
do: When extending Dragonflight with a write path for map data, add `save_map(game_map, path, *, metadata)` and a `MapSaveError(ValueError)` in `src/dragonflight/map_loader.py`, returning plain dataclasses on the read side and accepting plain dataclasses on the write side.
dont: Add `.save()` or `.to_json()` methods to `GameMap`/`Tile`, or scatter JSON I/O across `render.py`, `movement_playtest.py`, or `__main__.py` — the SoT dataclass must stay JSON-agnostic and frozen.
rationale: The frozen SoT + single-boundary-module design lets every other system read `GameMap` without ever importing Pydantic; co-locating save with load is what preserves that invariant across read and write.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260511-7c3d09
created: 2026-05-11
updated: 2026-05-11
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: research
tags: [map-loader, pydantic, validation, serde]
trigger: Designing a save path for Dragonflight maps raised the question of whether the saved JSON should be revalidated before being written to disk, or trusted because the in-memory `GameMap` is already validated.
evidence: `load_map` in `src/dragonflight/map_loader.py` uses `_RawMapModel.model_validate(raw_data)` as the sole schema gate; the loader's own constants (`_MAX_HEXES`, `_MAX_MAP_DIMENSION`, etc.) make the boundary the only place schema knowledge lives.
observed_count: 1
lesson: A `save_map` for Dragonflight must round-trip the candidate output through the same `_RawMapModel.model_validate(...)` / `model_dump()` used on load before writing the JSON.
do: In `save_map(game_map, path, metadata)`, build the dict, call `_RawMapModel.model_validate(candidate)` to enforce schema, then `model.model_dump()` and write the validated dict; wrap `ValidationError` (and `OSError`) in `MapSaveError` with the file path in the message.
dont: Skip validation on save because the in-memory `GameMap` "is already valid" — `GameMap` is the simulation contract, not the JSON contract, and writes can still drift (custom-type ids, schemaVersion, custom colour, optional editor fields).
rationale: Reusing the same Pydantic v2 models for both directions keeps the JSON contract owned by exactly one module and makes save/load round-trip tests sufficient to pin compatibility with the bundled editor.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260511-2b5e8a
created: 2026-05-11
updated: 2026-05-11
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: research
tags: [cli, entry-point, ui-separation, map-selection]
trigger: Picking the right seam for "which map does the game load?" required separating map-selection from the Pygame session loop.
evidence: `src/dragonflight/__main__.main` hardcodes `_example_map_path()` then calls `run_movement_playtest(game_map)`; `run_movement_playtest` in `movement_playtest.py` is already path-agnostic and only consumes a `GameMap`.
observed_count: 1
lesson: Add map selection (custom path, save slot, etc.) as an `argparse` argument in `src/dragonflight/__main__.py`, not as a file picker inside the Pygame session.
do: Extend `main()` with `argparse` (e.g. `--map PATH`) that defaults to `_example_map_path()`, resolve the path before calling `load_map`, and keep `run_movement_playtest(game_map)` unchanged so the session loop stays unaware of files.
dont: Add file open/save dialogs, path resolution, or `load_map` calls inside `movement_playtest.py` or `render.py` — those modules must remain pure presentation that consume an already-loaded `GameMap`.
rationale: Keeping the selection seam in the entry point preserves the rule that the UI never reaches across the simulation API boundary, and lets headless tests construct or load any `GameMap` and call `run_movement_playtest` directly.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260511-9d1f47
created: 2026-05-11
updated: 2026-05-11
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: research
tags: [terrain, serde, map-schema, editor-compat]
trigger: Sketching the reverse `Terrain → hexType` map for `save_map` exposed that the forward and reverse mappings in `map_loader.py` are not symmetric in their string vocabularies.
evidence: `_BUILT_IN_TERRAIN` in `map_loader.py` maps `forest` to `Terrain.WOODLAND`, and `Documentation/map-schema.md` notes that `woodland` is not accepted as a `hexType`. Reserved terrains resolve via `customHexTypes` entry ids documented in the schema.
observed_count: 1
lesson: When serialising `Terrain` back to `hexType`, use the editor's vocabulary (`forest`, not `woodland`) for built-ins and reuse the four documented `custom-*` ids for reserved terrains; check `Documentation/map-schema.md` rather than inferring strings from `Terrain` enum names.
do: Add reverse tables in `map_loader.py` matching the schema doc exactly and add a round-trip test that saves and reloads the bundled example map while preserving every `Tile` (coord + terrain).
dont: Construct the reverse mapping by lowercasing enum names or write `woodland` as a `hexType`; the loader rejects it and editor round-trips break.
rationale: The editor/loader vocabulary is intentionally asymmetric; reverse-mapping by enum identity silently breaks editor round-trips and is hard to notice without explicit tests.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260511-5a6217
created: 2026-05-11
updated: 2026-05-11
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: research
tags: [spec-vs-code, investigation, terrain, armies]
trigger: Answering "how does terrain impact armies" required distinguishing rules that exist in the spec from rules that exist in code.
evidence: The current `src/dragonflight/` package has no implemented army system; terrain usage in code is presently limited to rendering, citadel discovery in the playtest, and a settlement raid guard.
observed_count: 1
lesson: When asked "how does X impact Y" in Dragonflight, first confirm whether the consumer exists in `src/dragonflight/` and explicitly label behavior as "spec-only" vs "implemented".
do: Search `src/dragonflight/` for the consuming system before answering, and cite the implementing file/function when it exists.
dont: Quote the specification as if it described current code behavior when the system hasn't been implemented yet.
rationale: The MVP slices intentionally ship partial systems; separating spec from code prevents recommendations that depend on non-existent modules.
confidence: medium
status: active
superseded_by: null
review_after: null
```
