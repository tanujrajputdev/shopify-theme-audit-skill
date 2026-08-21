# Scoring Methodology

## How to calculate the score

Start at 100. Deduct points for each issue found across both checklists (`audit-checklist.md` and `seo-aeo-geo-checklist.md`).

**Retrieval checks are scored first and reported first.** The `R-` tier in `seo-aeo-geo-checklist.md` gates whether any AEO or GEO work can pay off at all. A store failing `R-C1` (retrieval crawler blocked) or `R-C2` (no on-domain page to answer a scoped query) cannot be cited regardless of its schema coverage, so those findings lead the report and are never traded off against page-level polish.

| Severity | Points deducted per issue |
|---|---|
| Critical | 10 |
| High | 5 |
| Medium | 2 |
| Low | 1 |

**Cite the limit, or do not state one.** Several checks are anchored to published, verifiable ceilings — Shopify's 10-point App Store Lighthouse limit (`APP-C5`), the 128KB `json` metafield cap in API 2026-04 (`C7`), the Built for Shopify Core Web Vitals thresholds, the FTC order behind `APP-C6`. When a finding rests on one of these, link it. When a threshold is this audit's own judgement rather than a platform limit — the ~15KB custom-pixel heuristic in `APP-H7`, for example — say so in the finding. Never present a self-chosen number as a platform rule; a merchant who repeats it to a developer will be corrected, and the whole report loses credibility with it.

Minimum score is 0. Score cannot go below 0.

## Audit modes

Choose mode based on the user's request:

**Full audit (default):** Single score 0–100 covering performance, accessibility, CRO, SEO, AEO, and GEO. Use when the user asks for a "theme audit" without further qualification.

**Split-score mode:** Two scores reported side by side — a Technical & CRO score (from `audit-checklist.md` + `apps-audit.md`) and an SEO/AEO/GEO score (from `seo-aeo-geo-checklist.md`). Each starts at 100 and is deducted independently. Use when the user explicitly asks for an SEO audit, AEO audit, GEO audit, or "search visibility" review.

**Quick-wins mode:** Re-rank the findings by `impact ÷ effort` instead of raw severity. Trigger words: "quick wins", "what should I fix first", "biggest bang for buck", "highest ROI", "Friday afternoon fixes". In this mode, still compute the standard score and present the standard sections, but **lead the report with a Quick-Wins table** before the Critical Issues section:

```
### Quick Wins — Top 10 by Impact ÷ Effort

| # | Issue | Severity | Est. effort | Est. impact | ROI |
|---|---|---|---|---|---|
| 1 | R-C1 — Allow OAI-SearchBot in robots.txt | Critical | 5 min | Unblocks ChatGPT Search citation entirely | ★★★★★ |
| 2 | H3 — Add LCP preload to hero image | High | 5 min | LCP -400ms | ★★★★★ |
| 3 | C6 — Add width/height to header logo | Critical | 5 min | CLS removed above fold | ★★★★★ |
| 4 | R-C2 — Publish the missing policy/sizing page as real text | Critical | 30 min | Answers site:-scoped fanouts that currently return nothing | ★★★★★ |
| 5 | M4 — Add Product JSON-LD via {{ product \| structured_data }} | Medium | 10 min | Rich snippets unlocked | ★★★★ |
| 6 | APP-C5 — Scope page builder script to its own templates | Critical | 20 min | Recovers Lighthouse points on PDP + collection | ★★★★★ |
| 7 | AEO-C1 — Add FAQPage schema to existing FAQ block | Critical | 15 min | Tie-breaker once retrieval works | ★★★ |
| ... |

**Ordering rule:** `R-` findings outrank everything else at equal effort. Fixing a robots.txt line is five minutes and can be the difference between eligible and invisible; no amount of schema work substitutes for it. Never place `AEO-C1` above an open `R-C1`.

**Do not list `APP-C6` (accessibility overlay) as a quick win.** Removing the script takes two minutes, but the finding is not resolved until the underlying markup is fixed. Listing it as a quick win implies a two-minute path to accessibility, which is the exact false promise the overlay itself makes. Report it as Critical with the structural fixes attached.

**Do not list `llms.txt` as a quick win.** It was ranked as one in v2.0. The evidence says AI crawlers do not fetch it (see `GEO-L3`). It is a fast task with no measured effect, which is the worst possible ROI profile — it looks productive and is not.
```

Effort buckets: 5 min / 15 min / 1 hour / half-day / multi-day. Impact buckets: site-wide vs single-template, blocking vs visual. ROI rating is one to five stars derived from `impact_bucket ÷ effort_bucket`.

After the Quick Wins table, still emit the standard Critical → High → Medium → Low sections in full. Do not omit anything.

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

For Quick-Wins mode specifically, sort strictly by `impact_bucket ÷ effort_bucket` — a 5-minute High beats a half-day Critical. Use this when the merchant is time-boxed (pre-BFCM, pre-launch, agency sprint).

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
**Retrieval eligibility:** [X] blocking issues resolved — crawler access, on-domain answer coverage, server-rendered content
**AEO (ChatGPT, Claude, Perplexity citations):** [X] schema and content-structure improvements — FAQ, HowTo, factual summaries
**GEO (AI Overviews, LLM-powered search):** [X] crawlability and trust improvements — Organization schema, AI crawler access, BreadcrumbList
```

Note: These are estimates, not guarantees. Actual Lighthouse scores depend on server response time and third-party scripts outside theme control. AEO and GEO outcomes also depend on factors outside the theme — domain authority, backlinks, and the AI provider's retrieval choices on any given day.

**State this honestly in every AEO/GEO report.** The August 2026 ChatGPT changes — the `site:` fanout shift on the 8th and the Reddit citation collapse on the 14th — landed overnight, unannounced, and the second one still has no confirmed mechanism. Publishers found out by watching their numbers move. Do not promise a citation outcome. What an audit can honestly promise is *eligibility*: the store is retrievable, its domain can answer the questions buyers ask, and its content is first-party rather than dependent on a surface someone else can revalue without notice. Frame every AEO estimate that way.

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
