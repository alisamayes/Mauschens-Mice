# Memory: security-specialist / whiskers

Lessons captured by `security-specialist` while working on `whiskers`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: security-specialist-whiskers-20260505-3ecfc6
created: 2026-05-05
updated: 2026-05-05
agent_type: security-specialist
scope: project
project_slug: whiskers
task_type: bugfix
tags: [deserialization, ml-model, integrity, joblib]
trigger: Risk review identified joblib.load on a default supervised model path as pickle-class deserialization exposure.
evidence: Mitigation shipped: model files must resolve under models/ and load runs only after SHA-256 verification against a sibling hash file; training script writes the hash on each dump.
observed_count: 1
lesson: Load supervised joblib bundles in Whiskers only from paths under models/ after SHA-256 verification against a sibling hash file.
do: Keep models/ip_supervised_rf.joblib.sha256 in sync via train_supervised_ip_classifier when the bundle changes; refuse load on missing hash or mismatch.
dont: Call joblib.load on a supervised artifact that bypasses resolve_models_file or secure_load_supervised_bundle.
rationale: Reduces arbitrary-code execution from a replaced bundle without blocking normal retrain workflows.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-whiskers-20260505-94ac8a
created: 2026-05-05
updated: 2026-05-05
agent_type: security-specialist
scope: project
project_slug: whiskers
task_type: bugfix
tags: [path-traversal, filesystem, cli]
trigger: Review noted directory plus filename join could escape intended folders when arguments contained traversal segments.
evidence: save_logs now resolves via safe_output_path_for_save with project-root checks, single-component filenames for three-arg form, and rejection of literal .. in directory arguments.
observed_count: 1
lesson: Resolve Whiskers save destinations with safe_output_path_for_save so outputs stay under the project root and cannot escape via mixed directory and filename arguments.
do: Use the helper for every save path; keep three-arg filename as one path component.
dont: Use raw os.path.join on user directory and filename tokens without containment checks relative to the Whiskers project root.
rationale: Prevents writing copies outside the intended workspace.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-whiskers-20260505-6b9775
created: 2026-05-05
updated: 2026-05-05
agent_type: security-specialist
scope: project
project_slug: whiskers
task_type: bugfix
tags: [cli, repl, interactive, operator-error]
trigger: Interactive REPL used input() to drive the same command parser as argv, increasing mistaken destructive actions.
evidence: _interactive_repl gates confirmation prompts on save and shred in the REPL only; startup argv processing leaves confirmation off.
observed_count: 1
lesson: Require explicit yes confirmation for Whiskers save and shred when driven from the interactive REPL, not from non-interactive argv.
do: Set _interactive_repl only around await_input process_commands and use _interactive_confirm for save_logs and shred_logs.
dont: Add extra prompts on argv startup paths where automation or scripting is expected.
rationale: Reduces confused-deputy mistakes in the shell without breaking scripted or GUI flows.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-whiskers-20260506-256cd5
created: 2026-05-06
updated: 2026-05-06
agent_type: security-specialist
scope: project
project_slug: whiskers
task_type: bugfix
tags: [filesystem, destructive-action, allowlist, cli, gui]
trigger: Security hardening work identified that deleting log files via shred could target arbitrary paths.
evidence: Updated Whiskers to enforce a default boundary under data/ for shred targets and to require explicit bypass flags for deleting outside that boundary (implemented in path_security.py and simulator/file_manager.py).
observed_count: 1
lesson: Enforce an allowlisted directory boundary for destructive file operations in Whiskers (default: data/) and require explicit bypass flags for anything outside it.
do: Use a single helper to resolve-and-validate targets under <repo>/data for destructive operations (and make bypass an explicit, named flag).
dont: Accept a raw path (directly or via GUI selection) as a shred/delete target without first proving it lies under the intended allowlisted directory.
rationale: Prevents arbitrary file deletion if a configured path or UI input points outside the Whiskers workspace boundary.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-whiskers-20260506-6ab27d
created: 2026-05-06
updated: 2026-05-06
agent_type: security-specialist
scope: project
project_slug: whiskers
task_type: bugfix
tags: [filesystem, safe-defaults, overwrite, cli, gui]
trigger: Security hardening work found that save could overwrite existing destinations or write outside intended directories without an explicit opt-in.
evidence: Updated Whiskers save to refuse overwriting by default unless --overwrite is provided, and to refuse writing outside data/ unless explicitly allowed (simulator/file_manager.py plus GUI call site adjustments).
observed_count: 1
lesson: Default Whiskers save behavior must refuse overwriting existing files and refuse writing outside data/ unless explicitly allowed.
do: Make overwrite and outside-data writes opt-in via explicit flags (and add extra confirmation/force gating for the risky paths).
dont: Use save-as style flows that silently overwrite or write to arbitrary absolute destinations without a deliberate overwrite/allow flag.
rationale: Safe defaults reduce the blast radius of operator error and limit abuse through path injection or accidental selection.
confidence: high
status: active
superseded_by: null
review_after: null
```
