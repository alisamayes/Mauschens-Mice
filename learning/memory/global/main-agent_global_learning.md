# Memory: main-agent / global

Lessons captured by `main-agent` while working across projects. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.

## Lessons

<!-- Append new YAML blocks below this line. Separate blocks with --- on its own line. -->

---
id: main-agent-global-20260506-2dfb35
created: 2026-05-06
updated: 2026-05-06
agent_type: main-agent
scope: global
project_slug: global
task_type: other
tags: [orchestration, tooling, indexing]
trigger: Role discovery via directory glob/search returned an incomplete `agents/subagents/` list despite the files existing and being directly readable.
evidence: Direct-path reads (for example `agents/subagents/gui-developer.md`) succeeded while directory scanning reported only a subset, which would have blocked correct role delegation if trusted.
observed_count: 1
lesson: When enumerating available role subagents, verify by direct-path reads (or a shell directory listing) if glob/index results look incomplete.
do: Keep a canonical list of expected role filenames (or ask the user for `ls` output) and attempt direct reads before concluding a role is missing.
dont: Assume `agents/subagents/*.md` glob/search results are complete; treat them as hints and validate when results conflict with user evidence.
rationale: Prevents incorrect claims about available roles and avoids routing work to the wrong subagent due to indexing glitches.
confidence: high
status: active
superseded_by: null
review_after: null

---
id: main-agent-global-20260508-bf704e
created: 2026-05-08
updated: 2026-05-08
agent_type: main-agent
scope: global
project_slug: global
task_type: other
tags: [orchestration, subagent-interruption, recovery, partial-state]
trigger: Slice 1 Wave 3 (Dragonflight, GUI Developer round 0) was interrupted mid-task; on resumption the orchestrator initially assumed no files had landed yet, but a Glob+Read sweep revealed `render.py`, `__main__.py`, `dragonflight.py`, and `tests/test_render.py` had already been written before the interruption.
evidence: Recovery from the interruption took ~5–10 tool calls of Glob+Read inspection before the orchestrator could confidently re-dispatch; the alternative — re-running the subagent from scratch — would have produced duplicated or conflicting writes.
observed_count: 1
lesson: When a subagent task is interrupted mid-flight, reconstruct partial workspace state via Glob plus Read on the touched paths before resuming or re-dispatching; do not assume the subagent's prior turn had no side effects.
do: As the first action after detecting an interrupted subagent, Glob the directories named in the subagent's brief and Read every file that returned, comparing against the known pre-task state to enumerate what landed; only then decide between resume (use the existing files) and re-dispatch (start over with explicit "ignore prior partial work" instructions).
dont: Resume an interrupted subagent without reconstructing partial state, or re-dispatch it without explicit instructions about prior writes; both options produce duplicated or conflicting writes that surface as merge conflicts in the next handoff.
rationale: Subagent tools have side effects on the filesystem that survive the interruption; treating an interrupted subagent as a no-op corrupts the workspace and forces a more expensive recovery later.
confidence: medium
status: active
superseded_by: null
review_after: null

