# Memory: architectural-lead / dragonflight

Lessons captured by `architectural-lead` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: architectural-lead-dragonflight-20260508-c0c109
created: 2026-05-08
updated: 2026-05-08
agent_type: architectural-lead
scope: project
project_slug: dragonflight
task_type: feature
tags: [contracts, schema, loader, ssot, drift]
trigger: Slice 1 round 0 architectural memo named the loader's accepted vocabulary in passing (built-ins `grassland`/`forest`/`mountain`; customs `Settlement`/`Citadel`/`Bridge`/`River`) but did not require that this list and the schema doc be the single source of truth; the schema doc and loader subsequently drifted on the `woodland` alias before Documentation later reconciled them.
evidence: Slice-level Revision History recorded the schema-doc-vs-loader `woodland`-alias discrepancy and its later reconciliation by the Documentation role; backend coder's round-0 follow-up explicitly asked the AL to nominate a single authority for the loader vocabulary contract.
observed_count: 1
lesson: When approving a Dragonflight architectural contract whose vocabulary lives in two places (a schema doc and a loader allowlist), name one of them as the single source of truth and require the other to cite it explicitly.
do: For each Dragonflight contract that enumerates accepted strings (built-in `hexType` allowlist, reserved `customHexTypes` names, terrain enum), pick the loader's named-constant set as the authority, mark the schema doc's tables as "rendered from `<module>._<CONSTANT>`", and forbid additions to either side until the other has been updated in the same slice.
dont: Approve a contract that lists accepted vocabulary in both the schema doc and the loader code as independently authoritative; one of them will silently overstate the contract within a slice.
rationale: Pre-game data files travel further than code (community-shared maps, hand-edited scenarios), so an over-permissive doc costs more than a strict loader; pinning the loader's allowlist constants as the single source of truth keeps the schema doc honest by construction.
confidence: medium
status: active
superseded_by: null
review_after: null
```
