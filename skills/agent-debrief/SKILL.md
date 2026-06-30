---
name: agent-debrief
description: Create a strong audit debrief after agent-assisted work. Use when the user asks to debrief, review what happened, understand what they learned, reduce out-of-control feelings from Codex/Claude Code/agent collaboration, audit decisions, produce an end-of-task debrief, produce a checkpoint debrief, or optionally append a compressed entry to agent-debrief.md.
---

# Agent Debrief

Turn agent-assisted work into a clear audit trail produced by an independent auditor - not by the agent grading its own work.

Core principle: the agent that did the work does NOT grade its own work. An independent auditor does, from evidence. The deciding agent compiles a neutral brief of what happened; a fresh auditor (or a memory-disabled self-audit on degraded runtimes) produces the debrief body from git evidence and the brief alone.

The four stages are: (0) main agent compiles a neutral decision brief -> (1) independent auditor produces the debrief body -> (2) main agent presents the debrief -> (3) main agent drafts memory candidates and writes them only after user confirmation.

This skill is not a handoff. `handoff.md` answers "how does the next session continue?" `agent-debrief` answers "what did the agent decide, how well-supported were those decisions, and what should I own next time?"

## Modes

- `end debrief` (default): use when a task is done or mostly done. Emphasize final decisions, residual risks, transferable lessons, and what the user should retain.
- `checkpoint debrief`: use when the user asks for a stage review, "review up to here", "checkpoint", or a mid-task debrief. Emphasize what is confirmed, what is still unstable, and what the user should watch in the next stage.

Default to chat output. Append to `agent-debrief.md` only when the user explicitly asks to save, log, write, append, or persist the debrief.

---

## Stage 0 - Neutral Decision Brief

The MAIN agent (the one that did the work) compiles a neutral decision brief before any audit begins. This brief is the isolation key: it separates "what happened and who decided it" (facts + attribution) from "was it a good decision" (the auditor's job).

### Grounding

Run these commands to gather git evidence:

1. `git status` - working tree state.
2. `git log --oneline -10` - recent commits.
3. For uncommitted changes: `git diff` and `git diff --staged`.
4. For committed changes: `git diff <base>..HEAD` or `git show <commit>` for each relevant commit.

WARNING: `git diff` alone is often EMPTY after the agent has committed its work. Always check committed changes via `git diff <base>..HEAD` or `git show <commit>` - do not assume `git diff` covers the full changeset.

Read relevant files when the audit depends on implementation details.

If no repo is available, ground from conversation context and note that the result is conversation-grounded rather than git-grounded.

### Decision Brief Format

The brief contains one line per decision. Format each line as:

```text
"<decision>" - [user-chosen | agent-default]
```

Examples:

```text
"Use pytest fixtures instead of setup/teardown methods" - [agent-default]
"Keep the existing API surface, no breaking changes" - [user-chosen]
"Skip integration tests in this pass" - [user-chosen]
"Split into two modules rather than one large file" - [agent-default]
```

Rules for the brief:
- Write FACTS and ATTRIBUTION only.
- Write NO reasoning, justification, or "why" - those belong to the auditor.
- Include every non-trivial decision the agent made, including agent defaults.
- Omit micro-decisions with no meaningful alternative (e.g. "named the variable `x`").

---

## Stage 1 - Portable Audit Dispatch

The audit MUST be produced by an entity that has not seen the agent's reasoning.

### If the runtime can dispatch a subagent (e.g. Claude Code with Task tool)

Dispatch a FRESH auditor subagent. The auditor receives:
- Git evidence: the output of `git status`, `git log --oneline -10`, `git diff <base>..HEAD`, and `git show <commit>` for relevant commits.
- Final artifacts: current content of relevant files (read them; do not assume the auditor can access the repo).
- The neutral decision brief from Stage 0.

The auditor does NOT receive:
- Chat history.
- The main agent's reasoning or explanations.
- Any self-justification from the deciding agent.

The auditor is instructed to follow the Output Structure in this skill and to apply the forced fact/inference rule (see below). The auditor subagent exits after producing the debrief body; it does not present, confirm, or write memory.

### Otherwise (e.g. Codex, no subagent dispatch available)

The main agent performs a degraded self-audit. The main agent MUST:

1. Declare at the TOP of the debrief output: "DEGRADED SELF-AUDIT: this debrief was produced by the same agent that did the work, not an independent auditor. Treat all 'why' explanations as inference."
2. Treat its own memory of "why I chose things" as forbidden. Re-derive conclusions ONLY from git evidence and the decision brief, exactly as a fresh auditor would.
3. Flag any claim about reasoning or intent with `(inference - source: self-memory, not evidence)`.

---

## Output Structure

This section defines what the auditor (subagent or degraded self) produces. The main agent follows this structure when presenting in Stage 2.

### Flow

Write a 5-9 node text flow of the work. Allow and encourage failure branches and rollbacks. Use `->` for progression and `-(fail)->` for a failed attempt that required rerouting.

Example:

```text
Goal -> Explore repo -> Try approach A -(fail)-> Diagnose -(fail)-> Try approach B -> Implement -> Verify -> Result
```

Do not narrate chat turns. Focus on the logical shape of the work.

### What Changed

List concrete facts: files, modules, behavior, docs, commands, tests, or decisions completed. Do not explain rationale here. Reference commits or `file:line` rather than re-pasting large diffs.

### Decisions Under Audit

Audit each important decision with this template:

```text
Decision:
Evidence:
Alternatives:
Tradeoff:
Risk / Weak Assumption:
Who decided: [user-chosen | agent-default]
```

The `Who decided` field is sourced from the neutral decision brief. The goal is NOT to defend the agent. The goal is to expose where the agent made judgments, how well supported they were, and where the user may want to challenge them.

### Problems & Fixes

List problems encountered, root-cause evidence, the fix, and residual risk. If the root cause was not proven by evidence, label it as inference.

### Control Points

Name the points the user should own, question, or verify. Include decisions that were agent defaults, tests that may not cover real risk, architecture boundaries that may affect future work, and assumptions that need user confirmation.

### Transferable Lessons

Extract reusable judgment, not project trivia. Write lessons that could help on a future task. These are the primary candidates for Stage 3 memory promotion.

### Knowledge Gaps

Name specific concepts the user could learn to feel more in control. Avoid vague advice such as "learn frontend" or "study architecture".

### Next Questions

Give 3-5 specific questions the user can ask next to turn passive acceptance into active understanding.

### Forced Fact/Inference Rule

Apply this rule throughout ALL sections:
- Git evidence (diffs, commit messages, file contents, test output) = fact. State as fact.
- Any claim about "why" something was chosen, root causes, or intent that has NO direct evidence = inference. Label it: `(inference)`.

The auditor cannot state the deciding agent's reasoning as fact because the auditor was never given the reasoning. This is structural enforcement, not a courtesy label.

### Small-Task Minimal Set

For a single-step or small task, emit ONLY: Flow, What Changed, Control Points, Next Questions. Omit all other sections entirely. Do NOT emit empty headings. A short output with no empty sections is better than a long output with placeholders.

---

## Stage 2 - Present the Debrief

The MAIN agent (not the auditor subagent, which has exited) presents the debrief to the user.

- Default to chat output.
- Do not re-introduce the agent's own reasoning or self-justification when presenting.
- Reference commits or `file:line` for specifics; do not re-paste large diffs.
- If the runtime produced a degraded self-audit, keep the declaration visible at the top.

---

## Stage 3 - Memory Promotion Loop

After presenting the debrief, the main agent reviews Transferable Lessons and any other long-lived items (user preferences revealed during the task, repo-level pitfalls that will recur) as memory candidates.

### Distill Test

Before promoting any item, apply this test:

- Expires when this task ends -> NOT memory (at most a handoff note).
- Is a time-point action for the user right now -> stays in debrief as a Control Point.
- Any future session in any project should know it -> memory candidate.

### Memory Candidate Draft Format

Draft each memory candidate as a complete entry with frontmatter, type, and Why/How lines:

```markdown
---
title: <slug-style title>
date: <YYYY-MM-DD>
tags: [<relevant-tags>]
type: lesson | preference | pitfall
---

**Why:** <one sentence on why this matters across sessions and projects>

**How:** <one sentence on what to do or avoid>
```

### Confirmation Step

Show ALL drafted memory candidates to the user before writing anything. Ask explicitly:

```text
Memory candidates ready. For each, reply y / n / edit.

1. [<slug-style title>]: <one-line summary>
2. [<slug-style title>]: <one-line summary>
...
```

Write a memory file and update `MEMORY.md` ONLY after the user confirms with `y` for that entry. If the user replies `n`, discard. If the user replies `edit`, accept their edited version before writing.

NEVER silently write memory. Draft -> show -> confirm -> write.

---

## Dedup Boundaries

Debrief owns these content areas:
- Decision audit (Decisions Under Audit section)
- Control points (what the user should verify)
- Knowledge gaps
- Next questions

Debrief REFERENCES but does not duplicate:
- Code changes: reference commits or `file:line`; do not re-paste large diffs.

Debrief does NOT write:
- Current branch name, dirty file list, or next-step execution state - that is handoff territory.
- In-task temporary pitfalls (they belong in handoff if anywhere).

Debrief does NOT permanently store long-lived lessons:
- Long-lived lessons, user preferences, and repo-level pitfalls become memory candidates in Stage 3.
- The debrief itself does not need to retain them; memory is their home.

---

## Saving to agent-debrief.md

Only save when the user explicitly requests it.

Write `agent-debrief.md` at the repo root. Append a new entry; NEVER overwrite the file. Keep saved entries more compressed than the chat version unless the user asks for a deep saved debrief.

If a lesson was promoted to memory in Stage 3, write only:
`promoted to memory: [[slug]]` - do NOT duplicate the lesson content.

Use this entry format:

```markdown
## <YYYY-MM-DD> - <task title>

- Mode: end | checkpoint
- Git: <branch>, <latest commit>

### Flow
<5-9 node flow>

### Decisions Under Audit
- <decision> - evidence: <evidence>; who decided: [user-chosen | agent-default]; risk: <risk>

### Problems & Fixes
- <problem> -> <fix> -> <residual risk>

### Control Points
- <what the user should verify or own>

### Transferable Lessons
- <lesson> (or: promoted to memory: [[slug]])

### Knowledge Gaps
- <specific concept>

### Next Questions
1. <question>
```

If `agent-debrief.md` already exists, read the end of the file first so the new entry does not duplicate the immediately previous entry.

---

## Quality Rules

- Do not write a transcript.
- Do not present unverified inference as fact; label it `(inference)`.
- Do not hide weak assumptions.
- Do not skip alternatives for major decisions.
- Do not write API keys, tokens, secrets, or personal sensitive data.
- Do not `git add`, `git commit`, or `git push`.
- Do not silently write memory; always use the Stage 3 draft -> confirm -> write loop.
