# Agent Debrief Independent-Audit Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `skills/agent-debrief/SKILL.md` so the debrief is produced by an independent auditor (portable, with graceful Codex degrade) and so long-lived lessons flow into memory through a draft-confirm-write loop.

**Architecture:** The skill stays a single prose `SKILL.md`. The workflow changes from "one agent self-audits" to four stages: (0) the main agent compiles a neutral decision brief, (1) an independent auditor produces the debrief body — dispatched as a fresh subagent where the runtime supports it, else a memory-disabled self-audit, (2) the main agent presents it, (3) the main agent drafts memory candidates and writes them only after user confirmation. No scripts. Verification is structural grep plus a walkthrough against a real commit.

**Tech Stack:** Markdown skill file; git for grounding; bash/grep for structural verification; `quick_validate.py` for frontmatter + ASCII validation.

## Global Constraints

- SKILL.md MUST be ASCII-only. `quick_validate.py` runs on Python 3.6 (ASCII default) and crashes on any non-ASCII byte. Use `-` for dashes, `->` for arrows.
- frontmatter uses ONLY portable `name` and `description`. No Claude-only fields (e.g. `disable-model-invocation`). Platform differences live in the body as conditional branches.
- The agent NEVER silently writes memory: memory entries are drafted, shown to the user, and written only on explicit confirmation.
- The agent NEVER runs `git add` / `commit` / `push`, and NEVER writes secrets/tokens/PII.
- `quick_validate.py` must pass: `python3 ~/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief`

---

## File Structure

- Modify (full rewrite): `skills/agent-debrief/SKILL.md` - the complete redesigned skill. All four stages, the auditor interface, dedup boundaries, memory promotion loop, quality rules, and the optional `agent-debrief.md` entry format live inline in this one file.
- No separate template or script files - the workflow is judgment-heavy prose; deterministic helpers add no value.

The rewrite is one file, but split into tasks by section so each carries its own structural check and a reviewer can reject one section while approving its neighbor. Tasks are sequential edits to the same file; commit after each.

---

### Task 1: Capture the baseline failure (RED)

This is the writing-skills "watch it fail first" gate. The redesign exists to kill three behaviors; record that the CURRENT skill exhibits them so the rewrite is verifiable, not faith-based.

**Files:**
- Create: `docs/superpowers/plans/notes/2026-06-30-debrief-baseline.md` (scratch evidence, not shipped)

- [ ] **Step 1: Pick a real fixture commit**

Use `fe92d14` (the spec commit) or any recent substantive commit as the work-under-debrief.

Run: `git show fe92d14 --stat`
Expected: a diff with files changed - this is the material a debrief would audit.

- [ ] **Step 2: Run the CURRENT skill against it and record behavior**

Read `skills/agent-debrief/SKILL.md` (current version) and produce a debrief of `fe92d14` following it as written. Capture verbatim into the baseline note:
- Did the same agent both decide and audit? (expected: yes - the conflict)
- Were any "why this was chosen" claims stated as fact without an inference label? (expected: yes)
- After producing Transferable Lessons, was there any path to persist them? (expected: no - dead end)

- [ ] **Step 3: Write the baseline note**

Record the three observed failures with quoted evidence. This is the "test fails" artifact the rewrite must flip.

- [ ] **Step 4: Commit the baseline note**

```bash
git add docs/superpowers/plans/notes/2026-06-30-debrief-baseline.md
git commit -m "test(agent-debrief): capture baseline self-audit failures"
```

---

### Task 2: Rewrite frontmatter, overview, and modes

**Files:**
- Modify: `skills/agent-debrief/SKILL.md` (top section, through Modes)

**Interfaces:**
- Produces: the `## Modes` vocabulary (`end debrief`, `checkpoint debrief`) and the four-stage framing that Tasks 3-5 fill in.

- [ ] **Step 1: Define the structural checks for this section**

The finished top section must satisfy:
- Frontmatter has only `name: agent-debrief` and a `description:` that still triggers on: debrief, review what happened, audit decisions, reduce out-of-control feelings, end/checkpoint debrief, optional `agent-debrief.md` logging. Description states WHEN to use, not the workflow.
- Overview states the new core principle in 1-2 sentences: the debrief is produced by an INDEPENDENT auditor, not the agent grading itself.
- Keeps the handoff-vs-debrief boundary line.
- `## Modes` retains `end debrief` (default) and `checkpoint debrief`.
- ASCII-only.

- [ ] **Step 2: Write the section**

Replace the current overview so it names the four stages (decision brief -> independent audit -> present -> promote to memory) at a high level and states the principle: "The agent that did the work does not grade its own work; an independent auditor does, from evidence."

Keep: the "not a handoff" paragraph, the default-to-chat rule, both modes.

- [ ] **Step 3: Validate frontmatter and ASCII**

Run: `python3 ~/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief`
Expected: `Skill is valid!`

Run: `grep -nP '[^\x00-\x7F]' skills/agent-debrief/SKILL.md`
Expected: no output (no non-ASCII bytes).

- [ ] **Step 4: Commit**

```bash
git add skills/agent-debrief/SKILL.md
git commit -m "refactor(agent-debrief): reframe overview around independent audit"
```

---

### Task 3: Write Stage 0 (decision brief) and Stage 1 (portable audit dispatch)

**Files:**
- Modify: `skills/agent-debrief/SKILL.md` (add `## Stage 0` and `## Stage 1` sections after Modes)

**Interfaces:**
- Consumes: the four-stage framing from Task 2.
- Produces: the auditor INPUT contract (git evidence + final artifacts + neutral decision brief) that Task 4 consumes as the auditor's output structure.

- [ ] **Step 1: Define the structural checks**

Stage 0 + Stage 1 must satisfy:
- Stage 0 instructs the main agent to compile a NEUTRAL decision brief: each decision as `"<decision>" - [user-chosen | agent-default]`, facts and attribution only, explicitly NO reasoning/why.
- Stage 0 grounding covers committed work: `git status`, `git log --oneline -10`, uncommitted via `git diff`/`git diff --staged`, AND committed via `git diff <base>..HEAD` / `git show <commit>` - with the explicit warning that `git diff` alone may be empty after the agent commits.
- Stage 1 has an explicit portability branch:
  - If the runtime can dispatch a subagent: dispatch a fresh auditor; its inputs are git evidence + final artifacts + the decision brief; it does NOT receive chat history, the agent's reasoning, or self-justification.
  - Else (e.g. Codex): the main agent self-audits but is forbidden to use its memory of "why I chose things" - it must re-derive from git evidence + the decision brief, and must state at the top of the output that this is a degraded self-audit.
- ASCII-only; arrows as `->`.

- [ ] **Step 2: Write Stage 0**

Write the decision-brief instructions and the grounding steps (absorbing the committed-changes fix). Show the brief's exact line format as a fenced `text` example.

- [ ] **Step 3: Write Stage 1**

Write the portability branch as prose with a clear `if dispatch is available ... otherwise ...` structure. State the auditor's input contract and the explicit "does not receive" list. State the degraded-self-audit declaration requirement.

- [ ] **Step 4: Validate**

Run: `python3 ~/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief`
Expected: `Skill is valid!`

Run: `grep -nc 'agent-default\|user-chosen' skills/agent-debrief/SKILL.md`
Expected: >= 1 (the attribution vocabulary is present).

Run: `grep -nP '[^\x00-\x7F]' skills/agent-debrief/SKILL.md`
Expected: no output.

- [ ] **Step 5: Commit**

```bash
git add skills/agent-debrief/SKILL.md
git commit -m "feat(agent-debrief): add decision brief and portable audit dispatch"
```

---

### Task 4: Write Stage 1 auditor output structure + forced fact/inference + small-task minimal set

**Files:**
- Modify: `skills/agent-debrief/SKILL.md` (add `## Output Structure` / auditor output)

**Interfaces:**
- Consumes: the auditor input contract from Task 3.
- Produces: the section vocabulary (Flow, What Changed, Decisions Under Audit, Problems & Fixes, Control Points, Transferable Lessons, Knowledge Gaps, Next Questions) that Task 5 routes into memory and dedup.

- [ ] **Step 1: Define the structural checks**

The output section must satisfy:
- Lists all eight sections in order.
- `Decisions Under Audit` shape includes a `Who decided:` field with `[user-chosen | agent-default]` (new vs the old five-field shape).
- Flow allows and encourages failure/rollback branches, e.g. `Try A -(fail)-> Try B`.
- A forced fact/inference rule: git evidence = fact; "why/root cause" with no direct evidence MUST be labeled inference. Note that the auditor cannot state reasoning as fact because it was never given the reasoning.
- Small-task minimal set: a single-step task emits only Flow / What Changed / Control Points / Next Questions; omit empty sections rather than emitting empty headings.
- ASCII-only.

- [ ] **Step 2: Write the section**

Write the eight-section structure with the `Decisions Under Audit` template including `Who decided:`. Add the Flow failure-branch guidance, the forced fact/inference rule, and the small-task minimal-set rule.

- [ ] **Step 3: Validate**

Run: `python3 ~/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief`
Expected: `Skill is valid!`

Run: `grep -nc 'Who decided\|inference\|Control Points' skills/agent-debrief/SKILL.md`
Expected: >= 3 (each appears).

Run: `grep -nP '[^\x00-\x7F]' skills/agent-debrief/SKILL.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add skills/agent-debrief/SKILL.md
git commit -m "feat(agent-debrief): add auditor output structure with forced attribution"
```

---

### Task 5: Write Stage 2/3 (present + memory loop), dedup boundaries, save format, quality rules

**Files:**
- Modify: `skills/agent-debrief/SKILL.md` (add `## Stage 2/3`, `## Dedup Boundaries`, `## Saving to agent-debrief.md`, `## Quality Rules`)

**Interfaces:**
- Consumes: the section vocabulary from Task 4 (specifically Transferable Lessons as the memory-candidate source).

- [ ] **Step 1: Define the structural checks**

This section must satisfy:
- Stage 2: the MAIN agent (not the auditor subagent, which has exited) presents the debrief; default chat output.
- Stage 3 memory loop: the main agent drafts long-lived lessons/user-preferences/repo-level pitfalls as complete memory entries (frontmatter + `type` + Why/How), shows them to the user, and writes the file + updates `MEMORY.md` ONLY after confirmation `[y/n/edit]`. Explicit: never silently write memory.
- Stage 3 distill test stated: expires at task end -> not memory (at most handoff); time-point action for the user -> stays in debrief; any future session in any project should know it -> memory candidate.
- Dedup boundaries table/list: debrief owns decision audit / control points / knowledge gaps / next questions; references commits or `file:line` instead of re-pasting diffs; does NOT write current branch/dirty/next-steps (handoff territory); long-lived items become candidates, not debrief-resident content.
- Save-to-`agent-debrief.md`: only on explicit request; append-only; compressed; if a lesson was promoted to memory, write only `promoted to memory: [[slug]]` without duplicating content.
- Quality Rules: no transcript; no unverified inference as fact; no hidden weak assumptions; no skipped alternatives; no secrets/PII; no `git add`/`commit`/`push`.
- ASCII-only.

- [ ] **Step 2: Write the sections**

Write Stage 2/3, the dedup boundary list, the save format, and the quality rules. Show the memory-entry draft format and the `[y/n/edit]` confirmation step explicitly.

- [ ] **Step 3: Validate**

Run: `python3 ~/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief`
Expected: `Skill is valid!`

Run: `grep -nc 'MEMORY.md\|confirm\|promoted to memory' skills/agent-debrief/SKILL.md`
Expected: >= 2.

Run: `grep -nP '[^\x00-\x7F]' skills/agent-debrief/SKILL.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add skills/agent-debrief/SKILL.md
git commit -m "feat(agent-debrief): add memory promotion loop and dedup boundaries"
```

---

### Task 6: Integration smoke walkthrough (GREEN) and README sync

This flips the Task 1 baseline: prove the rewritten skill kills the three failures.

**Files:**
- Modify: `docs/superpowers/plans/notes/2026-06-30-debrief-baseline.md` (append GREEN results)
- Modify: `README.md` only if the agent-debrief one-liner is now inaccurate

- [ ] **Step 1: Walk the rewritten skill against the same fixture**

Read the rewritten `skills/agent-debrief/SKILL.md` and produce a debrief of fixture commit `fe92d14` following it exactly. In a runtime with subagent dispatch, actually dispatch the auditor with only git evidence + final artifacts + the decision brief.

- [ ] **Step 2: Verify the three failures are gone**

Confirm and record in the baseline note:
- Independence: the audit was produced from evidence + decision brief, not from the deciding agent's narrative (or, on a no-dispatch runtime, the degraded-self-audit declaration is present).
- Forced fact/inference: every "why" claim is either backed by evidence or labeled inference; none stated as bare fact.
- Closed loop: at least one Transferable Lesson was drafted as a memory entry and offered for `[y/n/edit]` confirmation (not written silently, not dropped).

- [ ] **Step 3: Sync README if needed**

Read the `agent-debrief` line in `README.md`. If it still describes the old self-debrief behavior, update it to mention independent audit + memory promotion. If accurate, leave unchanged.

- [ ] **Step 4: Final full validation**

Run: `python3 ~/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief`
Expected: `Skill is valid!`

Run: `grep -nP '[^\x00-\x7F]' skills/agent-debrief/SKILL.md`
Expected: no output.

- [ ] **Step 5: Commit**

```bash
git add docs/superpowers/plans/notes/2026-06-30-debrief-baseline.md README.md
git commit -m "test(agent-debrief): verify independent-audit redesign passes walkthrough"
```

---

## Self-Review

**Spec coverage** (against `2026-06-30-agent-debrief-independent-audit-design.md`):
- D1 portability + Codex degrade -> Task 3 Stage 1 branch.
- D2 auditor inputs (evidence + neutral brief, no reasoning) -> Task 3 Stage 0/1.
- D3 memory draft+confirm+write -> Task 5 Stage 3.
- D4 scope = debrief only, handoff untouched -> no task modifies handoff; dedup boundary (Task 5) only references handoff's territory.
- Spec section 5 auditor output + forced fact/inference + small-task set -> Task 4.
- Spec section 6 dedup + save format -> Task 5.
- Spec section 1 three root flaws -> Task 1 (baseline) + Task 6 (verify flipped).
- Spec section 8 known limitations (post-hoc, residual brief bias, degraded independence) -> carried as prose in the Stage sections; not separate tasks (documentation of limits, not behavior).

**Placeholder scan:** no TBD/TODO; each task states exact checks and commands. Prose-content steps describe the required content and structural greps rather than pasting the full final prose, matching the repo's existing prose-skill plan style (`2026-06-26-agent-debrief-skill.md`).

**Type consistency:** attribution vocabulary `[user-chosen | agent-default]` used identically in Tasks 3 and 4; `MEMORY.md`, `agent-debrief.md`, `[[slug]]` consistent across Tasks 5-6.
