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
| Total page weight (all assets) | < [calculated from image count x avg size + CSS + JS + fonts] KB |
| All images combined | < [total image budget based on image count] KB |
| CSS (minified) | < 30KB |
| JS (minified) | < 20KB |
| Fonts (WOFF2) | < 50KB per family |

[Calculate total page weight as: (hero images × 120KB) + (content images × 40KB) + 30KB CSS + 20KB JS + (font families × 50KB). Use this calculated figure for the "Total page weight" row. Use the image subtotal for the "All images combined" row.]

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

---

## Final Pipeline Statement

End with this exact statement:

> "Performance specifications locked. Pipeline complete. All context ready for development phase."

Then immediately produce the full **HWD PROJECT BRIEF** by merging all PROJECT_CONTEXT sections (from Agents 1–6) into the brief format defined in the orchestrator SKILL.md.
