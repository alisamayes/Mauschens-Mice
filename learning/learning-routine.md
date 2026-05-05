# Post-Task Learning Routine

This routine is mandatory. Every agent (main agent and every role subagent) runs it once per task, but ONLY after the user has explicitly approved the result.

## When To Run

Run only when ALL of the following are true:

- The task has reached state `user-approved` as defined in `learning/task-lifecycle.md`.
- The main agent has issued the explicit "task approved, run learning" signal (subagents must NOT run learning on their own initiative).
- This routine has not yet been run for this task by this agent.

Do NOT run this routine when:

- The agent has merely finished its own immediate piece of work.
- The main agent has summarised changes to the user but the user has not approved.
- The user response is ambiguous, silent, or partial.
- The user has abandoned the task without approval (record "task abandoned, no lessons captured" and exit).

Run exactly once per task per agent. Do not run inside intermediate steps or revision rounds.

## Inputs

- The full task brief and acceptance criteria.
- Your own context history for this task (decisions, tool calls, retries, dead ends).
- Any user interventions, corrections, or clarifications during the task, including every revision round.
- The `Revision History` block from the handoff note (highest-signal source of evidence).
- The agent type running the routine (e.g. `coder-backend-python`).
- The project slug (look it up in `learning/project-registry.md`; if missing, stop and ask the main agent to add it before learning is written).

## Procedure

### Step 1 - Reflect

Honestly answer the following questions in plain prose first (do not skip):

1. What was the goal? What did the user-approved "done" actually look like vs the initial brief?
2. What approach did I take? Was there a simpler one available?
3. Which tools did I use, and which were unnecessary or redundant?
4. Where did I get stuck, retry, or change direction?
5. Where did the user intervene or clarify? What did each revision round reveal?
6. Which corrections were about preference/convention vs functionality? (Both matter, but they produce different lesson types.)
7. What pitfalls or surprising behavior did I encounter?
8. What worked unusually well and is worth repeating?
9. What would I do differently next time on a similar task?

Pay particular attention to user corrections that were not in the original brief but were applied. These almost always reveal an unwritten convention worth capturing as a lesson.

### Step 2 - Generate Candidate Lessons

Convert the reflection into atomic candidate lessons using the format in `learning-schema.md`. Rules:

- One lesson per discrete, reusable insight. Do not bundle.
- Each lesson must be a single imperative rule (Do X / Avoid Y).
- Each lesson must include concrete evidence from this task.
- Prefer fewer, higher-quality lessons over many vague ones.
- If you have nothing reusable to add, that is a valid outcome. Record it in the handoff note as "no new lessons" and stop.

### Step 3 - Validate

Run every candidate through `learning-validator.md`. For each candidate, decide:

- `approved` - merge into the appropriate memory file.
- `quarantined` - append to `learning/memory/quarantine/` for human review.
- `rejected` - drop, but record the reason in the handoff note.

### Step 4 - Decide Scope

For each `approved` lesson, decide whether it belongs in:

- Project memory (`<agentType>_<projectSlug>_learning.md`) - default for anything tied to this codebase, stack, conventions, or product behavior.
- Global memory (`<agentType>_global_learning.md`) - only if the lesson is genuinely stack-agnostic and project-agnostic, and you can name at least one other project where it would apply unchanged.

When in doubt, choose project scope.

### Step 5 - Merge

For each approved lesson:

1. If the target memory file does not exist, create it from `learning/memory/_template.md`.
2. Check for duplicates and conflicts (see `learning-validator.md`).
   - If a near-duplicate exists, increment its `observed_count` and update `updated`. Do not append a second copy.
   - If a conflicting lesson exists, do NOT auto-overwrite. Move the new candidate to `quarantine/` and flag the conflict in the handoff note for user review.
3. Otherwise, append the new lesson under the `## Lessons` section of the memory file.

### Step 6 - Report

In the final handoff note, fill in the `Lessons Captured` section with:

- Count of approved lessons (and which file each was written to).
- Count of quarantined lessons (and the reason).
- Count of rejected lessons (and the reason).
- Any conflicts that need user review.

## Hard Stops

Do NOT write a lesson if any of the following is true:

- It contains user-identifying data, secrets, tokens, paths to private files, or any content that looks sensitive.
- It is a restatement of a generic best practice already implied by the role file.
- It is based on a single ambiguous event with no concrete evidence.
- It contradicts an existing active lesson without explicit user approval.
- The agent type or project slug cannot be determined.

If a hard stop applies to all candidates, record "no new lessons" in the handoff and exit cleanly.
