# Project Registry

Canonical project slugs used by the learning system. Memory file names depend on these, so they must stay stable. Display names can change; slugs cannot, except by explicit migration.

## Why Slugs

Project display names drift, get reorganised, or differ between machines. The learning memory layer uses a stable slug so files like `coder-backend-python_whiskers_learning.md` keep meaning the same thing across all environments.

## Registered Projects

| Slug | Display Name | Brief File | Notes |
| --- | --- | --- | --- |
| `whiskers` | Whiskers | `briefs/whiskers_brief.md` | Cybersecurity log analysis toolkit, Python + ML + PyQt. |
| `health-app` | Mindful Mäuschen (Health App) | `briefs/health_app_brief.md` | Python + PyQt6 desktop health tracker. |
| `dragonflight` | Dragonflight | `briefs/dragonflight_brief.md` | Single-player turn-based dragon raiding strategy game, Python + Pygame. Engages Coder (Game Systems) and Game Map Designer roles. |
| `mauschens-mice` | Mäuschen's Mice | (this repo) | The agent operating manual itself. Lessons here are about agent process, not application code. |
| `global` | (cross-project) | (none) | Reserved slug. Used only for `<agentType>_global_learning.md` files. |

## Slug Rules

- Lowercase, kebab-case, ASCII only, no spaces.
- Stable for the life of the project. If you must rename, add a new row and mark the old one `deprecated`; do not delete history.
- Use `global` only for cross-project lessons. Do not create a project named `global`.

## Adding A New Project

1. Pick a slug following the rules above.
2. Add a row to the table.
3. Create the project brief in `briefs/<slug>_brief.md` (or update the existing brief filename if you adopt this convention later).
4. The first agent to write a lesson for the project will create memory files lazily from `learning/memory/_template.md`.

## Renaming Or Deprecating

If a project must change slugs:

1. Add a new row with the new slug.
2. Mark the old slug `deprecated` in this file with the date and replacement.
3. The user (not the agent) decides whether to copy old memory into the new file. Agents must not auto-migrate.
