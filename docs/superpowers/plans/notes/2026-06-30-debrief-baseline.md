# Baseline (RED) — current agent-debrief self-audit failures

Fixture: commit `fe92d14` (the redesign spec) debriefed by a fresh subagent
following the CURRENT `skills/agent-debrief/SKILL.md` verbatim, without being
told it was a test. Full output captured in controller context on 2026-06-30.

The redesign must flip three failures. Observed in the baseline run:

## Failure 1 — auditor = auditee (conflict of interest) — DEMONSTRATED

The same agent that summarized the work also produced "Decisions Under Audit"
on it. The current skill has no independent party and no isolation: one agent,
one context, grading its own narrative. The structured five-field template made
the self-assessment look rigorous regardless of independence. This is the
structural flaw the redesign's Stage 0/1 split must remove.

GREEN target (Task 6): the audit body is produced from git evidence + the
neutral decision brief, NOT from the deciding agent's narrative — or, on a
no-dispatch runtime, the degraded-self-audit declaration is present at the top.

## Failure 3 — no closed loop — DEMONSTRATED

The baseline produced four Transferable Lessons and several Knowledge Gaps,
then ended. There is no mechanism in the current skill to persist any of it;
the lessons evaporate when the chat ends and would be re-discovered from
scratch next time. Confirmed dead end.

GREEN target (Task 6): at least one Transferable Lesson is drafted as a memory
entry and offered for [y/n/edit] confirmation — neither written silently nor
dropped.

## Failure 2 — unenforceable fact/inference split — STRUCTURAL, under-stressed by fixture

The current skill's rule ("mark as inference when no direct evidence exists")
relies entirely on agent self-discipline; nothing enforces it. In this run the
agent actually labeled several inferences correctly — but only because a SPEC
commit makes "evidence" trivial (Evidence fields just quote the spec text). A
fixture with real implementation choices and no written rationale would stress
this far harder. So the failure is present by-design (no enforcement) but this
fixture does not expose it behaviorally.

GREEN target (Task 6): verify the fix is STRUCTURAL, not behavioral luck — the
auditor is never given the agent's reasoning, so it physically cannot state a
"why" as fact; any "why" must be evidence-backed or labeled inference. Check
the mechanism, not just whether labels happened to appear.

## Conclusion

#1 and #3 are robustly demonstrated. #2 is structural and must be verified by
mechanism in GREEN. Proceed with the rewrite.

---

# GREEN — rewritten skill walkthrough (same fixture)

The controller acted as the main agent following the REWRITTEN skill on the
same fixture `fe92d14`: compiled a Stage 0 neutral decision brief (facts +
[user-chosen | agent-default] only), then dispatched a FRESH independent
auditor subagent with git evidence + final artifact + the brief, and
explicitly NOT the conversation or any reasoning.

## Failure 1 — auditor = auditee — FLIPPED

The audit was produced by a separate subagent with no access to the
conversation in which the work was decided. It surfaced findings the deciding
agent would be motivated to omit: that the spec was committed "pending review"
while implementation commits landed immediately (review window ~ zero), and
the Chinese-spec / English-SKILL.md fidelity gap. An actor grading itself does
not volunteer these. Independence is real, not cosmetic.

## Failure 2 — fact/inference — FLIPPED BY MECHANISM

The auditor labeled `(inference)` exactly where it lacked direct evidence for a
"why" ("may be intentional to keep the design simple (inference)"). Crucially,
it was never given our actual rationale, so it physically could not state our
reasoning as fact - it had to cite spec text (evidence) or label inference.
This is the structural enforcement the baseline run could not exhibit.

## Failure 3 — closed loop — FLIPPED

The auditor produced Transferable Lessons; Stage 3 (main agent) drafts the
long-lived ones as memory candidates and offers [y/n/edit] before any write.
Demonstration candidate drafted from the run (NOT written - this was a test,
not a user-requested debrief):

```markdown
---
name: separate-actor-from-auditor
description: When an actor must evaluate its own output, a separate auditor reduces motivated reasoning
metadata:
  type: feedback
---
Separating "what happened + who decided" (compiled by the actor) from "was it
good" (judged by a separate entity given only evidence) structurally cuts
self-justification. **Why:** the actor cannot launder its rationale into the
audit if the auditor never receives the rationale. **How to apply:** for any
self-review, hand the reviewer evidence + a neutral fact list, withhold the
"why". Links: [[agent-debrief]]
```

This would be presented for [y/n/edit]; on `y` it is written and MEMORY.md
updated; on `n` it is dropped. Neither silent-write nor dead-end.

## Conclusion

All three baseline failures flipped. The independent auditor produced sharper,
less self-serving findings than the baseline self-audit - the intended effect.
Redesign verified.
