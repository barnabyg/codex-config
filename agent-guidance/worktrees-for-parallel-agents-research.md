# Git worktrees for parallel coding agents

Research date: 2026-08-26

## Conclusion

The common first-party pattern is one branch-backed Git worktree for each concurrent session or agent that may edit files. Git defines a worktree as a separate working tree with its own `HEAD` and index while sharing the underlying repository. Anthropic's agent documentation applies that mechanism directly to parallel coding agents: each session receives a separate checkout so its edits do not touch another session's files.

That isolation boundary is narrower than a process, container, or virtual machine. A worktree separates checkout-local files, but not host-level resources such as fixed TCP ports, databases, globally configured caches or temporary paths, Docker resources, devices, or external services. Concurrent validation therefore also needs per-run resource names/paths/ports, or serialization when a resource cannot be isolated.

## Evidence from primary sources

- Git supports multiple linked working trees so more than one branch can be checked out at once. Each linked worktree has per-worktree files such as `HEAD` and `index`, but shares the rest of the repository. Git normally refuses to check out a branch that is already checked out in another worktree. [Git: `git-worktree`](https://git-scm.com/docs/git-worktree.html)
- Git recommends creating a linked worktree with `git worktree add` and removing it with `git worktree remove`; `git worktree prune` cleans stale administrative records left by missing directories. A detached worktree is suitable for throwaway experiments or testing, but branch-backed worktrees are the natural choice when an agent's changes must be retained and integrated. [Git: `git-worktree`](https://git-scm.com/docs/git-worktree.html)
- Anthropic documents worktrees as the file-isolation mechanism for parallel coding sessions and subagents. It recommends a separate worktree per session, initialization of the development environment inside each fresh checkout, and normal cleanup after the session. [Claude Code: parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees)
- Anthropic explicitly says worktrees isolate file edits while agent mechanisms coordinate the work. Its isolation checks keep edits and command working directories out of the main checkout, illustrating the useful operational rule that every command must remain rooted in the assigned worktree. [Claude Code: parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees)
- A fresh worktree contains tracked files, not ordinary untracked or ignored local configuration, so each worktree needs deliberate setup. Anthropic recommends initializing dependencies per worktree and selectively copying required ignored files such as environment configuration. [Claude Code: parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees)
- Isolation must extend beyond source files. For concrete examples, pytest says concurrent invocations need unique base temporary directories, and Docker Compose says distinct project names let multiple copies of the same application run on one host with separately identified resources. [pytest: temporary directories](https://docs.pytest.org/en/stable/how-to/tmp_path.html) and [Docker Compose: application model](https://docs.docker.com/compose/intro/compose-application-model/)

## Recommended policy

1. Give every concurrent write-capable agent or task its own branch-backed worktree. Read-only research agents may share a checkout if they do not run commands that mutate it.
2. Create every worktree from an explicit intended base ref. Put the worktree's absolute path, branch, and ownership in the delegation; do not make an agent infer them.
3. Before its first mutation, have the agent verify `git rev-parse --show-toplevel` and `git branch --show-current`. Keep all edits, dependency installation, code generation, builds, tests, and Git commands inside that worktree.
4. Treat worktrees as file isolation only. Give concurrent runs unique temp/build/cache directories, ports, database names or schemas, Docker Compose project names, and other mutable resource identifiers derived from the task or worktree. If the project cannot isolate a shared resource, run the affected commands serially.
5. Initialize dependencies and required ignored configuration in each worktree. Copy only what is needed; do not assume untracked files or secrets are present.
6. Use one designated integrator or integration worktree. Integrate completed branches there and run the canonical full verification with exclusive access to any remaining shared resources. Worktrees prevent live checkout collisions, not later semantic merge conflicts.
7. Remove a worktree only after its changes are committed, transferred, or intentionally discarded and its status is clean. Use `git worktree remove`, followed by `git worktree prune` when necessary; do not delete linked worktree directories manually.
8. If separate worktrees are unavailable, restrict parallel agents to read-only work and serialize edits, builds, tests, and other stateful commands.

## Suggested `AGENTS.md` text

```markdown
## Parallel agents and Git worktrees

- When multiple agents may edit files or run stateful build/test commands concurrently in the same repository, give each agent or task its own branch-backed Git worktree. Read-only agents may share a checkout only if their commands do not mutate it.
- Create each worktree from the explicit intended base ref and give the agent its absolute worktree path and branch. Before the first mutation, verify both with `git rev-parse --show-toplevel` and `git branch --show-current`.
- Keep all edits, dependency installation, code generation, builds, tests, and Git commands inside the assigned worktree. Do not switch branches in, modify, or run commands against another agent's worktree or the main checkout.
- Worktrees isolate checkout files, not host-level resources. For concurrent runs, use task-unique temp/build/cache directories, ports, database names or schemas, container/Compose project names, and other mutable resource identifiers. If a shared resource cannot be isolated safely, serialize the affected commands.
- Initialize each worktree's dependencies and required ignored configuration explicitly. Do not assume untracked files, generated files, local environments, or secrets exist in a fresh worktree.
- Use a designated integration worktree or agent to integrate completed branches and run the full canonical verification. Do not integrate another agent's in-progress branch.
- After work is committed or otherwise safely transferred and the worktree is clean, remove it with `git worktree remove`; use `git worktree prune` only to clean stale metadata. Do not delete linked worktree directories manually.
- If worktrees are unavailable, do not run parallel write, build, or test work in one checkout; keep parallel agents read-only and serialize stateful operations.
```

## Scope and inference

Git and agent-tool documentation directly establish separate checkout isolation, per-worktree setup, shared repository metadata, and safe cleanup. The recommendation to namespace host-level test resources is an inference from that documented boundary, supported by the pytest and Docker Compose primary documentation. The designated-integrator rule is an operational safeguard: separate worktrees prevent agents from changing each other's live files but cannot guarantee conflict-free integration of overlapping changes.
