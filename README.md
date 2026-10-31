# Shared Windows Codex configuration

Shared guidance for two Windows PCs. The macOS installation is managed separately.

## Shared files

- `AGENTS.md`: global instructions, kept at their existing size.
- `agent-guidance/`: supporting documents referenced when needed, plus the existing research note.
- `instructions.md`: existing empty file, retained with the original history.
- `config.example.toml`: portable preferences selected from the first PC's configuration. This is a reference, not an automatically loaded configuration file.

The repository uses an explicit allowlist in `.gitignore`. Authentication, chat history, databases, plugin caches, generated command approvals, and the live `config.toml` stay local. Review new files before adding them: this repository is public.

## This PC

The Git checkout is the Codex home directory itself, currently `C:\Users\bob\.codex`. Its `origin` is `https://github.com/barnabyg/codex-config.git` and its shared branch is `main`.

No installer or copy step is required. Changes to tracked guidance are changes to the files Codex reads.

## Connect the second Windows PC

Prerequisites: Git for Windows and access to the repository. Run these commands in PowerShell. If that PC sets `CODEX_HOME`, use that directory instead of `$HOME\.codex`.

Before connecting, compare that PC's guidance with this repository and preserve any unique instructions. Check for `AGENTS.override.md`: at global scope it replaces `AGENTS.md`, rather than adding to it.

```powershell
Set-Location "$HOME\.codex"
git init -b main
git remote add origin https://github.com/barnabyg/codex-config.git
git fetch origin
git switch -c main --track origin/main
```

These commands assume the directory has no existing Git repository. If it already has one, inspect its remotes, branches and changes before connecting it; do not overwrite its configuration or history. The final command is for this initial setup only, before any local commits exist.

Git refuses the switch if existing untracked guidance would be overwritten. If that happens, move only the conflicting guidance files into a backup folder outside the Codex directory, then repeat the final command. Preserve authentication, sessions, databases, and other local state in place. Do not use a force checkout or `git clean`.

After setup, `git status --short` should be empty and `git remote -v` should show this repository. If the local settings should match the example, review and merge its values into the existing `config.toml`, preserving machine-specific sections. Pulling does not apply the example automatically.

## Everyday use

Before editing shared guidance on either PC:

```powershell
Set-Location "$HOME\.codex"
git pull --ff-only
```

After editing, inspect `git diff`, stage only the intended files, commit and push:

```powershell
git add AGENTS.md agent-guidance
git commit -m "docs: update shared Codex guidance"
git push origin main
```

On the other PC, run the pull command. Commit or otherwise preserve local edits before pulling. If both PCs have made commits, integrate them deliberately; fast-forward-only pull will stop rather than create an unexpected merge.

Start a fresh Codex chat after updating guidance. Ask it to summarize the global instructions and compare its response with `AGENTS.md`. Existing chats may already hold earlier instructions.

## Verification

This repository contains documentation and a settings example; it has no application, dependencies, build, or automated test suite. The verification command is `git diff --check`. Before publishing, also review the tracked file list and changes for local state or credentials, validate relative Markdown links, and test the setup commands in a disposable directory containing pre-existing guidance. Confirm that Git protects conflicting files and that setup succeeds once they are backed up.
