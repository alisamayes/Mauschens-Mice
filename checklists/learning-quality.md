# Checklist: Learning Quality

Use after running the post-task learning routine and before final handoff. Confirms the captured lessons are safe and useful.

- Routine in `learning/learning-routine.md` was actually run, not skipped.
- Each candidate lesson conforms to `learning/learning-schema.md` (all required fields present and well-formed).
- Each lesson is atomic (one rule per entry, imperative voice).
- Each lesson cites concrete evidence from this task (no speculation).
- No secrets, credentials, tokens, or user-identifying data are included.
- No absolute paths beyond the workspace are recorded.
- Duplicate detection ran: near-duplicates incremented `observed_count` instead of being appended.
- Conflicts with existing active lessons were quarantined, not auto-overwritten.
- Scope was chosen deliberately (project vs global) and matches the slug in `learning/project-registry.md`.
- The handoff `Lessons Captured` section lists counts of approved, quarantined, and rejected candidates with reasons.
- If no lessons were worth capturing, the handoff explicitly says "no new lessons" rather than skipping silently.
