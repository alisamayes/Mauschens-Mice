# Quarantine

Candidate lessons that did not fully pass the validator in `learning/learning-validator.md`. NOT loaded by agents during tasks. For human review only.

## What Lands Here

- Low-confidence candidates (`confidence: low` without explicit user endorsement).
- Candidates that conflict with an existing active lesson.
- Candidates that look plausible but lack strong evidence.

## File Pattern

`<agentType>_<projectSlug>_quarantine.md`

Each entry uses the same YAML format as `learning-schema.md`, with `status: quarantined` and (where relevant) a `superseded_by` or conflict reference.

## Review Workflow

The user (not an agent) decides whether to:

- Promote the lesson to active by moving the YAML block into the matching memory file under `global/` or `projects/`, and changing `status: active`.
- Refine and merge with an existing lesson.
- Archive it (move to memory file with `status: archived`).
- Delete it.

Agents must not auto-promote quarantined lessons.
