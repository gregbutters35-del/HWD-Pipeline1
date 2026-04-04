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

## AGENT 3 OUTPUT — Front End Design

### Build Type
[Custom build | Template: [template name and URL]]

### Colour System (CSS Custom Properties)
[Paste the full :root {} block from ckm:design-system output here]

Example structure:
```css
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
```

### Typography Pairing
[Google Fonts @import + CSS variables]

```css
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
```

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

---

## Handoff Statement

End with this exact phrase:

> "Design direction locked. Build type: [custom build | template: [template name]]. Colour system and typography generated via ckm:design-system. Passing to Visual Assets Agent."

Then immediately begin Agent 4 by loading `agent-4-visual-assets.md`.
