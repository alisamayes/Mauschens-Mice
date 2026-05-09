# Memory: documentation / dragonflight

Lessons captured by `documentation` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

<!-- Append new YAML blocks below this line. Separate blocks with --- on its own line. -->

---
id: "documentation-dragonflight-20260508-4596d9"
created: 2026-05-08
updated: 2026-05-08
agent_type: documentation
scope: project
project_slug: dragonflight
task_type: docs
tags: [readme, slice-cadence, durability, scope]
trigger: Slice 1 ("See the map") shipped a new top-level README; after my round-0 sign-off the user revised the slice (rhombus → rectangle map shape, "Dev Map" rename, Pydantic bounds) and the schema doc was further edited by the Game Map Designer in round 2.
evidence: The README I wrote intentionally omitted coordinate-system and terrain-vocabulary detail — pointing at `Documentation/map-schema.md` instead — and survived those slice-level revisions unchanged because it made no claim the revisions invalidated.
observed_count: 1
lesson: Scope the Dragonflight top-level README to durable facts (entry points, requirements, setup, doc index) and defer moving-target content (coordinates, terrain rules, schema vocabulary) to the schema and spec docs.
do: When writing or revising `Dragonflight/README.md` at any slice, restrict it to title + one-line summary, status pointer, project layout, requirements, Windows/PowerShell setup, both run commands, quality gates, map-data pointer, and a doc index — and link to `Documentation/map-schema.md` and `Documentation/Dragonflight Specification and MVP.md` for everything else.
dont: Inline coordinate semantics, terrain vocabulary tables, schema-version history, or rule details into the README; those churn across slices and the README becomes the project's worst stale doc.
rationale: Dragonflight's slice cadence (game-feature playbook) revises rules, schema, and map shape between waves; concentrating volatile detail in the schema/spec docs means slice revisions touch one doc, not the README.
confidence: medium
status: active
superseded_by: null
review_after: null
