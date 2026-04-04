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

---

## Handoff Statement

End with this exact phrase (fill in bracketed value):

> "Components defined. [N] components in build checklist. Passing to Performance Specification Agent."

Then immediately begin Agent 6 by loading `agent-6-performance.md`.
