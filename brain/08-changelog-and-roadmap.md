# Changelog and Roadmap

## Version history

### v2.0 — June 2026 (current)

The audit went from a single performance + accessibility checklist to a full search-era audit covering eight surfaces: performance, accessibility, CRO, third-party app overhead, classic SEO, AEO, GEO, and structured data depth.

**Added in v2.0:**

1. `seo-aeo-geo-checklist.md` — 35+ new checks across SEO, AEO, GEO
   - SEO: title, meta, canonical, sitemap, JSON-LD, H1s, heading hierarchy, OG, Twitter Cards, alt text, hreflang, pagination, length, URL handles, internal linking, robots meta, logo
   - AEO: FAQ schema, factual summaries, PropertyValue specs, HowTo, Speakable, E-E-A-T, dateModified, Review schema, comparison content
   - GEO: AI crawler access (GPTBot, ClaudeBot, PerplexityBot, Google-Extended, CCBot), llms.txt, Organization schema, BreadcrumbList, shipping/returns schema, materials/origin, image URLs

2. `apps-audit.md` — 20+ Shopify app detection (Klaviyo, Judge.me, Loox, Yotpo, Rebuy, Gorgias, Recharge, Smile, LoyaltyLion, and more)

3. `before-after.md` — 25 strict WRONG → RIGHT Liquid code pairs keyed by check ID

4. Quick-wins mode in `scoring.md` — ranks by impact ÷ effort, leads report with star-rated table

5. Split-score mode in `scoring.md` — two independent scores: Technical & CRO vs SEO/AEO/GEO

6. Three sample audit reports in `examples/`:
   - Dawn custom build — C grade
   - Legacy OS 1.0 theme — F grade (migration prep)
   - Clean Dawn — A grade with Quick-Wins mode demo

7. Landing page in `landing/` — static marketing site (Tailwind CDN, Inter Tight + Instrument Serif)

**Removed in v2.0:**
- "Shopify MCP coming soon" placeholder (MCP is live)
- Developer Bundle / Gumroad section (skill is free + open source)

**Rewritten in v2.0:**
- `README.md` — full rewrite with AEO/GEO context, split-score sample, eight-file install tree
- `CLAUDE.md` — read order expanded from 5 to 7 files, quick-wins trigger phrases added
- `deprecated-apis.md` — updated with current replacements

---

### v1.0 — 2025

Initial release. Performance, accessibility, and CRO checks only.

Files: `CLAUDE.md`, `audit-checklist.md`, `liquid-patterns.md`, `deprecated-apis.md`, `scoring.md`
No SEO/AEO/GEO, no app overhead, no before-after gallery, no quick-wins mode, no split scores.

---

## What to build next (potential roadmap)

These are ideas — none are committed. Evaluate each against the core constraint: the skill must remain simple to install (copy files, no setup).

### High value, low complexity
- **More `before-after.md` pairs** — currently 25; every check ID should have a matching WRONG → RIGHT pair. Gap-fill starting from SEO and GEO checks.
- **More `apps-audit.md` entries** — Northbeam, Triple Whale, Elevar, Black Crow AI, Enquire Labs are not covered yet
- **Check C7: no `fetchpriority="high"` on LCP image** — distinct from H3 (preload); this is the newer complementary signal

### Medium value, medium complexity
- **`performance-budgets.md`** — per-template weight budgets (home page, product, collection) with pass/fail thresholds instead of just flagging large assets
- **Accessibility checklist expansion** — WCAG 2.2 AA gaps: focus-visible on custom components, touch target size, reflow at 400% zoom
- **`metafield-patterns.md`** — common metafield namespace patterns for product specs, size guides, FAQs; audit checks that reference these would flag when metafields exist but are not rendered in schema

### Lower priority / needs validation
- **Automated score diff mode** — audit the same theme twice (before/after a sprint) and output only what changed. Complex to implement as a pure instruction file.
- **Claude Code marketplace listing** — distributes the skill without requiring GitHub. Needs marketplace to be available.
- **Companion scheduled audit** — runs the skill weekly via a Claude Code hook and emails a diff report. Requires Claude Code hooks + external email trigger.
