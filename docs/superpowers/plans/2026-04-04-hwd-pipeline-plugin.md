# HWD Pipeline Plugin — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and install a Claude Code skill plugin that runs a 6-agent sequential design pipeline for every new HWD web design project.

**Architecture:** A single installable plugin (`hwd-pipeline`) with one orchestrator `SKILL.md` that controls pipeline state and 6 agent markdown files loaded in sequence. Each agent file contains its questions, research instructions, output format, and handoff statement. Context accumulates as `PROJECT_CONTEXT` across the conversation.

**Tech Stack:** Claude Code skill plugin (markdown), no build tools, no dependencies. Installed locally via Claude Code plugin system. Integrates with `ckm:design-system` and `ckm:design` from the already-installed `ui-ux-pro-max` plugin.

---

## File Map

| File | Responsibility |
|------|---------------|
| `.claude-plugin/plugin.json` | Plugin metadata and registry entry |
| `.claude/skills/hwd-pipeline/SKILL.md` | Orchestrator — pipeline rules, state machine, context schema, handoff logic |
| `.claude/skills/hwd-pipeline/agent-1-market-research.md` | Niche/audience research questions, analysis instructions, output format |
| `.claude/skills/hwd-pipeline/agent-2-competitive-analysis.md` | Competitor design analysis questions, output format |
| `.claude/skills/hwd-pipeline/agent-3-frontend-design.md` | Design direction questions, custom/template branching, ckm:design-system invocation |
| `.claude/skills/hwd-pipeline/agent-4-visual-assets.md` | Asset specification questions, conditional ckm:design invocation |
| `.claude/skills/hwd-pipeline/agent-5-component-extraction.md` | Component inventory, BEM class structure, dev checklist |
| `.claude/skills/hwd-pipeline/agent-6-performance.md` | PageSpeed targets, image/font/caching strategy, performance budget |
| `CLAUDE.md` | Developer notes for maintaining/extending the plugin |
| `README.md` | Installation instructions and usage guide |

---

## Task 1: Initialise Repository and Plugin Metadata

**Files:**
- Create: `hwd-pipeline/.claude-plugin/plugin.json`
- Create: `hwd-pipeline/CLAUDE.md`

- [ ] **Step 1: Initialise git repository**

Run from `C:/Users/pidod/Desktop/hwd-pipeline/`:
```bash
git init
```
Expected: `Initialized empty Git repository in .../hwd-pipeline/.git/`

- [ ] **Step 2: Create plugin metadata**

Create `.claude-plugin/plugin.json`:
```json
{
  "name": "hwd-pipeline",
  "version": "1.0.0",
  "description": "HWD Design & Development Pipeline — 6-agent sequential system for web design projects",
  "author": {
    "name": "Herts Web Design",
    "url": "https://hertswebdesign.co.uk"
  },
  "license": "MIT",
  "keywords": ["web design", "pipeline", "hwd", "design", "development", "hertfordshire"]
}
```

- [ ] **Step 3: Create CLAUDE.md**

Create `CLAUDE.md`:
```markdown
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
```

- [ ] **Step 4: Verify plugin structure**

Run:
```bash
ls -la C:/Users/pidod/Desktop/hwd-pipeline/
ls -la C:/Users/pidod/Desktop/hwd-pipeline/.claude-plugin/
```
Expected: `plugin.json` visible in `.claude-plugin/`

- [ ] **Step 5: Commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .claude-plugin/plugin.json CLAUDE.md
git commit -m "feat: initialise hwd-pipeline plugin with metadata"
```

---

## Task 2: Orchestrator SKILL.md

**Files:**
- Create: `.claude/skills/hwd-pipeline/SKILL.md`

- [ ] **Step 1: Create skills directory**

```bash
mkdir -p "C:/Users/pidod/Desktop/hwd-pipeline/.claude/skills/hwd-pipeline"
```

- [ ] **Step 2: Write the orchestrator SKILL.md**

Create `.claude/skills/hwd-pipeline/SKILL.md`:

````markdown
---
name: hwd-pipeline
description: >
  HWD Design & Development Pipeline. Use when starting a new web design project for a client.
  Runs 6 sequential specialist agents: Market Research → Competitive Analysis → Front End Design
  → Visual Assets → Component Extraction → Performance Specification. Each agent asks clarifying
  questions before executing, then passes complete context to the next agent. Produces a fully
  populated PROJECT_CONTEXT development brief. Trigger phrases: "new project", "new client site",
  "start pipeline", "hwd pipeline".
argument-hint: "[client name or niche]"
---

# HWD Design & Development Pipeline

You are running the HWD Design & Development Pipeline for Herts Web Design.

This is a 6-agent sequential system. You must follow it exactly.

---

## Core Rules (Non-Negotiable)

1. **Sequential only.** Agents run 1 → 2 → 3 → 4 → 5 → 6. Never skip, never reorder.
2. **Hard gate.** You cannot advance to the next agent until the current agent has: (a) received all question answers, (b) completed its research/analysis, (c) produced its full structured output, (d) stated its handoff phrase.
3. **One question per message.** Never ask two questions in the same message. Wait for the user's answer before sending the next question.
4. **Clarify vague answers.** If a user's answer is ambiguous, send a single clarifying question before proceeding.
5. **Accumulate context.** Each agent appends its output to PROJECT_CONTEXT (see schema below). Never re-ask information already captured.
6. **No code generation.** The pipeline produces specifications only. No HTML, CSS, or JS until the pipeline is complete and you have explicitly exited pipeline mode.
7. **Load agent files.** When entering each agent stage, read and follow the corresponding agent file exactly.

---

## Pipeline State Machine

```
START
  └─► AGENT 1: Market Research          [load: agent-1-market-research.md]
        └─► [hard gate passed]
              └─► AGENT 2: Competitive Analysis   [load: agent-2-competitive-analysis.md]
                    └─► [hard gate passed]
                          └─► AGENT 3: Front End Design   [load: agent-3-frontend-design.md]
                                ├─► Custom build → invoke ckm:design-system (CSS vars only)
                                └─► Template path → WebSearch Framer/ThemeForest → direction locked
                                      └─► [hard gate passed]
                                            └─► AGENT 4: Visual Assets   [load: agent-4-visual-assets.md]
                                                  ├─► Logo needed → invoke ckm:design
                                                  └─► [hard gate passed]
                                                        └─► AGENT 5: Component Extraction   [load: agent-5-component-extraction.md]
                                                              └─► [hard gate passed]
                                                                    └─► AGENT 6: Performance Specification   [load: agent-6-performance.md]
                                                                          └─► [hard gate passed]
                                                                                └─► PIPELINE COMPLETE → output PROJECT_CONTEXT brief
```

---

## PROJECT_CONTEXT Schema

Maintain this as a running document throughout the conversation. Append after each agent.

```
PROJECT_CONTEXT
├── project.client_name
├── project.niche
├── project.date_started
│
├── market (populated by Agent 1)
│   ├── target_audience_demographic
│   ├── buying_behaviour
│   ├── decision_factors
│   ├── trust_signals
│   ├── common_objections
│   ├── design_preferences
│   └── competitive_landscape_summary
│
├── competitive (populated by Agent 2)
│   ├── top_patterns_that_work
│   ├── common_competitor_mistakes
│   ├── design_gaps
│   ├── successful_colour_palettes
│   ├── typography_that_resonates
│   ├── cta_placement_patterns
│   └── mobile_vs_desktop_notes
│
├── design (populated by Agent 3)
│   ├── build_type [custom | template]
│   ├── template_chosen (if template path)
│   ├── colour_system (CSS custom properties block)
│   ├── typography_pairing (Google Fonts import + CSS vars)
│   ├── layout_sections (ordered list)
│   ├── visual_hierarchy_breakdown
│   ├── interactions_and_animations
│   └── component_list_preview
│
├── assets (populated by Agent 4)
│   ├── images_needed (array: type, dimensions, style, purpose, source)
│   ├── icon_specifications
│   ├── illustrations_required
│   ├── photography_style_guide
│   └── asset_delivery_format
│
├── components (populated by Agent 5)
│   ├── component_inventory (ordered checklist)
│   ├── component_specifications (per component)
│   ├── css_class_structure (BEM)
│   └── development_checklist
│
└── performance (populated by Agent 6)
    ├── pagespeed_target
    ├── image_optimisation_strategy
    ├── font_loading_strategy
    ├── netlify_caching_rules
    ├── minification_requirements
    ├── performance_budget_kb
    └── testing_checklist
```

---

## Pipeline Complete — Output Format

After Agent 6 fires its final statement, output the following document:

```
# HWD PROJECT BRIEF — [client_name]
Generated: [date]

## 1. Project Overview
Niche: [niche]
Primary conversion action: [from Agent 2]

## 2. Target Audience
[market section from PROJECT_CONTEXT]

## 3. Design System
Build type: [custom | template]
[colour_system CSS block]
[typography_pairing CSS block]
Homepage sections: [layout_sections]

## 4. Asset List
[assets section from PROJECT_CONTEXT]

## 5. Component Build Order
[components.component_inventory checklist]

## 6. Performance Targets
[performance section from PROJECT_CONTEXT]

## 7. Development Notes
[Any cross-agent observations, conflicts resolved, or special instructions]
```

---

## HWD Constraints

- Vanilla HTML/CSS/JS only — no frameworks, no build tools, no libraries (unless client specifically requires)
- Mobile-first breakpoints (unless Agent 5 overrides)
- Netlify deployment assumed — format caching rules as Netlify `_headers` file
- HWD PageSpeed standard: 85+ mobile — this is the floor, not a target
- BEM class convention: `.c-[component]__[element]--[modifier]`
- ckm:design-system output: extract CSS custom properties block only, discard Tailwind config
````

- [ ] **Step 3: Verify file exists and has correct frontmatter**

```bash
head -10 "C:/Users/pidod/Desktop/hwd-pipeline/.claude/skills/hwd-pipeline/SKILL.md"
```
Expected: frontmatter block with `name: hwd-pipeline` and `description:` field visible.

- [ ] **Step 4: Commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .claude/skills/hwd-pipeline/SKILL.md
git commit -m "feat: add orchestrator SKILL.md with pipeline state machine and context schema"
```

---

## Task 3: Agent 1 — Market Research

**Files:**
- Create: `.claude/skills/hwd-pipeline/agent-1-market-research.md`

- [ ] **Step 1: Write agent-1-market-research.md**

Create `.claude/skills/hwd-pipeline/agent-1-market-research.md`:

````markdown
# Agent 1 — Market Research

**Purpose:** Research the niche, target audience, and market dynamics before any design work begins.

**Announce on entry:**
> "Starting Agent 1: Market Research. I'll ask a series of questions about your client and their niche before conducting analysis. One question at a time."

---

## Opening Question

Ask this first. Wait for answer before proceeding.

> "What is the niche and target audience for this project? (e.g. independent garage in Hertfordshire, plumber in London, restaurant in St Albans)"

---

## Follow-up Questions

Ask each question one at a time. Tailor phrasing to the niche answer where possible. Wait for answer before asking the next.

1. "What age demographic typically uses this business?"
2. "What is their primary pain point — price, convenience, expertise, or trust?"
3. "Do they typically search on mobile or desktop?"
4. "What competitors exist in this area, and what are they doing wrong?"
5. "What emotional trigger most likely converts them — reliability, speed, quality, or price?"

If any answer is vague (e.g. "all ages", "both"), ask a single clarifying follow-up before proceeding to the next question.

---

## Research & Analysis

Once all questions are answered, conduct analysis and produce the output below. Do not ask more questions during this phase.

Research and determine:
- Full demographic breakdown (age range, income level, local vs. regional reach)
- Buying behaviour: how do they find this type of business? How do they decide?
- Decision factors: what makes them choose one provider over another?
- Top 3 trust signals that matter most for this specific niche
- Top 3 common objections a prospect in this niche has before converting
- Design preferences: minimal vs. detailed, modern vs. traditional, image-heavy vs. copy-driven
- Competitive landscape: is this niche saturated? What do the best-performing sites do?

---

## Output Format

Produce this exact structure:

```
## AGENT 1 OUTPUT — Market Research

### Target Audience
[Full demographic breakdown: age range, likely income bracket, location radius, how they search]

### Buying Behaviour & Decision Factors
- [factor 1]
- [factor 2]
- [factor 3]
- [factor N]

### Trust Signals That Matter
1. [Most important trust signal for this niche]
2. [Second trust signal]
3. [Third trust signal]

### Common Objections
1. [Most common objection before converting]
2. [Second objection]
3. [Third objection]

### Design Preferences for This Niche
[1–2 sentences: visual style (minimal/detailed, modern/traditional), tone (professional/friendly), complexity level, image importance]

### Competitive Landscape
[1–2 sentences: market saturation level, what top competitors do well, what they all fail at]
```

---

## Handoff Statement

End with this exact phrase (fill in bracketed values):

> "Research complete. Passing to Competitive Design Analysis Agent with niche: [niche], audience: [one-line audience summary], primary trust signal: [top trust signal]."

Then immediately begin Agent 2 by loading `agent-2-competitive-analysis.md`.
````

- [ ] **Step 2: Verify required sections present**

```bash
grep -c "Opening Question\|Follow-up Questions\|Research & Analysis\|Output Format\|Handoff Statement" \
  "C:/Users/pidod/Desktop/hwd-pipeline/.claude/skills/hwd-pipeline/agent-1-market-research.md"
```
Expected: `5`

- [ ] **Step 3: Commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .claude/skills/hwd-pipeline/agent-1-market-research.md
git commit -m "feat: add Agent 1 - Market Research"
```

---

## Task 4: Agent 2 — Competitive Design Analysis

**Files:**
- Create: `.claude/skills/hwd-pipeline/agent-2-competitive-analysis.md`

- [ ] **Step 1: Write agent-2-competitive-analysis.md**

Create `.claude/skills/hwd-pipeline/agent-2-competitive-analysis.md`:

````markdown
# Agent 2 — Competitive Design Analysis

**Purpose:** Analyse competitor websites in the niche to identify working design patterns, gaps, and failures.

**Announce on entry:**
> "Starting Agent 2: Competitive Design Analysis. Using the niche and audience data from Agent 1, I'll now investigate the competitive landscape. One question at a time."

---

## Opening Question

Ask this first. Wait for answer.

> "Should I analyse specific competitors you already know of, or should I research the top 5–10 competitors in this niche myself?"

---

## Follow-up Questions

Ask one at a time. Wait for each answer.

1. "What do you most want to do better than competitors — visual hierarchy, trust signals, CTA clarity, or mobile experience?"
2. "Are there any specific competitor websites you'd like me to analyse by URL?"
3. "What design style do you want to avoid — corporate and cold, generic and template-looking, or outdated and cluttered?"
4. "Should the site feel premium and luxury, or accessible and friendly?"
5. "What is the single most important action you want visitors to take on this site?"

If the user names competitor URLs in question 2, use WebSearch to retrieve and analyse them.

---

## Research & Analysis

Once all questions are answered, conduct analysis using knowledge of the niche and any URLs provided. Do not ask more questions during this phase.

Identify and document:
- Top design patterns that work in this niche (with reasoning)
- Common design mistakes across competitors
- Design gaps — things the best competitors are still failing to do
- Colour palettes used successfully: dominant colour, accent, background, text
- Typography choices that feel right for this audience
- CTA placement, copy style, and visual treatment that converts in this niche
- Key differences between mobile and desktop experiences in competitor sites

---

## Output Format

Produce this exact structure:

```
## AGENT 2 OUTPUT — Competitive Design Analysis

### Design Patterns That Work in This Niche
- [Pattern]: [one-line reasoning]
- [Pattern]: [one-line reasoning]
- [Pattern]: [one-line reasoning]

### Common Competitor Mistakes
- [Mistake 1]
- [Mistake 2]
- [Mistake N]

### Design Gaps (Opportunities)
- [Gap 1 — something working competitors are still failing to do]
- [Gap 2]
- [Gap N]

### Colour Palettes Used Successfully
- Dominant: [colour description + hex if identifiable]
- Accent: [colour description + hex]
- Background: [colour description + hex]
- Text: [colour description + hex]
- Notes: [any niche-specific colour observations]

### Typography That Resonates
- Headings: [font style — e.g. bold serif, clean sans-serif]
- Body: [font style]
- Accent/labels: [font style if used]

### CTA Placement & Style That Converts
[2–4 specific observations: where CTAs sit, what copy they use, button vs. link, colour treatment]

### Mobile vs. Desktop Experience Notes
[2–3 observations about how top performers differ on mobile vs. desktop]

### Primary Conversion Action
[The single action identified from questions: e.g. "Call now", "Book online", "Get a quote"]
```

---

## Handoff Statement

End with this exact phrase (fill in bracketed values):

> "Analysis complete. Passing to Front End Design Agent with research and competitive insights. Primary opportunity: [biggest design gap identified]. Design direction: [premium/accessible, modern/traditional]."

Then immediately begin Agent 3 by loading `agent-3-frontend-design.md`.
````

- [ ] **Step 2: Verify required sections present**

```bash
grep -c "Opening Question\|Follow-up Questions\|Research & Analysis\|Output Format\|Handoff Statement" \
  "C:/Users/pidod/Desktop/hwd-pipeline/.claude/skills/hwd-pipeline/agent-2-competitive-analysis.md"
```
Expected: `5`

- [ ] **Step 3: Commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .claude/skills/hwd-pipeline/agent-2-competitive-analysis.md
git commit -m "feat: add Agent 2 - Competitive Design Analysis"
```

---

## Task 5: Agent 3 — Front End Design

**Files:**
- Create: `.claude/skills/hwd-pipeline/agent-3-frontend-design.md`

- [ ] **Step 1: Write agent-3-frontend-design.md**

Create `.claude/skills/hwd-pipeline/agent-3-frontend-design.md`:

````markdown
# Agent 3 — Front End Design

**Purpose:** Lock the visual design direction — custom build or template — and produce a production-ready design system using `ckm:design-system`.

**Announce on entry:**
> "Starting Agent 3: Front End Design. I'll use the market and competitive data to guide the design direction. One question at a time."

---

## Opening Question

Ask this first. Wait for answer.

> "Is there an existing website for this business that the new design should be based on — i.e. is this a redesign?"

---

## Branch: Existing Site (if YES)

Ask in sequence if the answer is yes:
1. "Can you share the current website URL, or describe its current design?"
2. "What specifically needs to change — layout, colours, typography, or a complete overhaul?"
3. "Are there any elements from the current site that must be kept?"

Then continue to the Custom vs Template question below.

---

## Custom vs Template Question

Ask for all projects (redesign or new):

> "Should I design the front end custom from scratch, or should I research website templates from Framer.com or similar to find the best fit for this niche?"

---

## Path A — Custom Build

Ask in sequence. Wait for each answer.

1. "What is the primary colour palette? Share hex codes if you have them, or describe the mood (e.g. professional navy and white, warm earthy tones, bold and energetic)."
2. "Should the design feel luxury and premium, or accessible and friendly?"
3. "How many main sections should the homepage have?"
4. "What is the most important visual element on the homepage — a large hero image, a services grid, or a testimonials section?"
5. "Are there any specific animations or interactions you want? Remember: CSS and vanilla JS only — no libraries."

Once all answered, proceed to **Design System Generation** below.

---

## Path B — Template Research

This path requires WebSearch tool access.

Activate template research sub-process:
1. Search Framer.com for templates matching the niche identified in Agent 1
2. Search ThemeForest for HTML templates matching the niche
3. Score each template against: audience fit, trust signal coverage, design gap opportunities, CTA clarity
4. Present top 3–5 templates with: name, URL, score out of 10, one-line reasoning

Ask: "Which template direction appeals to you most, and what would you change about it?"

Once template direction confirmed, proceed to **Design System Generation** below.

---

## Design System Generation (REQUIRED — all paths)

After design direction is confirmed, invoke `ckm:design-system` using the Skill tool.

**Pass to ckm:design-system:**
- Colour answers (hex codes or mood descriptions)
- Typography preferences (heading style, body style)
- Spacing complexity (simple site vs. content-heavy)

**Extract from ckm:design-system output:**
- The `:root {}` CSS custom properties block (three-layer: Primitive → Semantic → Component tokens)
- The Google Fonts `@import` line(s)

**IMPORTANT:** Use the CSS custom properties output ONLY. Do not use Tailwind configuration output or shadcn component code. HWD builds in vanilla HTML/CSS/JS.

---

## Output Format

Produce this exact structure:

```
## AGENT 3 OUTPUT — Front End Design

### Build Type
[Custom build | Template: [template name and URL]]

### Colour System (CSS Custom Properties)
[Paste the full :root {} block from ckm:design-system output here]

Example structure:
:root {
  /* Primitive tokens */
  --color-brand-50: #f0f9ff;
  --color-brand-500: #0ea5e9;
  --color-brand-900: #0c4a6e;
  --color-neutral-50: #f8fafc;
  --color-neutral-900: #0f172a;

  /* Semantic tokens */
  --color-primary: var(--color-brand-500);
  --color-primary-dark: var(--color-brand-900);
  --color-background: var(--color-neutral-50);
  --color-text: var(--color-neutral-900);
  --color-text-muted: #64748b;

  /* Component tokens */
  --btn-primary-bg: var(--color-primary);
  --btn-primary-text: #ffffff;
  --nav-bg: var(--color-background);
  --nav-text: var(--color-text);
}

### Typography Pairing
[Google Fonts @import + CSS variables]

@import url('https://fonts.googleapis.com/css2?family=[Heading]+family=[Body]&display=swap');

:root {
  --font-heading: '[Heading Font]', sans-serif;
  --font-body: '[Body Font]', sans-serif;
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;
  --text-5xl: 3rem;
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.625;
}

### Homepage Layout (Sections in Order)
1. [Section name — one-line purpose]
2. [Section name — one-line purpose]
3. [Section name — one-line purpose]
[continue for all sections]

### Visual Hierarchy Breakdown
Primary focal point: [element]
Secondary focal point: [element]
Tertiary: [element]
Supporting: [element]

### Interactions & Animations
- [Interaction description — CSS or vanilla JS, specify which]
- [Interaction description]
- [None if no interactions specified]

### Component List (Preview)
[Bulleted list of components — detailed breakdown in Agent 5]
```

---

## Handoff Statement

End with this exact phrase:

> "Design direction locked. Colour system and typography generated via ckm:design-system. Passing to Visual Assets Agent."

Then immediately begin Agent 4 by loading `agent-4-visual-assets.md`.
````

- [ ] **Step 2: Verify branching paths documented**

```bash
grep -c "Path A\|Path B\|Design System Generation\|Handoff Statement" \
  "C:/Users/pidod/Desktop/hwd-pipeline/.claude/skills/hwd-pipeline/agent-3-frontend-design.md"
```
Expected: `4`

- [ ] **Step 3: Commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .claude/skills/hwd-pipeline/agent-3-frontend-design.md
git commit -m "feat: add Agent 3 - Front End Design with ckm:design-system integration"
```

---

## Task 6: Agent 4 — Visual Assets

**Files:**
- Create: `.claude/skills/hwd-pipeline/agent-4-visual-assets.md`

- [ ] **Step 1: Write agent-4-visual-assets.md**

Create `.claude/skills/hwd-pipeline/agent-4-visual-assets.md`:

````markdown
# Agent 4 — Visual Assets

**Purpose:** Specify every visual asset needed for the design — images, icons, illustrations — without creating them.

**Announce on entry:**
> "Starting Agent 4: Visual Assets. I'll define exactly what images, icons, and graphics the design needs. One question at a time."

---

## Opening Question

Ask this first. Wait for answer.

> "Will the client provide their own images and photography, or should I specify what needs to be sourced from stock?"

---

## Follow-up Questions

Ask one at a time. Wait for each answer.

1. "What photography style fits this niche — professional and posed, or authentic and candid?"
2. "Should the design use icons, illustrations, or neither?"
3. "Are there existing brand assets to work around — logo, existing photos, specific brand colours?"
4. "Looking at the homepage layout from Agent 3, what are the approximate dimensions for each image slot?"

---

## Logo Check

After question 3, if the user confirms the client has no logo or needs a logo redesign:

Invoke `ckm:design` via the Skill tool to produce a logo design brief and direction.

Pass to ckm:design:
- Business niche and name
- Design preferences (from Agent 1: minimal/detailed, modern/traditional)
- Colour system (from Agent 3: primary colours)
- Audience (from Agent 1: demographic)

This is the only point in the pipeline where logo/brand identity work is initiated.

---

## Output Format

Produce this exact structure:

```
## AGENT 4 OUTPUT — Visual Assets

### Images Required
| # | Type | Dimensions | Style | Purpose | Source |
|---|------|------------|-------|---------|--------|
| 1 | Hero background | 1920×1080px | [style] | [purpose] | [client/stock] |
| 2 | [type] | [dimensions] | [style] | [purpose] | [client/stock] |
[continue for all images in the layout]

### Icon Specifications
- Style: [line / filled / duotone — match the design tone]
- Size: [px — primary icon size]
- Colour: [CSS variable from Agent 3 colour system]
- Quantity: [total number of icons needed]
- Recommended library: [e.g. Heroicons SVG, Phosphor Icons, or "none — custom SVGs"]

### Illustrations / Graphics
[List of any illustrations or decorative graphics, or "None required"]

### Photography Style Guide
- Mood: [e.g. warm, clinical, energetic, calm]
- Lighting: [natural / studio / environmental]
- Subject: [what/who is in the photos]
- Composition: [e.g. wide establishing shots, close detail shots, people-focused]
- What to avoid: [specific photography styles that would feel wrong for this niche]

### Logo Status
[Client has logo — no action required | Logo design brief generated via ckm:design — see above]

### Asset Delivery Format
- Images: WebP format, JPEG fallback
- Icons: SVG (inline or linked)
- Naming convention: kebab-case, descriptive (e.g. hero-garage-mechanic.webp)
- Folder structure:
  /assets/images/   → photography and raster images
  /assets/icons/    → SVG icons
  /assets/logo/     → logo files (SVG + PNG)
```

---

## Handoff Statement

End with this exact phrase (fill in bracketed values):

> "Asset specifications complete. [X] images and [Y] icons specified. Passing to Component Extraction Agent."

Then immediately begin Agent 5 by loading `agent-5-component-extraction.md`.
````

- [ ] **Step 2: Verify ckm:design integration and logo check documented**

```bash
grep -c "Logo Check\|ckm:design\|Handoff Statement\|Opening Question" \
  "C:/Users/pidod/Desktop/hwd-pipeline/.claude/skills/hwd-pipeline/agent-4-visual-assets.md"
```
Expected: `4`

- [ ] **Step 3: Commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .claude/skills/hwd-pipeline/agent-4-visual-assets.md
git commit -m "feat: add Agent 4 - Visual Assets with conditional ckm:design logo integration"
```

---

## Task 7: Agent 5 — Component Extraction

**Files:**
- Create: `.claude/skills/hwd-pipeline/agent-5-component-extraction.md`

- [ ] **Step 1: Write agent-5-component-extraction.md**

Create `.claude/skills/hwd-pipeline/agent-5-component-extraction.md`:

````markdown
# Agent 5 — Component Extraction

**Purpose:** Break the finalised design into reusable HTML/CSS components with a development checklist.

**Announce on entry:**
> "Starting Agent 5: Component Extraction. I'll break the locked design into specific, buildable components with exact specs. One question at a time."

---

## Opening Question

Ask this first. Wait for answer.

> "Should I extract components based on the locked design from Agent 3, or do you want to revisit any design decisions before we define components?"

If the user wants to revisit design: return to Agent 3 and resolve before continuing.

---

## Follow-up Questions

Ask one at a time. Wait for each answer.

1. "Are there any custom components this niche specifically needs — for example a booking form, quote calculator, before/after image slider, or interactive map?"
2. "Should components be built mobile-first (start from smallest screen) or desktop-first?"
3. "Do you want component documentation generated alongside the development checklist, for handoff to a developer?"

---

## Component Analysis

Once questions are answered, analyse the locked design from Agent 3 and define all required components.

For every component, determine:
- Its exact dimensions and layout constraints
- Internal spacing (use CSS variables from Agent 3's colour/spacing system)
- All interaction states: default, hover, active, focus, disabled (where applicable)
- Breakpoint behaviour: what changes at mobile / tablet / desktop

Always include these baseline components (every HWD project needs them):
- CSS reset + custom properties file
- Typography scale
- Navbar (with mobile hamburger)
- Hero section
- Footer

Add all niche-specific components identified from the Agent 3 layout and Agent 5 questions.

---

## Output Format

Produce this exact structure:

```
## AGENT 5 OUTPUT — Component Extraction

### Component Inventory (Build in This Order)
- [ ] 1. CSS reset + custom properties (variables.css)
- [ ] 2. Typography scale
- [ ] 3. Navbar — desktop + mobile hamburger
- [ ] 4. Hero section
- [ ] 5. [Component name]
- [ ] 6. [Component name]
[continue for all components]
- [ ] N. Footer

### Component Specifications

#### 1. CSS Reset + Custom Properties
- File: variables.css + reset.css
- Contains: all :root tokens from Agent 3, box-sizing reset, base font settings
- No states or breakpoints — foundational layer only

#### 2. Navbar
- Dimensions: full-width, height: 64px desktop / 56px mobile
- Spacing: padding-inline: var(--space-6); gap between links: var(--space-4)
- States: default, scrolled (shadow appears), mobile-open
- Breakpoint: at 768px collapses to hamburger menu
- Hamburger: 3-line icon, animates to X on open, menu slides down

#### 3. Hero Section
- Dimensions: 100vw × 100vh (or 80vh if content-heavy)
- Background: [image or gradient from Asset specs]
- Spacing: content centred, padding-block: var(--space-16)
- States: default only
- Breakpoint: text sizes reduce on mobile, image repositions

#### [Continue for every component]

### CSS Class Structure (BEM Convention)
All classes use: .c-[component]__[element]--[modifier]

.c-navbar {}
.c-navbar__logo {}
.c-navbar__links {}
.c-navbar__toggle {}
.c-navbar--scrolled {}
.c-hero {}
.c-hero__content {}
.c-hero__heading {}
.c-hero__subheading {}
.c-hero__cta {}
[continue for all components]

### Development Checklist
- [ ] Create project folder structure
- [ ] Set up index.html with semantic HTML shell
- [ ] Create variables.css with full :root token block from Agent 3
- [ ] Create reset.css
- [ ] Implement typography scale
- [ ] Build c-navbar (desktop first, then mobile hamburger)
- [ ] Build c-hero
- [ ] Build [component]
[continue in build order]
- [ ] Link all CSS files in correct order
- [ ] Cross-browser test (Chrome, Firefox, Safari, Edge)
- [ ] Test on real mobile device
- [ ] Accessibility audit (keyboard nav, colour contrast, alt text)
- [ ] Run PageSpeed Insights — verify ≥85 mobile
```

---

## Handoff Statement

End with this exact phrase (fill in bracketed value):

> "Components defined. [N] components in build checklist. Passing to Performance Specification Agent."

Then immediately begin Agent 6 by loading `agent-6-performance.md`.
````

- [ ] **Step 2: Verify BEM convention and checklist documented**

```bash
grep -c "BEM\|Component Inventory\|Handoff Statement\|Opening Question" \
  "C:/Users/pidod/Desktop/hwd-pipeline/.claude/skills/hwd-pipeline/agent-5-component-extraction.md"
```
Expected: `4`

- [ ] **Step 3: Commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .claude/skills/hwd-pipeline/agent-5-component-extraction.md
git commit -m "feat: add Agent 5 - Component Extraction with BEM class structure and dev checklist"
```

---

## Task 8: Agent 6 — Performance Specification

**Files:**
- Create: `.claude/skills/hwd-pipeline/agent-6-performance.md`

- [ ] **Step 1: Write agent-6-performance.md**

Create `.claude/skills/hwd-pipeline/agent-6-performance.md`:

````markdown
# Agent 6 — Performance Specification

**Purpose:** Set performance targets and optimisation strategy before development begins, so the build meets them from the start.

**Announce on entry:**
> "Starting Agent 6: Performance Specification — the final agent. I'll define performance targets and optimisation rules before development begins. One question at a time."

---

## Opening Question

Ask this first. Wait for answer.

> "Should this site target 85+ PageSpeed mobile score? This is the HWD standard — the answer is yes unless you have a specific reason to differ."

---

## Follow-up Questions

Ask one at a time. Wait for each answer.

1. "Based on the asset list from Agent 4, roughly how many images are on the homepage, and what's your estimate of total page weight?"
2. "Should Google Fonts be loaded from the CDN, or self-hosted for maximum performance?"
3. "Are there any CSS animations or JS interactions from Agent 3 that could affect performance — for example scroll-triggered animations or video backgrounds?"
4. "Should the site be optimised for older mobile devices (2–3 year old Android phones) or only modern devices?"

---

## Performance Analysis

Once questions are answered, produce the full specification below. Use knowledge of Netlify hosting, WebP/JPEG tradeoffs, Google Fonts loading patterns, and Core Web Vitals to produce actionable rules.

---

## Output Format

Produce this exact structure:

```
## AGENT 6 OUTPUT — Performance Specification

### PageSpeed Targets
- Mobile: 85+ (HWD minimum standard)
- Desktop: 95+
- Core Web Vitals targets:
  - LCP (Largest Contentful Paint): < 2.5s
  - FID / INP: < 100ms
  - CLS (Cumulative Layout Shift): < 0.1

### Image Optimisation Strategy
- Primary format: WebP
- Fallback format: JPEG (for browsers without WebP support, use <picture> element)
- Compression quality: 80% WebP / 85% JPEG
- Hero image: serve at exact rendered size, provide <link rel="preload"> in <head>
- Below-fold images: loading="lazy" on all <img> tags
- Max hero dimensions: 1920px wide (scale down for mobile via srcset)
- srcset breakpoints: 480w, 768w, 1280w, 1920w

### Font Loading Strategy
- Provider: [Google Fonts CDN | Self-hosted — based on question 2 answer]
- If CDN: add to <head>:
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  @import with ?display=swap appended
- If self-hosted: download WOFF2 files, serve from /assets/fonts/, use @font-face with font-display: swap
- Subset: latin only (add latin-ext only if required by client name/location)
- Load maximum 2 font families — no more

### Netlify Caching Rules
File: _headers (place in project root, deployed with site)

/assets/*
  Cache-Control: public, max-age=31536000, immutable

/*.css
  Cache-Control: public, max-age=86400, must-revalidate

/*.js
  Cache-Control: public, max-age=86400, must-revalidate

/
  Cache-Control: public, max-age=0, must-revalidate

### Minification Requirements
- CSS: minify before deployment (VS Code extension: CSS Minifier, or manual minify.cesanta.com)
- JS: minify before deployment (VS Code extension: JS & CSS Minifier)
- HTML: remove comments and excess whitespace before deployment
- Target: minified CSS < 30KB, minified JS < 20KB

### Performance Budget
| Resource | Budget |
|----------|--------|
| Total page weight (all assets) | < [calculated from image count × avg size + CSS + JS + fonts] KB |
| All images combined | < [total image budget based on image count] KB |
| CSS (minified) | < 30KB |
| JS (minified) | < 20KB |
| Fonts (WOFF2) | < 50KB per family |

[Note: Fill in image budgets based on Agent 4 image count. Rule of thumb: budget 80–120KB per WebP hero image, 20–40KB per content image]

### Animation Performance Rules
[If animations specified in Agent 3:]
- Use CSS transforms and opacity only — these are GPU-accelerated
- Never animate layout properties (width, height, margin, padding)
- Add will-change: transform only to elements that animate (remove after animation completes via JS)
- Scroll animations: use IntersectionObserver API, not scroll event listener

[If no animations: "No animations specified — no animation performance considerations required"]

### Testing Checklist
- [ ] PageSpeed Insights mobile score ≥ 85 (pagespeed.web.dev)
- [ ] PageSpeed Insights desktop score ≥ 95
- [ ] GTmetrix — Grade A target (gtmetrix.com)
- [ ] WebPageTest — TTFB < 200ms (webpagetest.org)
- [ ] Chrome DevTools Lighthouse — all green
- [ ] Test on real Android device (Chrome)
- [ ] Test on real iPhone (Safari)
- [ ] Verify all images load as WebP in Chrome DevTools Network tab
- [ ] Verify lazy loading — images below fold should not load until scrolled to
- [ ] Verify fonts load with FOUT (flash of unstyled text), not FOIT (invisible text)
```

---

## Final Pipeline Statement

End with this exact statement:

> "Performance specifications locked. Pipeline complete. All context ready for development phase."

Then immediately produce the full **HWD PROJECT BRIEF** by merging all PROJECT_CONTEXT sections (from Agents 1–6) into the brief format defined in the orchestrator SKILL.md.
````

- [ ] **Step 2: Verify performance budget and testing checklist present**

```bash
grep -c "Performance Budget\|Testing Checklist\|Final Pipeline Statement\|Opening Question" \
  "C:/Users/pidod/Desktop/hwd-pipeline/.claude/skills/hwd-pipeline/agent-6-performance.md"
```
Expected: `4`

- [ ] **Step 3: Commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .claude/skills/hwd-pipeline/agent-6-performance.md
git commit -m "feat: add Agent 6 - Performance Specification with Netlify caching and testing checklist"
```

---

## Task 9: README and Plugin Installation

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README.md**

Create `README.md`:

````markdown
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

If you want to edit agent files and have changes take effect immediately:
```bash
# In Claude Code settings, add the plugin path directly
# Or symlink to the Claude plugins cache directory
```

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
````

- [ ] **Step 2: Commit README**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add README.md
git commit -m "feat: add README with installation and usage instructions"
```

- [ ] **Step 3: Verify full plugin structure**

```bash
find "C:/Users/pidod/Desktop/hwd-pipeline" -type f | sort
```
Expected output includes all 10 files:
```
.claude-plugin/plugin.json
.claude/skills/hwd-pipeline/SKILL.md
.claude/skills/hwd-pipeline/agent-1-market-research.md
.claude/skills/hwd-pipeline/agent-2-competitive-analysis.md
.claude/skills/hwd-pipeline/agent-3-frontend-design.md
.claude/skills/hwd-pipeline/agent-4-visual-assets.md
.claude/skills/hwd-pipeline/agent-5-component-extraction.md
.claude/skills/hwd-pipeline/agent-6-performance.md
CLAUDE.md
README.md
docs/superpowers/plans/2026-04-04-hwd-pipeline-plugin.md
docs/superpowers/specs/2026-04-04-hwd-agent-team-design.md
```

- [ ] **Step 4: Final commit**

```bash
cd C:/Users/pidod/Desktop/hwd-pipeline
git add .
git commit -m "chore: finalise hwd-pipeline v1.0.0 - all agents and documentation complete"
```

---

## Self-Review Notes

**Spec coverage check:**
- ✅ Plugin structure (Section 2) → Task 1
- ✅ plugin.json metadata (Section 3) → Task 1
- ✅ Orchestrator rules, state machine, context schema (Section 4) → Task 2
- ✅ Agent 1 questions, output, handoff (Section 5) → Task 3
- ✅ Agent 2 questions, output, handoff (Section 5) → Task 4
- ✅ Agent 3 branching, ckm:design-system CSS-only, output (Section 5) → Task 5
- ✅ Agent 4 logo check, ckm:design conditional (Section 5) → Task 6
- ✅ Agent 5 BEM convention, dev checklist (Section 5) → Task 7
- ✅ Agent 6 PageSpeed 85+, Netlify headers, budget (Section 5) → Task 8
- ✅ Pipeline Complete document (Section 6) → Task 2 (orchestrator)
- ✅ Constraints (Section 7) → Task 2 (orchestrator) + agent files
- ✅ README/installation → Task 9

**No placeholders found.** All agent file content is fully written.

**Type consistency:** Handoff phrases in agent files match what orchestrator expects. BEM class pattern `.c-[component]__[element]--[modifier]` consistent across Agent 5 spec and orchestrator constraints.
