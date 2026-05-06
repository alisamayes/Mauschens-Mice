# Quarantine: security-specialist / whiskers

Candidates that failed the learning validator for automatic merge. These are **not loaded** into active behavior until a human reviews them.

## Quarantined Lessons

```yaml
id: security-specialist-whiskers-20260506-195c05
created: 2026-05-06
updated: 2026-05-06
agent_type: security-specialist
scope: project
project_slug: whiskers
task_type: bugfix
tags: [cli, gui, repl, confirmation, force-flag]
trigger: Security hardening work found that relying on REPL-only confirmation left argv/startup invocations and GUI-triggered actions without an equivalent safety gate.
evidence: Implemented a non-interactive force gate for risky filesystem actions (e.g., overwrite/outside-boundary saves and shredding) so the gate applies even when invoked via startup args; this contradicts an existing active lesson that discouraged argv prompts.
observed_count: 1
lesson: Gate destructive or risky filesystem actions in Whiskers behind explicit user confirmation in REPL or an explicit --force flag in non-interactive invocation paths.
do: Require either typed yes (REPL) or an explicit --force/--yes flag for destructive/risky file operations in non-interactive contexts (CLI startup args or GUI-to-backend calls).
dont: Rely on REPL-only prompts when the same operations can be triggered from startup args or the GUI without an operator at a terminal.
rationale: Ensures the safety gate exists across all invocation paths while keeping automation explicit.
confidence: medium
status: quarantined
superseded_by: null
review_after: 2026-06-06
```

