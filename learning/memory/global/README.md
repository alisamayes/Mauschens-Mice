# Global Memory

Cross-project lessons per agent type. Files here are loaded for an agent regardless of which project the current task targets.

## When To Write Here

- The lesson is genuinely stack-agnostic and project-agnostic.
- You can name at least one other project (besides the current one) where it would apply unchanged.
- The lesson is about the agent's own process or generic engineering hygiene, not project-specific behavior.

When in doubt, write to `projects/` instead. Global memory should grow slowly.

## File Pattern

`<agentType>_global_learning.md`

Files are created lazily on first write.
