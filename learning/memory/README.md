# Memory

Stored lessons used by Mäuschen's Mice agents.

## Subfolders

- `global/` - lessons that apply across every project for a given agent type.
- `projects/` - lessons scoped to one agent type and one project. Highest priority on load.
- `quarantine/` - candidate lessons that did not fully pass the validator. NOT loaded by agents during tasks. For human review only.
- `_template.md` - the skeleton used when creating a new memory file.

## File Naming

Project memory: `<agentType>_<projectSlug>_learning.md`
Global memory: `<agentType>_global_learning.md`
Quarantine: `<agentType>_<projectSlug>_quarantine.md`

`agentType` matches the role file name in `agents/subagents/` (or `main-agent`). `projectSlug` must come from `learning/project-registry.md`.

## Lazy Creation

Memory files are created on first use. If an agent has nothing to write, no file is created. Empty files should not be checked in.

## Load Priority At Task Start

1. `global/<agentType>_global_learning.md` (lower priority).
2. `projects/<agentType>_<projectSlug>_learning.md` (higher priority - wins on conflict).
3. Never load `quarantine/`.

Lessons with `status: archived`, `superseded`, or `quarantined` are NOT loaded into active behavior even if present in an active file.
