# HWD Design & Development Pipeline — Agent Team System
**Date:** 2026-04-04
**Project:** hwd-pipeline Claude Code Plugin
**Author:** Herts Web Design (HWD)
**Status:** Approved for Implementation

---

## 1. Overview

A Claude Code skill plugin that runs a 6-agent sequential pipeline for every new HWD web design project. Each agent asks clarifying questions before executing, passes structured context to the next agent, and together they produce a complete development brief before a single line of code is written.

**Invocation:** `Skill tool → hwd-pipeline`
**Trigger phrases:** "new project", "start pipeline", "new client site", "hwd pipeline"
**Outcome:** A fully populated `PROJECT_CONTEXT` document covering market, design, assets, components, and performance — ready to hand to a developer.

---

## 2. Plugin File Structure

```
hwd-pipeline/
├── .claude-plugin/
│   └── plugin.json
├── .claude/
│   └── skills/
│       └── hwd-pipeline/
│           ├── SKILL.md                          ← Orchestrator
│           ├── agent-1-market-research.md
│           ├── agent-2-competitive-analysis.md
│           ├── agent-3-frontend-design.md
│           ├── agent-4-visual-assets.md
│           ├── agent-5-component-extraction.md
│           └── agent-6-performance.md
├── CLAUDE.md
└── README.md
```

---

## 3. Plugin Metadata (`plugin.json`)

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

---

## 4. Orchestrator (`SKILL.md`)

### 4.1 Frontmatter

```yaml
name: hwd-pipeline
description: >
  HWD Design & Development Pipeline. Use when starting a new web design project for a client.
  Runs 6 sequential specialist agents: Market Research → Competitive Analysis → Front End Design
  → Visual Assets → Component Extraction → Performance Specification. Each agent asks questions
  before executing. Produces a complete PROJECT_CONTEXT brief for development.
argument-hint: "[client name or niche]"
```

### 4.2 Orchestrator Behaviour Rules

1. **Sequential execution only.** Agents run 1 → 2 → 3 → 4 → 5 → 6 in strict order. No skipping.
2. **Hard gate per agent.** Claude cannot advance to the next agent until the current agent has: (a) received all question answers, (b) completed its research/analysis, (c) output its structured section, (d) stated the handoff phrase.
3. **One question at a time.** Never ask multiple questions simultaneously. Wait for user response before asking the next question.
4. **Clarify vague answers.** If a user answer is ambiguous, ask a focused clarification before proceeding.
5. **Context accumulation.** Each agent appends its output to `PROJECT_CONTEXT`. This object is maintained across the full conversation. Never re-ask information already captured.
6. **Reference agent files.** When entering each agent stage, load and follow the corresponding agent file exactly.
7. **No code generation during pipeline.** The pipeline produces specifications only. No HTML, CSS, or JS is written until the pipeline is complete.

### 4.3 Pipeline State Machine

```
START
  └─► AGENT 1: Market Research
        └─► [questions answered + output produced]
              └─► AGENT 2: Competitive Analysis
                    └─► [questions answered + output produced]
                          └─► AGENT 3: Front End Design
                                ├─► [custom build path] → invoke ckm:design-system
                                └─► [template path] → Template Research Sub-Agent
                                      └─► [design direction locked]
                                            └─► AGENT 4: Visual Assets
                                                  └─► [questions answered + output produced]
                                                        ├─► [logo needed] → invoke ckm:design
                                                        └─► AGENT 5: Component Extraction
                                                              └─► [questions answered + output produced]
                                                                    └─► AGENT 6: Performance Specification
                                                                          └─► [questions answered + output produced]
                                                                                └─► PIPELINE COMPLETE
```

### 4.4 `PROJECT_CONTEXT` Schema

A running document appended by each agent. Final document is the development brief.

```
PROJECT_CONTEXT
├── project.client_name
├── project.niche
├── project.date_started
│
├── market (Agent 1)
│   ├── target_audience_demographic
│   ├── buying_behaviour
│   ├── decision_factors
│   ├── trust_signals
│   ├── common_objections
│   ├── design_preferences
│   └── competitive_landscape_summary
│
├── competitive (Agent 2)
│   ├── top_patterns_that_work
│   ├── common_competitor_mistakes
│   ├── design_gaps
│   ├── successful_colour_palettes
│   ├── typography_that_resonates
│   ├── cta_placement_patterns
│   └── mobile_vs_desktop_notes
│
├── design (Agent 3)
│   ├── build_type [custom | template]
│   ├── template_chosen (if template)
│   ├── colour_system (CSS custom properties block)
│   ├── typography_pairing (Google Fonts import + variables)
│   ├── layout_sections (ordered list)
│   ├── visual_hierarchy_breakdown
│   ├── interactions_and_animations
│   └── component_list_preview
│
├── assets (Agent 4)
│   ├── images_needed (array: type, size, style, purpose)
│   ├── icon_specifications
│   ├── illustrations_required
│   ├── photography_style_guide
│   └── asset_delivery_format
│
├── components (Agent 5)
│   ├── component_inventory (ordered list)
│   ├── component_specifications (dimensions, spacing, states)
│   ├── css_class_structure
│   └── development_checklist
│
└── performance (Agent 6)
    ├── pagespeed_target
    ├── image_optimisation_strategy
    ├── font_loading_strategy
    ├── netlify_caching_rules
    ├── minification_requirements
    ├── performance_budget_kb
    └── testing_checklist
```

---

## 5. Agent Specifications

### Agent 1 — Market Research

**File:** `agent-1-market-research.md`
**Purpose:** Research the niche, target audience, and market dynamics before any design work begins.

#### Opening Question
> "What is the niche and target audience for this project? (e.g. independent garage in Hertfordshire, plumber in London, restaurant in St Albans)"

#### Follow-up Questions (ask one at a time, tailored to niche answer)
1. "What age demographic typically uses this business?"
2. "What is their primary pain point — price, convenience, expertise, or trust?"
3. "Do they typically search on mobile or desktop?"
4. "What competitors exist in this area, and what do they do wrong?"
5. "What emotional trigger most likely converts them — reliability, speed, quality, or price?"

#### Research & Analysis Instructions
After questions answered, Claude must:
- Build a demographic breakdown of the target audience
- Identify buying behaviour patterns and decision factors
- List the top 3 trust signals for this niche
- List the top 3 common objections
- Define design preferences (minimal vs. detailed, modern vs. traditional, etc.)
- Assess market saturation and competitive landscape

#### Output Format
```
## AGENT 1 OUTPUT — Market Research

### Target Audience
[Demographic breakdown]

### Buying Behaviour & Decision Factors
[Bulleted list]

### Trust Signals That Matter
1. [signal]
2. [signal]
3. [signal]

### Common Objections
1. [objection]
2. [objection]
3. [objection]

### Design Preferences for This Niche
[Description: visual style, tone, complexity level]

### Competitive Landscape
[Summary of market saturation and what competitors do well/poorly]
```

#### Handoff Statement
> "Research complete. Passing to Competitive Design Analysis Agent with niche: [X], audience: [Y], primary trust signal: [Z]."

---

### Agent 2 — Competitive Design Analysis

**File:** `agent-2-competitive-analysis.md`
**Purpose:** Analyse competitor websites in the niche to identify working design patterns, gaps, and failures.

#### Opening Question
> "Should I analyse specific competitors you know, or research the top 5–10 competitors in this niche?"

#### Follow-up Questions
1. "What do you want to do better than competitors — visual hierarchy, trust signals, CTA clarity, or mobile experience?"
2. "Are there any specific competitor websites you want me to analyse?"
3. "What design style do you want to avoid — corporate, generic, or outdated?"
4. "Should the site feel premium/luxury, or accessible/friendly?"
5. "What is the ONE action you want visitors to take on this site?"

#### Research & Analysis Instructions
After questions answered, Claude must:
- Identify top design patterns that work in this niche
- Document common mistakes competitors make
- Identify design gaps (things working competitors are missing)
- Extract colour palettes used successfully in this niche
- Note typography choices that resonate with the audience
- Analyse CTA placement and style that converts
- Compare mobile vs. desktop experience patterns

#### Output Format
```
## AGENT 2 OUTPUT — Competitive Design Analysis

### Design Patterns That Work in This Niche
[Bulleted list with brief reasoning]

### Common Competitor Mistakes
[Bulleted list]

### Design Gaps (Opportunities)
[Bulleted list — things competitors are failing to do]

### Colour Palettes Used Successfully
[Colour descriptions + hex codes where identifiable]

### Typography That Resonates
[Font style observations for headings, body, accents]

### CTA Placement & Style That Converts
[Specific observations]

### Mobile vs. Desktop Experience Notes
[Key differences observed]

### Primary Conversion Action
[The one action identified]
```

#### Handoff Statement
> "Analysis complete. Passing to Front End Design Agent with research and competitive insights. Primary opportunity: [X]. Design direction: [Y]."

---

### Agent 3 — Front End Design

**File:** `agent-3-frontend-design.md`
**Purpose:** Lock the visual design direction — custom build or template — and produce a production-ready design system using `ckm:design-system`.

#### Opening Question
> "Is there an existing website for this business that the design should be based on as a redesign?"

#### Branching: Existing Site
If YES:
- "Can you share the current website URL or describe its current design?"
- "What specifically needs to change — layout, colours, typography, or all of it?"
- "Are there any elements from the current site you want to keep?"

#### Next Question (all paths)
> "Should Claude design the front end custom itself, or should Claude research website templates from Framer.com or similar sites to find the best fit?"

#### Path A — Custom Build
Ask in sequence:
1. "What is the primary colour palette? (provide hex codes if possible, or describe the mood)"
2. "Should the design feel luxury/premium or accessible/friendly?"
3. "How many main sections should the homepage have?"
4. "What is the most important visual element — hero image, service grid, or testimonials?"
5. "Any specific animations or interactions? (CSS and vanilla JS only — no libraries)"

#### Path B — Template Research (Sub-Agent)
Claude activates a Template Research sub-process (requires WebSearch tool access):
- Search Framer.com and ThemeForest for templates matching the niche
- Score templates against market research data (audience, trust signals, design preferences from Agents 1–2)
- Score templates against competitive analysis (gaps, patterns, CTA needs)
- Present top 3–5 templates with: name, URL, score, reasoning
- Ask: "Which template direction appeals to you most, and what would you change?"

#### ui-ux-pro-max Integration (REQUIRED for all paths)
After design direction is confirmed, invoke `ckm:design-system` via the Skill tool.
Pass: colour answers, typography preferences, spacing complexity level.
Receive: three-layer token architecture (Primitive → Semantic → Component) output as **CSS custom properties only** — do NOT use Tailwind config or shadcn components, as HWD builds in vanilla HTML/CSS/JS. Extract the `:root {}` CSS variables block and Google Fonts `@import` only.

#### Output Format
```
## AGENT 3 OUTPUT — Front End Design

### Build Type
[Custom | Template: name]

### Colour System (CSS Custom Properties)
```css
:root {
  /* Primitive tokens */
  --color-brand-500: #XXXXXX;
  ...
  /* Semantic tokens */
  --color-primary: var(--color-brand-500);
  --color-background: ...;
  --color-text: ...;
  /* Component tokens */
  --btn-primary-bg: var(--color-primary);
  ...
}
```

### Typography Pairing
```css
@import url('https://fonts.googleapis.com/...');
:root {
  --font-heading: 'FontName', sans-serif;
  --font-body: 'FontName', sans-serif;
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;
}
```

### Homepage Layout (Sections in Order)
1. [Section name + purpose]
2. ...

### Visual Hierarchy Breakdown
[Description of primary, secondary, tertiary focal points]

### Interactions & Animations
[List of CSS/vanilla JS interactions with descriptions]

### Component List (Preview)
[High-level list — detailed in Agent 5]
```

#### Handoff Statement
> "Design direction locked. Colour system and typography generated via ckm:design-system. Passing to Visual Assets Agent."

---

### Agent 4 — Visual Assets

**File:** `agent-4-visual-assets.md`
**Purpose:** Specify every visual asset needed — images, icons, illustrations — without creating them.

#### Opening Question
> "Will the client provide their own images and photography, or should I specify what needs to be sourced?"

#### Follow-up Questions
1. "What photography style fits this niche — professional/posed or authentic/candid?"
2. "Should the design use icons, illustrations, or neither?"
3. "Are there specific brand assets to work around — logo, existing imagery, brand colours?"
4. "What image sizes and aspect ratios does the layout require?"

#### ckm:design Integration (Conditional)
If the opening question or brand assets question reveals the client has no logo or needs a logo redesign, invoke `ckm:design` via the Skill tool to produce a logo design brief and direction. This is the only point in the pipeline where logo work is initiated.

#### Output Format
```
## AGENT 4 OUTPUT — Visual Assets

### Images Required
| # | Type | Dimensions | Style | Purpose | Source |
|---|------|------------|-------|---------|--------|
| 1 | Hero background | 1920×1080 | [style] | [purpose] | [client/stock] |
...

### Icon Specifications
- Style: [line / filled / duotone]
- Size: [px]
- Colour: [CSS variable]
- Quantity: [n icons needed]
- Library recommendation: [if applicable]

### Illustrations / Graphics
[List or "None required"]

### Photography Style Guide
[Mood, lighting, subject, composition notes]

### Asset Delivery Format
- Images: WebP with JPEG fallback
- Icons: SVG
- Naming convention: [kebab-case, descriptive]
- Folder structure: /assets/images/, /assets/icons/
```

#### Handoff Statement
> "Asset specifications complete. [X] images, [Y] icons specified. Passing to Component Extraction Agent."

---

### Agent 5 — Component Extraction

**File:** `agent-5-component-extraction.md`
**Purpose:** Break the finalised design into reusable HTML/CSS components with a development checklist.

#### Opening Question
> "Should components be extracted based on the locked design, or do you want to adjust the design before we define components?"

#### Follow-up Questions
1. "Are there any custom components this niche specifically needs? (e.g. booking form, price calculator, before/after slider)"
2. "Should components be mobile-first or desktop-first?"
3. "Do you want component documentation generated for development handoff?"

#### Output Format
```
## AGENT 5 OUTPUT — Component Extraction

### Component Inventory (Build in This Order)
1. [ ] Base reset + CSS custom properties
2. [ ] Typography scale
3. [ ] Navbar (desktop + mobile hamburger)
4. [ ] Hero section
5. [ ] [component name]
...
N. [ ] Footer

### Component Specifications

#### [Component Name]
- Dimensions: [width, height, constraints]
- Spacing: [padding, margin values using CSS vars]
- States: [default, hover, active, focus, disabled]
- Breakpoints: [mobile / tablet / desktop behaviour]
- Notes: [any specific behaviour]

[Repeat for each component]

### CSS Class Structure
```
.c-navbar {}
.c-navbar__logo {}
.c-navbar__links {}
.c-hero {}
.c-hero__heading {}
...
```
(BEM convention: .c-[component]__[element]--[modifier])

### Development Checklist
[ ] Set up file structure
[ ] Implement CSS custom properties
[ ] Build [component 1]
...
[ ] Cross-browser test
[ ] Accessibility audit
```

#### Handoff Statement
> "Components defined. [N] components in build checklist. Passing to Performance Specification Agent."

---

### Agent 6 — Performance Specification

**File:** `agent-6-performance.md`
**Purpose:** Set performance targets and optimisation strategy before development begins.

#### Opening Question
> "Should this site target 85+ PageSpeed mobile score? (HWD standard — answer is yes unless specified otherwise)"

#### Follow-up Questions
1. "What is the expected image count and approximate total page weight?"
2. "Will Google Fonts be loaded from CDN or self-hosted?"
3. "Are there any animations or interactions that might impact performance?"
4. "Should the site be optimised for older mobile devices or modern only?"

#### Output Format
```
## AGENT 6 OUTPUT — Performance Specification

### PageSpeed Target
- Mobile: 85+
- Desktop: 95+

### Image Optimisation Strategy
- Format: WebP primary, JPEG fallback
- Compression: [quality level]
- Lazy loading: All images below fold use loading="lazy"
- Hero image: preloaded via <link rel="preload">
- Max dimensions: [per image type]

### Font Loading Strategy
- Provider: [Google Fonts CDN | self-hosted]
- Method: @import in CSS with font-display: swap
- Preconnect: <link rel="preconnect" href="https://fonts.googleapis.com">
- Subset: [latin only unless otherwise needed]

### Netlify Caching Rules (_headers file)
```
/assets/*
  Cache-Control: public, max-age=31536000, immutable
/*.css
  Cache-Control: public, max-age=86400
/*.js
  Cache-Control: public, max-age=86400
```

### Minification Requirements
- CSS: Minified at build (no build tool = manual or VS Code extension)
- JS: Minified at build
- HTML: Whitespace removed

### Performance Budget
| Resource | Budget |
|----------|--------|
| Total page weight | < [X]KB |
| Images (total) | < [X]KB |
| CSS | < 30KB |
| JS | < 20KB |
| Fonts | < 50KB |

### Testing Checklist
- [ ] PageSpeed Insights (mobile + desktop)
- [ ] GTmetrix
- [ ] WebPageTest
- [ ] Chrome DevTools Lighthouse
- [ ] Test on real mobile device
```

#### Final Pipeline Statement
> "Performance specifications locked. Pipeline complete. All context ready for development phase."

---

## 6. Pipeline Complete Document

After Agent 6 fires, Claude outputs the full `PROJECT_CONTEXT` as a formatted development brief:

```
# HWD PROJECT BRIEF — [Client Name]
Generated: [date]

## 1. Project Overview
[Niche, client, primary conversion action]

## 2. Target Audience
[From Agent 1]

## 3. Design Decisions
[Build type, colour system, typography — from Agent 3]

## 4. Asset List
[From Agent 4]

## 5. Component Build Order
[From Agent 5]

## 6. Performance Targets
[From Agent 6]

## 7. Development Notes
[Any cross-agent observations or conflicts resolved]
```

---

## 7. Constraints & Principles

- **CSS + vanilla JS only.** No frameworks, no libraries, no build tools (unless client project requires otherwise).
- **Mobile-first breakpoints** unless Agent 5 specifies desktop-first.
- **Netlify deployment** assumed — caching rules formatted accordingly.
- **HWD PageSpeed standard: 85+ mobile** — this is the default, not an option.
- **No code generation during pipeline.** Spec only until Pipeline Complete fires.
- **One question per message.** Hard rule, no exceptions.
- **Context never lost.** `PROJECT_CONTEXT` carried across all 6 agents in the same conversation.

---

## 8. Out of Scope

- Actual HTML/CSS/JS code generation (happens after pipeline, in development phase)
- CMS integration (separate pipeline not covered here)
- E-commerce functionality (separate pipeline)
- SEO meta content (separate agent to be added in v2)
- Copywriting (separate agent to be added in v2)
