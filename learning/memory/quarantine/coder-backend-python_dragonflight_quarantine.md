# Quarantine: coder-backend-python / dragonflight

Candidates that failed the learning validator for automatic merge. These are **not loaded** into active behavior until a human reviews them.

## Quarantined Lessons

```yaml
id: coder-backend-python-dragonflight-20260508-26eea8
created: 2026-05-08
updated: 2026-05-08
agent_type: coder-backend-python
scope: project
project_slug: dragonflight
task_type: feature
tags: [pydantic, extra-allow, defensive, follow-up]
trigger: Slice 1 used `model_config = ConfigDict(extra="allow")` on every Pydantic v2 boundary model so editor-only fields (`regions`, `paths`, `notes`, fog flags, `edgeData`) pass through without crashing the loader. The same setting silently accepts typos in our owned field names (e.g. `hex-Type` instead of `hexType`).
evidence: Round-0 and round-1 handoffs both flagged this as a follow-up; no user correction has been applied yet, so the trade-off is unverified — keeping `extra='allow'` everywhere may be the right call until the schema fully stabilises.
observed_count: 1
lesson: For Dragonflight Pydantic v2 boundary models, narrow `extra='allow'` to the top-level model only; substructures with stable required field names should use `extra='ignore'` (or `extra='forbid'` for our owned schemas) so authoring typos surface as validation errors instead of being silently dropped.
do: Set `model_config = ConfigDict(extra='ignore')` on `_HexModel`, `_SettingsModel`, `_CustomHexTypeModel` once the Slice 1 schema fields are confirmed stable; keep `extra='allow'` only on `_RawMapModel` where the editor adds top-level metadata.
dont: Apply `extra='allow'` uniformly across every nested Pydantic v2 boundary model — it weakens the validation gate in places where the loader actually owns the field set.
rationale: The Slice 1 trade-off was forward-compatibility with editor metadata, but uniform `extra='allow'` also disables typo detection on fields the loader DOES own; selective narrowing keeps both wins.
confidence: low
status: quarantined
superseded_by: null
review_after: 2026-08-08
```
