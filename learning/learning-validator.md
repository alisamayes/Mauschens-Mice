# Learning Validator

Every candidate lesson must pass this validator before being merged into a memory file. The validator is a checklist; failing any required gate moves the candidate to `quarantine/` or rejects it outright.

## Decision Outcomes

- `approved` - all required gates pass. Merge into the chosen memory file.
- `quarantined` - candidate is plausible but cannot be trusted yet. Append to `learning/memory/quarantine/<agentType>_<projectSlug>_quarantine.md` for human review.
- `rejected` - candidate fails a hard rule. Drop and log the reason in the handoff.

## Required Gates

A candidate must pass ALL of the following or it is NOT `approved`.

### G1 - Schema Compliance
- All required fields from `learning-schema.md` are present and well-formed.
- `agent_type` matches a real role file or `main-agent`.
- `project_slug` exists in `project-registry.md` (or is `global`).
- Failure: rejected.

### G2 - Specificity
- `lesson`, `do`, and `dont` are concrete, actionable, and verifiable.
- No vague filler ("be careful", "think harder", "use best practices").
- Failure: rejected.

### G3 - Evidence
- `trigger` and `evidence` reference something that actually happened in this task.
- Speculative or hypothetical lessons are not allowed.
- Failure: rejected.

### G4 - Non-Redundancy
- The lesson does not duplicate guidance already present in:
  - The agent's role file in `agents/subagents/`.
  - The relevant project brief in `briefs/`.
  - An active lesson already in the same memory file with the same intent.
- If a near-duplicate active lesson exists (same `do`/`dont` semantics), increment that lesson's `observed_count` and update `updated` instead of writing a new one.
- Failure on duplicate: merge into existing lesson.
- Failure on already-implied-by-role: rejected.

### G5 - Sensitivity Scrub
- No secrets, tokens, credentials, or environment values.
- No absolute paths beyond the workspace.
- No user-identifying data.
- Failure: rejected.

### G6 - Conflict Check
- Lesson does not contradict an active lesson in the same memory file or the loaded global file.
- If a conflict exists, set `status: quarantined`, link the conflicting `id`, and surface the conflict in the handoff for user review. Do NOT auto-supersede.
- Failure: quarantined.

### G7 - Minimum Confidence Floor
- Lessons with `confidence: low` are written with `status: quarantined` by default unless the user explicitly approved during this task.
- Failure: quarantined (not rejected - low confidence has value when reviewed).

### G8 - User Correction Boost
- A candidate lesson whose `evidence` is a user correction recorded in the handoff `Revision History` AND that produced a real change applied to the work is treated as `confidence: high`.
- A user correction about preference, convention, or single-source-of-truth patterns is high-value: write it as `active`, not quarantined, even if it appears for the first time.
- Do NOT downgrade such a lesson just because `observed_count == 1`.
- This gate only fires AFTER the task has reached `user-approved`. Subagents must not pre-empt the lifecycle.

## Recommended Gates (soft)

These do not block approval but should be considered:

- `R1` - The lesson would change observable agent behavior. If it would not change anything, prefer rejecting it.
- `R2` - The lesson is small enough to fit in working memory alongside the role file. If a lesson sprawls, split it.
- `R3` - The lesson includes at least one tag that matches expected retrieval queries.

## Memory Hygiene Rules

These run periodically (when triggered by the user, or when a memory file exceeds size limits) but are documented here for completeness:

- Lessons with `observed_count == 1` and older than 90 days with no further hits should be downgraded to `archived` unless explicitly endorsed.
- Lessons whose `review_after` date has passed should be re-evaluated against the current stack and either renewed or archived.
- A memory file should not exceed roughly 60 active lessons. If it does, the lowest-confidence and oldest-unused entries should be archived first.

## Conflict Resolution Procedure

When G6 fails:

1. Do not modify the existing active lesson.
2. Write the new candidate to `quarantine/` with `status: quarantined` and `superseded_by: <existing id?>` left null.
3. In the handoff, include a `Conflict Detected` block:
   - The two lesson IDs.
   - A one-sentence summary of each.
   - A request for user decision: keep current, replace with new, or merge into a refined rule.
4. The user (not the agent) marks the resolved outcome on the next run.

## Failure Logging

Every rejected or quarantined candidate must produce a one-line entry in the handoff note's `Lessons Captured` section explaining why. This prevents silent data loss and gives the user a chance to overrule the validator.
