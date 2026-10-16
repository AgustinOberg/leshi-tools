# Delivery: branches, tests, PRs and Ready for Review (shared)

## Branches and worktrees

- Work on a branch per item: `delivery.branchPrefix.feature` (default `feature/`) or `.bug` (default `fix/`) + the TM id + a short slug, e.g. `feature/123-voice-chat`.
- Branch from the up-to-date target branch (`delivery.targetBranch`, default `main`).
- For independent parallel writes, use worktrees under `delivery.worktreesDir` (default `../<repo>-worktrees/`). Never delete a worktree with unmerged changes.

## Test policy (defined in `bootstrap-technology`, stored in `testing`)

- Backend tests are always required, whatever `testing` says: every backend change ships with tests through the real entry point against a local database only for tests (craft §9). `testing` only names their tools and commands.
- While implementing: targeted tests for what changed (`testing.unitCommand` scoped to files/modules; backend tests with `testing.integrationCommand` scoped to the touched module; plus typecheck/lint when relevant).
- Before the final commit/push: every command in `testing.requiredBeforePush` (at least the configured unit tests) must pass.
- Feature-level: integration checks and the applicable E2E (`testing.e2e`); visual-heavy features on every target platform (Android **and** iOS when both are targets; tablet when relevant); functional features may use one representative device if the project's policy allows it.
- Never remove, skip or weaken a required check to make it pass. Disclose any check that cannot run here and why.

## Commits

- Small, meaningful commits referencing the TM id. No secrets, no generated noise, no unrelated changes.
- Follow the repository's existing commit conventions.

## Pull requests

- Push the branch and open a PR against the target branch.
- PR body: summary, link to the TM item + approved Spec revision (`<!-- leshi:pr url=... spec=<item>@<rev> -->` goes in the TM, the human-readable link in the PR), what changed, test results (commands + pass/fail), visual evidence when relevant (screenshots), and **manual test instructions** a human can follow.
- Never merge automatically. Never open an empty PR: if nothing needs to change, report findings in the TM instead.

## Ready for Review — [CHECK:pr-before-ready-for-review]

Set *Ready for Review* only when all are true:
1. a PR is open against the target branch;
2. required local validations passed (and are listed in the PR);
3. required GitHub checks (CI) pass — check with the GitHub MCP or `gh pr checks`; if CI is still running, wait or leave the item *In Progress* with a note;
4. the PR is genuinely testable by a human (instructions + environment).

Then save in the TM: PR URL, test instructions, final summary and evidence.
