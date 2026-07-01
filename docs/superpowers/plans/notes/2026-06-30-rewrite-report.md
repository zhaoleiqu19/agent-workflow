# Rewrite Report: agent-debrief SKILL.md

- Date: 2026-06-30
- Commit: d940499
- Branch: agent-debrief-independent-audit
- File rewritten: `skills/agent-debrief/SKILL.md`

---

## Sections Written

The rewrite produced a full replacement of the prior 131-line skill with a 245-line
structured skill covering all four stages and supporting sections.

### Frontmatter + Overview + Modes

- Frontmatter retains only `name: agent-debrief` and `description:`. No Claude-only fields.
- Description wording preserved verbatim from the old skill (triggers on: debrief, review what
  happened, reduce out-of-control feelings, audit decisions, end/checkpoint debrief, optional
  agent-debrief.md logging).
- Overview states the core principle in two sentences: the deciding agent does not grade its
  own work; an independent auditor does, from evidence.
- Four-stage pipeline named at overview level:
  (0) neutral decision brief -> (1) independent audit -> (2) present -> (3) promote to memory.
- "Not a handoff" boundary line kept.
- Both modes retained: `end debrief` (default) and `checkpoint debrief`.
- Default-to-chat rule kept.

### Stage 0 - Neutral Decision Brief

- Grounding steps cover: `git status`, `git log --oneline -10`, uncommitted changes via
  `git diff` and `git diff --staged`, AND committed changes via `git diff <base>..HEAD` and
  `git show <commit>`.
- Explicit warning: "`git diff` alone is often EMPTY after the agent has committed its work."
- Decision brief format shown as fenced `text` block with examples.
- Rules: facts + attribution only; NO reasoning or "why"; include every non-trivial decision;
  omit micro-decisions.

### Stage 1 - Portable Audit Dispatch

- Explicit branch: "If the runtime can dispatch a subagent" / "Otherwise (e.g. Codex)".
- Subagent path: auditor receives git evidence + final artifacts + neutral decision brief.
- Explicit "does NOT receive" list: chat history, agent reasoning, self-justification.
- Degraded path: main agent self-audits but MUST declare at top "DEGRADED SELF-AUDIT" and
  is forbidden to use its memory of "why I chose things".
- Auditor subagent exits after producing debrief body (does not present, confirm, or write memory).

### Output Structure

Eight sections in order: Flow, What Changed, Decisions Under Audit, Problems & Fixes,
Control Points, Transferable Lessons, Knowledge Gaps, Next Questions.

- `Decisions Under Audit` template includes the new `Who decided: [user-chosen | agent-default]`
  field alongside Decision/Evidence/Alternatives/Tradeoff/Risk.
- Flow section explicitly allows and encourages failure/rollback branches with `-(fail)->` syntax.
- Example Flow shows a failure branch: "Try approach A -(fail)-> Diagnose -(fail)-> Try approach B".
- Forced fact/inference rule stated as its own subsection: git evidence = fact; "why/root cause"
  with no direct evidence = inference labeled as `(inference)`. Auditor cannot state reasoning
  as fact because it was never given the reasoning.
- Small-task minimal set: single-step tasks emit only Flow / What Changed / Control Points /
  Next Questions; empty headings are forbidden.

### Stage 2 - Present the Debrief

- Clarifies the main agent (not the auditor subagent, which has exited) presents the debrief.
- Default chat output.
- Degraded self-audit declaration kept visible at top.
- No re-introduction of agent reasoning when presenting.

### Stage 3 - Memory Promotion Loop

- Distill test stated with three cases:
  - expires at task end -> not memory (at most handoff)
  - time-point action for user now -> stays in debrief as Control Point
  - any future session in any project should know it -> memory candidate
- Memory candidate draft format shown: frontmatter + `type` + Why/How lines.
- Confirmation step shown with exact `[y/n/edit]` prompt format.
- Explicit: "NEVER silently write memory. Draft -> show -> confirm -> write."
- Write file + update MEMORY.md ONLY after user confirmation.

### Dedup Boundaries

- What debrief owns: decision audit, control points, knowledge gaps, next questions.
- What debrief references but does not duplicate: code changes (commits or file:line).
- What debrief does NOT write: current branch, dirty file list, next-step execution state
  (handoff territory).
- What debrief does NOT permanently store: long-lived lessons go to memory candidates in Stage 3.

### Saving to agent-debrief.md

- Only on explicit user request.
- Append-only; never overwrite.
- Compressed format.
- If a lesson was promoted to memory: write only `promoted to memory: [[slug]]`, do NOT
  duplicate the lesson content.
- Concrete entry-format example in fenced markdown block.
- Read-before-append guard (check end of file to avoid duplicating the previous entry).

### Quality Rules

- No transcript.
- No unverified inference as fact (label `(inference)`).
- No hidden weak assumptions.
- No skipped alternatives.
- No secrets/PII.
- No `git add` / `git commit` / `git push`.
- No silent memory writes; always use Stage 3 draft -> confirm -> write loop.

---

## Validation Command Outputs

### ASCII check

Command: `grep -nP '[^\x00-\x7F]' skills/agent-debrief/SKILL.md`
Output: (no output - ASCII clean)

### Skill validator

Command: `python3 ~/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief`
Output: `Skill is valid!`

### Structural grep checks

Command: `grep -nc 'agent-default\|user-chosen' skills/agent-debrief/SKILL.md`
Output: `7` (expected >= 1)

Command: `grep -nc 'Who decided\|inference\|Control Points' skills/agent-debrief/SKILL.md`
Output: `11` (expected >= 3)

Command: `grep -nc 'MEMORY.md\|confirm\|promoted to memory' skills/agent-debrief/SKILL.md`
Output: `10` (expected >= 2)

---

## Commit

SHA: d940499
Message: feat(agent-debrief): rewrite around independent audit and memory loop

---

## Concerns and Ambiguities Resolved

1. The spec (section 9) and plan both stated to keep ONLY `name` and `description` in frontmatter,
   with no Claude-only fields. The old skill already satisfied this. The new skill preserves it.

2. The plan's Task 3 says to show the brief format as a "fenced text example" - done with two
   fenced blocks (the format line and multiple concrete examples).

3. The spec says "5-9 nodes" for Flow. The rewritten skill keeps this requirement and the small-task
   minimal set drops the constraint implicitly (small tasks get a simpler flow by definition).

4. "Degraded self-audit declaration at the TOP" - positioned in Stage 1 (where the self-audit
   happens) and in Stage 2 (where the declaration must remain visible when presenting). Both
   references are included.

5. The `git diff <base>..HEAD` grounding warning was placed prominently in Stage 0 as a WARNING
   paragraph to make it impossible to miss.

6. The old Quality Rules had "if a lesson looks long-lived, suggest that the user may want to
   promote it to memory; do not write memory yourself." The new Stage 3 replaces this with a
   full draft-confirm-write loop. The Quality Rules section now states the loop requirement
   instead of the old passive-suggestion wording.

7. No concerns about spec coverage gaps - all eight design decisions (D1-D4) and all spec
   sections (2-9) are represented in the skill body.
