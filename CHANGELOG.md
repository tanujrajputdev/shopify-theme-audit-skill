# Changelog — Shopify Theme Audit Skill

This file documents what changed between versions and how much more the current version covers.

---

## v2.3 — August 2026 (current)

**Correcting the same mistake twice: every AI provider runs three crawlers, not one.**

v2.1 fixed this on OpenAI and then made the identical error on Anthropic. Both are now corrected.

### What was wrong

**v2.0** recommended a robots.txt allow-block listing `GPTBot` and omitting `OAI-SearchBot`. `GPTBot` is OpenAI's *training* crawler; `OAI-SearchBot` builds the index behind ChatGPT Search citations. Stores that applied it opted **into** training and **out of** citations.

**v2.1** fixed OpenAI — and then repeated the mistake on Anthropic. It listed `ClaudeBot` as Anthropic's single crawler and placed it in the **retrieval** group. `ClaudeBot` is the **training** crawler. `Claude-SearchBot` (which gates Claude citations) and `Claude-User` (live user fetches) were missing entirely. Anthropic documented the three-crawler split in February 2026.

**The false claim both versions implied:** that blocking `GPTBot` removes a store from ChatGPT's answers. It does not. Blocking a training crawler has no effect on search visibility on either platform. Anything stating otherwise is wrong.

### The corrected model

| Group | Bots | What blocking costs |
|---|---|---|
| **Retrieval** | `OAI-SearchBot`, `Claude-SearchBot`, `PerplexityBot` | **Citations** |
| **User-initiated fetch** | `ChatGPT-User`, `Claude-User` | Live fetches in a user's session |
| **Training** | `GPTBot`, `ClaudeBot`, `CCBot` | Training-corpus inclusion **only** |
| **Dual** | `Google-Extended` | Gemini training **and** AI Overviews grounding |

### Also corrected

- **The two platforms are not symmetric on user-initiated fetches.** Anthropic states all three of its crawlers honour robots.txt, **including `Claude-User`**. robots.txt **may not apply** to `ChatGPT-User`, and generally does not to `Perplexity-User`. Do not promise a merchant they can robots.txt their way out of user-initiated fetches everywhere.
- **`Claude-Web` and `anthropic-ai` are deprecated.** Harmless to leave in a file; not a substitute for `Claude-SearchBot`.
- **Reporting rule:** "blocked from AI crawlers" is not an actionable finding and is usually wrong in detail. Name the bot and say what it gates.
- `CLAUDE.md` gains the v2.2 rules it never received — the web-pixel non-limit and the cite-the-limit-or-label-it rule.

**If you applied the robots.txt recommendation from v2.0 *or* v2.1, re-check `templates/robots.txt.liquid` against the corrected block in `GEO-C1`.**

*Credit: the Anthropic error was caught by the skill's author on review, not by the tool.*

---

## v2.2 — August 2026

**Four new checks, each anchored to a published limit rather than a rule of thumb.**

| Check | The limit | Source |
|---|---|---|
| `APP-C5` — Page builder script weight | App must not reduce storefront Lighthouse by **>10 points** to be listed | [shopify.dev/docs/apps/build/performance](https://shopify.dev/docs/apps/build/performance) |
| `APP-C6` — Accessibility overlay detection | **FTC $1M order** barring WCAG-compliance claims; WebAIM **67%** "not effective" | [FTC](https://www.ftc.gov/news-events/news/press-releases/2025/04/ftc-approves-final-order-requiring-accessibe-pay-1-million) · [WebAIM](https://webaim.org/projects/screenreadersurvey/) |
| `C7` — JSON metafield size | **128KB** cap on `json` metafields, API **2026-04**+ | [changelog](https://shopify.dev/changelog/reduced-metafield-value-sizes) |
| `APP-H7` — Web pixel sprawl | Built for Shopify **LCP 2.5s / CLS 0.1 / INP 200ms**, 10-point Lighthouse | [BFS requirements](https://shopify.dev/docs/apps/launch/built-for-shopify/requirements) |

### A correction worth stating plainly

`APP-H7` was scoped as "web pixel payload against the 128KB ceiling." **There is no published 128KB ceiling for web pixels.** Shopify's two 128KB figures belong elsewhere — `json` metafield writes on 2026-04+ (which is `C7`) and Shopify Functions input size. UI extensions cap at 64KB compressed. Web pixels have no published payload limit, and a Shopify Community thread asking precisely this went unanswered.

Rather than invent one, `APP-H7` is built on what *is* documented: the strict (web worker) vs lax sandbox distinction, duplicate-destination detection, and the Built for Shopify performance ceilings. It carries a ~15KB custom-pixel heuristic that is **explicitly labelled as this audit's threshold, not Shopify's.** `scoring.md` now carries a general rule requiring that labelling.

### Details worth knowing on `C7`

- Apps writing JSON metafields **before 1 April 2026 are grandfathered at 2MB.** Oversized values stay *readable* on all API versions and fail on the next *write* — so a store can look fine and be one sync away from breaking.
- The cap was originally announced at **16KB** and revised to 128KB after developer feedback. Work hardened against 16KB was aimed at a number that no longer applies.
- **Shopify Functions receive `null` for any metafield over 10,000 bytes.** The value is still stored and the Admin API still returns it — the function just silently gets nothing. No error is raised anywhere.

### On `APP-C5` and `APP-C6`

Published benchmarks put the major page builders at 260–340KB of injected JS and **16–35 point** mobile Lighthouse deltas — two to three and a half times Shopify's own published ceiling. `APP-C5` includes a measurement procedure using Shopify's weighting (Home 17% / Product 40% / Collection 43%) so the report can carry a measured number rather than a citation to someone's benchmark post.

The overlay check is deliberately written to be reported without moralising. Merchants were sold these as litigation protection by a vendor the FTC has since fined for that exact claim. The check cites the order and lists the underlying accessibility findings the overlay was masking.

### Also

- Detection table extended with six page builders, four overlay vendors, and the web pixel runtime.
- `before-after.md` gains paste-able WRONG → RIGHT pairs for all four.
- New positive findings: tracking migrated to strict-sandbox pixels, native OS 2.0 sections over a builder, accessibility handled in markup, `json` metafields split by access pattern.

---

## v2.1 — August 2026

**A retrieval-layer update, plus two corrections to v2.0 that were costing users accuracy.**

In one week of August 2026, ChatGPT Search changed how it retrieves. v2.0's AEO guidance was written for the world before that and audited the wrong layer. This release rebuilds the AEO/GEO model around retrieval and fixes two claims that the evidence no longer supports.

### What changed in the world

- **August 8, 2026** — ChatGPT Search began using the `site:` operator at scale. Domain-scoped fanout queries went from **0.37% to 16.8%** of all fanouts in a single day (~46×), and fanouts per response nearly doubled (**~1.08 → ~1.83**). The scoped searches are additive, not replacements. Retrieval is now two-stage: pick a domain, then search inside it.
- **August 14, 2026** — Reddit's ChatGPT Search citation share fell from **3.83%** (Jul 18–Aug 7) to **0.52%** (Aug 14–17), an **86.4%** relative drop. **The cause is unresolved.** The drop came in two phases six days apart; the August 8 change does not explain the larger second one; the measuring firm cannot rule out a data-collection issue on its own end; OpenAI has not commented. This skill does not assert the causal link, and says so explicitly.

### Added — the `R-` (retrieval) tier

A new tier that runs *before* AEO and GEO, on the principle that schema is a tie-breaker among pages that were already retrieved:

- **R-C1** — Retrieval crawler blocked or not explicitly allowed, with a bot-by-bot table separating retrieval crawlers from training crawlers
- **R-C2** — Domain cannot answer its own `site:`-scoped questions, with a mapping of likely fanouts to the pages that must exist to satisfy them
- **R-C3** — AI visibility depends on off-domain UGC the merchant does not control
- **R-H1** — Store not verifiably present in the Bing index `[merchant action]`
- **R-H2** — Answer content requires JavaScript to appear

### Fixed — two v2.0 errors

**1. The recommended robots.txt block omitted `OAI-SearchBot`.**
v2.0 told merchants to allow `GPTBot`, `ClaudeBot`, `PerplexityBot`, and `Google-Extended`. `GPTBot` is OpenAI's *training* crawler. `OAI-SearchBot` is the one that builds the index behind ChatGPT Search citations, and it was missing. A store that applied v2.0's recommendation opted **into** model training and **out of** ChatGPT Search citations — the inverse of what nearly every merchant wants. **If you applied v2.0's robots.txt fix, re-check `templates/robots.txt.liquid` against the corrected block in `GEO-C1`.** The August 8 change makes this more costly, not less: site:-scoped fanouts have to fetch from your domain.

**2. `llms.txt` was scored as Critical (−10). It has been demoted to Low, effectively retired.**
The 2026 evidence: no major AI provider reads it in production; an Ahrefs study of ~137,000 sites found **97% of `llms.txt` files received zero traffic**; one instrumented domain logged 84 requests out of 62,100 AI-bot visits (**0.1%**); Google's June 2026 docs state it has no effect on Search or AI Overviews. v2.0 deducted ten points from stores for not having a file that does nothing, and listed creating it as a quick win. Both are corrected — see `GEO-L3`.

### Recalibrated

- **AEO-C1 (FAQ schema)** — v2.0 called it "the single highest-leverage AEO signal." Overstated. Google retired FAQ rich results for most sites, and schema cannot rescue a page retrieval never reached. Still worth emitting; no longer the headline.
- **AEO-H3 → AEO-L2 (Speakable)** — demoted from High. Google restricts `speakable` to a narrow set of news publishers; scoring it at −5 for an ecommerce store was not defensible.
- **Scoring** — `R-` findings lead the report and are never traded against page-level polish. Quick-wins table reordered; `llms.txt` removed from it.
- **Honesty rule** — AEO estimates now promise *eligibility*, not citation outcomes. August 2026 showed a provider can revalue an entire source class overnight with no announcement and no confirmed explanation.

### Note for AEO practitioners

The durable takeaway from August is not "Reddit is dead" — that claim may not survive revision. It is that **AI visibility resting on a surface you do not own is revocable by someone else's config change.** First-party content on an owned domain is both the hedge against that and exactly what the `site:` fanout shift rewards. Both August events point the same way even though only one has a confirmed mechanism.

---

## v2.0 — June 2026

The audit went from a single performance + accessibility checklist to a full search-era audit covering eight surfaces: performance, accessibility, CRO, third-party app overhead, classic SEO, Answer Engine Optimization, Generative Engine Optimization, and structured data depth.

### What was added

**1. SEO / AEO / GEO checklist (`seo-aeo-geo-checklist.md`)**
A dedicated 35+ check file covering:
- **SEO** — `<title>`, meta description, canonical, sitemap accessibility, Product JSON-LD, multiple H1s, heading hierarchy, Open Graph, Twitter Cards, descriptive alt text, hreflang, pagination handling, title/description length, URL handle quality, internal linking, robots meta, semantic logo markup
- **AEO** — FAQ schema, factual product summaries, `PropertyValue` specifications, HowTo schema, Speakable hints, author/publisher (E-E-A-T), `dateModified`, individual Review schema, comparison content
- **GEO** — AI crawler accessibility (GPTBot, ClaudeBot, PerplexityBot, Google-Extended, CCBot), `llms.txt`, Organization schema with `sameAs`, BreadcrumbList, machine-readable shipping/returns, materials/origin, descriptive image URLs

**2. Third-party app overhead audit (`apps-audit.md`)**
Detects 20+ Shopify apps in compiled HTML/JS and scores each by loading strategy, bundle size, and above-fold impact:
- Email/popups: Klaviyo, Privy, Justuno
- Reviews: Judge.me, Loox, Yotpo, Stamped.io
- Upsell/cart: Rebuy, ReConvert
- Chat: Gorgias, Tidio, Zendesk
- Loyalty: Smile.io, LoyaltyLion
- Subscriptions: Recharge, Bold Subscriptions
- Recommendations: Nosto, LimeSpot
- Search/filter: Searchanise, Boost AI Search
- Analytics: Hotjar, Microsoft Clarity, GTM duplication
- Plus video player, fonts, and A/B testing tool checks

Most Shopify stores die from app bloat, not theme bloat. This catches it.

**3. Before / After code gallery (`before-after.md`)**
25 strict `WRONG → RIGHT` Liquid pairs keyed by check ID. The audit pastes the exact RIGHT block into the report instead of paraphrasing. Removes the failure mode where AI gives vague fixes ("add lazy loading") instead of complete working code.

**4. Quick-Wins mode (in `scoring.md`)**
Triggered by phrases like "what should I fix first" / "biggest bang for buck" / "highest ROI". Re-ranks findings by `impact ÷ effort` instead of raw severity and leads the report with a star-rated table before the standard breakdown. A 5-minute High beats a half-day Critical.

**5. Split-score mode (in `scoring.md`)**
When the user requests an SEO/AEO/GEO-focused audit, the report returns two independent scores side-by-side instead of one merged number — Technical & CRO vs SEO/AEO/GEO. Each starts at 100 and is deducted independently.

**6. Three example audit reports**
- `examples/sample-audit-report.md` — Dawn custom build, C-grade (the original)
- `examples/sample-audit-legacy-1.0.md` — Vintage 1.0 theme being prepped for migration, F-grade
- `examples/sample-audit-clean-dawn.md` — Production-quality Dawn theme, A-grade, demos Quick-Wins mode

**7. Landing page (`landing/`)**
A self-contained marketing site for the skill — Tailwind via CDN, Inter Tight + Instrument Serif, mobile-first, 10 sections covering hero, three-layer search reality, what it checks, sample report, by the numbers, install, use cases, FAQ, author, and footer. Deploys to GitHub Pages, Vercel, or Netlify as static.

### What was removed

- "Shopify MCP coming soon" placeholder (MCP is live — references updated throughout)
- "Shopify Developer Bundle" / Gumroad section in the README (skill is fully free + open source)
- Em dashes across the landing page copy (replaced with commas for readability on small screens)

### What was rewritten

- `README.md` — full rewrite. New "Why AEO and GEO matter now" section, expanded checks table, split-score sample report, eight-file install tree, new use-case list, updated author links (tanujrajput.com, @tanujbuilds, /in/tanuj-rajput)
- `CLAUDE.md` — read-order expanded from 5 files to 7, plus quick-wins trigger phrases
- `scoring.md` — three modes now (full / split / quick-wins), each documented with example output formats

---

## v1.0 — earlier (baseline)

The original skill shipped with:

- `CLAUDE.md` — master instructions
- `audit-checklist.md` — 22 checks across Critical / High / Medium / Low severity, focused on performance, accessibility, and CRO
- `liquid-patterns.md` — 15 before/after code examples
- `deprecated-apis.md` — deprecated Shopify APIs and replacements
- `scoring.md` — single-score 0–100 formula
- One sample report (`examples/sample-audit-report.md`)
- A README documenting it

It worked, but it only covered classic web-performance auditing. SEO was a single check ("missing structured data"). There was no AEO. There was no GEO. There was no third-party app audit. The score was a single number with no prioritization for time-boxed merchants.

---

## Quantitative comparison

| Surface | v1.0 | v2.0 | Change |
|---|---|---|---|
| Total files in skill | 6 | 9 | +50% |
| Total documented checks | 22 | 80+ | +263% |
| Severity categories covered | 4 (C / H / M / L) | 4 + app-specific + SEO-specific + AEO-specific + GEO-specific | broader axis coverage |
| Audit modes | 1 (full, single score) | 3 (full / split-score / quick-wins) | +2 modes |
| Working code fixes per finding | 1 example per pattern (~15 patterns) | 25 strict WRONG → RIGHT pairs keyed to check IDs | +67% direct paste-in fixes |
| Search-era coverage | none (no SEO/AEO/GEO) | 35+ checks across all three layers | new capability |
| Third-party app coverage | none | 20+ apps detected and scored | new capability |
| Example reports | 1 | 3 (C / F / A grade) | +200% |
| Landing page | none | full marketing site | new |

---

## Capability improvement, qualitatively

| Question a merchant might ask | v1.0 answer | v2.0 answer |
|---|---|---|
| "Will my store get cited by ChatGPT?" | not covered | AEO checklist diagnoses it |
| "Am I visible in Google AI Overviews?" | not covered | GEO checklist diagnoses it |
| "Which of my apps is killing performance?" | not covered | Apps audit names each one with severity |
| "What should I fix in the next hour?" | not covered | Quick-Wins mode ranks fixes by ROI |
| "Show me the exact Liquid I need to paste" | partial — patterns referenced, not always pasted | before-after.md pasted verbatim |
| "I want SEO only, not performance" | full audit only | split-score mode separates the two |
| "Is my robots.txt set up for AI crawlers?" | not covered | GEO-C1 check |
| "Do I have an `llms.txt`?" | not covered | ~~GEO-C2 check~~ — *retired in v2.1, see above* |
| "Are my product specs machine-readable?" | not covered | AEO-H1 check |
| "Is my product schema complete enough for shopping AI?" | partial — Product schema check existed | GEO-M1 adds shipping + returns required by AI shopping |
| "What grade is my theme?" | one number 0–100 | one or two numbers + letter grade + sample comparisons |

---

## Estimated audit deliverable improvement

A v1.0 audit on a typical custom Dawn theme produced ~12 findings across performance + accessibility, surfaced as a single 0–100 score.

A v2.0 audit on the same theme produces:
- ~12 performance and accessibility findings (unchanged)
- 6–10 SEO findings
- 4–8 AEO findings
- 3–6 GEO findings
- 2–6 third-party app overhead findings
- A Quick-Wins table prioritising 8–10 highest-ROI fixes if quick-wins mode is requested
- Split or merged score depending on mode

Total: **~30 sourced, file-and-line-cited findings** per audit instead of ~12. Each with a working code fix pasted verbatim.

---

## Migration notes (for users on v1.0)

1. Pull the latest from `main`.
2. Copy the three new files into your theme root: `seo-aeo-geo-checklist.md`, `apps-audit.md`, `before-after.md`.
3. The existing `audit-checklist.md`, `liquid-patterns.md`, `deprecated-apis.md`, and `scoring.md` are backwards-compatible — your existing prompts continue to work, they just get richer reports.
4. To use the new modes, change your prompt:
   - Quick wins: "Run the audit skill in quick-wins mode."
   - SEO/AEO/GEO only: "Run an SEO, AEO, and GEO audit. Use split-score mode."
   - Apps only: "Audit only third-party app overhead using apps-audit.md."

---

## What's planned for v2.1+

Not committed, but on the table:

- **Core Web Vitals budgets per template** — flag specific KB overages, not just generic "this is heavy"
- **CRO heuristics beyond technical** — above-fold trust signals, PDP buy-box hierarchy, sticky ATC, abandoned-cart triggers
- **Checkout Extensibility audit** — checkout.liquid sunset is forcing every store to migrate; no skill currently covers it
- **i18n / Shopify Markets audit** — hreflang correctness, market-aware schema, AI-crawler behavior across locales
- **JSON output mode** — emit machine-readable JSON alongside markdown for dashboards
- **Re-audit / diff mode** — compare against a previous audit and surface only deltas
- **Companion fix skill** — applies the audit findings as a branch of patches

If you want one of these next, open an issue and say so.
