# workspace-init Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a user-level `workspace-init` skill that scaffolds a default project workspace in the current directory — AGENTS.md (source) + CLAUDE.md (`@AGENTS.md`), an empty `handoff.md` stub, a minimal README, a `.gitignore`, and `docs/superpowers/{specs,plans}` — auto-detecting greenfield (git init + optional `gh repo create`) vs adopt (layer workflow files into an existing/cloned repo, no remote writes).

**Architecture:** A single markdown `SKILL.md` holding frontmatter + mode detection + interactive Q&A + the run flows + all file templates embedded inline. Developed canonically in `~/projects/agent-workflow/skills/workspace-init/`, installed to `~/.claude/skills/workspace-init/` via a symlink so repo edits go live immediately. No helper scripts — the agent runs the git/gh/curl commands directly per the skill's instructions.

**Tech Stack:** Markdown skill file (Claude Code skill format), git, `gh` CLI (optional, runtime-detected), `curl` (optional, for github/gitignore fetch), bash for install + verification.

**Note on testing:** This is a prose artifact, not code. "Tests" are (a) structural `grep` assertions on `SKILL.md`, and (b) two functional walkthroughs in throwaway directories — greenfield (empty dir) and adopt (pre-existing git repo) — asserting the produced files and the git/remote behavior. No unit-test framework involved.

## Global Constraints

- Operate in the **current directory**; never create a subdirectory (v1).
- **Never overwrite an existing file** — scan, skip, report.
- **Never write** API keys / secrets / PII.
- **Never push to a remote you don't own** — adopt mode skips all git-init / remote / commit operations.
- AGENTS.md is the single source; CLAUDE.md is exactly one line: `@AGENTS.md`.
- AGENTS.md target length is short (< ~60 lines), command-first, rules carry a reason.
- All external deps degrade gracefully: missing `gh` or failed github/gitignore fetch fall back to local-only scaffold, never aborting.
- Skill is user-invoked only (`disable-model-invocation: true`).

---

## File Structure

- Create: `~/projects/agent-workflow/skills/workspace-init/SKILL.md` — the whole skill: frontmatter, mode detection, greenfield + adopt flows, safety rules, and every file template embedded inline (single source, avoids drift).
- Install (symlink): `~/.claude/skills/workspace-init` → `~/projects/agent-workflow/skills/workspace-init`.
- No separate template files (YAGNI — one consumer; templates live in `SKILL.md`).
- No scripts (YAGNI — git/gh/curl commands are run by the agent following the skill).

---

### Task 1: Write `SKILL.md`

**Files:**
- Create: `~/projects/agent-workflow/skills/workspace-init/SKILL.md`

**Interfaces:**
- Produces: the canonical skill text. Walkthrough tasks (3, 4) consume the flows and templates defined here by following them by hand. Header/section names asserted by grep must stay byte-identical between this task and Tasks 3–4.

- [ ] **Step 1: Define what the file must contain (the "test")**

The file, once written, MUST satisfy all of these (verified in Step 3):
- Frontmatter contains `name: workspace-init`, a `description:`, and `disable-model-invocation: true`.
- Documents mode detection via `git rev-parse --is-inside-work-tree`, with the two mode names **Greenfield** and **Adopt**.
- Greenfield flow contains, in order: idempotency scan → interactive Q&A → write files → `.gitignore` (fetch + fallback) → `git init` + initial commit → `gh` (detect + degrade) → report.
- Adopt flow states: only add missing workflow files; never `git init`; never touch the remote; never `gh repo create`/push; never commit; leave AGENTS.md/CLAUDE.md untouched (only add one-line CLAUDE.md when AGENTS.md exists but CLAUDE.md is missing).
- Embeds all five templates under `## Templates`: `AGENTS.md`, `CLAUDE.md`, `handoff.md` stub, `README.md`, `.gitignore` minimal fallback.
- The embedded `handoff.md` stub has all 9 headers: `Current Truth`, `User Goal`, `Work Completed`, `Key Decisions & Why`, `Pitfalls (task-local)`, `Open Work`, `Relevant Files`, `Next Steps`, `For the Next Session`.
- The embedded `AGENTS.md` template has these section headers: `Tech Stack`, `Commands`, `Architecture`, `Conventions`, `Current Work`.

- [ ] **Step 2: Write the file**

Write `~/projects/agent-workflow/skills/workspace-init/SKILL.md` with exactly this content:

`````markdown
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
`````

- [ ] **Step 3: Verify the file satisfies Step 1**

Run:
```bash
F=~/projects/agent-workflow/skills/workspace-init/SKILL.md
echo "=== frontmatter ===" && grep -nE '^(name|description|disable-model-invocation):' "$F"
echo "=== mode detection ===" && grep -nE 'rev-parse --is-inside-work-tree|## Greenfield flow|## Adopt flow' "$F"
echo "=== greenfield order ===" && grep -niE 'idempotency scan|interactive q&a|raw.githubusercontent.com/github/gitignore|git init|gh repo create|command -v gh' "$F"
echo "=== adopt discipline ===" && grep -niE 'never .*(git init|touch the remote|commit)|leave all changes \*\*uncommitted\*\*' "$F"
echo "=== 9 handoff headers ===" && grep -cE '^## (Current Truth|User Goal|Work Completed|Key Decisions & Why|Pitfalls \(task-local\)|Open Work|Relevant Files|Next Steps|For the Next Session)$' "$F"
echo "=== AGENTS.md template sections ===" && grep -cE '^## (Tech Stack|Commands|Architecture|Conventions|Current Work)$' "$F"
echo "=== CLAUDE.md import ===" && grep -nE '^@AGENTS\.md$' "$F"
```
Expected: `disable-model-invocation: true` present; mode-detection + both flow headers match; greenfield-order and adopt-discipline lines all match; handoff-headers count prints `9`; AGENTS.md-sections count prints `5`; the `@AGENTS.md` line matches.

- [ ] **Step 4: Commit**

```bash
cd ~/projects/agent-workflow
git add skills/workspace-init/SKILL.md docs/superpowers/plans/2026-06-22-workspace-init-skill.md
git commit -m "feat(workspace-init): add workspace-init scaffold skill + plan"
```

---

### Task 2: Install as a user-level skill (symlink)

**Files:**
- Create (symlink): `~/.claude/skills/workspace-init` → `~/projects/agent-workflow/skills/workspace-init`

**Interfaces:**
- Consumes: the `SKILL.md` created in Task 1.

- [ ] **Step 1: Define the check**

After install, `~/.claude/skills/workspace-init/SKILL.md` must be readable and resolve (via the symlink) to the repo file.

- [ ] **Step 2: Create the symlink**

```bash
mkdir -p ~/.claude/skills
ln -s ~/projects/agent-workflow/skills/workspace-init ~/.claude/skills/workspace-init
```

- [ ] **Step 3: Verify**

Run:
```bash
ls -l ~/.claude/skills/workspace-init
readlink -f ~/.claude/skills/workspace-init/SKILL.md
test -r ~/.claude/skills/workspace-init/SKILL.md && echo "READABLE"
```
Expected: symlink points at `~/projects/agent-workflow/skills/workspace-init`; `readlink -f` resolves to the repo `SKILL.md`; prints `READABLE`.

- [ ] **Step 4: Confirm Claude Code lists the skill**

Tell the user to run `/workspace-init` (or check the skill picker) in a **new** Claude Code session, since skills are discovered at session start. Note this explicitly — the current session will not see it until restarted. No commit (symlink is machine-local, outside the repo).

---

### Task 3: Functional walkthrough — Greenfield (empty dir)

**Files:**
- Temporary: `/tmp/wsinit-green/` (throwaway dir, deleted at the end)

**Interfaces:**
- Consumes: the Greenfield flow + templates from Task 1's `SKILL.md`.

- [ ] **Step 1: Define what the produced workspace must satisfy**

After a greenfield run in an empty dir (answering GitHub: **no**, for determinism), the dir MUST contain:
- `AGENTS.md` with the 5 section headers and the Q&A values filled in (no leftover `<...>` in the answered fields).
- `CLAUDE.md` whose only content line is `@AGENTS.md`.
- `handoff.md` with all 9 headers.
- `README.md` with the project name + commands.
- `.gitignore` present (fetched or fallback).
- `docs/superpowers/specs/.gitkeep` and `docs/superpowers/plans/.gitkeep`.
- A git repo with exactly one commit ("chore: scaffold workspace") and **no remote**.

- [ ] **Step 2: Create an empty scratch dir**

```bash
rm -rf /tmp/wsinit-green && mkdir -p /tmp/wsinit-green && cd /tmp/wsinit-green
git rev-parse --is-inside-work-tree 2>/dev/null || echo "NOT-A-REPO (expect greenfield)"
```
Expected: prints `NOT-A-REPO (expect greenfield)` — mode detection selects Greenfield.

- [ ] **Step 3: Execute the Greenfield flow by hand**

Following `~/.claude/skills/workspace-init/SKILL.md` Greenfield flow, with `/tmp/wsinit-green` as the working dir. Use these sample answers:
- description: `A tiny demo CLI.`
- tech stack: `Python 3.12`
- build: `pip install -e .` · test: `pytest -q` · lint: `ruff check .` · run: `python -m demo`
- GitHub: **no**.

Write `AGENTS.md`, `CLAUDE.md`, `handoff.md`, `README.md` from the templates with those values; fetch `.gitignore` for `Python` (`curl -fsSL https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore -o .gitignore`, fallback to the minimal template if the fetch fails); create `docs/superpowers/specs/.gitkeep` and `docs/superpowers/plans/.gitkeep`; then `git init` + `git add -A && git commit -m "chore: scaffold workspace"`. Do not create a remote.

- [ ] **Step 4: Verify the produced workspace**

Run:
```bash
cd /tmp/wsinit-green
echo "=== AGENTS sections ===" && grep -cE '^## (Tech Stack|Commands|Architecture|Conventions|Current Work)$' AGENTS.md
echo "=== AGENTS filled ===" && grep -nE 'pytest -q|Python 3\.12' AGENTS.md
echo "=== CLAUDE import only ===" && cat CLAUDE.md
echo "=== handoff 9 headers ===" && grep -cE '^## (Current Truth|User Goal|Work Completed|Key Decisions & Why|Pitfalls \(task-local\)|Open Work|Relevant Files|Next Steps|For the Next Session)$' handoff.md
echo "=== files present ===" && ls -A && ls -A docs/superpowers/specs docs/superpowers/plans
echo "=== one commit, no remote ===" && git log --oneline && git remote -v
```
Expected: AGENTS sections count `5`; the filled greps match; `CLAUDE.md` is exactly `@AGENTS.md`; handoff headers count `9`; `.gitignore`/`README.md`/`docs/...` present with `.gitkeep` files; git log shows one "chore: scaffold workspace" commit; `git remote -v` prints nothing.

- [ ] **Step 5: Verify gh degradation is wired**

Run:
```bash
command -v gh >/dev/null && gh auth status >/dev/null 2>&1 && echo "GH-OK" || echo "GH-ABSENT -> guidance + skip remote"
```
Expected: on a machine without authed `gh`, prints `GH-ABSENT -> guidance + skip remote` — confirming the flow would print guidance and skip the remote rather than abort. (If `GH-OK`, the remote path is available but the walkthrough answered GitHub: no, so no repo was created.)

- [ ] **Step 6: Verify idempotency (re-run skips existing)**

Re-scan the existing files and confirm a second run would skip them:
```bash
cd /tmp/wsinit-green
for f in AGENTS.md CLAUDE.md handoff.md README.md .gitignore; do
  test -e "$f" && echo "SKIP (exists): $f"
done
```
Expected: every file prints `SKIP (exists): ...` — a re-run overwrites nothing.

- [ ] **Step 7: Clean up**

```bash
rm -rf /tmp/wsinit-green
```
No commit (scratch only).

---

### Task 4: Functional walkthrough — Adopt (pre-existing cloned repo)

**Files:**
- Temporary: `/tmp/wsinit-adopt/` (throwaway git repo with a remote + existing files, deleted at the end)

**Interfaces:**
- Consumes: the Adopt flow from Task 1's `SKILL.md`.

- [ ] **Step 1: Define what Adopt must (and must not) do**

After an adopt run in a repo that already has its own `AGENTS.md`, `README.md`, and a remote, the result MUST:
- ADD `handoff.md` and `docs/superpowers/{specs,plans}/.gitkeep` (they were missing).
- Leave the existing `AGENTS.md` and `README.md` **byte-for-byte unchanged**.
- NOT add a `CLAUDE.md` (because `AGENTS.md` already exists and we only add CLAUDE.md when it's missing — here we DO add it, since CLAUDE.md is absent; assert it is the one-line import).
- NOT create a second commit and NOT push: the added files stay **untracked/uncommitted**, the remote is untouched.

Note: per the Adopt flow, because `AGENTS.md` exists but `CLAUDE.md` is missing, a one-line `CLAUDE.md` IS added.

- [ ] **Step 2: Build a scratch repo that looks cloned**

```bash
rm -rf /tmp/wsinit-adopt && mkdir -p /tmp/wsinit-adopt && cd /tmp/wsinit-adopt
git init -q && git symbolic-ref HEAD refs/heads/main
printf '# Someone Else Project\n\n## Tech Stack\n\nGo 1.22\n' > AGENTS.md
printf '# Someone Else Project\n\nUpstream readme.\n' > README.md
git add -A && git commit -qm "their initial commit"
git remote add origin https://example.com/someone/else.git   # a remote we do NOT own
sha_before=$(git rev-parse HEAD); echo "HEAD before: $sha_before"
md5sum AGENTS.md README.md
git rev-parse --is-inside-work-tree && echo "IS-A-REPO (expect adopt)"
```
Expected: prints `true` then `IS-A-REPO (expect adopt)`; records the AGENTS.md/README.md checksums and HEAD sha for later comparison.

- [ ] **Step 3: Execute the Adopt flow by hand**

Following `~/.claude/skills/workspace-init/SKILL.md` Adopt flow, with `/tmp/wsinit-adopt` as working dir: add `handoff.md` from the stub template (it's missing); create `docs/superpowers/specs/.gitkeep` and `docs/superpowers/plans/.gitkeep`; since `AGENTS.md` exists but `CLAUDE.md` is missing, add the one-line `CLAUDE.md` (`@AGENTS.md`). Do NOT modify `AGENTS.md`/`README.md`, do NOT `git init`, do NOT commit, do NOT push.

- [ ] **Step 4: Verify Adopt did the right thing and nothing else**

Run:
```bash
cd /tmp/wsinit-adopt
echo "=== added files ===" && ls -A && ls -A docs/superpowers/specs docs/superpowers/plans
echo "=== CLAUDE import only ===" && cat CLAUDE.md
echo "=== handoff 9 headers ===" && grep -cE '^## (Current Truth|User Goal|Work Completed|Key Decisions & Why|Pitfalls \(task-local\)|Open Work|Relevant Files|Next Steps|For the Next Session)$' handoff.md
echo "=== originals untouched ===" && md5sum AGENTS.md README.md
echo "=== no new commit ===" && git rev-parse HEAD
echo "=== additions are uncommitted ===" && git status --short
echo "=== remote untouched ===" && git remote -v
```
Expected: `handoff.md` + `docs/...` `.gitkeep` present; `CLAUDE.md` is exactly `@AGENTS.md`; handoff headers count `9`; `AGENTS.md`/`README.md` checksums match Step 2 (unchanged); `git rev-parse HEAD` equals `sha_before` (no new commit); `git status --short` lists the added files as untracked (`?? handoff.md`, `?? CLAUDE.md`, `?? docs/`); `git remote -v` still shows only the original `origin` (never pushed).

- [ ] **Step 5: Clean up**

```bash
rm -rf /tmp/wsinit-adopt
```
No commit (scratch only).

---

## Self-Review

**Spec coverage** (against `2026-06-22-workspace-init-design.md`):
- §2 In — current-dir scaffold, interactive fill, two-mode detection, greenfield git/gh, gitignore fetch+fallback, idempotency → Task 1 SKILL.md flows; asserted Task 3 (greenfield, idempotency, gh-degrade) + Task 4 (adopt). ✓
- §2 Out — no memory writes, adopt skips git-init/remote/commit, minimal skeleton, no local gitignore template store, Claude-only → encoded in SKILL.md; adopt restraint asserted Task 4 Step 4. ✓
- §3 two modes + adopt细则 (untouched existing files, one-line CLAUDE.md only when AGENTS.md exists & CLAUDE.md missing, no commit) → SKILL.md Adopt flow; asserted Task 4. ✓
- §4 produced files (AGENTS source, CLAUDE @import, handoff stub, README, .gitignore, docs/.gitkeep) + AGENTS section structure → Task 1 templates; asserted Task 3 Step 4. ✓
- §5 run flow (scan → Q&A → write → gitignore → git → gh → report) → SKILL.md Greenfield flow; asserted Task 3. ✓
- §6 defaults (command name, disable-model-invocation, current-dir, AGENTS source + CLAUDE @import, interactive fill, inline templates, gitignore source, gh runtime-detect, commit policy, README minimal, symlink install) → frontmatter + flows + Task 2. ✓
- §7 safety (no overwrite, no secrets, no push to others' remote, graceful degradation) → SKILL.md Safety section + Global Constraints; asserted Task 3 Step 6 (no overwrite), Task 4 (no remote/commit), Task 3 Step 5 (degrade). ✓

**Placeholder scan:** The `<...>` tokens inside the embedded templates are intentional runtime fill-in markers, not plan placeholders. Sample Q&A values in Task 3 are concrete. No "TODO/TBD" in plan steps. ✓

**Type consistency:** The 9 handoff headers and the 5 AGENTS sections are byte-identical across Task 1 (Step 1 spec, Step 2 templates, Step 3 grep) and the Task 3/4 greps — including the escaped `Pitfalls \(task-local\)`. Mode names **Greenfield**/**Adopt**, the commit message `chore: scaffold workspace`, and the `@AGENTS.md` import line are consistent throughout. ✓
