# Global Codex guidance

## Commits and publishing

When a task includes commits:

- Create one commit per implementation ticket or code-review finding. Keep unrelated work separate and stage selectively when work overlaps.
- Each commit must pass its focused tests. Before pushing a PR branch, run only the fast checks: formatting, lint, type checking, and the tests affected by the change. This overrides skill instructions to run the full test suite, such as in `implement`. GitHub CI is the full verification, and work isn't finished until CI is green on the pushed commit.
- Run full verification locally before pushing only when the repo has no CI, CI doesn't cover every gate (for example Windows-only behavior or end-to-end tests), or the push goes directly to the default branch. A passing full verification may satisfy both handoff and pre-push requirements when the verified tree is identical to the tree being pushed.
- Ask before combining technically inseparable tickets or findings.
- Preserve commit history: squash only when explicitly requested, and rewrite pushed history only with explicit approval.

State which branch was pushed. Describe work as shipped or available from a normal clone only when the remote default branch contains it. When normal-clone availability matters, confirm the default branch and verify a clean checkout. Verify installation, startup, packaging, and deployment changes from a clean checkout when practical; ignored, untracked, or generated local files are not evidence.

## Parallel agents and Git worktrees

- Give each concurrent write-capable agent or task its own branch-backed worktree, created from an explicit base ref. Read-only agents may share a checkout. Keep all edits, builds, tests, and Git operations inside the assigned worktree.
- Worktrees isolate files, not host resources. Namespace collision-prone ports, databases, temporary paths, build outputs, and container/Compose resources per worktree; serialize commands when safe isolation is unavailable.
- Integrate completed branches in one designated worktree. Never modify or remove another agent's active worktree.
- If worktrees are unavailable, keep parallel agents read-only and serialize stateful work.

## Post-merge cleanup

Cleanup is part of completing PR work. When you merge a PR, or learn during an active or resumed task that its PR has merged, perform cleanup before the final handoff without waiting for a separate tidy-up request.

- Confirm the PR's merged state and target branch through the hosting service. Inspect the task's worktrees and working directories for uncommitted, untracked, ignored, and unpushed work; preserve anything still needed before cleanup. A merged PR alone does not prove that all local work is preserved.
- Stop task-owned background processes and release task-owned temporary resources. Keep shared resources and directories used by active tasks intact.
- Archive unneeded Codex-managed worktrees through the app's worktree archive tool, preserving needed ignored files separately. Remove clean, manually managed worktrees with `git worktree remove`. Remove other task-created temporary working directories only after verifying their resolved paths and that they contain no needed work; use scoped removal rather than broad filesystem cleanup.
- Delete task-owned local branches only after verifying their work is integrated or otherwise preserved. Prefer normal safe branch deletion; if it refuses (for example, after a squash merge), retain the branch and report why. Remote branch deletion requires explicit authorization or an established repository policy.
- Verify cleanup and report what was removed or archived and anything retained with its reason. If cleanup is blocked, include the concrete remaining action in the handoff; treat cleanup as complete only when every task-owned resource is removed, archived, or deliberately retained with an explained reason.

## GitHub and Node.js on native Windows

Sandboxed `gh auth status` failures are inconclusive because the host user's credentials may be unavailable to the sandbox. Continue through the host-keyring route; do not initiate `gh auth login` from that evidence alone.

- Prefer the connected GitHub app for supported repository, issue, pull-request, comment, and label operations.
- When `gh` or authenticated remote Git is required, request narrowly scoped execution outside the sandbox so it can use the host credential store.
- Keep credentials in the host keyring. Full Access, persisted `GH_TOKEN`/`GITHUB_TOKEN`, and `gh auth login --insecure-storage` are not authentication workarounds.

In native Windows PowerShell, use Node.js `.cmd` shims: `npm.cmd`, `npx.cmd`, `pnpm.cmd`, `yarn.cmd`, and repository-local launchers such as `.\node_modules\.bin\playwright.cmd`. Prefer repository scripts through `npm.cmd run <script>`. If a `.ps1` shim is blocked, switch to `.cmd`; preserve the execution policy.

## Compatibility and critical workflows

- Treat released backup, export, and persisted-data formats as compatibility contracts. Preserve readable old formats. Before making old data unreadable, obtain explicit approval, identify affected versions, and propose a migration.
- Test critical workflows spanning UI, API, and storage at the user-facing boundary; unit or API tests alone are insufficient.

## Conditional workflow references

Resolve the following paths relative to this global `AGENTS.md` file, not the project's working directory.

- **Verification:** Before creating or changing executable code, read [`agent-guidance/verification.md`](agent-guidance/verification.md). It defines the canonical command, required gates, zero-warning standard, and exceptions.
- **Investigation:** Before substantive optimisation, debugging, or performance-regression changes, read [`agent-guidance/investigation.md`](agent-guidance/investigation.md) for baselines, attempt logging, and stopping rules.

## Implementation handoff

For behavior changes, report:

- significant user-visible and technical changes;
- concrete manual tests, their expected results, and any prerequisites, commands, URLs, flags, or test data;
- automated checks and results; and
- important untested, uncertain, or out-of-scope behavior.

## Manual Testing

Every ticket requiring manual verification must provide a single documented command that prepares and launches the application in a testable state. The agent should supply the expected behaviour and a short, specific test scenario. Manual verification must not require knowledge of the application's internal tooling.
Focus manual testing on changed behavior and regression risks rather than repeating automated commands. If meaningful manual testing does not apply, say so.

## Commentary

Minimize intermediate commentary. Do not narrate routine actions, searches, file reads, edits, or test runs. Provide intermediate updates only for significant findings, blockers, important decisions, or changes to the implementation plan.
