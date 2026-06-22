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
