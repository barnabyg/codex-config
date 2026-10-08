# Global Codex guidance

## Commits and publishing

When a task includes commits:

- Create one commit per implementation ticket or code-review finding. Keep unrelated work separate and stage selectively when work overlaps.
- Each commit must pass its focused tests. Run full verification before pushing the complete series.
- A passing full verification may satisfy both handoff and pre-push requirements when the verified tree is identical to the tree being pushed.
- Ask before combining technically inseparable tickets or findings.
- Preserve commit history: squash only when explicitly requested, and rewrite pushed history only with explicit approval.

State which branch was pushed. Describe work as shipped or available from a normal clone only when the remote default branch contains it. When normal-clone availability matters, confirm the default branch and verify a clean checkout. Verify installation, startup, packaging, and deployment changes from a clean checkout when practical; ignored, untracked, or generated local files are not evidence.

## Parallel agents and Git worktrees

- Give each concurrent write-capable agent or task its own branch-backed worktree, created from an explicit base ref. Read-only agents may share a checkout. Keep all edits, builds, tests, and Git operations inside the assigned worktree.
- Worktrees isolate files, not host resources. Namespace collision-prone ports, databases, temporary paths, build outputs, and container/Compose resources per worktree; serialize commands when safe isolation is unavailable.
- Integrate completed branches and run full verification in one designated worktree. Never modify or remove another agent's active worktree; remove clean, preserved worktrees with `git worktree remove`.
- If worktrees are unavailable, keep parallel agents read-only and serialize stateful work.

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

- **Verification:** Before creating or changing executable code, read [`agent-guidance/verification.md`](agent-guidance/verification.md). It defines the canonical command, required gates, zero-warning standard, and exceptions.
- **Live dashboard:** When creating a software project with automated verification, or materially changing an existing build or test workflow, also read [`agent-guidance/verification-dashboard.md`](agent-guidance/verification-dashboard.md).
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
