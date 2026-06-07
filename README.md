# Shopify Theme Audit Skill for Claude Code

> Drop five files into your Shopify project. Ask Claude Code to audit the theme. Get a scored report with exact file references, line numbers, and working code fixes in under 5 minutes.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Works with Claude Code](https://img.shields.io/badge/Works%20with-Claude%20Code-orange)](https://claude.ai/code)
[![Shopify](https://img.shields.io/badge/Shopify-Online%20Store%202.0-96BF48)](https://shopify.dev)

---

## What this is

A **Claude Code skill** — a set of markdown instruction files that teach Claude Code how to audit Shopify themes the way a senior Shopify developer does.

Once installed, one prompt gives you a structured audit report with:
- A score out of 100 with a letter grade
- Every issue found, with the exact file name and line number
- Working Liquid code fixes for each issue
- A prioritized fix order with time estimates
- Estimated Lighthouse impact after fixes

No API calls, no SaaS, no subscription. It runs inside Claude Code using your existing subscription.

---

## Real output example

Here is an actual report this skill produced on a live Shopify theme:

```
Theme: Bloom (Dawn-based, Online Store 2.0)
Score: 69/100 — Grade: C

Critical Issues (1)

C6: Images missing height attribute (CLS)
Files: sections/header.liquid:192,237 — sections/bloom-hero.liquid:18,44
Impact: Visible page jump on first load. Header logo shifts as it loads.
Fix:
<img src="{{ 'logo.png' | asset_url }}"
     alt="{{ shop.name }}"
     width="180"
     height="60"
     fetchpriority="high">

High Priority (3)
H3: No preload hint for LCP hero image — LCP +300–800ms on mobile
H-custom: Render-blocking Google Fonts + Hugeicons CDN in <head>
H4: base.css at 81KB loaded on every page

What This Theme Does Well (8)
- All JS uses defer — zero render-blocking first-party scripts
- Skip-to-content link present as first focusable element
- Online Store 2.0 throughout — no deprecated img_url or {% include %}
- Product structured data via {{ product | structured_data }}
- Cart drawer implements trapFocus() for keyboard accessibility
[...]

Estimated Impact After Fixes
Lighthouse Performance: +8 to +15 points
LCP: -400 to -800ms
CLS: substantial reduction
```

[Full example report →](examples/sample-audit-report.md)

---

## What it checks

**22 checks across four severity levels:**

| Severity | Checks | Points each |
|---|---|---|
| Critical | C1–C6: render-blocking scripts, deprecated img_url, no lazy loading, N+1 queries, missing skip link, missing image dimensions | −10 |
| High | H1–H7: JS without defer, no WebP, no hero preload, oversized global CSS, no focus trap, canonical issues, heavy app blocks | −5 |
| Medium | M1–M7: repeated Liquid expressions, missing alt text, inline styles, no structured data, no breadcrumbs, poor zero-results state, unbranded password page | −2 |
| Low | L1–L6: console.log in prod, commented-out code, unminified assets, no CSS custom properties, bare 404 page, footer without trust elements | −1 |

It also generates a **"What This Theme Does Well"** section — so the report is credible, not just a list of problems.

---

## Installation

**Step 1:** Clone or download this repo

```bash
git clone https://github.com/tanujrajputdev/shopify-theme-audit-skill.git
```

**Step 2:** Copy the skill files into your Shopify theme project

```
your-shopify-theme/
├── CLAUDE.md                ← copy here
├── audit-checklist.md       ← copy here
├── liquid-patterns.md       ← copy here
├── deprecated-apis.md       ← copy here
├── scoring.md               ← copy here
├── layout/
├── sections/
├── snippets/
└── ...
```

**Step 3:** Open the project in Claude Code. That is the complete setup.

---

## Usage

With the files in place, open Claude Code and say:

```
Audit this Shopify theme for performance, accessibility, and conversion issues.
Start with layout/theme.liquid and work through all files in sections/ and snippets/.
```

Claude Code will read your theme files, run every check from the checklist, calculate the score, and return a formatted report.

**For large themes**, audit in two passes to avoid token limits:

```
Pass 1: "Audit layout/ and sections/ only. Use the audit skill."
Pass 2: "Continue the audit — check snippets/ and assets/. Add findings to the previous report."
```

---

## What Claude Code needs

Claude Code needs file access to your theme directory. If you are using the [Shopify MCP Server](https://github.com/tanujrajputdev/shopify-mcp-server) *(coming soon)*, Claude Code can read theme files directly from your live store without a local clone.

---

## Adding to an existing project

If you already have a `CLAUDE.md` in your project, add this to the bottom instead of replacing it:

```markdown
## Shopify Theme Audit

When asked to audit this theme, read the following skill files:
- audit-checklist.md
- liquid-patterns.md
- deprecated-apis.md
- scoring.md

Then follow the audit methodology in those files exactly.
```

---

## Files in this repo

| File | What it contains |
|---|---|
| `CLAUDE.md` | Master instruction — how Claude Code runs the audit, step by step |
| `audit-checklist.md` | All 22 checks with severity ratings, what to look for, where to look |
| `liquid-patterns.md` | 15 before/after code examples: images, metafields, schemas, accessibility, structured data |
| `deprecated-apis.md` | Every deprecated Shopify API with exact current replacements |
| `scoring.md` | 0–100 scoring formula, grade thresholds, report format |
| `examples/` | Sample audit reports from real themes |

---

## Shopify Developer Bundle

The full Developer Bundle includes three additional skills:

**Liquid Code Quality** — Write correct, modern Shopify Liquid. Section schemas, snippet conventions, performance rules, translation strings, CSS architecture.

**Shopify App Development** — Build Shopify apps with the Remix template. Authentication, GraphQL in apps, Polaris components, webhooks, Billing API, App Store submission checklist.

**GraphQL Query Library** — 40+ production-ready queries for products, metafields, collections, orders, customers, inventory, themes, and bulk operations.

[Get the Developer Bundle →](https://gumroad.com/tanujrajputdev) *(₹4,499)*

---

## Contributing

Found a deprecated API that is not in the list? A Liquid pattern that should be covered? PRs welcome.

- Open an issue describing the check you want added
- PRs should follow the existing format in `audit-checklist.md`
- All checks must include: where to look, what to find, why it matters, and a code fix

---

## Who built this

**Tanuj Rajput** — Shopify developer and founder of [EcomLifters](https://ecomlifters.com), a Shopify development agency. Built custom themes and apps for 32+ brands across India and Canada over 5+ years.

- Website: [tanujrajput.dev](https://tanujrajput.dev)
- X: [@tanujrajput](https://x.com/tanujrajput)
- LinkedIn: [linkedin.com/in/tanujrajput](https://linkedin.com/in/tanujrajput)
- Agency: [ecomlifters.com](https://ecomlifters.com)

---

## License

MIT — use it, modify it, sell audits with it. Attribution appreciated but not required.

---

*If this saved you time on a theme audit, a star helps others find it.*
