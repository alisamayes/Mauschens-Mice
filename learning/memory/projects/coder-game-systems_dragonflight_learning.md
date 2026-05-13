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

---

```yaml
id: coder-game-systems-dragonflight-20260512-a8f31c
created: 2026-05-12
updated: 2026-05-12
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: feature
tags: [settlements, healing, phase-rules, user-correction]
trigger: First-pass settlement phase logic healed only cities; the user revised the requirement to heal all settlement types the same way.
evidence: Revision History records the user correction that all settlements should heal (not just cities); `on_settlement_phase_end` now applies `SETTLEMENT_HEAL_PERCENT_OF_MAX` whenever `hp < max_hp` for every type.
observed_count: 1
lesson: Apply one shared damaged-settlement heal branch for every settlement type unless the brief explicitly type-gates healing.
do: Implement heal-vs-growth using `hp < max_hp` without branching on `SettlementType` unless the design doc names different recovery per type.
dont: Encode a heal rule for only one archetype when the spec or brief says "settlements" generically.
rationale: Type-specific heals without an explicit brief invite rework when designers mean global settlement rules.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260512-62e9b0
created: 2026-05-12
updated: 2026-05-12
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: feature
tags: [settlements, growth, hp, user-correction]
trigger: The user added a growth rule to raise max HP each turn while keeping undamaged settlements logically "full".
evidence: User asked to increase `max_hp` by 50 on growth ticks; implementation adds `SETTLEMENT_GROWTH_MAX_HP_BONUS` to both `max_hp` and `hp` on the undamaged growth path and surfaces `max_hp_delta` on `SettlementPhaseOutcome`.
observed_count: 1
lesson: When undamaged settlement growth increases max_hp, increase current hp by the same increment so the entity stays at full health.
do: On growth ticks that bump `max_hp`, add the same delta to `hp` when the settlement started the tick at full health; record both deltas on the phase outcome for tests and UI.
dont: Raise `max_hp` without adjusting `hp` on a full-health settlement unless the brief explicitly wants a partial-health state after growth.
rationale: Silent drops below max after a "growth" tick confuse phase logic that keys off `hp == max_hp`.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260512-4d1c7e
created: 2026-05-12
updated: 2026-05-12
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: feature
tags: [combat, settlements, retreat, raid, user-correction]
trigger: The user needed a hard separation between losing a fight and retreating mid-combat.
evidence: Revision History states that if the dragon retreats, settlement stats other than HP from resolved rounds must not change; `run_settlement_combat_loop` documents this and only calls `on_raid_defeat` when settlement HP reaches zero.
observed_count: 1
lesson: Never apply raid-defeat economy or aggression mutations when the player ends combat via retreat; reserve that bundle for terminal defeat at `hp == 0` or an explicit defeat hook.
do: Gate `on_raid_defeat` on settlement HP depletion (or a dedicated "raid success" API), not on loop exit via `should_continue` false.
dont: Fire eco or aggression penalties automatically whenever combat ends—tie them to the documented loss condition only.
rationale: Mixing retreat with defeat breaks risk/reward tuning and contradicts player expectations.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260512-7b2e91
created: 2026-05-12
updated: 2026-05-12
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: feature
tags: [dragon, hp, api-clarity]
trigger: The user asked for explicit current vs max HP language on the dragon as well as settlements.
evidence: `Dragon` already had `hp` and `max_hp`; revision added a documented `current_hp` property with a clamping setter so callers can speak the same vocabulary as the design doc.
observed_count: 1
lesson: When the design vocabulary distinguishes current and max HP, expose a `current_hp` alias (mirroring `hp`) with a clamping setter on the dragon entity.
do: Add `current_hp` read/write that delegates to `hp` and clamps to `[0, max_hp]`; document that combat mutates current HP, not the ceiling, unless a progression system changes `max_hp`.
dont: Introduce a second mutable HP field; keep a single source field and treat `current_hp` as API sugar.
rationale: Duplicate fields desynchronise; an alias keeps terminology clear without splitting state.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260512-91c4ae
created: 2026-05-12
updated: 2026-05-12
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: feature
tags: [pygame, layout, map-viewport, input]
trigger: User reported the hex map and left dragon panel overlapping and clipping after introducing left/right side panels and splitters.
evidence: `movement_playtest` used `layout_map_on_canvas` with width `client - panels` but placed the map origin without adding `dragon_panel_w`, shifting the grid under the left column; fix added that offset, `set_clip(map_viewport)`, and splitter-aware picking.
observed_count: 1
lesson: When the map is laid out in a reduced-width canvas between side panels, add the left panel width to the map origin, clip rendering to the map viewport, and align mouse hit-tests with the same viewport and splitter hit zones.
do: Set `origin = (dragon_panel_w + ox, time_bar + oy)` when `map_canvas_w = client_w - dragon_panel_w - inspector_panel_w`; call `surf.set_clip(map_viewport)` around `render_map`; gate hex clicks on viewport and ignore splitter strips during drag.
dont: Pass panel-subtracted width into `layout_map_on_canvas` but treat the returned `(ox, oy)` as if the surface origin were `(0, 0)` for the full window.
rationale: Layout helpers usually return coordinates in the canvas space you gave them, not absolute surface space; mixing the two draws under UI chrome.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260512-72dbe3
created: 2026-05-12
updated: 2026-05-12
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: feature
tags: [ui, economy, settlement, single-source-of-truth]
trigger: User asked to replace vague raid spoils copy with the actual gold number and to narrow the inspector column further.
evidence: Added `raid_victory_gold_from_eco` in `settlement.py` for HUD and `apply_raid_victory_loot`; inspector shows `Raid Spoils: N` only for live settlements and sizes min width using a sample string including a wide digit run.
observed_count: 1
lesson: Show raid gold preview in the UI via the same helper used for victory payout, and include that string when computing minimum inspector width so labels do not clip.
do: Export `raid_victory_gold_from_eco(eco)` next to `RAID_VICTORY_GOLD_PERCENT_OF_ECO`, call it from the inspector and from loot application, and add the longest expected preview line to font-metric width calculations.
dont: Display percentage-only spoils text while computing loot elsewhere with a duplicated `eco * percent // 100` expression, or shrink panel mins without measuring the new label.
rationale: One function keeps preview and payout identical; width sizing must cover the real string you render.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260513-c7e21b
created: 2026-05-13
updated: 2026-05-13
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: feature
tags: [abilities, timers, dragonflight, state]
trigger: User correction split Greengon Vivify into a 5-hour sacrifice-to-attack window versus a +20% max HP buff that lasts until the citadel day boundary; the first implementation used one `active_ability_hours["Vivify"]` entry for the full day for both behaviors.
evidence: Revision History round 2; `try_use_ability` now sets `active_ability_hours["Vivify sacrifice"]` for five hours while max HP delta stays in `passive_stacks["Vivify max hp bonus"]` cleared in `begin_new_turn`.
observed_count: 1
lesson: Give each independent duration of the same named ability its own state key instead of multiplexing one active-effect timer.
do: Use separate `active_ability_hours` keys (or stack fields plus day hooks) when one cast applies both a short window mechanic and a day-scoped stat change.
dont: Let one `active_effect_hours_remaining(dragon, "AbilityName")` gate two rules that intentionally expire at different boundaries.
rationale: Single-key multiplexing hides which rule expired and forces incorrect player-facing timing.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: coder-game-systems-dragonflight-20260513-8b4d2a
created: 2026-05-13
updated: 2026-05-13
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: feature
tags: [hud, dragonflight, stats, vivify]
trigger: User follow-up required temporary Vivify max HP to appear only under Combat Stats while Base Stats showed the persistent ceiling.
evidence: `_draw_dragon_panel` computes `base_max_hp` from `dragon.max_hp` minus `passive_stacks["Vivify max hp bonus"]` for the Base Stats HP line and uses full `dragon.max_hp` for Combat Stats with green when boosted.
observed_count: 1
lesson: When day-scoped buffs mutate `max_hp` in place, subtract the tracked bonus delta for any HUD label advertised as base or persistent max.
do: Store the removable bonus (e.g. Vivify stack total) and render base max as `max_hp - bonus` wherever the design calls for a non-temporary ceiling.
dont: Render the same `dragon.max_hp` in both base and combat columns when part of the pool is explicitly temporary until `begin_new_turn`.
rationale: Players cannot tell temporary power from upgrades if both panels read the same inflated ceiling.
confidence: high
status: active
superseded_by: null
review_after: null
```
