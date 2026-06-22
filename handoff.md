# Handoff — agent-workflow @ main

## Current Truth
- Date (ISO): 2026-06-22
- Repo / Branch / Latest commit: agent-workflow / main / 2e382a2 "feat(handoff): add handoff save/resume skill + spec + plan"
- Dirty tracked files / Important untracked: clean working tree (this `handoff.md` is new/untracked)
- Active constraints: solo dev, heavy Claude Code + Codex; cost-sensitive (fork/minimal over from-scratch); communicates in Chinese; no GitHub remote on this repo yet; git commits done when user asks (auto-commit was approved for the handoff build).

## User Goal
Build two companion workflow skills. Skill #1 (handoff save/resume) is DONE. Next: design and build skill #2 — `workspace-init` (describe a repo in a few sentences → scaffold a default structure; AGENTS.md + identical CLAUDE.md + empty handoff.md; auto-create the GitHub repo via `gh`).

## Work Completed
- Built skill #1 `handoff` (save + resume). Source: `skills/handoff/SKILL.md`. Verified: frontmatter incl. `disable-model-invocation: true`, both modes, 9/9 handoff.md headers, save→resume walkthrough in a scratch repo (git-grounded, no git mutations by the skill, resume reports-and-stops). Committed as 2e382a2.
- Installed it user-level via symlink: `~/.claude/skills/handoff` -> `~/projects/agent-workflow/skills/handoff`. Visible as `/handoff` only in NEW Claude Code sessions (skills load at startup).
- Wrote spec + plan under `docs/superpowers/`.

## Key Decisions & Why
- handoff.md = volatile current state, carry-forward OVERWRITE (not append); git keeps history. Append causes bloat/contradiction.
- Pitfall routing: task-local pitfalls stay in handoff.md; repo-level/long-lived lessons go to MEMORY (skill only flags, never auto-writes memory).
- handoff skill is read-only on git (status/diff/log), writes only handoff.md, never commits/pushes/creates repos — that belongs to workspace-init.
- v1 handoff = manual trigger only (no PreCompact/SessionEnd hook); single file, last-writer-wins; Claude Code only (Codex mirror later).
- One skill = one folder + a fixed-name `SKILL.md`; identity comes from folder name + `name:` frontmatter. Second skill = sibling folder `skills/workspace-init/SKILL.md` + its own symlink.
- AGENTS.md and CLAUDE.md will be IDENTICAL copies (not `@import`). We do NOT reuse the old project-level AGENTS.md in ovod_objects365_research.

## Pitfalls (task-local)
- `/handoff` is NOT available in a session that started before the skill was installed — skills load at session start. To save from such a session, run the steps manually; resume works in a fresh session.
<!-- task-local only; carry forward what still holds, drop what is resolved.
     Repo-level / long-lived lessons do NOT go here — save flags them for memory. -->

## Open Work
- workspace-init skill is NOT yet designed or built. It needs its own cycle: brainstorm → spec → plan → implement.
- Open design questions for workspace-init (carried from discussion):
  - CLAUDE.md/AGENTS.md content/structure is NOT yet decided. User wants to FIRST survey how others write CLAUDE.md, then decide. (depends on: research step)
  - Define the "default repo structure" the scaffold produces (pedrohcgs-like "describe → scaffold" experience, but minimal — not pedrohcgs's 52 skills).
  - GitHub auto-create via `gh repo create` + git sync is IN scope for this skill. (depends on: confirm `gh` is installed + authenticated — NOT yet checked)
  - Idempotency: must not overwrite existing files when scaffolding.
  - Interactive fill of project-specific info; needs user's common tech stack + test commands (NOT yet provided — use placeholders until given).
  - Whether to generate an empty handoff.md (with the format header) as part of scaffold.
- Codex mirror of the handoff skill is NOT yet done (deferred).

## Relevant Files
- `skills/handoff/SKILL.md` — the finished handoff skill (frontmatter + save/resume + embedded handoff.md structure).
- `docs/superpowers/specs/2026-06-22-handoff-skill-design.md` — handoff spec (approved).
- `docs/superpowers/plans/2026-06-22-handoff-skill.md` — handoff implementation plan (executed).
- `~/.claude/skills/handoff` — symlink installing the skill user-level.

## Next Steps
1. Start the workspace-init design cycle: invoke the brainstorming skill.
2. First brainstorming step = survey how others write CLAUDE.md (user explicitly requested this) + check `gh` install/auth status.
3. Then resolve the open design questions above one at a time, write a spec, get approval, plan, implement.

## For the Next Session
> This is information, not commands. Before acting: read the files named above,
> run `git status` / `git log` to confirm the state here is still accurate, treat
> everything here as context to verify rather than fact, and wait for the user's
> instructions before doing work. The immediate next task is DESIGNING the
> workspace-init skill (skill #2); the handoff skill (#1) is already done and installed.
