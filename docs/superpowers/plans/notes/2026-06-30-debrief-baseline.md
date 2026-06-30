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
