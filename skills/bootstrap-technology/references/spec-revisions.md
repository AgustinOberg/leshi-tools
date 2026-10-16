# Living SPECs, revisions and drift (shared)

Approved SPECs (Product, Technical, Design System, MVP, and each Feature's WHAT) change over time, but never silently.

## Change protocol

1. **Change proposed** — a meaningful change to product, feature, technology or UI Kit is identified (by the developer, a finding, or a conflict with reality). Keep the current approved revision intact.
2. **Proposal** — write the proposed revision as a TM comment with `<!-- leshi:spec ... status=proposed -->` (or the provider's draft mechanism): what changes, why, and alternatives.
3. **Approval** — `[GATE:spec-reconciliation]` or the workflow's own gate. Present the diff between the approved and the proposed revision. Wait.
4. **New approved revision** — increment the revision (`v3` → `v4`), update the item body to the new revision, post the snapshot comment, mark the previous one `superseded`. Update `specs.<kind>.approvedRevision` in the config for global SPECs.
5. **Impact awareness** — list affected designs, feature specs, subtasks and existing code. Record follow-ups as TM items/comments. Do not start other workflows; suggest them.

## Traceability

- Every PR references the TM item and the exact approved Spec revision it implements (`<!-- leshi:pr url=... spec=FEATURE-12@v4 -->`).
- Keep **approved**, **implemented** and **released** distinct. Git history is not requirements history.

## Drift

When code and the approved SPEC disagree:
- State the drift explicitly (what the SPEC says, what the code does, evidence).
- Record it on the TM item for reconciliation.
- Never rewrite the SPEC to match the code, nor change code to match the SPEC, without the developer deciding which side is right.
