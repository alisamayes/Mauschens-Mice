# Memory: security-specialist / dragonflight

Lessons captured by `security-specialist` while working on `dragonflight`. Loaded at task start. Updated only by the post-task learning routine.

## How To Read This File

- Each lesson is a YAML block following `learning/learning-schema.md`.
- Only lessons with `status: active` are applied to behavior.
- Lessons with `status: quarantined`, `archived`, or `superseded` are kept for traceability but ignored by active agents.
- Do not edit by hand without updating `updated` and recording rationale in the project's handoff history.

## Lessons

```yaml
id: security-specialist-dragonflight-20260508-132fc4
created: 2026-05-08
updated: 2026-05-08
agent_type: security-specialist
scope: project
project_slug: dragonflight
task_type: review
tags: [pydantic, validation, dos-hardening, loader, security-default]
trigger: Slice 1 Wave-4 review found `map_loader.py`'s Pydantic v2 boundary models accepted unbounded numeric values for `width`/`height`/`hexSize`/`q`/`r`/`schemaVersion`; logged as Low (L1). User explicitly accepted L1 as a defect to fix this round, and the Backend Coder added explicit `Field(ge=…, le=…)` bounds plus named MAX constants in revision round 1.
evidence: `coder-backend-python_dragonflight_learning.md` lesson `…-85501c` records the same bound-by-default pattern from the implementer side; `tests/test_map_loader.py::TestPydanticBounds` pins five rejection paths after the fix; G8 user-correction boost applied because the user-recorded Revision-History fix produced concrete code.
observed_count: 1
lesson: For any Pydantic v2 boundary model in Dragonflight, attach explicit numeric bounds (`Field(ge=…, le=…)` and a named MAX constant) by default rather than as a security retrofit.
do: When reviewing or adding a new Pydantic v2 model that ingests untrusted JSON in Dragonflight, treat the absence of explicit numeric bounds on every `int`/`float` field as a defect on first sight, and require named module constants (`_MAX_…`) and at least one rejection test per field before approving the slice.
dont: Treat unbounded numerics on a Pydantic v2 boundary model as "fine because the data is local and trusted" — Dragonflight expects to load community-authored or shared maps in later slices, and loaders are reviewed in retrospect rather than refactored under release pressure.
rationale: Bounds defend the parse-time CPU and memory ceiling, surface authoring typos with field-name detail in the Pydantic error, and remove a class of finding from every future security review without any per-field thinking.
confidence: high
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-dragonflight-20260508-992aa8
created: 2026-05-08
updated: 2026-05-08
agent_type: security-specialist
scope: project
project_slug: dragonflight
task_type: review
tags: [error-handling, json, recursion, loader, contract]
trigger: Slice 1 Wave-4 review found that `map_loader.load_map` translates `pydantic.ValidationError` and `json.JSONDecodeError` into `MapLoadError`, but a `RecursionError` raised by deeply nested JSON parsing (or by a malformed Pydantic discriminator) escapes as a raw Python exception, breaking the loader's "all expected failures surface as MapLoadError" contract (logged as Low L2).
evidence: Review traced the `try/except` around `json.loads` and `model_validate`; only `JSONDecodeError`/`ValidationError` are caught; Python's stdlib raises `RecursionError` (not `JSONDecodeError`) when a JSON file's nesting depth exceeds the recursion limit, so the loader contract is leakier than its docstring implies.
observed_count: 1
lesson: Wrap the entire load path including `json.load(s)` in the `MapLoadError` translation layer so `RecursionError` from deeply nested JSON cannot escape as a raw Python exception in Dragonflight.
do: In `map_loader.load_map`, extend the existing `except` clause to also catch `RecursionError` (and any other top-level parser exception class) and translate it into a `MapLoadError` whose message names the offending file path and notes "JSON nesting too deep"; cover with a parametrised negative-path test that constructs a deeply nested fixture.
dont: Limit the `MapLoadError` translation to `JSONDecodeError`/`ValidationError`; that pair leaves recursion-depth and other parser-internal exceptions to surface as raw tracebacks, which both leaks Python internals into log files and breaks the documented loader contract.
rationale: A loader that promises "all expected failures are MapLoadError" must defend that promise against the full set of stdlib parser exceptions, not just the headline two — otherwise downstream callers (renderer, future save/load, future content-pack manager) cannot rely on the type.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-dragonflight-20260508-3cb902
created: 2026-05-08
updated: 2026-05-08
agent_type: security-specialist
scope: project
project_slug: dragonflight
task_type: review
tags: [methodology, ripgrep, review-process, negative-evidence]
trigger: Slice 1 Wave-4 review needed to evidence the absence of forbidden APIs (`eval`, `exec`, `pickle`, `marshal`, `subprocess`, network calls) and unsafe filesystem sinks across the full slice surface; the most reliable way to make that absence concrete in the report was to run two scoped ripgrep sweeps and cite their (zero-result) output in the findings rather than describe the search prose-only.
evidence: Wave-4 security report explicitly stated "no `eval`/`exec`/`pickle`/`marshal`/`subprocess`/network usage anywhere in `src/`, `tests/`, `scripts/`, or the top-level shim"; that statement is only as defensible as the search method behind it, which was a forbidden-API ripgrep pattern plus a filesystem-sink ripgrep pattern across the same paths.
observed_count: 1
lesson: Before drafting a Dragonflight security review, run two ripgrep negative-existence sweeps — one for the forbidden-API set and one for the filesystem/network sink set — across the in-scope paths and cite the zero-result evidence in the report.
do: For every Dragonflight security review, prepare two ripgrep patterns (forbidden-API set and filesystem-sink set), run them across `src/`, `tests/`, `scripts/`, and any top-level shim, and quote the zero-result outcome in the "Scope reviewed" section so the absence claim is reproducible.
dont: Assert "no `eval`/`exec`/etc. anywhere" in a Dragonflight finding without naming the search pattern and the searched paths; reviewers and future maintainers cannot tell whether absence was verified or assumed.
rationale: Negative-existence claims are the most often hand-waved part of a security report; pre-committed ripgrep patterns make them cheap to verify, cheap to re-run on later slices, and durable evidence in the handoff.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-dragonflight-20260508-8aa395
created: 2026-05-08
updated: 2026-05-08
agent_type: security-specialist
scope: project
project_slug: dragonflight
task_type: review
tags: [threat-model, severity, deferral, review-process]
trigger: Slice 1's threat model is a single-player desktop game where today only the bundled trusted map is loaded; the Wave-4 review filed M1 (loader DoS) and L1-L3 with severity reasoning that referenced this threat model explicitly, and the user accepted the deferral of M1, L2, L3 to a later slice without dispute because the deferral rationale survived a re-read.
evidence: Wave-4 security report's M1 finding explicitly tied severity to "not exploitable today (the only map we open is the bundled trusted one), but should be hardened before any feature that lets users open arbitrary or shared maps"; the user's revision round accepted L1 for fix and deferred M1/L2/L3 without challenging the severity calls.
observed_count: 1
lesson: In every Dragonflight security finding, anchor the severity sentence to the project's threat-model attributes (single-player desktop, only the bundled trusted map is loaded today, no save/load, no network) so deferral decisions remain defensible when the threat model changes.
do: For each finding, write the severity rationale as one sentence that names the current threat-model attributes that constrain exploitability AND the future slice that would relax them; this lets the user defer with confidence and lets a future review re-evaluate quickly.
dont: File a Dragonflight finding whose severity reasoning is generic ("Medium because DoS"); a generic severity sentence forces the user to re-derive the threat model on every triage.
rationale: Severity in MVP-stage projects is mostly a function of which slice introduces the exploitable surface; explicit threat-model anchoring makes deferral durable and re-review cheap, and discourages severity inflation when the bundled-trusted-data assumption still holds.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-dragonflight-20260511-assetpt
created: 2026-05-11
updated: 2026-05-11
agent_type: security-specialist
scope: project
project_slug: dragonflight
task_type: review
tags: [filesystem, path-traversal, assets, save]
trigger: This task added a “Save map to assets/” requirement, but the repo has no existing safe-write helper and current code only demonstrates safe asset path reading via `Path(__file__).resolve()` rooted joins.
evidence: Repo pattern: `src/dragonflight/__main__.py` resolves project root from `__file__` and joins `assets/example_hexmap.json`; `src/dragonflight/map_loader.py` reads arbitrary paths passed in, so the new Save sink must enforce containment explicitly rather than trusting caller input.
observed_count: 1
lesson: Enforce `assets/` confinement by resolving paths and rejecting any save target that is not `relative_to(assets_root)`.
do: Compute a canonical `assets_root` from `__file__`, build `target = assets_root / filename`, then compare `target.resolve()` against `assets_root.resolve()` using `relative_to`; reject on failure before any file operation.
dont: Accept user-provided paths (absolute, `..`, or separator-containing strings) or rely on `joinpath` alone; without canonical containment checks, traversal or symlink escape can write outside `assets/`.
rationale: Canonicalization + `relative_to` is a simple, robust barrier against traversal and “write anywhere” vulnerabilities.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-dragonflight-20260511-fnameok
created: 2026-05-11
updated: 2026-05-11
agent_type: security-specialist
scope: project
project_slug: dragonflight
task_type: review
tags: [filesystem, input-validation, windows, save]
trigger: Save requires a user-chosen map filename; filenames are an input channel distinct from JSON schema validation and need their own allowlist.
evidence: This review identified the need to prevent traversal and OS quirks (notably Windows reserved names and trailing dot/space behavior) while saving into `assets/`; there is no repo-wide filename validation helper today.
observed_count: 1
lesson: Validate map filenames with a strict allowlist (chars, length, extension) and reject Windows-reserved device names.
do: Require a simple filename pattern like `^[A-Za-z0-9][A-Za-z0-9._-]{0,63}\\.json$`, reject any separators, reject trailing dot/space, and block `CON/PRN/AUX/NUL/COM1…/LPT1…` (case-insensitive).
dont: “Sanitize” arbitrary user strings into filenames or accept paths/Unicode confusables by default; silent rewriting makes it hard to reason about where the file went and can hide abuse.
rationale: A strict allowlist removes ambiguity across OSes and closes multiple classes of path and device-name abuses with minimal complexity.
confidence: medium
status: active
superseded_by: null
review_after: null
```

---

```yaml
id: security-specialist-dragonflight-20260511-atomicw
created: 2026-05-11
updated: 2026-05-11
agent_type: security-specialist
scope: project
project_slug: dragonflight
task_type: review
tags: [filesystem, atomic-write, overwrite, integrity]
trigger: Save can overwrite an existing map JSON in `assets/`, creating integrity risk (partial writes) and UX risk (silent destructive overwrite).
evidence: This task’s review recommended explicit overwrite confirmation and atomic replace because the repo currently only reads the bundled asset and does not yet have a safe write pattern to prevent corruption on crash/interruption.
observed_count: 1
lesson: Perform map saves as atomic writes and require explicit overwrite confirmation when the target exists.
do: If target exists, require a deliberate overwrite confirmation; write to a temp sibling in `assets/` and then `replace()` to the final filename so failures don’t corrupt the existing file.
dont: Write directly into the final path without confirmation or atomicity; a crash mid-write can leave a truncated JSON and destroy the prior good map.
rationale: Atomic replace preserves last-known-good data and makes overwrite behavior predictable and auditable.
confidence: medium
status: active
superseded_by: null
review_after: null
```
