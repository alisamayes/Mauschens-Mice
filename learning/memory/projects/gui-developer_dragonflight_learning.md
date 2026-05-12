# Memory: gui-developer / dragonflight

Lessons captured by `gui-developer` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: gui-developer-dragonflight-20260508-4b47c5
created: 2026-05-08
updated: 2026-05-08
agent_type: gui-developer
scope: project
project_slug: dragonflight
task_type: bugfix
tags: [renderer, hex-coord, coordinate-space, pygame]
trigger: User feedback on Slice 1 round 0 said the rendered map looked like a rhombus because each odd column was shifted down from the right; render.py was still calling axial_to_pixel after Wave A switched Tile.coord from AxialCoord to OffsetCoord.
evidence: Round-1 fix in src/dragonflight/render.py imported offset_to_pixel and replaced _axial_extent / _pixel_bbox_size / _origin_for with offset-aware versions; the 30x30 example map's height/width ratio went from ~1.63 (rhombus) to ~1.15 (near-square) and the user explicitly approved Slice 1 immediately after.
observed_count: 1
lesson: In Dragonflight, render.py is offset-native — every per-tile projection must go through hex_coord.offset_to_pixel because Tile.coord is keyed by OffsetCoord (odd-q flat-top, col/row), and axial helpers are reserved for simulation math (distance, neighbours, A* per spec section 4 / section 14).
do: Import and call offset_to_pixel from dragonflight.hex_coord for any per-tile pixel placement in render.py; iterate offset extents on coord.col / coord.row; keep render.py's "renderer is offset-native" docstring line in sync if Tile.coord ever changes again.
dont: Use axial_to_pixel inside render.py while Tile.coord is OffsetCoord, even after a refactor that papers over field names; treating offset (col, row) values as axial (q, r) shears every column down by an extra q/2 hex and produces a rhombus for any map wider than one column.
rationale: The map is authored, stored, and keyed in offset coordinates because that matches the editor's vocabulary and the visual layout the user expects (rectangular bounding box); only the projection helper that matches that space yields a rectangular render of a rectangular dataset.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: gui-developer-dragonflight-20260508-98350a
created: 2026-05-08
updated: 2026-05-08
agent_type: gui-developer
scope: project
project_slug: dragonflight
task_type: bugfix
tags: [testing, layout, regression, renderer, pytest]
trigger: The rhombus regression slipped through Slice 1's pre-revision render tests (which asserted bbox sizing in pixels and a never-up-scale rule but never that the rendered window was actually rectangular for a rectangular dataset); the user caught it visually only after the demo opened.
evidence: Added tests/test_render.py::TestComputeWindowSize::test_offset_layout_is_roughly_square_for_square_map asserting 0.75 <= height/width <= 1.5 on the 30x30 example map at compute_render_hex_size; the pre-fix axial layout produced ratio ~1.63 (would fail the bound) while the post-fix offset layout produces ~1.15 (passes), and all six quality gates now go green.
observed_count: 1
lesson: Guard the Dragonflight renderer with an aspect-ratio assertion on the bundled example map's compute_window_size output (target 0.75 <= height/width <= 1.5) so future coordinate-space regressions fail CI instead of waiting for the next manual demo run.
do: Keep test_offset_layout_is_roughly_square_for_square_map alive; if MAX_WINDOW_*, MARGIN_PX, or the example map's column/row count change, recompute the expected ratio from the intrinsic odd-q flat-top geometry (sqrt(3) * (row_extent + zigzag_extra + 1) / (1.5 * col_extent + 2)) and update the bound rather than dropping the assertion.
dont: Rely on mypy or per-helper unit tests alone to cover renderer geometry; the pre-revision test_render.py asserted "fits inside MAX_WINDOW_*" and "monotonic in hex_size" yet still let a sheared layout ship, because neither captured the "rectangle stays rectangular" contract the user articulated in round-0 feedback.
rationale: An aspect-ratio bound is the cheapest layout-correctness contract — it directly encodes the rectangular-render-of-a-rectangular-map requirement, costs one assert plus one example-map fixture, and covers any future projection bug class (rhombus, mirror, off-by-half-zigzag) that mypy cannot catch.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: gui-developer-dragonflight-20260511-b6c1a2
created: 2026-05-11
updated: 2026-05-11
agent_type: gui-developer
scope: project
project_slug: dragonflight
task_type: feature
tags: [pygame, gui, screens, architecture]
trigger: Adding a Settings/Map Creator flow required finding existing screen/scene abstractions.
evidence: Exploration showed `src/dragonflight/movement_playtest.py` is a single monolithic Pygame loop with direct event handling and redraw, and the codebase contains no existing `screen/scene/ui` modules to extend.
observed_count: 1
lesson: When adding new UI flows in Dragonflight, implement a minimal in-loop screen router rather than searching for or inventing a heavy scene framework.
do: Add a small `mode`/screen object that owns `handle_event` + `draw`, and dispatch from the existing `pygame.event.get()` loop in `movement_playtest.py`.
dont: Bolt new UI pages directly into the playtest loop without a mode boundary; it will quickly tangle input routing and redraw logic.
rationale: The current app is immediate-mode and loop-centric; a tiny router matches existing style and keeps new flows testable and separable.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: gui-developer-dragonflight-20260511-8fd3e1
created: 2026-05-11
updated: 2026-05-11
agent_type: gui-developer
scope: project
project_slug: dragonflight
task_type: feature
tags: [pygame, gui, layout, hit-testing]
trigger: The requested Settings button must sit in new bottom chrome while preserving correct map hit-testing.
evidence: `movement_playtest.py` already reserves `TIME_BAR_HEIGHT` and adjusts `origin` by it plus ignores clicks in that region; adding bottom chrome must also shrink the map canvas and keep hit-testing aligned.
observed_count: 1
lesson: Any new UI chrome in Dragonflight must reserve pixels in layout and in click filtering to keep map hit-testing consistent.
do: Introduce `BOTTOM_BAR_HEIGHT` and compute map canvas height as `client_h - TIME_BAR_HEIGHT - BOTTOM_BAR_HEIGHT`, adjusting draw origin consistently with the reserved regions.
dont: Draw a bottom bar “on top” of the map without changing the layout inputs to `layout_map_on_canvas` or filtering clicks; it will cause misaligned painting/movement picks.
rationale: Rendering and hit-testing share the same origin/viewport math; inconsistent chrome handling breaks interaction correctness.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: gui-developer-dragonflight-20260511-52aa7c
created: 2026-05-11
updated: 2026-05-11
agent_type: gui-developer
scope: project
project_slug: dragonflight
task_type: feature
tags: [editor, terrain, consistency, gui]
trigger: The Map Creator toolbar must list “each tile type currently in game” with recognizable visuals.
evidence: Tile identity is centralized in `src/dragonflight/terrain.py::Terrain`, and the canonical palette is already in `src/dragonflight/render.py::TERRAIN_COLORS`.
observed_count: 1
lesson: Drive editor toolbars from `Terrain` and `TERRAIN_COLORS` to keep tile lists and colors consistent with the runtime.
do: Enumerate `Terrain` for tool buttons and use `render.TERRAIN_COLORS` for swatches in the Map Creator UI.
dont: Hardcode terrain names or colors inside the editor screen; it will drift from runtime rules and palettes.
rationale: Single-source-of-truth enums and palettes prevent silent mismatches between editor output and runtime rendering.
confidence: medium
status: active
superseded_by: null
review_after: null
```
