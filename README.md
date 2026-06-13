# Shopify Theme Audit Skill for Claude Code

> Drop eight files into your Shopify project. Ask Claude Code to audit the theme. Get a scored report covering performance, accessibility, conversion, third-party app overhead, SEO, AEO (ChatGPT / Claude / Perplexity citations), and GEO (AI Overviews) — with exact file references, line numbers, and working code fixes, in under five minutes.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Works with Claude Code](https://img.shields.io/badge/Works%20with-Claude%20Code-orange)](https://claude.ai/code)
[![Shopify](https://img.shields.io/badge/Shopify-Online%20Store%202.0-96BF48)](https://shopify.dev)

---

## What this is

A **Claude Code skill** — a set of markdown instruction files that teach Claude Code to audit Shopify themes the way a senior Shopify developer does.

Once installed, a single prompt gives you a structured audit report with:

- A score out of 100 with a letter grade (or two split scores: Technical/CRO and SEO/AEO/GEO)
- Every issue found, with the exact file name and line number
- Working Liquid code fixes for each issue
- A prioritized fix order with impact estimates
- Estimated Lighthouse, Core Web Vitals, and search-visibility improvements after fixes

No API calls, no SaaS, no subscription. It runs inside Claude Code using your existing subscription. Works fully offline against local theme files, and works with the live Shopify MCP for in-store data when you want it.

---

## What changed in this version

- **SEO / AEO / GEO audits added.** A dedicated `seo-aeo-geo-checklist.md` covers traditional search, Answer Engine Optimization (ChatGPT, Claude, Perplexity, Gemini citations), and Generative Engine Optimization (Google AI Overviews, Bing Copilot, LLM-powered shopping).
- **35+ new checks** spanning meta tags, hreflang, FAQ schema, HowTo schema, Speakable, Organization schema, AI crawler accessibility (GPTBot, ClaudeBot, PerplexityBot), llms.txt, factual product summaries, machine-readable specifications, and policy schema.
- **Third-party app overhead audit** — a new `apps-audit.md` detects and scores 20+ common Shopify apps (Klaviyo, Judge.me, Loox, Yotpo, Rebuy, Gorgias, Recharge, Smile, LoyaltyLion, and more) by loading strategy, bundle size, and above-fold impact. Most stores die from app bloat, not theme bloat — this catches it.
- **Before / After code gallery** — `before-after.md` provides strict `WRONG → RIGHT` Liquid pairs keyed by check ID. Claude pastes exact working code into the report instead of vague descriptions.
- **Quick-Wins mode** — say "what should I fix first" or "biggest bang for buck" and the report leads with a ranked Impact ÷ Effort table before the full breakdown.
- **Split-score mode** — request an SEO-only or AEO-only audit and get two independent scores.
- Updated to assume **Shopify MCP is live** for live-store reads. No "coming soon" placeholders.

---

## Why AEO and GEO matter for Shopify now

Search has split into three layers:

1. **Classic SEO** — Google, Bing, DuckDuckGo. Still the largest traffic source for most stores.
2. **AEO — Answer Engine Optimization** — ChatGPT, Claude, Perplexity, and Gemini increasingly answer product-research questions directly. The user never sees a results page. If your product page is not structured to be quotable, you are invisible at the moment of decision.
3. **GEO — Generative Engine Optimization** — Google's AI Overviews, Bing Copilot, and LLM-powered shopping assistants either cite your store or do not. Citation depends on schema completeness, crawlability for AI bots, and machine-readable trust signals.

This skill audits all three.

---

## Real output example

An actual report this skill produced on a live Shopify theme:

```
Theme: Bloom (Dawn-based, Online Store 2.0)
Mode: Full audit
Technical & CRO Score: 69/100 — Grade: C
SEO / AEO / GEO Score:  58/100 — Grade: D

Critical Issues — Technical (1)

C6: Images missing height attribute (CLS)
Files: sections/header.liquid:192,237 — sections/bloom-hero.liquid:18,44
Impact: Visible page jump on first load. Header logo shifts as it loads.
Fix:
<img src="{{ 'logo.png' | asset_url }}"
     alt="{{ shop.name }}"
     width="180"
     height="60"
     fetchpriority="high">

Critical Issues — SEO / AEO / GEO (3)

S-C2: Meta description missing on collection pages — layout/theme.liquid:34
AEO-C1: FAQ accordion present but no FAQPage schema — sections/faq.liquid:1-80
GEO-C1: robots.txt.liquid blocks GPTBot, ClaudeBot, PerplexityBot — templates/robots.txt.liquid:12-18

High Priority (5)
H3: No preload hint for LCP hero image — LCP +300–800ms on mobile
H4: base.css at 81KB loaded on every page
S-H1: Header logo wrapped in <h1> on every page — competes with content H1
AEO-H1: Product specs buried in prose, no PropertyValue schema
GEO-H1: Organization schema missing — no sameAs social profile links

What This Theme Does Well (9)
- All JS uses defer — zero render-blocking first-party scripts
- Skip-to-content link present as first focusable element
- Online Store 2.0 throughout — no deprecated img_url or {% include %}
- Product structured data via {{ product | structured_data }}
- Cart drawer implements trapFocus() for keyboard accessibility
- Hreflang correctly emitted for all five Shopify Markets locales
- Canonical handles ?variant= and pagination correctly
- dateModified present on article schema
[...]

Estimated Impact After Fixes
Lighthouse Performance: +8 to +15 points
LCP: -400 to -800ms
CLS: substantial reduction
AEO citations: meaningful uplift — FAQ schema + factual summaries are highest leverage
GEO visibility: allowing AI crawlers + Organization schema unlocks brand-disambiguation in LLM answers
```

[Full example report →](examples/sample-audit-report.md)

---

## What it checks

### Technical, accessibility, and CRO — `audit-checklist.md`

| Severity | Checks | Points each |
|---|---|---|
| Critical | C1–C6: render-blocking scripts, deprecated `img_url`, no lazy loading, N+1 queries, missing skip link, missing image dimensions | −10 |
| High | H1–H7: JS without `defer`, no WebP, no hero preload, oversized global CSS, no focus trap, canonical issues, heavy app blocks | −5 |
| Medium | M1–M7: repeated Liquid expressions, missing alt text, inline styles, no product structured data, no breadcrumbs, poor zero-results state, unbranded password page | −2 |
| Low | L1–L6: `console.log` in prod, commented-out code, unminified assets, no CSS custom properties, bare 404 page, footer without trust elements | −1 |

### SEO, AEO, and GEO — `seo-aeo-geo-checklist.md`

| Group | What it covers |
|---|---|
| **SEO Critical (S-C1 → S-C5)** | `<title>`, meta description, canonical, sitemap accessibility, Product JSON-LD |
| **SEO High (S-H1 → S-H7)** | Multiple H1s, heading hierarchy, Open Graph, Twitter Card, descriptive alt text, hreflang, pagination handling |
| **SEO Medium (S-M1 → S-M6)** | Title/description length, URL handle quality, internal linking from product pages, robots meta on non-indexable pages, semantic logo markup |
| **AEO Critical (AEO-C1, AEO-C2)** | FAQ schema missing, product pages opening with marketing fluff instead of factual definitions |
| **AEO High (AEO-H1 → AEO-H4)** | Specs not structured as `PropertyValue`, no HowTo schema, no Speakable hints, no author/publisher (E-E-A-T) signals |
| **AEO Medium (AEO-M1 → AEO-M3)** | Missing `dateModified`, no individual `Review` schema, no comparison content |
| **GEO Critical (GEO-C1, GEO-C2)** | AI crawlers blocked in robots.txt, no `llms.txt` overview |
| **GEO High (GEO-H1 → GEO-H4)** | Organization schema absent, BreadcrumbList missing, critical content JS-rendered, thin brand description |
| **GEO Medium (GEO-M1 → GEO-M3)** | No shipping/returns schema in offers, no machine-readable materials/origin, undescriptive image URLs |

It also generates a **"What This Theme Does Well"** section so the report is credible — not just a list of problems.

---

## Installation

**Step 1.** Clone or download this repo.

```bash
git clone https://github.com/tanujrajputdev/shopify-theme-audit-skill.git
```

**Step 2.** Copy the skill files into the root of your Shopify theme project.

```
your-shopify-theme/
├── CLAUDE.md                       ← copy here
├── audit-checklist.md              ← copy here
├── seo-aeo-geo-checklist.md        ← copy here
├── apps-audit.md                   ← copy here
├── liquid-patterns.md              ← copy here
├── before-after.md                 ← copy here
├── deprecated-apis.md              ← copy here
├── scoring.md                      ← copy here
├── layout/
├── sections/
├── snippets/
└── ...
```

**Step 3.** Open the project in Claude Code. That is the complete setup.

---

## Usage

With the files in place, open Claude Code and pick the prompt that matches what you want.

**Full audit (default — performance, accessibility, CRO, SEO, AEO, GEO):**

```
Audit this Shopify theme using the audit skill. Cover performance,
accessibility, conversion, SEO, AEO, and GEO. Start with layout/theme.liquid
and work through sections/ and snippets/.
```

**SEO + AEO + GEO only:**

```
Run an SEO, AEO, and GEO audit on this theme using the audit skill.
Use split-score mode. Skip the performance-only sections.
```

**Quick wins — what should I fix first:**

```
Run the audit skill in quick-wins mode. Show me the top 10 fixes ranked by
impact divided by effort, then the full report.
```

**Third-party app overhead:**

```
Audit only third-party app overhead using the audit skill. Use apps-audit.md.
List every app detected, its loading strategy, and its measurable cost.
```

**One specific area:**

```
Audit only the product page for AEO — focus on Product schema, FAQ schema,
factual summary opening, and specification structure.
```

**For large themes**, audit in two passes to avoid token limits:

```
Pass 1: "Audit layout/ and sections/. Use the audit skill, full coverage."
Pass 2: "Continue — check snippets/ and templates/. Add to the previous report."
```

---

## Auditing a live store with the Shopify MCP

The Shopify MCP server is live. If you have it connected in Claude Code, you can audit a store without cloning the theme locally — Claude can read shop info, products, collections, and run analytics queries directly.

Typical prompt:

```
Connect to my Shopify store via the MCP. Pull shop info and a sample product
page. Then run the full audit skill against what you see. For checks that
require theme file access (Liquid source), note which checks were skipped.
```

When MCP-only auditing, expect SEO/AEO/GEO output-level checks (rendered HTML, schema in DOM, robots.txt, sitemap, meta tags) to be reliable, and source-level checks (Liquid patterns, file structure, `img_url` usage) to be skipped unless you also provide theme files.

For best results, do both: connect the MCP for live store context, and put the theme files in the project directory for source-level checks.

---

## Adding to an existing CLAUDE.md

If you already have a `CLAUDE.md` in your project, append this block instead of replacing it:

```markdown
## Shopify Theme Audit

When asked to audit this theme — for performance, accessibility, CRO, SEO,
AEO, GEO, or app overhead — read the following skill files in order:

- audit-checklist.md
- seo-aeo-geo-checklist.md
- apps-audit.md
- liquid-patterns.md
- before-after.md
- deprecated-apis.md
- scoring.md

Then follow the audit methodology in those files exactly. Always include
file names and line numbers for every finding, and paste the exact RIGHT
block from before-after.md as the fix — do not paraphrase.
```

---

## Files in this repo

| File | What it contains |
|---|---|
| `CLAUDE.md` | Master instructions — how Claude Code runs the audit, step by step |
| `audit-checklist.md` | Performance, accessibility, and CRO checks with severity ratings and locations |
| `seo-aeo-geo-checklist.md` | SEO, Answer Engine Optimization, and Generative Engine Optimization checks |
| `apps-audit.md` | Third-party app overhead — Klaviyo, Judge.me, Loox, Yotpo, Rebuy, Gorgias, Recharge, and 15+ more |
| `liquid-patterns.md` | 15+ before/after code examples: images, metafields, schemas, accessibility, structured data |
| `before-after.md` | Strict WRONG → RIGHT code gallery keyed by check ID — the exact fixes the audit pastes into reports |
| `deprecated-apis.md` | Every deprecated Shopify API with exact current replacements |
| `scoring.md` | 0–100 scoring formula, grade thresholds, full / split-score / quick-wins modes, report format |
| `examples/` | Three sample audit reports — Dawn custom (C grade), legacy 1.0 (F grade), clean Dawn (A grade) |

---

## What you can do with this skill

- **Pre-launch theme review** — run before going live to catch regressions
- **Agency client reports** — produce a credible, sourced audit deliverable in minutes instead of days
- **Migration QA** — audit a newly migrated theme against Online Store 2.0 expectations
- **Pre-Black-Friday hardening** — find the LCP, CLS, and render-blocking issues before peak traffic
- **AI-search readiness** — confirm the store can be cited by ChatGPT, Claude, Perplexity, and Google AI Overviews
- **Structured data sweep** — verify Product, FAQ, BreadcrumbList, Organization, Article, and HowTo schemas across the catalog
- **Robots.txt review** — confirm the merchant's intent for AI crawlers is reflected in `templates/robots.txt.liquid`
- **Section-by-section review** — point Claude at one section file for a focused audit

---

## Contributing

Found a deprecated API that is not in the list? A new schema pattern that should be covered? A check that misses an edge case? PRs welcome.

- Open an issue describing the check you want added
- PRs should follow the existing format in `audit-checklist.md` or `seo-aeo-geo-checklist.md`
- All checks must include: where to look, what to find, why it matters, and a code fix

See `CONTRIBUTING.md` for details.

---

## Who built this

**Tanuj Rajput** — Shopify developer and founder of [EcomLifters](https://ecomlifters.com), a Shopify development agency. Built custom themes and apps for 32+ brands across India and Canada over 5+ years.

- Website: [tanujrajput.com](https://tanujrajput.com)
- X: [@tanujbuilds](https://x.com/tanujbuilds)
- LinkedIn: [linkedin.com/in/tanuj-rajput](https://www.linkedin.com/in/tanuj-rajput/)
- Agency: [ecomlifters.com](https://ecomlifters.com)

---

## License

MIT — use it, modify it, sell audits with it. Attribution appreciated but not required.

---

*If this saved you time on a theme audit, a star helps others find it.*
