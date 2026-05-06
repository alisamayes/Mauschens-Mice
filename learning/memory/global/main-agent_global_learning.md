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

