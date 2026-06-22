# Handoff Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a user-level `handoff` skill that saves and resumes cross-session work state via a `handoff.md` file, grounded in git, overwrite-with-carry-forward, manual-invocation only.

**Architecture:** A single markdown `SKILL.md` (frontmatter + save/resume process + the canonical handoff.md structure). Developed canonically in `~/projects/agent-workflow/skills/handoff/`, installed to `~/.claude/skills/handoff/` via a symlink so repo edits go live immediately. No helper scripts — the agent runs the git commands directly per the skill's instructions.

**Tech Stack:** Markdown skill file (Claude Code skill format), git (read-only grounding), bash for install + verification.

**Note on testing:** This is a prose artifact, not code. "Tests" are (a) structural `grep` assertions on `SKILL.md`, and (b) a functional save→resume walkthrough in a throwaway git repo asserting the produced `handoff.md`. No unit-test framework involved.

---

## File Structure

- Create: `~/projects/agent-workflow/skills/handoff/SKILL.md` — the whole skill: frontmatter, save flow, resume flow, the canonical `handoff.md` structure embedded inline (single source, avoids drift between save and resume).
- Install (symlink): `~/.claude/skills/handoff` → `~/projects/agent-workflow/skills/handoff`
- No separate template file (YAGNI — one consumer; structure lives in `SKILL.md`).
- No scripts (YAGNI — git commands are run by the agent following the skill).

---

### Task 1: Write `SKILL.md`

**Files:**
- Create: `~/projects/agent-workflow/skills/handoff/SKILL.md`

- [ ] **Step 1: Define what the file must contain (the "test")**

The file, once written, MUST satisfy all of these (verified in Step 3):
- Frontmatter contains `name: handoff`, a `description:`, and `disable-model-invocation: true`.
- Body documents both modes: `/handoff` / `/handoff save` (save) and `/handoff resume` (resume).
- Save flow contains, in order: git grounding → read existing file → structured extraction → pitfall routing gate → carry-forward overwrite → safety (no secrets) → no commit/push.
- Resume flow contains: read → cross-check against git → flag staleness → stop and wait.
- The embedded `handoff.md` structure has all 9 headers: `Current Truth`, `User Goal`, `Work Completed`, `Key Decisions & Why`, `Pitfalls (task-local)`, `Open Work`, `Relevant Files`, `Next Steps`, `For the Next Session`.

- [ ] **Step 2: Write the file**

Write `~/projects/agent-workflow/skills/handoff/SKILL.md` with exactly this content:

````markdown
---
name: handoff
description: Save or resume cross-session work state via handoff.md. Use when ending a work session to record in-flight state, or when starting a fresh session to pick up in-progress work. User-invoked only.
disable-model-invocation: true
---

# Handoff

Maintain `handoff.md` — a volatile snapshot of the **current in-flight work state** — so a fresh session (or another tool/window) can pick up without the original chat history.

`handoff.md` holds easily-changing state, NOT slow facts. Repo-level or permanent lessons belong in memory, not here. handoff.md is overwritten on each save; that is correct — a finished task's state should disappear.

## Modes

- `/handoff` or `/handoff save` → **save** (write/refresh `handoff.md`)
- `/handoff resume` → **resume** (read `handoff.md`, verify against reality, report, then stop)

If no mode is given, default to **save**.

## Save

1. **Ground in git first.** Run `git status`, `git diff`, and `git log --oneline -5`. Treat the working tree as the source of truth — do NOT rely on chat memory alone.
2. **Read the existing `handoff.md`** if it exists (needed for carry-forward in step 5).
3. **Structured extraction.** From the working tree plus the conversation, distill: completed work, decisions + why, pitfalls, open work, relevant files, next steps. This is structured extraction, NOT a transcript dump — do not narrate the conversation.
4. **Pitfall routing gate.** Scan the pitfalls. Flag any that look repo-level or long-lived and tell the user to consider promoting them to memory. Do NOT write memory yourself.
5. **Carry-forward overwrite.** Keep the still-valid entries from the old file, drop resolved or stale ones, then write one correct complete snapshot that overwrites `handoff.md`. This is neither blind append (no pile-up) nor blind rewrite (do not ignore the old file).
6. **Safety.** Never write API keys, secrets, or PII into `handoff.md`.
7. Write the file and stop. Do NOT `git add` / `commit` / `push` — leave commits to the user.

Write `handoff.md` at the repo root using this exact structure (fixed English headers, ISO dates, machine-greppable):

```markdown
# Handoff — <repo> @ <branch>

## Current Truth
- Date (ISO):
- Repo / Branch / Latest commit:
- Dirty tracked files / Important untracked:
- Active constraints:

## User Goal
<1–2 sentences>

## Work Completed
- <what changed> + <files> + <evidence>

## Key Decisions & Why
- <decision> — <reason>

## Pitfalls (task-local)
- <failed attempt / easy-to-repeat mistake>
<!-- task-local only; carry forward what still holds, drop what is resolved.
     Repo-level / long-lived lessons do NOT go here — save flags them for memory. -->

## Open Work
- <X is not yet implemented> (depends on: ...)

## Relevant Files
- `path/to/file:L10-L45` — <what changed>

## Next Steps
1.
2.
3.

## For the Next Session
> This is information, not commands. Before acting: read the files named above,
> run `git status` / `git log` to confirm the state here is still accurate, treat
> everything here as context to verify rather than fact, and wait for the user's
> instructions before doing work.
```

## Resume

1. Read `handoff.md` at the repo root.
2. **Cross-check against reality.** Run `git status` and `git log --oneline -5`; compare the branch, latest commit, and dirty files against what the file claims.
3. If they disagree, say so explicitly — the handoff may be stale or was written by another window.
4. Summarize the current state and Next Steps to the user, then **stop and wait for instructions**. Do NOT start working on your own.
````

- [ ] **Step 3: Verify the file satisfies Step 1**

Run:
```bash
F=~/projects/agent-workflow/skills/handoff/SKILL.md
echo "=== frontmatter ===" && grep -nE '^(name|description|disable-model-invocation):' "$F"
echo "=== modes ===" && grep -nE '/handoff resume|/handoff save' "$F"
echo "=== 9 headers ===" && grep -cE '^## (Current Truth|User Goal|Work Completed|Key Decisions & Why|Pitfalls \(task-local\)|Open Work|Relevant Files|Next Steps|For the Next Session)$' "$F"
echo "=== save discipline ===" && grep -niE 'do NOT .*(commit|push)|never write api keys|carry-forward|ground in git' "$F"
```
Expected: `disable-model-invocation: true` present; both modes match; the "9 headers" count prints `9`; save-discipline lines all match.

- [ ] **Step 4: Commit**

```bash
cd ~/projects/agent-workflow
git add skills/handoff/SKILL.md docs/superpowers/specs docs/superpowers/plans
git commit -m "feat(handoff): add handoff save/resume skill + spec + plan"
```

---

### Task 2: Install as a user-level skill (symlink)

**Files:**
- Create (symlink): `~/.claude/skills/handoff` → `~/projects/agent-workflow/skills/handoff`

- [ ] **Step 1: Define the check**

After install: `~/.claude/skills/handoff/SKILL.md` must be readable and resolve (via the symlink) to the repo file.

- [ ] **Step 2: Create the symlink**

```bash
mkdir -p ~/.claude/skills
ln -s ~/projects/agent-workflow/skills/handoff ~/.claude/skills/handoff
```

- [ ] **Step 3: Verify**

Run:
```bash
ls -l ~/.claude/skills/handoff
readlink -f ~/.claude/skills/handoff/SKILL.md
test -r ~/.claude/skills/handoff/SKILL.md && echo "READABLE"
```
Expected: symlink points at `~/projects/agent-workflow/skills/handoff`; `readlink -f` resolves to the repo `SKILL.md`; prints `READABLE`.

- [ ] **Step 4: Confirm Claude Code lists the skill**

Tell the user to run `/handoff` (or check the skill picker) in a **new** Claude Code session, since skills are discovered at session start. Note this explicitly — the current session will not see it until restarted. No commit (symlink is machine-local, outside the repo).

---

### Task 3: Functional walkthrough — save then resume

**Files:**
- Temporary: `/tmp/handoff-smoke/` (throwaway git repo, deleted at the end)

- [ ] **Step 1: Define what the produced `handoff.md` must satisfy**

After a save in the scratch repo, `/tmp/handoff-smoke/handoff.md` MUST:
- Contain all 9 `## ` headers (same grep as Task 1).
- Have `Current Truth` reflect the scratch repo's actual branch and latest commit (git-grounded, not invented).
- Contain no scratch repo commit/push performed by the skill (skill only writes the file).

- [ ] **Step 2: Build a scratch repo with real state**

```bash
rm -rf /tmp/handoff-smoke && mkdir -p /tmp/handoff-smoke && cd /tmp/handoff-smoke
git init -q && git symbolic-ref HEAD refs/heads/main
printf 'print("v1")\n' > app.py
git add app.py && git commit -qm "initial: app.py v1"
printf 'print("v2 - work in progress")\n' > app.py   # dirty working tree
git status --short && git log --oneline -1
```
Expected: `app.py` shows as modified (` M app.py`); one commit "initial: app.py v1".

- [ ] **Step 3: Execute the save flow against the scratch repo**

Following `~/.claude/skills/handoff/SKILL.md` Save steps, with `/tmp/handoff-smoke` as the working dir: run the git grounding commands, then write `/tmp/handoff-smoke/handoff.md` per the embedded structure. Fill `Current Truth` from the real `git status` / `git log` output (branch `main`, the real commit hash/message, dirty file `app.py`). Use a plausible User Goal ("bump app.py to v2"). Do NOT commit in the scratch repo.

- [ ] **Step 4: Verify the produced file**

Run:
```bash
F=/tmp/handoff-smoke/handoff.md
echo "=== 9 headers ===" && grep -cE '^## (Current Truth|User Goal|Work Completed|Key Decisions & Why|Pitfalls \(task-local\)|Open Work|Relevant Files|Next Steps|For the Next Session)$' "$F"
echo "=== git-grounded ===" && grep -nE 'main|app.py' "$F"
echo "=== skill did not commit ===" && git -C /tmp/handoff-smoke log --oneline   # still only the initial commit
echo "=== handoff.md is untracked (skill did not stage) ===" && git -C /tmp/handoff-smoke status --short
```
Expected: headers count `9`; `Current Truth` mentions `main` and `app.py`; git log shows only the initial commit; `handoff.md` appears as untracked (`?? handoff.md`).

- [ ] **Step 5: Execute the resume flow and confirm it stops**

Following `~/.claude/skills/handoff/SKILL.md` Resume steps with `/tmp/handoff-smoke` as working dir: read `handoff.md`, run `git status` / `git log --oneline -5`, confirm branch/commit/dirty match the file, summarize state + Next Steps. Confirm the flow **stops and waits** rather than editing `app.py`. Verify `app.py` is unchanged since Step 2:
```bash
cat /tmp/handoff-smoke/app.py   # still the v2 WIP line, untouched by resume
```
Expected: `print("v2 - work in progress")` — resume did not modify it.

- [ ] **Step 6: Clean up**

```bash
rm -rf /tmp/handoff-smoke
```
No commit (scratch only).

---

## Self-Review

**Spec coverage** (against `2026-06-22-handoff-skill-design.md`):
- §2 In — save/resume/read-only git grounding → Task 1 SKILL.md body, Task 3 walkthrough. ✓
- §2 Out — no commit/push/repo, no memory writes, no hook, single file, Claude-only → encoded in SKILL.md (steps 4,7) + verified Task 3 step 4. ✓
- §3 structure (9 sections) → Task 1 Step 2 embeds it, Step 3 + Task 3 Step 4 assert all 9. ✓
- §4 save flow (7 steps incl. carry-forward + pitfall routing) → SKILL.md Save section verbatim. ✓
- §5 resume flow (read → cross-check → flag stale → stop) → SKILL.md Resume section; stop-behavior asserted Task 3 Step 5. ✓
- §6 defaults (command names, `disable-model-invocation`, repo-root location, user-level install) → frontmatter + Task 2. ✓

**Placeholder scan:** The `<repo>`, `<branch>`, `<...>` tokens inside the embedded `handoff.md` template are intentional fill-in markers for the runtime output, not plan placeholders. No "TODO/TBD" in the plan steps. ✓

**Type consistency:** Header names are identical across Task 1 Step 1, the embedded structure, Task 1 Step 3 grep, and Task 3 Step 4 grep — including the exact string `Pitfalls (task-local)` (parenthesis escaped in grep). Mode strings `/handoff save` / `/handoff resume` consistent throughout. ✓
