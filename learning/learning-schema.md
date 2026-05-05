# Lesson Schema

Every lesson written to memory MUST follow this schema exactly. Lessons are stored as YAML blocks under the `## Lessons` section of a memory file. One YAML block per lesson, separated by `---`.

## Required Fields

```yaml
id: <agentType>-<projectSlug-or-global>-<YYYYMMDD>-<short-hash>
created: <ISO 8601 date, e.g. 2026-05-05>
updated: <ISO 8601 date>
agent_type: <e.g. coder-backend-python>
scope: project | global
project_slug: <slug from project-registry.md, or "global">
task_type: feature | bugfix | refactor | docs | review | research | other
tags: [<tag1>, <tag2>, ...]
trigger: <one sentence: what observation in this task produced this lesson>
evidence: <one or two sentences: concrete proof - failure, retry, user correction, measurable improvement>
observed_count: <integer, starts at 1, increments on duplicate detection>
lesson: <single imperative sentence. Do X. Avoid Y.>
do: <specific concrete action to take next time>
dont: <specific anti-pattern to avoid>
rationale: <one sentence: why this rule helps>
confidence: low | medium | high
status: active | quarantined | archived | superseded
superseded_by: <id or null>
review_after: <ISO date, optional, when this lesson should be re-evaluated>
```

## Field Rules

- `id`: deterministic and unique. The short-hash is a 6-char hash of the lesson text to make it stable for dedup.
- `agent_type`: must match exactly the file name (without `.md`) of the role in `agents/subagents/`, or `main-agent`.
- `project_slug`: must be present in `project-registry.md`. If `scope: global`, set `project_slug: global`.
- `tags`: 1 to 5 tags. Use lowercase, kebab-case (e.g. `tooling`, `git`, `pyqt`, `pytest`, `ml-model`, `cli`, `gui`).
- `trigger` and `evidence`: must reference something concrete from this task. No vague claims.
- `lesson`: imperative voice, one sentence, no compound rules.
- `do` and `dont`: actionable and specific. "Run `ruff check .` before declaring backend work complete" is good. "Be careful" is not.
- `confidence`:
  - `low` - inferred from one event without strong evidence. Default to `quarantined` status.
  - `medium` - clear evidence from this task and consistent with role expectations.
  - `high` - either repeated evidence (`observed_count >= 3`) or a user explicitly endorsed this rule.
- `status`:
  - `active` - loaded by future runs.
  - `quarantined` - stored but NOT loaded; awaits human review.
  - `archived` - kept for history but not loaded.
  - `superseded` - replaced by a newer lesson; `superseded_by` must be set.
- `review_after`: optional but recommended for stack-version-sensitive lessons (e.g. dependency upgrades likely to invalidate it).

## Example

```yaml
id: coder-backend-python-whiskers-20260505-a1b2c3
created: 2026-05-05
updated: 2026-05-05
agent_type: coder-backend-python
scope: project
project_slug: whiskers
task_type: bugfix
tags: [tooling, mypy, pydantic]
trigger: mypy failed on a function added without annotations even though the project enables check_untyped_defs.
evidence: Encountered mypy errors only after the user pointed out the missing annotations on the helper function in detectors/access.py.
observed_count: 1
lesson: Annotate all new helper functions in Whiskers, even private ones, before running mypy.
do: Add full type annotations on every new or modified function in Whiskers backend code, including private helpers.
dont: Rely on mypy passing for files where new helpers are unannotated; check_untyped_defs will still flag them.
rationale: The Whiskers project enables check_untyped_defs in mypy.ini, so partial annotation coverage produces noisy late-stage failures.
confidence: medium
status: active
superseded_by: null
review_after: null
```

## Forbidden Content

- Secrets, tokens, API keys, passwords, environment variable values.
- Absolute paths that include user directories or machine names beyond the workspace-relative path.
- Personally identifying information about the user.
- Code snippets longer than 10 lines (link to file/line in `evidence` instead).
- Speculation phrased as fact ("X is always slow"). State the observed evidence instead.
