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
