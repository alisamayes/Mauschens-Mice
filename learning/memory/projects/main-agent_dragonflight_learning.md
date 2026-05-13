# Memory: main-agent / dragonflight

Lessons captured by `main-agent` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: main-agent-dragonflight-20260508-87c2eb
created: 2026-05-08
updated: 2026-05-08
agent_type: main-agent
scope: project
project_slug: dragonflight
task_type: feature
tags: [orchestration, acceptance-gates, gui, regression-tests, see-the-x]
trigger: Slice 1 round 0 passed every dispatched gate (ruff, mypy, pytest including a headless render integration smoke test, and a full Wave-4 QA pass) and was declared GO; the user then ran the demo and saw a rhombus-shaped map. None of the orchestrator-dispatched gates encoded "rectangular dataset renders rectangularly".
evidence: Slice-level Revision History records the user's "the shape of the map is a rhombus rather than a square" feedback in revision round 1; the round-1 fix added `tests/test_render.py::test_offset_layout_is_roughly_square_for_square_map` asserting `0.75 ≤ height/width ≤ 1.5`, which is now part of the standard pytest run.
observed_count: 1
lesson: For every "see the X" visual slice in Dragonflight, require at least one structural-output gate (an aspect-ratio or shape assertion on the renderer's window size) as part of the orchestrator's GO criteria.
do: When dispatching a Dragonflight slice whose acceptance criteria include "the user can see X", explicitly require the GUI Developer to land at least one shape/aspect-ratio assertion against the example data, and refuse to declare GO until that test exists in the standard pytest run.
dont: Treat headless smoke-boots and scalar invariant tests (`compute_window_size` ≤ MAX, monotonic, includes margin) as sufficient evidence of visual correctness; they confirm clean lifecycle, not correct geometry.
rationale: Coordinate-system and aspect-ratio mistakes compile cleanly, type-check cleanly, and pass headless smoke-boots; only a structural-output assertion catches them before the user does, and they are cheap to write once the renderer's geometry is known.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: main-agent-dragonflight-20260508-d73319
created: 2026-05-08
updated: 2026-05-08
agent_type: main-agent
scope: project
project_slug: dragonflight
task_type: feature
tags: [orchestration, architectural-amendments, revision-rounds, ssot, contracts]
trigger: Revision round 1 of Slice 1 changed `Tile.coord` from `AxialCoord` to `OffsetCoord`, which is an architectural amendment to the AL's locked round-0 contract. The orchestrator drafted the amendment directly and dispatched the Backend Coder + GUI Developer without re-engaging the (readonly) Architectural Lead, on the grounds that the user's correction made the amendment unambiguous.
evidence: Wave A round-1 brief to the Backend Coder spelled out the new `Tile.coord: OffsetCoord` contract and the `offset_to_axial`/`axial_to_offset`/`offset_to_pixel` helpers; the AL was not resumed for this round even though `agents/main-agent.md` explicitly forbids "letting architecture decisions be made silently in coding threads".
observed_count: 1
lesson: When a Dragonflight user revision introduces an architectural amendment (a data-type change, a public-contract change, or a vocabulary change), re-engage the Architectural Lead (in readonly is fine) to vet the amendment in a one-shot consult before dispatching the implementing roles.
do: For any revision-round brief that touches a public type signature, a coordinate convention, a vocabulary allowlist, or a module boundary, dispatch a brief AL consult first that asks "ratify or refine the proposed amendment" and only proceed when the AL has signed off, even if the consult is one paragraph.
dont: Bypass the AL because the amendment "looks small" or because the AL is in readonly mode; readonly AL consults are still cheap and produce a written ratification that future audits can cite.
rationale: Architectural amendments compound — today's silent `OffsetCoord` switch becomes tomorrow's silent rewrite of pathfinding's distance assumptions; a one-paragraph AL ratification is the cheapest way to keep the architectural contract auditable across revisions.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: main-agent-dragonflight-20260508-1b3273
created: 2026-05-08
updated: 2026-05-08
agent_type: main-agent
scope: project
project_slug: dragonflight
task_type: feature
tags: [orchestration, waves, parallelism, contract-changes, dependencies]
trigger: Revision round 1 of Slice 1 needed three logical changes (offset coordinate refactor, Pydantic numeric bounds, GUI offset-native projection); naive dispatch as one parallel wave would have raced the Backend Coder's `Tile.coord` type change against the GUI Developer's renderer change.
evidence: Round 1 was sequenced as Wave A (Game Map Designer + Backend Coder in parallel — disjoint owned files) → Wave B (GUI Developer alone — depends on Wave A's `Tile.coord` type change) → Wave C (folded into orchestrator self-validation). All gates green at end of Wave B; if Wave B had run in parallel with Wave A, the renderer would have been written against the wrong type.
observed_count: 1
lesson: Within a single Dragonflight revision round, parallelize subagent waves where their owned files are disjoint, and serialise the next wave when a prior wave's output changes a data type or public contract that the next wave consumes.
do: Before dispatching a multi-role revision round, draw the owned-files-vs-consumed-types map for each pending change; group changes whose owned files are disjoint AND whose consumed types are unchanged into one parallel wave, and start a new sequential wave at every data-type or contract change.
dont: Dispatch every revision-round role in a single parallel wave just because they're "all part of the same fix"; a renderer that depends on a freshly-changed `Tile.coord` cannot be written in parallel with the type change itself.
rationale: Parallelism halves wall-clock time for orthogonal work, but parallelism across a contract change forces guesswork in the downstream role and produces "owned-only gates green" half-deliverables that mask cascading defects.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: main-agent-dragonflight-20260512-6e4a91
created: 2026-05-12
updated: 2026-05-12
agent_type: main-agent
scope: project
project_slug: dragonflight
task_type: feature
tags: [orchestration, pygame, layout, dragonflight]
trigger: After tile inspector and draggable splitters shipped, the user reported the map and left dragon panel clipping and overlapping until a layout revision fixed it.
evidence: Revision History round 2 described a bug where `layout_map_on_canvas` used map-canvas width but the draw origin omitted the left column offset, so the grid drew under the dragon panel; the fix shifted origin by `dragon_panel_w` and clipped draws to `map_viewport`.
observed_count: 1
lesson: When briefing Dragonflight Pygame layout overlap bugs, require an explicit audit of map origin against panel widths, `set_clip` on the map viewport, and pointer routing that matches the same rects.
do: In handoffs to Coder (Game Systems), list checks: `origin_x` includes `dragon_panel_w`, `map_canvas_w = client_w - left - right`, render and hit-testing use `_map_viewport_rect`, splitters consume drags before hex pick.
dont: Treat "panels look wrong" as purely cosmetic CSS-style tuning without verifying coordinate spaces (canvas-local vs surface-local).
rationale: Column layouts mix two coordinate conventions; a short checklist prevents regressions that pass tests but fail visually.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: main-agent-dragonflight-20260513-a3f81c
created: 2026-05-13
updated: 2026-05-13
agent_type: main-agent
scope: project
project_slug: dragonflight
task_type: feature
tags: [orchestration, dragonflight, revisions, pygame]
trigger: After the user manually verified draconic abilities, two revision rounds (panel base vs combat stats and Vivify timing) touched the same modules without re-scoping architecture.
evidence: Revision rounds 1–2 were briefed as checklists to the same resumed implementation subagent; changes landed in `dragon_abilities.py` and `movement_playtest.py` and the user approved without a third round.
observed_count: 1
lesson: Keep follow-up UX and rule tweaks for one Dragonflight feature on a single resumed implementation thread when one role owns both the Pygame panel and the simulation hooks.
do: After playable verification, send numbered acceptance tweaks in one dispatch to the subagent that already owns `movement_playtest.py` and `dragon_abilities.py` for that feature.
dont: Cold-start a new implementation thread for each micro-revision when file ownership and context are unchanged.
rationale: Repeated cold starts re-pay exploration cost and increase merge drift on hot files.
confidence: medium
status: active
superseded_by: null
review_after: null
```
