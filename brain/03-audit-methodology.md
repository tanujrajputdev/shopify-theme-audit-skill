# Audit Methodology

## The five steps (in order, always)

### Step 1 — Discover the theme structure
- List all files in `templates/`, `sections/`, `snippets/`, `layout/`, `assets/`
- Identify OS 1.0 vs 2.0: check `config/settings_schema.json` for 2.0 markers
- Check `layout/theme.liquid` for base HTML structure
- Note which templates exist: product, collection, page, blog, article, index, cart, search

### Step 2 — Read critical files first (in this order)
1. `layout/theme.liquid` — head structure, script loading, global CSS
2. `templates/product.json` or `templates/product.liquid` — the product template
3. `sections/main-product.liquid` — the primary product section
4. `sections/header.liquid` — navigation and above-fold elements
5. All snippets referenced more than 3 times across the theme

### Step 3 — Run every check
- Go through `audit-checklist.md` and `seo-aeo-geo-checklist.md` systematically
- Do not skip checks — every check runs on every audit
- For each issue found: record file name, line number, severity, specific bad code, correct fix
- Mobile and desktop are checked separately — some issues only appear at one viewport

### Step 4 — Calculate the score
- Start at 100
- Deduct: Critical −10, High −5, Medium −2, Low −1 per issue found
- Apply modifiers after base calculation:
  - Zero Critical issues: +5 bonus
  - More than 4 Critical issues: −5 additional penalty
- Score cannot go below 0

### Step 5 — Format the output
Exact section order:
1. Score and grade (top, immediately visible)
2. Critical issues
3. High priority issues
4. Medium priority issues
5. Low priority issues
6. What this theme does well (minimum 3 positives)
7. Recommended fix order (top 5, numbered)
8. Estimated impact

## Audit modes

### Full audit (default)
Single score 0–100 covering everything. Used when the user says "audit this theme" without qualifiers.

### Split-score mode
Two independent scores side by side:
- Technical & CRO score (from `audit-checklist.md` + `apps-audit.md`)
- SEO/AEO/GEO score (from `seo-aeo-geo-checklist.md`)

Each starts at 100 and is deducted independently. Triggered when the user asks for an SEO audit, AEO audit, GEO audit, or "search visibility" review.

### Quick-wins mode
Re-ranks findings by `impact ÷ effort` instead of raw severity. The report leads with a Quick Wins table (top 10, ranked by ROI) before the standard Critical → Low sections. Standard sections are still emitted in full after the table — nothing is omitted.

Trigger phrases: "quick wins", "what should I fix first", "biggest bang for buck", "highest ROI", "Friday afternoon fixes"

Effort buckets: 5 min / 15 min / 1 hour / half-day / multi-day
Impact buckets: site-wide vs single-template, blocking vs visual
ROI rating: 1–5 stars, derived from impact_bucket ÷ effort_bucket

## Recommended fix order calculation

Prioritize by combination of:
1. Severity (Critical first)
2. Scope (all-pages issues before single-page issues)
3. Effort (quick fixes before long rewrites)

A Critical that affects every page and takes 10 minutes beats a Critical that only affects the product page and needs a full section rewrite.

In Quick-Wins mode only: sort strictly by `impact_bucket ÷ effort_bucket`. A 5-minute High beats a half-day Critical.

## Hard rules (never violate these)

1. Never flag an issue without reading the specific file and confirming it exists. No assumptions.
2. Always include file name and line number for every finding.
3. Always include a working code example in the Fix — not just a description.
4. When a deprecated API is found, show the current replacement with a complete working example.
5. Check mobile separately from desktop.
6. Prioritize above-the-fold issues above everything else.
7. If a file cannot be read, say so explicitly. Do not guess at its contents.

## Common mistakes to avoid

- Do not flag `img_url` without confirming it exists in the file
- Do not assume a theme is OS 1.0 or 2.0 without checking `settings_schema.json`
- Do not recommend removing jQuery if the theme depends on it for other functionality
- Do not flag N+1 issues in loops iterating fewer than 5 items (negligible performance impact)
- Do not recommend changing brand fonts or colors — this is a technical audit, not a design review

## When only the live URL is available (no theme files)

Audit using:
1. Page source: scripts without defer/async, lazy loading presence, skip-to-content, JSON-LD
2. Browser DevTools Network tab: render-blocking resources, image formats, page weight
3. Google PageSpeed Insights: Core Web Vitals, Lighthouse scores

Always note in the report whether each finding is based on live page analysis or source file analysis. File-level analysis is more accurate.
