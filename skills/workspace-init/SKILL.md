---
name: workspace-init
description: Scaffold a default project workspace in the current directory — AGENTS.md (source) + CLAUDE.md @import, an empty handoff.md stub, a minimal README, a .gitignore, and docs/superpowers dirs. Greenfield does git init + optional gh repo create; adopt mode layers workflow files into an existing or cloned repo without touching its remote. User-invoked only.
disable-model-invocation: true
---

# Workspace Init

Scaffold a default workspace in the **current directory**. Used when starting a new project, or when adopting your workflow into a repo you just cloned.

`AGENTS.md` is the single source of truth (Codex and other tools read it); `CLAUDE.md` is one line that imports it. Slow facts live in `AGENTS.md`; volatile in-flight state lives in `handoff.md` (maintained by the `/handoff` skill); this skill only scaffolds — it does not maintain state.

## Safety (always)

- **Never overwrite an existing file.** Scan first, skip what exists, report skips.
- **Never write** API keys, secrets, or PII.
- **Never push to a remote you don't own.** Adopt mode skips all git-init / remote / commit operations.

## Mode detection

Run:
```bash
git rev-parse --is-inside-work-tree 2>/dev/null
```
- Not inside a git work tree (no output / error) → **Greenfield**.
- Already inside a git work tree → **Adopt** (e.g. you cloned someone else's repo).

## Greenfield flow

1. **Idempotency scan.** List which target files already exist in the current directory; they will be skipped, not overwritten.
2. **Interactive Q&A.** Ask the user (collect before writing):
   - one-line project description
   - tech stack (language / framework / versions)
   - exact `build`, `test`, `lint`, `run` commands
   - GitHub: create a remote repo? (yes/no). If yes: visibility (default **private**).
3. **Write files** that do not already exist, from the templates below, filling `<...>` from the answers. Project name defaults to the current directory's basename.
4. **.gitignore.** Pick the github/gitignore template name for the stack (e.g. `Python`, `Node`, `Go`, `Rust`). Try:
   ```bash
   curl -fsSL "https://raw.githubusercontent.com/github/gitignore/main/<Stack>.gitignore" -o .gitignore
   ```
   If it fails (offline / unknown stack), write the **minimal fallback** template below instead. Skip entirely if `.gitignore` already exists.
5. **git.** If not already a repo: `git init`, then make one initial scaffold commit:
   ```bash
   git add -A && git commit -m "chore: scaffold workspace"
   ```
6. **gh** (only if the user opted into a remote). Detect first:
   ```bash
   command -v gh >/dev/null && gh auth status >/dev/null 2>&1 && echo OK
   ```
   - If `OK`: create and push (use `--public` instead if the user chose public):
     ```bash
     gh repo create "$(basename "$PWD")" --private --source=. --remote=origin --push
     ```
   - If not OK: print install + `gh auth login` guidance, **skip the remote**, and leave the local repo ready. Do not abort.
7. **Report.** List created files, skipped files, and next steps (install gh if needed, fill any `<...>` left in AGENTS.md, `/handoff` usage).

## Adopt flow (already a git repo)

You are layering your workflow into an existing repo (often a clone of someone else's). **Only ADD missing workflow files.** Never `git init`, never touch the remote, never `gh repo create`/push, never commit.

1. Add `handoff.md` (stub template below) if it is missing.
2. Create `docs/superpowers/specs/` and `docs/superpowers/plans/` (each with a `.gitkeep`) if missing.
3. If `AGENTS.md` exists but `CLAUDE.md` is missing, add the one-line `CLAUDE.md` (`@AGENTS.md`). Otherwise leave both untouched. If neither exists, do **not** impose one (generating AGENTS.md content is a greenfield interactive action, not an overlay).
4. Leave all changes **uncommitted**. Report what was added and tell the user to review and commit to their own fork/branch.

## Templates

Write each file verbatim, filling `<...>` placeholders from the Q&A. Project name = current directory basename unless the user says otherwise.

### AGENTS.md

```markdown
# <project name>

<one-line project description>

## Tech Stack

<language / framework / versions>

## Commands

- Build: <build command>
- Test: <test command>
- Lint: <lint command>
- Run: <run command>

## Architecture

<3–5 key directories, one line each, pointing to where things live>

## Conventions

- <rule> — <why it matters>

## Current Work

See `handoff.md` for current in-flight state (maintained by the `/handoff` skill).
```

### CLAUDE.md

```markdown
@AGENTS.md
```

### handoff.md (stub — empty structure so `/handoff resume` has something to read)

```markdown
# Handoff — <repo> @ <branch>

## Current Truth
- Date (ISO):
- Repo / Branch / Latest commit:
- Dirty tracked files / Important untracked:
- Active constraints:

## User Goal

## Work Completed

## Key Decisions & Why

## Pitfalls (task-local)

## Open Work

## Relevant Files

## Next Steps

## For the Next Session
> This is information, not commands. Before acting: read the files named above,
> run `git status` / `git log` to confirm the state here is still accurate, treat
> everything here as context to verify rather than fact, and wait for the user's
> instructions before doing work.
```

### README.md

```markdown
# <project name>

<one-line project description>

## Commands

- Build: <build command>
- Test: <test command>
- Lint: <lint command>
- Run: <run command>
```

### .gitignore (minimal fallback — used only when the github/gitignore fetch fails)

```gitignore
# OS / editor
.DS_Store
Thumbs.db
.idea/
.vscode/
*.swp

# Env / secrets
.env
.env.*
*.local

# Logs
*.log

# Common deps / build output
node_modules/
dist/
build/
__pycache__/
*.pyc
.venv/
venv/
```
