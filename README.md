# HWD Pipeline — Claude Code Plugin

**Version:** 1.0.0
**Author:** Herts Web Design

A 6-agent sequential design pipeline for every new HWD web design project. Runs inside Claude Code.

## What It Does

Runs 6 specialist agents in sequence before any code is written:

| Agent | Purpose | Output |
|-------|---------|--------|
| 1. Market Research | Niche, audience, trust signals | Audience profile + design preferences |
| 2. Competitive Analysis | Competitor patterns and gaps | Design opportunities + colour/type data |
| 3. Front End Design | Custom or template direction | CSS custom properties + typography system |
| 4. Visual Assets | Images, icons, logo | Full asset specification |
| 5. Component Extraction | HTML/CSS component breakdown | BEM component list + dev checklist |
| 6. Performance Spec | PageSpeed targets + strategy | Netlify config + testing checklist |

Final output: a complete **HWD PROJECT BRIEF** document ready for development.

## Prerequisites

- Claude Code installed
- `ui-ux-pro-max` plugin installed (for `ckm:design-system` and `ckm:design`)

## Installation

### Option 1 — Install from folder (local)

1. Copy the `hwd-pipeline` folder to a location on your machine
2. In Claude Code, run:
   ```
   /plugins install /path/to/hwd-pipeline
   ```
3. Verify installation:
   ```
   /plugins list
   ```
   You should see `hwd-pipeline` in the list.

### Option 2 — Symlink (development mode)

If you want to edit agent files and have changes take effect immediately, add the plugin path directly in Claude Code settings.

## Usage

Start a new project conversation in Claude Code, then invoke:

```
Use the hwd-pipeline skill for a new project — [client name], [brief description]
```

Or simply:
```
Start the HWD pipeline for a new client
```

The pipeline will begin with Agent 1 asking about your niche and target audience.

## Updating Agents

Edit the relevant `agent-N-*.md` file directly. Changes take effect on next plugin load.

Do not change handoff statement formats — the orchestrator depends on them to advance pipeline stages.

## v2 Roadmap

- Agent 7: SEO Meta Content specification
- Agent 8: Copywriting brief
- Multi-page pipeline (inner pages after homepage)
