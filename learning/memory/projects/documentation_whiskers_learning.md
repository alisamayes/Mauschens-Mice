# Memory: documentation / whiskers

Lessons captured by `documentation` while working on `whiskers`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.

## Lessons

<!-- Append new YAML blocks below this line. Separate blocks with --- on its own line. -->

---
id: "documentation-whiskers-20260506-17736d"
created: 2026-05-06
updated: 2026-05-06
agent_type: documentation
scope: project
project_slug: whiskers
task_type: docs
tags: [python, docstrings, cli, codebase-audit]
trigger: The task required standardizing docstrings across Whiskers without adding noise to trivial functions.
evidence: Updated docstrings on CLI entrypoints and core engine/pipeline methods (e.g. `main.py`, `cli/main.py`, `whiskers.py`) while leaving the many simple `handle_*` flag toggles undocumented.
observed_count: 1
lesson: Prioritize entrypoints and core logic when standardizing docstrings; skip trivial handlers to avoid noise.
do: Add/standardize docstrings for public entrypoints and complex methods first, then selectively document helpers only when they carry non-obvious logic.
dont: Blanket-add docstrings to every small flag setter or one-liner handler in command parsing modules.
rationale: This keeps documentation high-signal and makes the public API and critical flows easier to understand without drowning the codebase in repetitive text.
confidence: medium
status: active
superseded_by: null
review_after: null

---
id: "documentation-whiskers-20260506-3075c1"
created: 2026-05-06
updated: 2026-05-06
agent_type: documentation
scope: project
project_slug: whiskers
task_type: docs
tags: [python, docstrings, accuracy]
trigger: The docstring policy required an Args/Returns structure that must match the live signatures.
evidence: Standardized docstrings on functions with optional parameters and keyword-only arguments (e.g. `cli/main.py: main(argv=...)`, `whiskers.py: run_generation(...)`) and ensured parameter names/default behaviors matched the signatures.
observed_count: 1
lesson: Verify docstring Args and defaults against the actual function signature before editing.
do: Read the function signature first and copy parameter names (including keyword-only parameters) into the Args section verbatim.
dont: Write or template docstrings from memory and risk mismatched parameter names or incorrect default descriptions.
rationale: Mismatched documentation is worse than missing documentation because it misleads callers and future maintainers.
confidence: medium
status: active
superseded_by: null
review_after: null

---
id: "documentation-whiskers-20260506-4d4c76"
created: 2026-05-06
updated: 2026-05-06
agent_type: documentation
scope: project
project_slug: whiskers
task_type: docs
tags: [python, docstrings, lint]
trigger: Multiple files received docstring-only edits and needed verification that formatting remained valid.
evidence: Ran lints on the edited modules and confirmed there were no new diagnostics after the docstring-only changes.
observed_count: 1
lesson: After docstring-only changes, run lints on the edited files to ensure formatting did not introduce warnings or errors.
do: Run the project's linter/diagnostics on the specific files touched by docstring updates before handing off.
dont: Assume docstring-only edits are always safe; formatting/indentation issues can still produce lint failures.
rationale: A quick lint pass prevents docstring formatting mistakes from breaking CI or developer workflows.
confidence: medium
status: active
superseded_by: null
review_after: null

