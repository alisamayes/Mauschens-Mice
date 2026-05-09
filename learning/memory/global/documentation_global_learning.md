# Memory: documentation / global

Lessons captured by `documentation` while working across projects. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

<!-- Append new YAML blocks below this line. Separate blocks with --- on its own line. -->

---
id: "documentation-global-20260508-1b4f0d"
created: 2026-05-08
updated: 2026-05-08
agent_type: documentation
scope: global
project_slug: global
task_type: docs
tags: [pyproject, packaging, readme, python]
trigger: During a top-level documentation pass on Dragonflight, `pyproject.toml` had `readme = "Documentation/Dragonflight Specification and MVP.md"`, pointing project metadata at a 678-line design spec instead of an actual README.
evidence: Changed `readme = "Documentation/Dragonflight Specification and MVP.md"` to `readme = "README.md"` in `Dragonflight/pyproject.toml` so packaging metadata (rendered by `pip show`, PyPI, IDEs) describes the project rather than its design history.
observed_count: 1
lesson: Audit `[project].readme` in `pyproject.toml` whenever updating top-level documentation; it must point at a user-facing README, not at a design spec or other long-form document.
do: When touching top-level docs in a Python project, open `pyproject.toml` and verify `readme` resolves to a README.md (or equivalent short user-facing entry doc); fix it if it points at a spec, changelog, or any long-form document.
dont: Leave `readme` pointing at design or specification documents — packaging tooling renders that file as the project description for distribution metadata, which is the wrong audience.
rationale: The `readme` field is consumed by packaging tools and registries (`pip show`, PyPI long description, IDE integrations); pointing it at a spec inflates the metadata and gives readers the wrong document.
confidence: medium
status: active
superseded_by: null
review_after: null

---
id: "documentation-global-20260508-a3af8c"
created: 2026-05-08
updated: 2026-05-08
agent_type: documentation
scope: global
project_slug: global
task_type: docs
tags: [schema, allowlist, drift, audit]
trigger: `Documentation/map-schema.md` listed `woodland` as an accepted built-in `hexType` alias for `forest`, but the loader's `_BUILT_IN_TERRAIN` allowlist contained only `grassland`, `forest`, and `mountain`, so any author trusting the doc would hit a `MapLoadError`.
evidence: Caught the drift by diffing the doc's §5/§6 enumerations against `map_loader.py`'s `_BUILT_IN_TERRAIN` and `_CUSTOM_NAME_TO_TERRAIN` dicts; removed the `woodland` row from §5 and the `woodland` reference from §6 step 1 of `map-schema.md`.
observed_count: 1
lesson: When auditing a schema or format doc that enumerates accepted vocabulary, diff every enumerated value against the corresponding allowlist literal in code.
do: For each enumerated value the doc claims is accepted (built-ins, reserved names, supported orientations, schema versions, file extensions), find the matching allowlist (dict keys, set members, enum values) in code and verify both sides agree before signing off.
dont: Trust schema-doc tables to match code just because the doc was recently edited or the code looks small; loader allowlists drift independently from prose, especially across waves.
rationale: A doc that lies about accepted vocabulary actively misleads authors, who only discover the drift when valid-by-doc input fails at the parser; cross-checking the literal allowlist is the cheapest way to catch this.
confidence: medium
status: active
superseded_by: null
review_after: null

---
id: "documentation-global-20260508-bbbd42"
created: 2026-05-08
updated: 2026-05-08
agent_type: documentation
scope: global
project_slug: global
task_type: docs
tags: [mvp, drift, doc-vs-code, scope-discipline]
trigger: While auditing Dragonflight Slice 1 docs I found drift between `map-schema.md` (claimed `woodland` was accepted) and `map_loader.py` (did not accept it); the brief explicitly stated "code is the source of truth at this stage", so fixing the doc rather than the code was the correct call but easy to get wrong.
evidence: Updated `map-schema.md` §5/§6 to remove the `woodland` alias and left `map_loader.py` untouched, keeping the documentation pass scoped strictly to documentation.
observed_count: 1
lesson: At MVP / pre-API-freeze stage, when documentation diverges from code during a docs-only pass, update the documentation to match observed code behaviour rather than introducing a code change.
do: When a docs-only task uncovers doc-vs-code drift, edit the doc to reflect actual code behaviour and surface the divergence as a follow-up if the code's behaviour itself is suspect; explicitly note the decision in the handoff.
dont: Silently change code to match a doc claim during a documentation-only pass — that crosses scope into a behaviour change and bypasses the relevant code-owner role's review.
rationale: Doc-only passes must not make behaviour changes; at MVP stage code typically moves faster than docs, so the doc is more often the stale party, and a quiet code change during a docs pass evades QA.
confidence: medium
status: active
superseded_by: null
review_after: null
