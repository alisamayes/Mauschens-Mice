# Memory: coder-game-systems / dragonflight (quarantine)

Quarantined lessons for `coder-game-systems` on `dragonflight`. These are NOT loaded by default. Review and either approve (move to project memory) or reject.

## Lessons

---

```yaml
id: coder-game-systems-dragonflight-20260511-c08b3e
created: 2026-05-11
updated: 2026-05-11
agent_type: coder-game-systems
scope: project
project_slug: dragonflight
task_type: research
tags: [map-loader, save, atomicity, file-io]
trigger: The save/load proposal recommends an atomic write (tempfile + `os.replace`) for `save_map`, but no such function has actually been implemented in Dragonflight yet.
evidence: Recommendation only; `src/dragonflight/map_loader.py` currently exposes `load_map` but no `save_map`. No test, retry, or user correction backs the atomicity rule in this task.
observed_count: 1
lesson: When `save_map` is implemented in Dragonflight, write to a sibling tempfile inside the destination directory and `os.replace` into place, wrapping `OSError`/`ValidationError` in `MapSaveError` with the offending path.
do: Use `tempfile.NamedTemporaryFile(delete=False, dir=path.parent)` followed by `os.replace(tmp, path)`; cover with a test that asserts the original file is untouched if validation fails after the temp file is written.
dont: Open the destination path directly with `open(path, \"w\")` for map JSON — a crash or `ValidationError` mid-write would corrupt the user's existing map file.
rationale: Map JSON is user-authored content where partial overwrites are unrecoverable; atomic replace gives the operation transactional semantics consistent with "reject invalid operations at the API boundary."
confidence: low
status: quarantined
superseded_by: null
review_after: null
```
