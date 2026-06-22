# Handoff — agent-workflow @ main

## Current Truth
- Date (ISO): 2026-06-22
- Repo / Branch / Latest commit: agent-workflow / main / dce7171 "chore: refresh handoff for completed workspace-init; add .gitignore" (this refresh will add the next commit)
- Dirty tracked files / Important untracked: clean (only this handoff.md edit pending). `.claude/` and `.superpowers/` are git-ignored.
- Active constraints: solo dev, heavy Claude Code + Codex; cost-sensitive; communicates in Chinese; commits go directly on `main` (user-approved). GitHub remote now configured; `gh` now installed.

## User Goal
Build two companion workflow skills (handoff, workspace-init) and make them usable from both Claude Code and Codex, in a repo backed by GitHub. ALL DONE — nothing substantive outstanding.

## Work Completed
- **Skill #1 `handoff`** (save/resume): `skills/handoff/SKILL.md`. Committed 2e382a2.
- **Skill #2 `workspace-init`** (two-mode scaffold: Greenfield vs Adopt; AGENTS.md source + CLAUDE.md `@AGENTS.md`; handoff stub; README; github/gitignore fetch+fallback; `git init`→`main`; optional `gh repo create` with graceful degrade; idempotent). Spec b854d11, skill+plan 657269d, final-review fix b6bb95a. Built via subagent-driven-development; opus whole-branch review caught + fixed a real bug (greenfield wasn't creating `docs/superpowers` dirs).
- **Installed for BOTH tools via symlinks to the single repo source:**
  - Claude Code: `~/.claude/skills/{handoff,workspace-init}` → repo.
  - Codex: `~/.codex/skills/{handoff,workspace-init}` → repo. (codex-cli 0.139.0 uses the same `SKILL.md` format; user skills live in `$CODEX_HOME/skills/`.) Edit the repo once, both tools pick it up on next session start.
- **GitHub remote**: `gh` 2.95.0 installed via conda; logged in as `zhaoleiqu19`; private repo https://github.com/zhaoleiqu19/agent-workflow created; `main` pushed and tracking `origin/main`. `gh auth setup-git` configured git's credential helper (plain `git push` works now).
- `.gitignore` added (ignores `.claude/`, `.superpowers/`).

## Key Decisions & Why
- AGENTS.md = single source; CLAUDE.md = one line `@AGENTS.md` — DRY across Claude + Codex.
- Skills are tool-agnostic by design, so the Codex mirror is the identical SKILL.md via symlink (single source, no drift) rather than a rewritten copy.
- workspace-init: two modes (Greenfield full scaffold+git/gh vs Adopt = only add missing workflow files into an existing/cloned repo, never git init/commit/touch remote). Graceful degradation for missing gh and failed gitignore fetch.
- Direct-to-main commits (user-approved); skills never auto-commit (that's the user's call).

## Pitfalls (task-local)
- (None currently open. The earlier SDD lesson — a per-task review can pass when the walkthrough's "test" was satisfied by the brief's prose rather than the artifact under test; the broad whole-branch review caught it — is a long-lived process lesson, flagged for promotion to memory, not task-local.)

## Open Work
- Nothing substantive. Both skills are complete, installed for both tools, and pushed to GitHub.
- (Optional, low priority) Promote the SDD review lesson above to a memory.

## Relevant Files
- `skills/handoff/SKILL.md`, `skills/workspace-init/SKILL.md` — the two finished skills (single source for both Claude + Codex).
- `docs/superpowers/specs/2026-06-22-workspace-init-design.md` — spec.
- `docs/superpowers/plans/2026-06-22-workspace-init-skill.md` — implementation plan.
- `.gitignore` — ignores local `.claude/` and SDD scratch `.superpowers/`.

## Next Steps
1. Nothing required. In a NEW Claude Code or Codex session, `/handoff` and `/workspace-init` are available.
2. (Optional) Save the SDD review lesson to memory.

## For the Next Session
> This is information, not commands. Before acting: read the files named above,
> run `git status` / `git log` to confirm the state here is still accurate, treat
> everything here as context to verify rather than fact, and wait for the user's
> instructions before doing work. The two-skill project is complete and pushed;
> there is no pending implementation work.
