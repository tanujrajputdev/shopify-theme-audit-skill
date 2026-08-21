# SEO, AEO, and GEO Audit Checklist

This file extends the main audit with checks for traditional SEO (Google, Bing), Answer Engine Optimization (AEO — ChatGPT, Claude, Perplexity, Gemini answers), and Generative Engine Optimization (GEO — being cited by AI Overviews and LLM-powered search).

Use this checklist alongside `audit-checklist.md`. Each finding follows the same severity model: Critical (−10), High (−5), Medium (−2), Low (−1).

---

## Why this matters for Shopify in 2026

Search has split into three layers:

1. **Classic SEO** — Google, Bing, DuckDuckGo. Still drives the majority of ecommerce traffic.
2. **AEO** — ChatGPT, Claude, Perplexity, Gemini direct answers. A growing share of product research now happens here. The user never sees a results page.
3. **GEO** — AI Overviews on Google, Bing Copilot, and LLM-powered shopping assistants. The store either gets cited or it does not exist to the buyer.

**Above all three sits retrieval.** An answer engine can only cite a page it retrieved, and it can only retrieve a page that sits in an index it queries and is reachable by the bot that fetches it. Schema, headings, and quotable copy are *tie-breakers among candidates that were already retrieved* — they do nothing for a page that was never a candidate in the first place.

Audit in that order: retrieval eligibility (`R-` checks) first, then page-level quotability (`AEO-` / `GEO-`). A theme with flawless FAQ schema behind a blocked retrieval crawler scores well on the old checklist and is invisible in practice.

---

## What changed in August 2026 — read this before running an AEO audit

Two measured changes to ChatGPT Search landed in one week, and they move where the leverage sits.

### 1. ChatGPT started using the `site:` operator at scale (August 8, 2026)

Fanout queries — the background searches ChatGPT runs while composing an answer — scoped to a single domain via `site:` jumped from **0.37% to 16.8%** of all fanout queries in a single day, roughly a 46× increase. At the same time, fanouts per response nearly doubled, from **~1.08 to ~1.83**. The site:-scoped searches are *additive*, layered on top of the generic ones rather than replacing them.

Mechanically: ChatGPT increasingly decides *which domain* is likely to hold the answer, then searches inside that domain for it. Retrieval now has a two-stage shape — entity selection, then on-domain lookup.

For a Shopify store that shifts the target in three concrete ways:

- **Entity recognition precedes page optimization.** If the model never thinks to run `site:yourstore.com`, no amount of schema on the page matters.
- **The answer must live on your own domain and be findable there.** A `site:yourstore.com return policy` fanout that finds nothing is a silent loss — no error, no fallback, just no citation.
- **Domain coverage breadth now outranks single-page polish.** A missing sizing page, materials page, or shipping page is a directly exploitable gap in a way it was not when retrieval was open-web only.

### 2. Reddit's ChatGPT citation share collapsed (August 14, 2026)

Reddit held a steady **3.83%** share of ChatGPT Search citations from July 18 through August 7, then averaged **0.52%** across August 14–17 — an **86.4%** relative drop.

**Treat the cause as unresolved.** It is tempting to pin this on the August 8 `site:` change, and that link is widely repeated, but the evidence does not support it cleanly:

- The decline came in two phases — a modest dip around August 8, then the sharp collapse on August 14, six days later. The `site:` change does not account for the second, larger drop.
- Promptwatch, the source of the measurement, states it cannot yet rule out a data-collection issue on its own end.
- OpenAI has not commented.
- There is precedent for a measurement artifact: a similar Reddit citation collapse in September 2025 was plausibly attributed to Google removing the `num=100` parameter — a change in the *measurement tooling's* visibility, not in the answer engine's behavior.
- Google's AI Overviews and AI Mode show a slower, far shallower Reddit decline, so whatever happened is OpenAI-specific rather than an industry-wide devaluation of forum content.

**Do not audit against "Reddit is dead."** That claim may not survive revision, and a checklist built on it would be wrong twice. Audit against the durable lesson underneath it, which holds either way:

> A store whose AI visibility depends on third-party UGC it does not control is exposed to overnight, unannounced, unexplained platform changes.

The hedge is first-party content on an owned domain — which is also precisely what the `site:` fanout change rewards. Both August events point the same direction even though only one of them has a confirmed mechanism. That convergence is why the `R-` checks below are worth running regardless of how the Reddit story resolves.

---

## SEO — CRITICAL SEVERITY (−10 each)

### S-C1: Missing or duplicate `<title>` tag
**Where to look:** `layout/theme.liquid` `<head>`
**What to find:** A single `<title>` element using the `page_title` global, with `shop.name` as fallback
**Flag if:** No `<title>` exists, multiple `<title>` tags present, or hardcoded title that does not vary per page
**Correct pattern:**
```liquid
<title>
  {{ page_title }}
  {%- if current_tags %} &ndash; {{ 'general.meta.tags' | t: tags: current_tags | join: ', ' }}{% endif -%}
  {%- if current_page != 1 %} &ndash; {{ 'general.meta.page' | t: page: current_page }}{% endif -%}
  {%- unless page_title contains shop.name %} &ndash; {{ shop.name }}{% endunless -%}
</title>
```

### S-C2: Missing meta description
**Where to look:** `layout/theme.liquid` `<head>`
**What to find:** `<meta name="description" content="...">` using `page_description` with sensible fallbacks
**Flag if:** No meta description tag, or content is empty on most pages
**Correct pattern:**
```liquid
{%- if page_description -%}
  <meta name="description" content="{{ page_description | escape }}">
{%- elsif template contains 'product' -%}
  <meta name="description" content="{{ product.description | strip_html | truncate: 160 | escape }}">
{%- elsif template contains 'collection' -%}
  <meta name="description" content="{{ collection.description | strip_html | truncate: 160 | escape }}">
{%- else -%}
  <meta name="description" content="{{ shop.description | default: shop.name | escape }}">
{%- endif -%}
```

### S-C3: Missing or incorrect canonical tag
**Where to look:** `layout/theme.liquid` `<head>`
**What to find:** `<link rel="canonical" href="{{ canonical_url }}">`
**Flag if:** No canonical tag, canonical points to a paginated URL, or canonical is hardcoded to a static URL
**Why it matters:** Without correct canonicals, Shopify variant URLs (`?variant=`), filter URLs, and pagination create duplicate content penalties

### S-C4: No XML sitemap reference
**Where to look:** `layout/theme.liquid`
**What to find:** Shopify auto-generates `/sitemap.xml`. Confirm robots.txt is not blocking it (check via `robots.txt.liquid` if customized).
**Flag if:** A custom `templates/robots.txt.liquid` exists and blocks `Sitemap: ` directive or disallows `/sitemap.xml`

### S-C5: Product pages missing `Product` JSON-LD
**Where to look:** `sections/main-product.liquid`, `templates/product.liquid`
**What to find:** Full Product schema with name, image, description, sku, brand, offers, aggregateRating
**Flag if:** No Product schema, schema missing `offers`, or schema missing `image`
**Note:** Shopify's `{{ product | structured_data }}` filter outputs valid Product schema. If the theme uses this, do not flag.

---

## SEO — HIGH SEVERITY (−5 each)

### S-H1: Multiple `<h1>` tags on a single page
**Where to look:** `sections/main-product.liquid`, `sections/main-collection-banner.liquid`, `sections/header.liquid`
**What to find:** More than one `<h1>` rendered on a page
**Flag if:** Header logo is wrapped in `<h1>` while page also has a content `<h1>`
**Why it matters:** Confuses search engines about the page's primary topic. Header logos should be `<div>` or `<p>` with brand styling, not `<h1>`.

### S-H2: No heading hierarchy
**Where to look:** Section files
**What to find:** Headings that skip levels (h1 → h4) or use heading tags for visual styling
**Flag if:** Any page has `<h3>` or `<h4>` without a parent `<h2>`, or `<h2>` without an `<h1>`

### S-H3: Open Graph tags missing or incomplete
**Where to look:** `layout/theme.liquid` `<head>`, often delegated to `snippets/social-meta-tags.liquid`
**What to find:** `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
**Flag if:** Any of these are missing on product or collection pages
**Correct pattern:**
```liquid
<meta property="og:site_name" content="{{ shop.name }}">
<meta property="og:url" content="{{ canonical_url }}">
<meta property="og:title" content="{{ page_title | escape }}">
<meta property="og:type" content="{% if template contains 'product' %}product{% elsif template contains 'article' %}article{% else %}website{% endif %}">
<meta property="og:description" content="{{ page_description | default: shop.description | default: shop.name | escape }}">
{%- if page_image -%}
  <meta property="og:image" content="http:{{ page_image | image_url: width: 1200 }}">
  <meta property="og:image:secure_url" content="https:{{ page_image | image_url: width: 1200 }}">
  <meta property="og:image:width" content="1200">
  <meta property="og:image:height" content="{{ 1200 | divided_by: page_image.aspect_ratio | round }}">
{%- endif -%}
```

### S-H4: Twitter Card tags missing
**Where to look:** `layout/theme.liquid` `<head>`
**What to find:** `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
**Flag if:** Twitter tags absent — limits social discovery on X and many AI crawlers that read Twitter Card data

### S-H5: Image alt text missing or generic
**Where to look:** All `<img>` tags
**What to find:** Descriptive alt text that includes product name or context, not "image" or "photo"
**Flag if:** Alt text is empty on content images, equals the file name, or is generic ("product image")
**Why it matters:** Alt text feeds Google Images AND is read by LLMs to understand the page

### S-H6: Hreflang missing on multi-region stores
**Where to look:** `layout/theme.liquid` `<head>`
**What to find:** `<link rel="alternate" hreflang="...">` for each language/region
**Flag if:** Store uses Shopify Markets but theme does not output hreflang
**Correct pattern:**
```liquid
{%- for locale in shop.published_locales -%}
  <link rel="alternate" hreflang="{{ locale.iso_code }}" href="{{ canonical_url | replace: request.locale.iso_code, locale.iso_code }}">
{%- endfor -%}
<link rel="alternate" hreflang="x-default" href="{{ canonical_url }}">
```

### S-H7: Pagination missing `rel="next"` / `rel="prev"` hints in canonical strategy
**Where to look:** `layout/theme.liquid`, collection templates
**What to find:** Either self-referencing canonical per page, OR a single canonical to page 1 with appropriate pagination meta
**Flag if:** All pages canonicalize to page 1 AND pages are non-trivially long (loses indexing depth)

---

## SEO — MEDIUM SEVERITY (−2 each)

### S-M1: Title tag length out of range
**Flag if:** Page titles are longer than 60 characters (Google truncates) or shorter than 30 (under-optimized)

### S-M2: Meta description length out of range
**Flag if:** Descriptions over 160 characters or under 70

### S-M3: URL handles include stop words or are over-long
**Where to look:** Product and collection handles
**Flag if:** Handles include "and", "the", "of" repeatedly, or are over 60 characters

### S-M4: Internal linking weak from product pages
**Where to look:** `sections/main-product.liquid`, related products area
**Flag if:** Product pages have no links to collections, related products, or category breadcrumbs

### S-M5: Robots meta tag missing on noindex-worthy pages
**Where to look:** `layout/theme.liquid`, `templates/search.liquid`, `templates/customers/*.liquid`
**Flag if:** Customer account pages, internal search results, or cart page are indexable
**Correct pattern for non-indexable pages:**
```liquid
{%- if template == 'search' or template contains 'customers' or template == 'cart' -%}
  <meta name="robots" content="noindex, follow">
{%- endif -%}
```

### S-M6: Header logo using `<img>` without semantic markup
**Flag if:** Logo `<img>` is not inside a link to `/` with proper alt text equal to `shop.name`

---

## RETRIEVAL — ELIGIBILITY TO BE CITED AT ALL

Run this tier first. Every AEO and GEO check below is a tie-breaker among pages that were already retrieved; these checks decide whether the page is a candidate.

Some items here are **store-level, not theme-level**. Flag them anyway, and label them `[merchant action]` in the report so the merchant knows it is not a code fix. An audit that stays silent about a blocking store-level problem because it is out of theme scope is not doing its job.

### R-C1 (Critical): Retrieval crawler blocked or not explicitly allowed
**Where to look:** `templates/robots.txt.liquid` (if customized)
**What to find:** `OAI-SearchBot` **and** `Claude-SearchBot` explicitly allowed
**Flag if:** Either retrieval bot is disallowed or absent — including the common case of a file that lists `GPTBot` and `ClaudeBot` (both training crawlers) and neither search crawler
**Why it matters:** These are different bots with different jobs, and conflating them is the most common AEO mistake in the wild:

Each major provider runs **three separate crawlers** with three different jobs. Blocking one does not block the others. Treating a provider as having a single bot is the single most common AEO mistake, and it fails in both directions — blocking citations while intending to block training, or the reverse.

| Bot | Operator | Job | Blocking it costs you |
|---|---|---|---|
| **Retrieval — these gate citations** | | | |
| `OAI-SearchBot` | OpenAI | Builds/refreshes the **search** index behind ChatGPT Search citations | **ChatGPT Search visibility** |
| `Claude-SearchBot` | Anthropic | Indexes content for **Claude's search results** | **Claude search citations** |
| `PerplexityBot` | Perplexity | Perplexity's search index | **Perplexity citations** |
| **User-initiated fetch** | | | |
| `ChatGPT-User` | OpenAI | Fetches a page when a ChatGPT **user** action requires it live | Live fetches during a user's session |
| `Claude-User` | Anthropic | Fetches a page when a Claude **user** asks about it | Live fetches during a user's session |
| **Training — separate decision** | | | |
| `GPTBot` | OpenAI | Crawls for **model training** | Training-corpus inclusion **only** |
| `ClaudeBot` | Anthropic | Crawls for **model training** | Training-corpus inclusion **only** |
| `CCBot` | Common Crawl | Open crawl corpus used by many trainers | Training-corpus inclusion |
| **Dual-purpose** | | | |
| `Google-Extended` | Google | Gemini training **and** grounding in AI Overviews / AI Mode | Both training and AI Overview eligibility |

**Read the table by column, not by vendor.** The two mistakes it prevents:

1. **Allowing `GPTBot` while blocking `OAI-SearchBot`** opts a store **into** training and **out of** citations — the exact inverse of what almost every merchant wants.
2. **Allowing or blocking `ClaudeBot` alone** and assuming it settles Anthropic. It does not. `ClaudeBot` is the **training** crawler. Blocking it has **no effect on Claude citations** — that is `Claude-SearchBot`. This is the mirror image of mistake 1, and it is just as common.

**Do not tell a merchant that blocking `GPTBot` removes them from ChatGPT's answers.** It does not. `GPTBot` is training-only; answer visibility is gated by `OAI-SearchBot` and `ChatGPT-User`. The same correction applies to `ClaudeBot` and Claude.

**One asymmetry worth knowing:** Anthropic states that **all three** of its crawlers honour robots.txt, **including `Claude-User`**. OpenAI and Perplexity draw a sharper line on user-initiated fetchers — robots.txt **may not apply** to `ChatGPT-User`, and generally does not apply to `Perplexity-User`. So a robots.txt block on `Claude-User` is reliable in a way the same block on `ChatGPT-User` is not. Do not promise a merchant that they can robots.txt their way out of user-initiated fetches on every platform.

**Legacy agents:** `Claude-Web` and `anthropic-ai` are deprecated. Harmless to leave in a robots.txt, but they are not a substitute for `Claude-SearchBot` — a file listing only those is functionally missing Anthropic's retrieval bot.

Since the August 8 shift, retrieval bots gate more than before: site:-scoped fanouts have to actually fetch from your domain.
**Severity note:** This supersedes the old `GEO-C1`, which omitted `OAI-SearchBot` and treated `ClaudeBot` as Anthropic's only crawler. See `GEO-C1` below for the corrected allow-block.

### R-C2 (Critical): Domain cannot answer its own site:-scoped questions
**Where to look:** `templates/`, `pages/`, store navigation — the set of pages that exist at all
**What to find:** A dedicated, crawlable, text-based page for each question a buyer asks before purchase
**How to check:** Enumerate the fanouts a model would plausibly run against this store and confirm a page exists to satisfy each:

| Likely fanout | Page that must exist |
|---|---|
| `site:store.com shipping` | Shipping policy with times, regions, costs as text |
| `site:store.com returns` | Returns/refunds policy with the window stated numerically |
| `site:store.com size guide` | Sizing page with measurements in a table, not an image |
| `site:store.com materials` | Materials/care page, or per-product metafield rendered as text |
| `site:store.com warranty` | Warranty terms |
| `site:store.com about` | Brand/about page with substantive prose |
| `site:store.com <product> review` | On-domain reviews rendered in HTML |

**Flag if:** Any of these has no page, is a PDF, is an image, is inside an accordion that only renders on click, or exists only as an app-injected widget
**Why it matters:** This is the check the August 8 change created. Before, a gap here meant the model fell back to the open web — possibly to a forum thread about you. Now the scoped search simply returns nothing and the citation goes to a competitor whose domain could answer.
**Severity guidance:** Missing shipping, returns, or sizing → Critical. Missing warranty, materials, or about → High.

### R-C3 (Critical): AI visibility depends on off-domain UGC
**Where to look:** Where the store's trust and detail content actually lives
**What to find:** Reviews, comparisons, sizing advice, and use-case content present as first-party HTML on the store's own domain
**Flag if:** The substantive buying information about this product exists mainly on Reddit, YouTube, TikTok, marketplace listings, or a review app's hosted subdomain rather than the store's domain
**Specifically flag:** Review apps that render only into a JS widget or an iframe on a third-party domain — the content is not on your domain for a site:-scoped search to find, and it is not in the HTML for a non-JS-executing crawler to read
**Why it matters:** August 2026 demonstrated that a platform can revalue an entire content source overnight, without announcement or explanation, and that even the analysts measuring it cannot always say why. Any AI-visibility strategy resting on a surface the merchant does not own carries that risk permanently. First-party content on an owned domain is the only position that is not revocable by someone else's config change.
**Fix direction:** Bring the substance on-domain — reviews rendered server-side into HTML, comparison content as real pages, sizing and materials as text. Off-domain presence is fine as *reinforcement*; it is dangerous as the *foundation*.

### R-H1 (High): Store not verifiably present in the Bing index `[merchant action]`
**What to find:** The store's key templates indexed in Bing (verify via Bing Webmaster Tools, or spot-check `site:` queries on Bing)
**Flag if:** Product and policy pages are absent from Bing while present in Google
**Why it matters:** ChatGPT Search retrieval runs on a blended stack — Bing's index plus OpenAI's own `OAI-SearchBot` crawl. Bing coverage is not the whole story, but a page missing from Bing loses one of the two main paths into a ChatGPT answer, and Bing indexation is the half a merchant can directly influence. Google rankings do not substitute.
**Note:** This is a store-level action (submit sitemap to Bing Webmaster Tools), not a theme fix. Report it as `[merchant action]`.

### R-H2 (High): Answer content requires JavaScript to appear
**Where to look:** Any content that satisfies an `R-C2` row — policies, specs, sizing, reviews
**Flag if:** The content is injected client-side, lives in an accordion whose panel is empty until clicked, or is loaded from a third-party script
**Why it matters:** Retrieval crawlers largely do not execute JavaScript. Content that a human sees and a crawler does not is content that cannot be cited. This overlaps `GEO-H3` but is scored here because since August 8 it now blocks *scoped* retrieval, not just general crawling.
**How to verify:** View source (not inspector) and search for the text. If it is absent from the raw HTML, it is invisible to retrieval.

---

## AEO — ANSWER ENGINE OPTIMIZATION

AEO optimizes the page for being quoted verbatim by ChatGPT, Claude, Perplexity, and Gemini when users ask product research questions. The mechanic is different from SEO: the model needs short, factual, well-attributed statements it can pull as a citation.

### AEO-C1 (Critical): No FAQ schema on product or info pages
**Where to look:** `sections/main-product.liquid`, `templates/page.faq.liquid`, FAQ accordion sections
**What to find:** `FAQPage` JSON-LD with question/answer pairs
**Flag if:** Theme renders FAQ accordion UI but does not emit `FAQPage` schema
**Why it matters:** FAQ schema gives answer engines clean, pre-segmented question/answer pairs to quote, and it is cheap to emit when the accordion UI already exists.
**Calibration (revised v2.1):** Earlier versions of this checklist called FAQ schema "the single highest-leverage AEO signal." That overstated it. Google retired FAQ rich results for most sites, and schema is a *tie-breaker among retrieved pages* — it cannot rescue a page that retrieval never reached. Since the August 8 fanout shift, the higher-leverage work is `R-C2`: making sure a page answering the question exists on the domain at all. Emit the schema — it is 15 minutes and it helps at the margin — but fix retrieval eligibility first.
**Note:** The *content* of the FAQ matters more than the markup. A `FAQPage` block wrapping vague marketing answers is worth less than plain HTML answering a real buyer question specifically.
**Correct pattern:**
```liquid
{%- if section.blocks.size > 0 -%}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {%- for block in section.blocks -%}
      {
        "@type": "Question",
        "name": {{ block.settings.question | json }},
        "acceptedAnswer": {
          "@type": "Answer",
          "text": {{ block.settings.answer | strip_html | json }}
        }
      }{%- unless forloop.last -%},{%- endunless -%}
    {%- endfor -%}
  ]
}
</script>
{%- endif -%}
```

### AEO-C2 (Critical): No clear, factual product summary block
**Where to look:** `sections/main-product.liquid`, top of product description
**What to find:** A short factual block (1–3 sentences) stating what the product is, who it is for, and the primary benefit — without marketing fluff
**Flag if:** Product description opens with promotional language ("Introducing the most...", "Get ready to...") rather than a definition LLMs can quote
**Why it matters:** LLMs prefer dictionary-style opening sentences. A product page that opens "The Aurora Backpack is a 22-liter waterproof commuter bag designed for cyclists." is far more quotable than "Discover the bag of your dreams."

### AEO-H1 (High): Missing structured product specifications
**Where to look:** Product templates, metafield-rendering snippets
**What to find:** A specifications section using `<dl><dt><dd>` semantic markup, OR a clear table, OR metafields surfaced as `additionalProperty` in Product schema
**Flag if:** Product specs (dimensions, materials, weight, capacity) are buried in prose paragraphs
**Correct pattern — specs as PropertyValue:**
```liquid
{%- assign specs = product.metafields.custom.specifications.value -%}
{%- if specs != blank -%}
<dl class="product-specs">
  {%- for spec in specs -%}
    <dt>{{ spec.name }}</dt>
    <dd>{{ spec.value }}</dd>
  {%- endfor -%}
</dl>
{%- endif -%}
```
And in Product schema:
```liquid
"additionalProperty": [
  {%- for spec in specs -%}
    {
      "@type": "PropertyValue",
      "name": {{ spec.name | json }},
      "value": {{ spec.value | json }}
    }{%- unless forloop.last -%},{%- endunless -%}
  {%- endfor -%}
]
```

### AEO-H2 (High): No `HowTo` schema on instruction or usage pages
**Where to look:** Blog articles, "how to use" pages, care instruction pages
**Flag if:** Step-by-step content exists in HTML but no `HowTo` schema is emitted
**Why it matters:** AI Overviews surface HowTo schema heavily for instructional queries

### AEO-L2 (Low): No Speakable schema — *demoted from High in v2.1*
**Where to look:** Blog articles, FAQ pages
**What to find:** `speakable` property on Article schema marking the summary CSS selector
**Status:** Google restricts `speakable` to a narrow set of news publishers and it has no demonstrated effect on ecommerce AI citations. Scoring it as High (−5) was not defensible.
**Flag if:** Only when the store publishes news-style editorial content and the merchant has asked about voice surfaces. Otherwise skip.
**Scoring:** Low (−1), or omit.
**Correct pattern:**
```liquid
"speakable": {
  "@type": "SpeakableSpecification",
  "cssSelector": [".article-summary", ".article-tldr"]
}
```

### AEO-H4 (High): No author or publisher signals (E-E-A-T)
**Where to look:** `sections/main-article.liquid`, `templates/blog.liquid`
**What to find:** `Article` schema with `author` (Person) and `publisher` (Organization) including `logo`
**Flag if:** Blog posts emit no Article schema or omit author/publisher
**Why it matters:** Google's E-E-A-T signals and LLM trust scoring both depend on identifiable authorship

### AEO-M1 (Medium): No `dateModified` on content pages
**Where to look:** Blog and article templates
**Flag if:** Article schema includes `datePublished` but not `dateModified` — freshness signals matter for both Google and AI citation choice

### AEO-M2 (Medium): Reviews not exposed as `Review` schema
**Where to look:** Product page, reviews section
**Flag if:** Reviews are rendered visually but only `aggregateRating` (not individual `Review` entries) is in schema
**Why it matters:** Individual reviews are quotable by LLMs as social proof

### AEO-M3 (Medium): Product page lacks comparison content
**Where to look:** Product description, "compare" snippet
**Flag if:** No comparison to alternatives, no "best for" framing, no use-case differentiation
**Why it matters:** When a buyer asks Claude or ChatGPT "should I get X or Y", pages with explicit comparison content get cited

### AEO-L1 (Low): No table of contents on long articles
**Flag if:** Articles over 1500 words have no jump links or TOC — both reduces AI parseability and hurts user experience

---

## GEO — GENERATIVE ENGINE OPTIMIZATION

GEO is about being included when an LLM-powered surface (Google AI Overviews, Bing Copilot, ChatGPT search, Perplexity) generates a shopping answer. The page needs to be crawlable, parseable, and trustable by the model's retriever.

### GEO-C1 (Critical): `robots.txt.liquid` blocks AI crawlers without intent
**Where to look:** `templates/robots.txt.liquid` (if customized)
**What to find:** Explicit rules for the **retrieval** bots (`OAI-SearchBot`, `PerplexityBot`, `Google-Extended`, `ClaudeBot`) as well as the training bot (`GPTBot`)
**Flag if:** AI crawlers are disallowed without the merchant having made a deliberate choice to opt out — **or** if the file allows `GPTBot` but omits `OAI-SearchBot` (see `R-C1` for why these are not the same thing)
**Why it matters:** Many themes carry over copy-pasted robots.txt blocks from 2023 SEO advice that fully blocked AI crawlers. If the merchant *wants* GEO visibility, these must be removed.

> **Corrected in v2.1, corrected again in v2.3.** Two rounds of the same mistake, stated plainly:
>
> - **v2.0** recommended an allow-block that omitted `OAI-SearchBot`, opting stores into training and out of ChatGPT Search citations.
> - **v2.1** fixed OpenAI but repeated the identical error on Anthropic — it listed `ClaudeBot` as Anthropic's single crawler and placed it in the *retrieval* group. `ClaudeBot` is the **training** crawler. `Claude-SearchBot` (citations) and `Claude-User` (user fetches) were missing entirely.
>
> **If you applied either version's robots.txt recommendation, re-check the file against the block below.**

**Default recommendation (allow):**
```
# In templates/robots.txt.liquid
{%- for group in robots.default_groups -%}
  {{ group | newline_to_br | strip_html }}
{%- endfor -%}

# --- RETRIEVAL: gates whether you can be CITED ---
User-agent: OAI-SearchBot
Allow: /

User-agent: Claude-SearchBot
Allow: /

User-agent: PerplexityBot
Allow: /

# --- USER-INITIATED FETCH ---
User-agent: ChatGPT-User
Allow: /

User-agent: Claude-User
Allow: /

# --- DUAL: Gemini training AND AI Overviews grounding ---
User-agent: Google-Extended
Allow: /

# --- TRAINING: separate decision, safe to omit to opt out of training ---
User-agent: GPTBot
Allow: /

User-agent: ClaudeBot
Allow: /

Sitemap: {{ shop.url }}/sitemap.xml
```
**Note on merchant intent:** Allowing the retrieval bots while disallowing `GPTBot` **and** `ClaudeBot` is a coherent, defensible position — cited but not trained on. Do not flag that combination as an error; it is a deliberate and increasingly common choice. The incoherent combination is the reverse: training allowed, citations blocked.

**When you report this, name the specific bot.** "Blocked from AI crawlers" is not an actionable finding and is usually wrong in detail. "Blocks `Claude-SearchBot`, so the store cannot be cited in Claude's search results — `ClaudeBot` is separately blocked, which only affects training" is.

### GEO-L3 (Low): No `llms.txt` — *demoted from Critical in v2.1; do not deduct 10 points for this*
**Where to look:** `templates/page.llms.liquid`, or a static `llms.txt` served from the domain
**Status:** **Retired as a Critical check. The evidence does not support it.**

Earlier versions of this checklist scored a missing `llms.txt` as Critical (−10). That was wrong, and it cost audited stores ten points for the absence of a file that does essentially nothing. What the 2026 data actually shows:

- No major AI provider reads `llms.txt` in production. GPTBot, ClaudeBot, PerplexityBot, `OAI-SearchBot`, and Google-Extended overwhelmingly skip it and crawl HTML directly.
- An Ahrefs study of ~137,000 sites found **97% of `llms.txt` files received zero traffic**. One instrumented domain logged 84 requests to `/llms.txt` out of 62,100 total AI-bot visits — **0.1%**.
- Google's June 2026 documentation update states `llms.txt` has **no effect, positive or negative**, on Search rankings or AI Overviews.
- Large-scale studies find no relationship between having the file and being cited.

**Flag if:** Nothing. Do not flag its absence.
**Mention only if:** The merchant asks about it directly, or already maintains one and is allocating real effort to it — in which case tell them the effort is better spent on `R-C2` (domain coverage gaps), which is the check that actually feeds ChatGPT's site:-scoped fanouts.
**The one real exception:** Documentation sites serving coding assistants (Cursor, Continue, Cline, MCP integrations) genuinely consume `llms.txt`. That is not a Shopify storefront use case.
**Scoring:** Low (−1) at most, and only when the merchant has explicitly stated they want it. Default: omit from the report entirely.

### GEO-H1 (High): Brand `Organization` schema absent or incomplete
**Where to look:** `layout/theme.liquid` or `snippets/organization-schema.liquid`
**What to find:** `Organization` schema with `name`, `url`, `logo`, `sameAs` (social profiles), `contactPoint`
**Flag if:** No Organization schema, or it omits `logo` or `sameAs`
**Why it matters:** LLMs need to disambiguate the brand. Without Organization schema and social profile links, the store risks being confused with similarly-named entities.
**Correct pattern:**
```liquid
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": {{ shop.name | json }},
  "url": {{ shop.url | json }},
  "logo": {{ settings.logo | image_url: width: 600 | prepend: 'https:' | json }},
  "sameAs": [
    {%- if settings.social_instagram_link != blank -%}{{ settings.social_instagram_link | json }},{%- endif -%}
    {%- if settings.social_twitter_link != blank -%}{{ settings.social_twitter_link | json }},{%- endif -%}
    {%- if settings.social_facebook_link != blank -%}{{ settings.social_facebook_link | json }}{%- endif -%}
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "contactType": "customer service",
    "email": {{ shop.email | json }}
  }
}
</script>
```

### GEO-H2 (High): No `BreadcrumbList` schema
**Where to look:** Product and collection templates
**Flag if:** Breadcrumb UI exists but no JSON-LD `BreadcrumbList` is emitted
**Why it matters:** Breadcrumbs are how LLM retrievers understand a product's place in the catalog hierarchy

### GEO-H3 (High): Critical content inside JavaScript-rendered DOM
**Where to look:** Product details, pricing, reviews
**Flag if:** Price, description, or reviews are hydrated by JS post-load
**Why it matters:** Most LLM crawlers (GPTBot, ClaudeBot) do not execute JavaScript. Content rendered after page load is invisible to them.

### GEO-H4 (High): Brand description thin or only in image form
**Where to look:** Homepage, `/pages/about`
**Flag if:** The "about" page is mostly imagery with little crawlable text describing what the brand sells, where it ships, founding story, materials
**Why it matters:** When an LLM is asked "tell me about [brand]", it needs text to quote. Image-only "about" pages produce empty or hallucinated answers.

### GEO-M1 (Medium): No shipping, returns, or policy page schema
**Where to look:** `shop.shipping_policy`, `shop.refund_policy`
**Flag if:** Policy pages exist but are not referenced from Product schema `shippingDetails` and `hasMerchantReturnPolicy`
**Correct pattern in Product schema:**
```liquid
"shippingDetails": {
  "@type": "OfferShippingDetails",
  "shippingDestination": {
    "@type": "DefinedRegion",
    "addressCountry": "US"
  },
  "shippingRate": {
    "@type": "MonetaryAmount",
    "value": "0",
    "currency": "USD"
  }
},
"hasMerchantReturnPolicy": {
  "@type": "MerchantReturnPolicy",
  "applicableCountry": "US",
  "returnPolicyCategory": "https://schema.org/MerchantReturnFiniteReturnWindow",
  "merchantReturnDays": 30,
  "returnMethod": "https://schema.org/ReturnByMail",
  "returnFees": "https://schema.org/FreeReturn"
}
```

### GEO-M2 (Medium): No mention of materials, origin, or specifics in machine-readable form
**Flag if:** Product copy says "ethically made" but no `countryOfOrigin`, `material`, or sustainability fields are in schema or metafields

### GEO-M3 (Medium): Product image URLs do not include descriptive context
**Flag if:** Image filenames are `IMG_0234.jpg` rather than `aurora-22l-waterproof-backpack-navy.jpg`
**Why it matters:** Image URLs and alt text are how multimodal models understand product visuals

### GEO-L1 (Low): No `Article` or `BlogPosting` schema on blog content
**Flag if:** Blog posts have no Article schema — limits citation in LLM-driven content research

### GEO-L2 (Low): No structured contact information in footer
**Flag if:** Footer omits machine-readable address, phone, or contact link

---

## POSITIVE FINDINGS — Add to "What This Theme Does Well"

Note when any of the following is correctly implemented:
- Title and meta description templating that varies per page
- Canonical tag correctly handling pagination and variants
- Product schema using `{{ product | structured_data }}` or hand-rolled equivalent
- FAQ schema emitted alongside FAQ accordion UI
- Organization schema with `sameAs` social profiles populated
- BreadcrumbList schema matching visible breadcrumb UI
- Hreflang tags on multi-region stores
- Open Graph and Twitter Card tags on every template
- **Retrieval crawlers allowed in robots.txt — specifically `OAI-SearchBot` and `Claude-SearchBot`, not just the training crawlers `GPTBot` and `ClaudeBot`**
- **A dedicated on-domain page exists for shipping, returns, sizing, and materials (satisfies site:-scoped fanouts)**
- **Reviews and specs rendered server-side into HTML rather than injected by a third-party widget**
- `dateModified` on article schema and visible "last updated" in UI
- Product description opens with a clear, definition-style factual sentence
- Specifications rendered as semantic `<dl>` and PropertyValue in schema
- Heading hierarchy is clean and single-H1 per page
