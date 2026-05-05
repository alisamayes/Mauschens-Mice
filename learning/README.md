# Learning System

A post-task reflection pipeline for Mäuschen's Mice agents. After every completed task, an agent runs the learning routine to evaluate its own work and (selectively, safely) write small, reusable lessons into its memory file so future runs can benefit.

## Goals

- Capture lessons that reduce repeat mistakes, convoluted tool use, and unnecessary user clarification.
- Keep lessons specific, actionable, and verifiable.
- Prevent cross-project contamination by scoping memory per project.
- Prevent self-reinforcement of bad habits with quality gates and quarantine.

## Folder Layout

- `task-lifecycle.md` - states a task moves through and the exact moment learning is allowed to fire.
- `learning-routine.md` - the procedure every agent runs after the task is `user-approved`.
- `learning-schema.md` - strict format for every lesson.
- `learning-validator.md` - safeguards, quality gates, and rejection rules.
- `project-registry.md` - canonical mapping of project slugs (so naming stays stable).
- `memory/` - actual stored lessons.
  - `global/` - cross-project lessons per agent type.
  - `projects/` - per-agent, per-project lessons (highest priority on load).
  - `quarantine/` - candidate lessons that did not fully pass the validator and need review.
  - `_template.md` - skeleton used when a memory file is created lazily.

## Naming Convention

Memory files follow the pattern requested by the project owner, with stable slugs from `project-registry.md`:

- Project memory: `<agentType>_<projectSlug>_learning.md`
- Global memory: `<agentType>_global_learning.md`

`agentType` matches the role file name in `agents/subagents/` (e.g. `coder-backend-python`, `quality-assurance`). `main-agent` uses `main-agent` as its agent type.

## Load Order (when starting a task)

1. `learning/memory/global/<agentType>_global_learning.md` (if exists).
2. `learning/memory/projects/<agentType>_<projectSlug>_learning.md` (if exists).
3. Project memory always wins over global memory on conflict.
4. Quarantined lessons are NOT loaded into active behavior; they are only reviewed by humans.

## Write-Back Rules (after explicit user approval)

Learning fires ONLY when the task reaches state `user-approved` (see `task-lifecycle.md`). Subagents persist between their direct work and the user's approval, including across revision rounds. The main agent issues an explicit "task approved, run learning" signal that triggers the routine.

1. Run the procedure in `learning-routine.md`.
2. Produce candidate lessons in the format from `learning-schema.md`.
3. Validate them with `learning-validator.md`. User corrections recorded in the handoff `Revision History` are treated as high-confidence evidence (validator G8).
4. Append approved lessons to the matching memory file. Quarantine borderline ones. Reject the rest with a reason recorded in the handoff note.

If the user abandons the task without approving, learning does NOT run and no memory is written.

## Why Per-Project Scope

Different projects have different stacks, conventions, and risk profiles. A lesson learned in Whiskers (security log analysis) may not apply to the Health App (PyQt desktop tracker). Per-project memory keeps guidance relevant; the global file is reserved for lessons that hold across projects (for example, generic git or test-discipline rules).
