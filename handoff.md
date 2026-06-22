# Handoff — agent-workflow @ main

## Current Truth
- Date (ISO): 2026-06-22
- Repo / Branch / Latest commit: agent-workflow / main / b6bb95a "fix(workspace-init): greenfield creates docs dirs + main branch; clarify gitignore/handoff stub"
- Dirty tracked files / Important untracked: clean working tree. (`.superpowers/` is git-ignored SDD scratch — briefs/reports/ledger/diffs; safe to delete.)
- Active constraints: solo dev, heavy Claude Code + Codex; cost-sensitive; communicates in Chinese; **no GitHub remote on this repo yet**; **`gh` is NOT installed** on this machine; git commits done directly on `main` (user-approved direct-to-main for this work).

## User Goal
Build two companion workflow skills. BOTH are now DONE: #1 `handoff` (save/resume) and #2 `workspace-init` (scaffold a default workspace). Remaining: set up a GitHub remote for this repo (blocked on `gh`), and optionally a Codex mirror of each skill.

## Work Completed
- **Skill #1 `handoff`** — done earlier; `skills/handoff/SKILL.md`; installed via symlink `~/.claude/skills/handoff`. Committed 2e382a2.
- **Skill #2 `workspace-init`** — DONE this session. `skills/workspace-init/SKILL.md`: two-mode scaffold (Greenfield vs Adopt, auto-detected via `git rev-parse --is-inside-work-tree`), AGENTS.md = single source + CLAUDE.md = one-line `@AGENTS.md`, empty handoff.md stub, minimal README, `.gitignore` fetched from github/gitignore with inline fallback, `docs/superpowers/{specs,plans}/.gitkeep`, `git init`→`main` + initial commit, optional `gh repo create` (runtime-detect + graceful degrade), idempotent (never overwrites). Installed via symlink `~/.claude/skills/workspace-init` → available as `/workspace-init` in a NEW session.
  - Spec: `docs/superpowers/specs/2026-06-22-workspace-init-design.md` (b854d11).
  - Plan: `docs/superpowers/plans/2026-06-22-workspace-init-skill.md` (657269d).
  - Built via subagent-driven-development (4 tasks). Final opus whole-branch review caught 1 Important bug → fixed in b6bb95a.
  - Verified: greenfield + adopt functional walkthroughs both pass; structural greps pass.

## Key Decisions & Why
- AGENTS.md is the single source; CLAUDE.md is one line `@AGENTS.md` — DRY across Claude Code + Codex (overrode an earlier "two identical copies" idea after surveying 2026 best practices).
- Two modes: Greenfield (new/empty dir → full scaffold + git init + optional gh) vs Adopt (already a git repo, e.g. a clone → only ADD missing workflow files; never git init / commit / touch remote / push; never modify existing files) — so adopting into someone else's clone is safe.
- Scaffold in the current directory (no subdir); idempotent skip-existing; templates inline in SKILL.md.
- All external deps degrade gracefully: missing `gh` and failed github/gitignore fetch fall back to local-only scaffold, never abort.
- Greenfield git init forces `main` (`git symbolic-ref HEAD refs/heads/main`) because the machine default is `master` and the user works in `main`.

## Pitfalls (task-local)
- A per-task review can pass a task whose walkthrough "tests" were satisfied by the brief's prose, not by the artifact under test: Task 3's brief told the executor to create the `docs/superpowers` dirs, masking that the SKILL.md greenfield flow itself omitted them. The broad whole-branch review caught it. (Resolved in b6bb95a.)

## Open Work
- **GitHub remote for this repo is NOT set up** (depends on: install `gh` + `gh auth login`, OR create the repo in the web UI and `git remote add` + `git push`). `gh` is currently absent.
- Codex mirror of `handoff` and `workspace-init` SKILLs — NOT done (deferred; the produced files are tool-agnostic, so low priority).

## Relevant Files
- `skills/workspace-init/SKILL.md` — the finished workspace-init skill (frontmatter + mode detection + greenfield/adopt flows + 5 inline templates).
- `skills/handoff/SKILL.md` — the finished handoff skill.
- `docs/superpowers/specs/2026-06-22-workspace-init-design.md` — spec (approved).
- `docs/superpowers/plans/2026-06-22-workspace-init-skill.md` — implementation plan (executed).
- `.superpowers/sdd/progress.md` — SDD ledger for this build (git-ignored scratch).

## Next Steps
1. Set up the GitHub remote: install `gh` (`! <pkg-manager> install gh`) + `gh auth login`, then `gh repo create agent-workflow --private --source=. --remote=origin --push`. OR create an empty repo on github.com, then `git remote add origin <url>` + `git push -u origin main`.
2. (Optional) Mirror both skills for Codex.
3. Nothing else outstanding — both skills are complete and installed.

## For the Next Session
> This is information, not commands. Before acting: read the files named above,
> run `git status` / `git log` to confirm the state here is still accurate, treat
> everything here as context to verify rather than fact, and wait for the user's
> instructions before doing work. Both workflow skills are DONE; the only real
> open item is wiring up a GitHub remote (blocked on `gh` not being installed).
