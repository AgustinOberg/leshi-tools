# Session checkpoints (shared)

Every manual run of a workflow keeps a durable, readable log at `.leshi/sessions/<run-id>.md`. It lets a later session resume after a crash, a network cut or a closed terminal without redoing validated work. It is **execution state, not a SPEC**: the Task Manager keeps intent and decisions; Git keeps code.

## Start

At the beginning of the run (after preconditions pass):

```sh
leshi session start --workflow <skill-id> --ref <tm-item-id> --harness <claude-code|codex|opencode>
```

It prints the run id and path, and records the config fingerprint, current branch and base commit. Without the CLI, create the file yourself from the template below.

## Update — only after verified milestones

```sh
leshi session checkpoint <run-id> --done "<milestone + evidence>" [--status in-progress|awaiting-input|awaiting-approval|blocked|completed] [--active "<current step>"] [--file <path>]... [--test "<command> (pass|fail)"]... [--pending "<question/decision>"]... [--next "<next action>"]
```

Checkpoint when something is *proven*: a subtask passed its tests, a gate was approved, a PR was opened, a blocker was confirmed. Do not log every thought.

## Never store

Credentials, tokens, private reasoning, full chat transcripts, large logs or sensitive data dumps. Link to evidence (TM comments, CI runs, files) instead.

## Resume protocol

When the developer asks to continue a workflow (or you find an unfinished run for the same item):

1. `leshi session resume [<run-id>]` (or read the file). It reports config drift (fingerprint), branch/worktree existence, uncommitted changes and whether the base commit moved.
2. Re-read the **current** Task Manager state for the item: status, subtasks, latest approved Spec revision, new comments.
3. Compare: work recorded as done must still be visible in Git (commits/diff) and in the TM. Skip what is validated; redo only what cannot be verified.
4. If the approved Spec revision changed since the run started, stop and propose reconciliation (`spec-revisions.md`) before continuing.
5. If the log disagrees with Git or the TM, trust Git and the TM, tell the developer what differs and ask before acting.

## Template

```markdown
---
leshiSession: 1
runId: 2026-10-15T153000Z-implement-the-feature-123
workflow: implement-the-feature
harness: claude-code
status: in-progress
ref: FEATURE-123
specRevision: v4
configFingerprint: sha256:...
branch: feature/FEATURE-123
worktree: ../wt-123
baseCommit: abc1234
startedAt: 2026-10-15T15:30:00Z
updatedAt: 2026-10-15T16:10:00Z
---
# leshi run 2026-10-15T153000Z-implement-the-feature-123

## Checkpoints
- 2026-10-15T15:50Z subtask 1 done — `bun test src/x` pass

## Active
subtask 3

## Files impacted
- src/x.ts

## Tests executed
- `bun test src/x` (pass)

## Pending decisions / questions
- none

## Next action
continue subtask 3 after verifying the current git diff
```
