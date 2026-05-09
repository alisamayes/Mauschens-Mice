# Memory: quality-assurance / dragonflight

Lessons captured by `quality-assurance` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: quality-assurance-dragonflight-20260508-b333c7
created: 2026-05-08
updated: 2026-05-08
agent_type: quality-assurance
scope: project
project_slug: dragonflight
task_type: review
tags: [pygame, rendering, regression-tests, hex-grid, slice-verification]
trigger: Slice 1 ("See the map") shipped a Pygame window whose rendered output looked rhombus-shaped to the user even though every numeric sizing test (compute_window_size <= MAX_WINDOW_*, monotonic, includes margin) passed and headless smoke-boots under SDL_VIDEODRIVER=dummy exited cleanly.
evidence: Round-1 revision added a layout aspect-ratio regression test to tests/test_render.py asserting 0.75 <= height/width <= 1.5 for the rendered example map; the user-found rhombus was the trigger and no prior test would have caught it.
observed_count: 1
lesson: For any Dragonflight Pygame rendering slice, require at least one structural shape regression test on the rendered output (e.g. an aspect-ratio guard on compute_window_size) before recommending GO.
do: When verifying a Dragonflight "see the X" slice, add (or insist on) a regression test that pins a structural property of the rendered geometry — bounding-box aspect ratio, non-empty pixel coverage, expected hex count along an axis — alongside the numeric sizing tests.
dont: Treat passing numeric sizing-helper tests plus a clean headless smoke-boot as sufficient evidence that the rendered map looks right; flat-top hex layout has enough degrees of freedom (axial extent, scale, margin, sqrt(3) factor) for the shape to drift while every scalar invariant still holds.
rationale: Hex-grid renderers couple axial coordinates, pixel projection, bbox computation, and window sizing through several multiplicative factors; a single off-by-a-factor in any of them produces a visually wrong but numerically valid window, so structural-shape tests close a gap that scalar tests cannot.
confidence: high
status: active
superseded_by: null
review_after: null
```
