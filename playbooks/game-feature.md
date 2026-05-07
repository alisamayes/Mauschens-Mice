# Playbook: Game Feature Delivery

Use this playbook for feature work on game projects (e.g. Dragonflight) where gameplay simulation and/or map content are part of scope. For non-game projects, use `feature-delivery.md` instead.

## When To Use

Pick this playbook when the change touches one or more of:

- Gameplay rules, formulas, turn flow, combat, AI, pathfinding, economy, or progression.
- Map data, terrain layouts, settlement placement, scenario balance, or map schema.
- Engine-level systems specific to gameplay (time budgets, hex math, aggression, army merging, etc.).

If the change is purely infrastructure (build tooling, data layer, generic utilities) and has no gameplay impact, prefer `feature-delivery.md`.

## Steps

1. Project Manager writes task brief (objective, scope, acceptance criteria), referencing the relevant section(s) of the project's specification.
2. Architectural Lead validates design boundaries, ownership split between Coder (Python Backend) and Coder (Game Systems), and any Game Map Designer involvement.
3. Game Map Designer authors or revises map data, schema, and validation rules **first** when the feature changes the map surface; otherwise this step is skipped.
4. Coder (Game Systems) implements gameplay rules, simulation, and rule-invariant tests in small reviewable increments. Coder (Python Backend) handles supporting infrastructure (loaders, persistence, plumbing) where needed.
5. GUI Developer implements presentation, HUD, confirmations, and game-over flow as required.
6. Documentation updates comments, docstrings, README, and the project specification for any behavior or structural change.
7. Security Specialist reviews changed and adjacent risk surfaces (e.g. unsafe file loading for map data, untrusted save formats once introduced) and reports findings by severity.
8. Quality Assurance validates behavior against acceptance criteria and the rule invariants in the spec (e.g. damage floors, illegal stranded-dragon rejection, army merge/order).
9. Main Agent summarises the outcome to the user. The task enters `awaiting-user-review`. Subagents persist; learning has NOT yet run.
10. If the user requests changes, Main Agent enters `revising`, re-engages the relevant subagents, captures the correction in `Revision History`, and returns to step 9.
11. Once the user explicitly approves (state `user-approved` per `learning/task-lifecycle.md`), Main Agent issues "task approved, run learning" to every participating subagent.
12. Each subagent runs `learning/learning-routine.md` once and returns its lesson summary.
13. Main Agent runs its own post-task learning routine, consolidates all lesson summaries into the handoff, and closes the task.

## Inputs

- Task brief
- Project specification (e.g. `Documentation/...` for Dragonflight)
- Existing architecture and rule constraints
- Relevant map data and existing scenario baselines (if a map-touching task)

## Outputs

- Implemented gameplay or map change
- Map validation results (when relevant)
- Updated tests, especially rule-invariant tests
- Updated documentation and spec where behavior changed
- QA report
- Security review notes
- Handoff notes

## Exit Criteria

- Acceptance criteria all pass or have approved exceptions.
- Rule invariants relevant to the change pass automated checks (or are documented as deferred with rationale).
- Map data, when touched, passes the validation suite (reachability, spawn legality, bridge/river logic, schema correctness).
- No simulation rule logic has leaked into rendering or UI code.
- Risks and follow-ups are documented.
- User has explicitly approved the result (state `user-approved`).
- Post-task learning routine has been run by every participating role AFTER user approval; the handoff records approved/quarantined/rejected lesson counts, the `Revision History`, and any conflicts needing user review.
