# Architecture — File Map

## How the skill works

`CLAUDE.md` is the entry point. When Claude Code loads a project, it reads `CLAUDE.md` automatically. That file contains the complete audit methodology and tells Claude to read the other files in order before starting an audit.

## File-by-file breakdown

### `CLAUDE.md` — Master controller
The skill's brain stem. Contains:
- Trigger conditions (when to use this skill)
- Ordered list of reference files to read before auditing
- The five-step audit process (discover → read → check → score → format)
- Exact output format structure
- Hard rules that must not be broken
- Common mistakes to avoid

Claude reads this automatically on project open. All other files are read on demand when an audit is triggered.

---

### `audit-checklist.md` — Technical and CRO checks
The complete list of performance, accessibility, and conversion-rate issues. Organized by severity:

- **Critical (C1–C6):** render-blocking scripts, deprecated `img_url`, missing lazy loading, N+1 queries in loops, missing skip-to-content link, missing image dimensions causing CLS
- **High (H1–H7):** JS without defer, no WebP, no LCP hero preload, oversized global CSS, no focus trap in modals/drawers, canonical issues, heavy third-party app blocks
- **Medium (M1–M7):** repeated Liquid expressions, missing alt text, inline styles, no product structured data, no breadcrumbs, poor zero-results state, unbranded password page
- **Low (L1–L6):** console.log in production, commented-out code, unminified assets, no CSS custom properties, bare 404 page, footer without trust elements

---

### `seo-aeo-geo-checklist.md` — Search visibility checks
Three layers of search optimization. Each check has an ID, a location to look, a signal to detect, and why it matters.

- **SEO Critical (S-C1 → S-C5):** title tag, meta description, canonical, sitemap, Product JSON-LD
- **SEO High (S-H1 → S-H7):** multiple H1s, heading hierarchy, Open Graph, Twitter Card, alt text, hreflang, pagination
- **SEO Medium (S-M1 → S-M6):** title/description length, URL handles, internal linking, robots meta, semantic logo
- **AEO Critical (AEO-C1, AEO-C2):** FAQ schema missing on FAQ blocks, product pages opening with marketing fluff instead of factual definitions
- **AEO High (AEO-H1 → AEO-H4):** specs not structured as PropertyValue, no HowTo schema, no Speakable, no E-E-A-T signals
- **AEO Medium (AEO-M1 → AEO-M3):** missing dateModified, no individual Review schema, no comparison content
- **GEO Critical (GEO-C1, GEO-C2):** AI crawlers blocked in robots.txt (GPTBot, ClaudeBot, PerplexityBot), no llms.txt
- **GEO High (GEO-H1 → GEO-H4):** Organization schema absent, BreadcrumbList missing, critical content JS-rendered, thin brand description
- **GEO Medium (GEO-M1 → GEO-M3):** no shipping/returns schema in Offers, no machine-readable materials/origin, undescriptive image URLs

---

### `apps-audit.md` — Third-party app overhead
Detects and scores 20+ common Shopify apps by loading strategy, bundle size, and above-fold impact. Apps audited include: Klaviyo, Judge.me, Loox, Yotpo, Rebuy, Gorgias, Recharge, Smile, LoyaltyLion, and more.

Key insight encoded here: most stores underperform because of app bloat, not theme code quality.

---

### `liquid-patterns.md` — Correct code patterns
15+ before/after code examples covering:
- Image tags (img_url vs image_url, lazy loading, width/height attributes)
- Metafield access patterns
- Schema/structured data patterns
- Accessibility patterns (skip links, focus traps, aria labels)
- Structured data (Product JSON-LD, FAQ, BreadcrumbList)

Used by Claude to recognize incorrect patterns in theme files.

---

### `before-after.md` — Working code fixes gallery
Strict `WRONG → RIGHT` Liquid code pairs keyed by check ID (e.g. `C2`, `H3`, `GEO-C1`).

When Claude finds an issue and flags it in the report, it looks up the check ID here and pastes the RIGHT block verbatim into the report's Fix section. This prevents vague descriptions like "add lazy loading" in favor of exact, copy-pasteable code.

---

### `deprecated-apis.md` — Deprecated API replacements
Every deprecated Shopify Liquid filter and tag with its exact current replacement and a complete working example. Covers img_url → image_url, include → render, and others.

---

### `scoring.md` — Score calculation and report format
- Deduction table: Critical −10, High −5, Medium −2, Low −1
- Grade thresholds: A (90–100), B (75–89), C (60–74), D (40–59), F (0–39)
- Score modifiers: +5 bonus for zero Critical issues, −5 penalty for more than 4 Critical issues
- Three audit modes: Full, Split-score, Quick-wins
- Report output format: exact structure Claude must follow
- Estimated impact section format

---

### `examples/` — Sample audit reports
Three sample reports showing what output looks like at different quality levels:
- Dawn custom (C grade)
- Legacy OS 1.0 theme (F grade)
- Clean Dawn (A grade)

Used to set quality expectations and show users what they will receive.

## Installation path

User copies all files from repo root into the root of their Shopify theme directory. `CLAUDE.md` must be at the project root so Claude Code auto-reads it. All other files must be in the same directory as `CLAUDE.md` (relative paths are used internally).

## Dependency graph

```
CLAUDE.md
  ├── audit-checklist.md
  ├── seo-aeo-geo-checklist.md
  ├── apps-audit.md
  ├── liquid-patterns.md
  ├── before-after.md  ← keyed to check IDs in audit-checklist.md + seo-aeo-geo-checklist.md
  ├── deprecated-apis.md
  └── scoring.md
```

`before-after.md` is the only file with hard coupling to specific check IDs. If you add a new check to `audit-checklist.md`, add a matching WRONG → RIGHT pair in `before-after.md` with the same check ID.
