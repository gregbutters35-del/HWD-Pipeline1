# CLAUDE.md — hwd-pipeline Plugin

## Overview

6-agent sequential pipeline for HWD web design projects.
Invoked via: `Skill tool → hwd-pipeline`

## Structure

- `.claude-plugin/plugin.json` — plugin registry metadata
- `.claude/skills/hwd-pipeline/SKILL.md` — orchestrator (pipeline rules, state machine)
- `.claude/skills/hwd-pipeline/agent-N-*.md` — individual agent files (1–6)

## Adding or Editing Agents

1. Edit the relevant `agent-N-*.md` file
2. Keep opening question first, follow-ups sequential
3. Do not change handoff statement format — orchestrator depends on it
4. If adding a new agent (v2+): add it to the state machine in SKILL.md, create its file, increment version in plugin.json

## Extending to v2

Planned additions (out of scope for v1):
- Agent 7: SEO Meta Content
- Agent 8: Copywriting Brief

## ui-ux-pro-max Integration

- `ckm:design-system` — invoked in Agent 3 after design direction locked. Use CSS custom properties output only. Do NOT use Tailwind config or shadcn components.
- `ckm:design` — invoked in Agent 4 if client needs logo creation or redesign.
