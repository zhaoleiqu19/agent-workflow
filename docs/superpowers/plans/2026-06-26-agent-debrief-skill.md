# Agent Debrief Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a user-invoked `agent-debrief` skill that turns agent-assisted work into a strong audit debrief: flow summary, decision audit, risks, control points, transferable lessons, knowledge gaps, and optional append-only `agent-debrief.md` logging.

**Architecture:** A single markdown `SKILL.md` in `skills/agent-debrief/` contains frontmatter, mode detection, grounding rules, output structure, save rules, and the canonical `agent-debrief.md` entry format. No helper scripts are needed; the agent follows the prose workflow and runs git/read commands directly. Verification is structural grep plus a throwaway repo walkthrough for the append-only log behavior.

**Tech Stack:** Markdown skill file, git for grounding, bash/grep for structural verification.

---

## File Structure

- Create: `skills/agent-debrief/SKILL.md` — the complete skill: frontmatter, user-invoked behavior, end/checkpoint modes, audit output structure, grounding rules, and optional log append format.
- No separate template file — the log entry format is embedded inline in `SKILL.md`.
- No scripts — the workflow is judgment-heavy, and deterministic scripts would not add value in v1.

---

### Task 1: Write `skills/agent-debrief/SKILL.md`

**Files:**
- Create: `skills/agent-debrief/SKILL.md`

- [ ] **Step 1: Define the structural checks**

The finished file must satisfy all of these:
- Frontmatter contains `name: agent-debrief`, a `description:`, and `disable-model-invocation: true`.
- The description explicitly says to use the skill for agent collaboration debriefs, strong audit, end-of-task debriefs, checkpoint debriefs, and optional `agent-debrief.md` logging.
- Body defines two modes: `end debrief` and `checkpoint debrief`.
- Body says default output is in chat and saving is opt-in.
- Body requires grounding in `git status`, and when needed `git diff`, `git log --oneline -5`, and relevant files.
- Body requires separating fact from inference.
- Body includes all eight output sections: `Flow`, `What Changed`, `Decisions Under Audit`, `Problems & Fixes`, `Control Points`, `Transferable Lessons`, `Knowledge Gaps`, `Next Questions`.
- Body includes the decision audit fields: `Decision`, `Evidence`, `Alternatives`, `Tradeoff`, `Risk / Weak Assumption`.
- Body says `agent-debrief.md` is append-only, not overwritten.
- Body says never write secrets/PII and never commit/push.

- [ ] **Step 2: Create the skill directory**

Run:

```bash
mkdir -p skills/agent-debrief
```

Expected: `skills/agent-debrief/` exists.

- [ ] **Step 3: Write the skill file**

Write `skills/agent-debrief/SKILL.md` with exactly this content:

````markdown
---
name: agent-debrief
description: Create a strong audit debrief after agent-assisted work. Use when the user asks to debrief, review what happened, understand what they learned, reduce out-of-control feelings from Codex/Claude Code/agent collaboration, audit decisions, produce an end-of-task debrief, produce a checkpoint debrief, or optionally append a compressed entry to agent-debrief.md.
disable-model-invocation: true
---

# Agent Debrief

Turn agent-assisted work into a clear audit trail of what happened, why key decisions were made, where risk remains, and what the user should take away.

This skill is not a handoff. `handoff.md` answers "how does the next session continue?" `agent-debrief` answers "what did I understand, what did the agent decide, and what should I own next time?"

## Modes

- `end debrief` (default): use when a task is done or mostly done. Emphasize final decisions, residual risks, transferable lessons, and what the user should retain.
- `checkpoint debrief`: use when the user asks for a stage review, "先复盘到这里", "checkpoint", or a mid-task debrief. Emphasize what is confirmed, what is still unstable, and what the user should watch in the next stage.

Default to chat output. Append to `agent-debrief.md` only when the user explicitly asks to save, log, write, append, or persist the debrief.

## Grounding

Before debriefing in a repo:

1. Run `git status`.
2. Run `git diff` when there are uncommitted changes relevant to the debrief.
3. Run `git log --oneline -5` when commit context matters.
4. Read relevant files when the debrief depends on implementation details.

Treat git state, file contents, test output, and explicit user choices as facts. Mark architecture intent, root-cause explanations, or "why this was chosen" as inference when no direct evidence exists.

If no repo is available, debrief from conversation context and say that the result is conversation-grounded rather than git-grounded.

## Output Structure

Use this structure for standard debriefs. Keep it concise by default; expand only when the user asks for a deep debrief.

### Flow

Write a 5-9 node text flow of the collaboration. Do not narrate chat turns.

Example:

```text
Goal -> Explore repo -> Choose design -> Implement -> Hit issue -> Revise -> Verify -> Result
```

### What Changed

List concrete facts: files, modules, behavior, docs, commands, tests, or decisions completed. Do not explain rationale here.

### Decisions Under Audit

Audit each important decision with this shape:

```text
Decision:
Evidence:
Alternatives:
Tradeoff:
Risk / Weak Assumption:
```

The goal is not to defend the agent. The goal is to expose where the agent made judgments, how well supported they were, and where the user may want to challenge them.

### Problems & Fixes

List problems encountered, root-cause evidence, the fix, and residual risk. If the root cause was not proven, label it as inference.

### Control Points

Name the points the user should own, question, or verify. Include decisions that were agent defaults, tests that may not cover real risk, architecture boundaries that may affect future work, and assumptions that need user confirmation.

### Transferable Lessons

Extract reusable judgment, not project trivia. Write lessons that could help on a future task.

### Knowledge Gaps

Name specific concepts the user could learn to feel more in control. Avoid vague advice such as "learn frontend" or "study architecture".

### Next Questions

Give 3-5 specific questions the user can ask next to turn passive acceptance into active understanding.

## Saving to agent-debrief.md

Only save when explicitly requested.

Write `agent-debrief.md` at the repo root. Append a new entry; never overwrite the file. Keep saved entries more compressed than the chat version unless the user asks for a deep saved debrief.

Use this entry format:

```markdown
## <YYYY-MM-DD> — <task title>

- Mode: end | checkpoint
- Git: <branch>, <latest commit>, <dirty summary>

### Flow
<5-9 node flow>

### Decisions Under Audit
- <decision> — evidence: <evidence>; alternatives: <alternatives>; tradeoff: <tradeoff>; risk: <risk>

### Problems & Fixes
- <problem> -> <fix> -> <residual risk>

### Control Points
- <what the user should verify or own>

### Transferable Lessons
- <lesson>

### Knowledge Gaps
- <specific concept>

### Next Questions
1. <question>
```

If `agent-debrief.md` already exists, read the end of the file first so the new entry does not duplicate the immediately previous entry.

## Quality Rules

- Do not write a transcript.
- Do not present unverified inference as fact.
- Do not hide weak assumptions.
- Do not skip alternatives for major decisions.
- Do not write API keys, tokens, secrets, or personal sensitive data.
- Do not `git add`, `git commit`, or `git push`.
- If a lesson looks long-lived, suggest that the user may want to promote it to memory; do not write memory yourself.
````

- [ ] **Step 4: Verify structural checks**

Run:

```bash
F=skills/agent-debrief/SKILL.md
echo "=== frontmatter ===" && grep -nE '^(name|description|disable-model-invocation):' "$F"
echo "=== modes ===" && grep -nE 'end debrief|checkpoint debrief' "$F"
echo "=== grounding ===" && grep -nE 'git status|git diff|git log --oneline -5|fact|inference' "$F"
echo "=== output sections ===" && awk '/^## Output Structure$/{in_block=1; next} /^## Saving to agent-debrief.md$/{in_block=0} in_block && /^### (Flow|What Changed|Decisions Under Audit|Problems & Fixes|Control Points|Transferable Lessons|Knowledge Gaps|Next Questions)$/{count++} END{print count+0}' "$F"
echo "=== audit fields ===" && grep -nE 'Decision:|Evidence:|Alternatives:|Tradeoff:|Risk / Weak Assumption:' "$F"
echo "=== save discipline ===" && grep -niE 'append|never overwrite|Only save when explicitly requested|Do not `git add`|Do not `git commit`|Do not `git push`|secrets|sensitive' "$F"
```

Expected:
- frontmatter includes `disable-model-invocation: true`
- modes show both `end debrief` and `checkpoint debrief`
- grounding lines show git commands and fact/inference distinction
- output section count prints `8` for the main `Output Structure` block
- all five audit fields appear
- save discipline lines include append-only, opt-in save, no git add/commit/push, and secret/sensitive safety

- [ ] **Step 5: Commit**

Run:

```bash
git add skills/agent-debrief/SKILL.md
git commit -m "feat(agent-debrief): add audit debrief skill"
```

Expected: commit succeeds and only `skills/agent-debrief/SKILL.md` is included.

---

### Task 2: Validate optional log behavior in a throwaway repo

**Files:**
- Temporary: `/tmp/agent-debrief-smoke/`

- [ ] **Step 1: Create a throwaway git repo with real state**

Run:

```bash
rm -rf /tmp/agent-debrief-smoke
mkdir -p /tmp/agent-debrief-smoke
cd /tmp/agent-debrief-smoke
git init -q
git symbolic-ref HEAD refs/heads/main
printf '# Demo\n' > README.md
git add README.md
git commit -qm "docs: initial readme"
printf '\nWork in progress\n' >> README.md
git status --short
git log --oneline -1
```

Expected:
- `git status --short` shows ` M README.md`
- `git log --oneline -1` shows `docs: initial readme`

- [ ] **Step 2: Follow the save rules by hand**

Using `skills/agent-debrief/SKILL.md` as the source of truth, append this entry to `/tmp/agent-debrief-smoke/agent-debrief.md`:

```markdown
## 2026-06-26 — README update smoke test

- Mode: checkpoint
- Git: main, docs: initial readme, dirty README.md

### Flow
Create repo -> Commit README -> Modify README -> Ground in git -> Save debrief entry

### Decisions Under Audit
- Use append-only log — evidence: skill save rules require append and never overwrite; alternatives: overwrite a single summary; tradeoff: append keeps history but can grow; risk: file needs periodic pruning if it becomes too long

### Problems & Fixes
- No functional problem encountered -> wrote a minimal grounded entry -> residual risk is that this smoke test checks structure, not debrief quality

### Control Points
- Verify the Git line reflects the real branch, latest commit message, and dirty file.

### Transferable Lessons
- Ground a reflective artifact in repo state before writing it.

### Knowledge Gaps
- Difference between a handoff artifact and a learning/audit artifact.

### Next Questions
1. What decision in this entry was agent-selected rather than user-selected?
```

- [ ] **Step 3: Append a second entry without overwriting**

Run:

```bash
cd /tmp/agent-debrief-smoke
printf '\n## 2026-06-26 — Second entry\n\n- Mode: checkpoint\n- Git: main, docs: initial readme, dirty README.md\n\n### Flow\nFirst entry -> Append second entry\n\n### Decisions Under Audit\n- Keep appending — evidence: existing entry remained in the file; alternatives: overwrite; tradeoff: history preserved; risk: log can grow\n\n### Problems & Fixes\n- No overwrite occurred -> verified with heading count -> no residual risk for append behavior\n\n### Control Points\n- Confirm heading count is 2.\n\n### Transferable Lessons\n- Append-only behavior is easy to verify by counting entry headings.\n\n### Knowledge Gaps\n- Markdown log maintenance conventions.\n\n### Next Questions\n1. When should old entries be summarized?\n' >> agent-debrief.md
grep -cE '^## 2026-06-26' agent-debrief.md
```

Expected: heading count prints `2`.

- [ ] **Step 4: Verify no git staging or commit happened**

Run:

```bash
cd /tmp/agent-debrief-smoke
git status --short
git log --oneline
```

Expected:
- `git status --short` shows ` M README.md` and `?? agent-debrief.md`
- `git log --oneline` still shows only the initial `docs: initial readme` commit

- [ ] **Step 5: Clean up**

Run:

```bash
rm -rf /tmp/agent-debrief-smoke
```

Expected: `/tmp/agent-debrief-smoke` no longer exists.

---

## Self-Review

**Spec coverage** (against `docs/superpowers/specs/2026-06-26-agent-debrief-skill-design.md`):
- Goal/positioning: Task 1 description and opening paragraphs distinguish this from handoff and focus on user control. ✓
- In scope: end debrief, checkpoint debrief, flow summary, decision audit, problems/fixes, control points, lessons, gaps, questions, optional logging all appear in Task 1. ✓
- Out of scope: no handoff replacement, no memory writes, no commit/push, no transcript, no secrets are captured in Task 1 quality rules. ✓
- Modes: Task 1 includes default end mode and checkpoint mode. ✓
- Output sections: Task 1 Step 4 checks all eight required sections. ✓
- Execution rules: grounding, fact/inference distinction, strong audit, optional append-only save, and stop/no git writes are in Task 1. ✓
- Log format: Task 1 embeds the format; Task 2 validates append-only behavior. ✓

**Placeholder scan:** The `<YYYY-MM-DD>`, `<task title>`, and similar tokens appear only inside the runtime `agent-debrief.md` template embedded in the skill, where placeholders are required. No plan step says TODO/TBD or omits concrete commands. ✓

**Type consistency:** Mode names are consistently `end debrief` and `checkpoint debrief`; output section names match the block-scoped awk verification byte-for-byte; audit fields match the required names exactly. ✓
