# Agent Workflow

Personal workflow skills for agent-assisted development across Codex and Claude Code.

This repository keeps small, reusable skills that help start projects, preserve in-flight state, and debrief work done with coding agents.

## Skills

- `workspace-init` - scaffold a new or adopted project workspace with agent instructions, handoff structure, README, gitignore, and Superpowers docs folders.
- `handoff` - save or resume volatile cross-session work state in `handoff.md`.
- `agent-debrief` - audit agent-assisted work after a task or checkpoint via an independent auditor (a fresh subagent where available, a memory-disabled self-audit otherwise) so decisions, risks, lessons, and knowledge gaps are explicit, then promote long-lived lessons into memory through a draft-confirm-write loop.

## Layout

```text
skills/
  agent-debrief/
  handoff/
  workspace-init/
docs/superpowers/
  specs/
  plans/
handoff.md
```

## Validation

For Codex-compatible skill metadata:

```bash
python3 /home/qushiduo/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief
```

For prose-only skills, most verification is structural review plus smoke walkthroughs documented in `docs/superpowers/plans/`.

## Notes

Skills intended for both Codex and Claude Code should use portable frontmatter (`name` and `description`) and put tool-specific behavior in the body instead of platform-only metadata fields.
