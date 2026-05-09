# Memory: quality-assurance / global

Lessons captured by `quality-assurance` while working across projects. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: quality-assurance-global-20260508-fb2d21
created: 2026-05-08
updated: 2026-05-08
agent_type: quality-assurance
scope: global
project_slug: global
task_type: review
tags: [gui, headless-testing, smoke-test, visual-verification, render-regression]
trigger: A Pygame "see the map" slice passed every quality gate (ruff, mypy, pytest) and a headless smoke-boot under SDL_VIDEODRIVER=dummy on both entry points, was given a GO recommendation, and was then rejected by the user because the rendered window was visibly the wrong shape.
evidence: The fix added a structural rendered-output property test (an aspect-ratio guard on the rendered window dimensions) that would have caught the issue; the headless dummy-driver run was sufficient to prove "the loop runs and exits cleanly" but did not — and could not — exercise visual correctness.
observed_count: 1
lesson: When verifying a GUI slice whose acceptance criterion is "the user can see the X", treat headless-driver smoke-boots as proof of clean lifecycle only, and require at least one structural property test on the rendered output before recommending GO.
do: Add (or insist on) at least one test that asserts a measurable structural property of the rendered output — bounding-box aspect ratio, non-empty pixel coverage on a known region, expected count of drawn primitives, dominant-colour distribution — to complement headless smoke-boots and numeric sizing tests.
dont: Conclude visual correctness from any combination of "loop exits 0", "no exception raised inside the renderer", and passing scalar sizing-helper tests; SDL_VIDEODRIVER=dummy, QT_QPA_PLATFORM=offscreen, and other headless GUI drivers all share this limit by design.
rationale: Headless drivers exist to exercise process lifecycle without a display, not to validate pixel-level layout; visual acceptance criteria therefore require visual or structural assertions that bypass the human eye, otherwise the gap closes only when a user runs the demo themselves.
confidence: high
status: active
superseded_by: null
review_after: null
```
