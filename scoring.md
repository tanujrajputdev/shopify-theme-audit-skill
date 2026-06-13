# Scoring Methodology

## How to calculate the score

Start at 100. Deduct points for each issue found across both checklists (`audit-checklist.md` and `seo-aeo-geo-checklist.md`).

| Severity | Points deducted per issue |
|---|---|
| Critical | 10 |
| High | 5 |
| Medium | 2 |
| Low | 1 |

Minimum score is 0. Score cannot go below 0.

## Two scoring modes

Choose mode based on the user's request:

**Full audit (default):** Single score 0–100 covering performance, accessibility, CRO, SEO, AEO, and GEO. Use when the user asks for a "theme audit" without further qualification.

**Split-score mode:** Two scores reported side by side — a Technical & CRO score (from `audit-checklist.md`) and an SEO/AEO/GEO score (from `seo-aeo-geo-checklist.md`). Each starts at 100 and is deducted independently. Use when the user explicitly asks for an SEO audit, AEO audit, GEO audit, or "search visibility" review.

Always state at the top of the report which mode you used.

## Grade thresholds

| Score | Grade | What it means |
|---|---|---|
| 90–100 | A | Production-ready. Minor polish items only. |
| 75–89 | B | Good theme. A few meaningful improvements available. |
| 60–74 | C | Average. Several issues affecting performance or conversions. |
| 40–59 | D | Below standard. Multiple high-impact issues needing immediate attention. |
| 0–39 | F | Significant problems. Performance or accessibility issues that are harming the store. |

## Score modifiers

**Apply these after the base calculation:**

- If the theme has zero Critical issues: add 5 points (bonus for clean foundation)
- If the theme has more than 4 Critical issues: additional -5 points (penalty for serious neglect)
- If the theme is a Shopify Theme Store theme: note this context — store themes have been through Shopify review, so any issues found are likely theme customizations by the merchant or developer, not the original theme author

## How to present findings

### Order of findings in the report

Always present in this order:
1. Score and grade (at the top, immediately visible)
2. Critical issues (must fix)
3. High priority issues
4. Medium priority issues
5. Low priority issues
6. What the theme does well (always include at least 3 positive findings)
7. Recommended fix order (a numbered list of the top 5 actions)
8. Estimated impact

### Recommended fix order calculation

Prioritize fixes by the combination of:
- Severity (Critical first)
- Impact scope (issues affecting all pages before issues on one page)
- Effort (quick fixes before long implementations)

A Critical issue that affects every page and takes 10 minutes to fix ranks above a Critical issue that only affects the product page and requires a full section rewrite.

### Estimated impact section

After the main findings, provide:

```
### Estimated Impact If All Critical + High Issues Fixed

**Google Lighthouse Performance Score:** Estimated +[X] points
(Current estimated score: [X] / After fixes: [X])

**Core Web Vitals:**
- LCP: [improvement if preload and image issues fixed]
- CLS: [improvement if image dimensions added]
- INP: [improvement if JavaScript deferred]

**Accessibility:** [X] WCAG AA violations resolved

**SEO (classic search):** [X] on-page improvements — title/meta/canonical/hreflang
**AEO (ChatGPT, Claude, Perplexity citations):** [X] schema and content-structure improvements — FAQ, HowTo, Speakable, factual summaries
**GEO (AI Overviews, LLM-powered search):** [X] crawlability and trust improvements — Organization schema, AI crawler access, BreadcrumbList
```

Note: These are estimates, not guarantees. Actual Lighthouse scores depend on server response time and third-party scripts outside theme control. AEO and GEO outcomes also depend on factors outside the theme — domain authority, backlinks, and the AI provider's retrieval choices on any given day.

## What to audit when you cannot access theme files

If you can only analyze the live store URL (no theme file access):

1. Use the Page Source to identify:
   - Scripts in `<head>` without defer/async
   - `img_url` filter output patterns (look for `cdn.shopify.com/s/files` URLs with old format parameters)
   - Missing `loading="lazy"` on product images
   - Presence of skip-to-content link
   - JSON-LD structured data

2. Check browser DevTools Network tab for:
   - Number of render-blocking resources
   - Image formats (WebP vs JPEG)
   - Total page weight

3. Run Google PageSpeed Insights on the store URL for:
   - Core Web Vitals scores
   - Specific Lighthouse recommendations

Note in the report when findings are based on live page analysis vs source file analysis. File-level analysis is more accurate.

## Sample score calculations

**Example 1: A typical custom theme**
- C3 (no lazy loading): -10
- H1 (JS without defer): -5
- H4 (large global CSS): -5
- M4 (no product schema): -2
- M5 (no breadcrumb schema): -2
- L1 (console.log left in): -1
- Base score: 100 - 25 = **75/100 — Grade B**

**Example 2: A neglected older theme**
- C1 (render-blocking scripts): -10
- C2 (img_url used throughout): -10
- C3 (no lazy loading): -10
- C5 (no skip to content): -10
- H1 (JS without defer): -5
- H2 (no WebP): -5
- H6 (canonical issues): -5
- M4 (no structured data): -2
- M5 (no breadcrumbs): -2
- L1, L2, L3: -3
- Base score: 100 - 62 = **38/100 — Grade F**
- More than 4 Critical issues: -5 additional
- Final score: **33/100 — Grade F**
