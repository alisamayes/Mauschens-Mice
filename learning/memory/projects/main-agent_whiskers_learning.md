# Memory: main-agent / whiskers

Lessons captured by `main-agent` while working on `whiskers`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: main-agent-whiskers-20260505-7017e1
created: 2026-05-05
updated: 2026-05-05
agent_type: main-agent
scope: project
project_slug: whiskers
task_type: review
tags: [scope, orchestration, security]
trigger: User first requested a multi-project security and documentation audit, then explicitly restricted implementation work to Whiskers-only findings after documentation was deferred.
evidence: Hardening work targeted findings 1, 3, and 7 only under the Whiskers tree; documentation backlog was explicitly set aside for separate follow-up.
observed_count: 1
lesson: Restate which project slug and finding IDs are in scope before writing code when the user narrows a prior broad audit.
do: When the user narrows from a workspace-wide review to one repo or numbered findings, confirm that list in the handoff and implement only there unless the user expands scope again.
dont: Mix deferred tracks (for example documentation fixes) into a security implementation batch after the user asked to postpone them.
rationale: Keeps the change set reviewable and respects explicit deferrals.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: main-agent-whiskers-20260505-e6cdd3
created: 2026-05-05
updated: 2026-05-05
agent_type: main-agent
scope: project
project_slug: whiskers
task_type: feature
tags: [validation, ml, smoke-test]
trigger: Integrity checks and path rules were added around supervised model loading after the security pass.
evidence: Ran an import instantiation of SupervisedIPClassifierDetector from the Whiskers project root to confirm the model still loads when the hash file matches.
observed_count: 1
lesson: Smoke-test the supervised detector constructor after tightening model path or integrity checks in Whiskers.
do: From the project root, instantiate SupervisedIPClassifierDetector and assert the model object loads when the artifact and hash file are present.
dont: Ship path-resolution or hash-gating changes without at least one local load check.
rationale: Catches broken default paths, missing hash files, or hash mismatches before users run detection.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: main-agent-whiskers-20260506-a1777b
created: 2026-05-06
updated: 2026-05-06
agent_type: main-agent
scope: project
project_slug: whiskers
task_type: feature
tags: [security, gui, cli]
trigger: File-operation hardening added non-interactive confirmation/force gates and a default `<repo>/data` boundary for `save`/`shred`.
evidence: The GUI paths call `save_logs`/`shred_logs` without REPL input, so the hardening required GUI-side confirmations plus passing `--force` and related flags to preserve UX while keeping safe defaults.
observed_count: 1
lesson: When hardening file ops, ensure GUI code paths satisfy new confirmation/force gates without weakening defaults.
do: After changing CLI confirmation or allowlist rules, identify non-REPL callers (GUI pages, startup args) and update them to pass explicit `--force`/opt-in flags only after a GUI confirmation dialog.
dont: Add non-interactive confirmation gates without checking GUI call sites; this will silently break core buttons or force unsafe default relaxations.
rationale: Keeps security hardening effective without regressing GUI usability.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: main-agent-whiskers-20260506-9363fb
created: 2026-05-06
updated: 2026-05-06
agent_type: main-agent
scope: project
project_slug: whiskers
task_type: other
tags: [tooling, windows, powershell]
trigger: A repo status/diff check failed due to using a Bash-style multi-command separator in a PowerShell terminal.
evidence: The command containing `&&` produced a PowerShell parser error; switching to `;` separators succeeded and allowed reviewing the working tree diff.
observed_count: 1
lesson: When running multi-command shell snippets on Windows PowerShell, use ; separators (not &&) to avoid parse errors.
do: For PowerShell sessions, chain commands with `;` (or use separate invocations) when collecting status/diff/test output.
dont: Paste Bash-style `&&` chains into PowerShell; it can fail before running any command.
rationale: Prevents avoidable command failures during Windows-based review and validation work.
confidence: medium
status: active
superseded_by: null
review_after: null
```
