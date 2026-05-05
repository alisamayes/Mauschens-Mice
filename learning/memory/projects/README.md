# Project Memory

Per-agent, per-project lessons. Highest-priority memory: project lessons override global lessons on conflict.

## File Pattern

`<agentType>_<projectSlug>_learning.md`

Examples:

- `coder-backend-python_whiskers_learning.md`
- `quality-assurance_health-app_learning.md`
- `main-agent_mauschens-mice_learning.md`

`projectSlug` must come from `learning/project-registry.md`. If the slug is not registered, the agent must stop and ask the main agent to register it before writing memory.

Files are created lazily on first write from `learning/memory/_template.md`.
