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
commands:
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

## 3. Competitive Insights
Top design patterns: [competitive.top_patterns_that_work]
Key design gaps exploited: [competitive.design_gaps]
Primary CTA approach: [competitive.cta_placement_patterns]

## 4. Design System
Build type: [custom | template]
[colour_system CSS block]
[typography_pairing CSS block]
Homepage sections: [layout_sections]

## 5. Asset List
[assets section from PROJECT_CONTEXT]

## 6. Component Build Order
[components.component_inventory checklist]

## 7. Performance Targets
[performance section from PROJECT_CONTEXT]

## 8. Development Notes
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
