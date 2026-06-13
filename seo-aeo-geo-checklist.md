# SEO, AEO, and GEO Audit Checklist

This file extends the main audit with checks for traditional SEO (Google, Bing), Answer Engine Optimization (AEO — ChatGPT, Claude, Perplexity, Gemini answers), and Generative Engine Optimization (GEO — being cited by AI Overviews and LLM-powered search).

Use this checklist alongside `audit-checklist.md`. Each finding follows the same severity model: Critical (−10), High (−5), Medium (−2), Low (−1).

---

## Why this matters for Shopify in 2026

Search has split into three layers:

1. **Classic SEO** — Google, Bing, DuckDuckGo. Still drives the majority of ecommerce traffic.
2. **AEO** — ChatGPT, Claude, Perplexity, Gemini direct answers. A growing share of product research now happens here. The user never sees a results page.
3. **GEO** — AI Overviews on Google, Bing Copilot, and LLM-powered shopping assistants. The store either gets cited or it does not exist to the buyer.

A theme that ignores AEO and GEO will lose visibility to competitors whose pages are structured in ways that LLMs can quote.

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

## AEO — ANSWER ENGINE OPTIMIZATION

AEO optimizes the page for being quoted verbatim by ChatGPT, Claude, Perplexity, and Gemini when users ask product research questions. The mechanic is different from SEO: the model needs short, factual, well-attributed statements it can pull as a citation.

### AEO-C1 (Critical): No FAQ schema on product or info pages
**Where to look:** `sections/main-product.liquid`, `templates/page.faq.liquid`, FAQ accordion sections
**What to find:** `FAQPage` JSON-LD with question/answer pairs
**Flag if:** Theme renders FAQ accordion UI but does not emit `FAQPage` schema
**Why it matters:** FAQ schema is the single highest-leverage AEO signal. Google AI Overviews and ChatGPT both pull from it.
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

### AEO-H3 (High): No Speakable schema for voice/assistant excerpts
**Where to look:** Blog articles, FAQ pages
**What to find:** `speakable` property on Article schema marking the summary CSS selector
**Flag if:** Long-form content has no speakable hint
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
**What to find:** Explicit `User-agent: GPTBot`, `User-agent: ClaudeBot`, `User-agent: PerplexityBot`, `User-agent: Google-Extended` rules
**Flag if:** AI crawlers are disallowed without the merchant having made a deliberate choice to opt out
**Why it matters:** Many themes carry over copy-pasted robots.txt blocks from 2023 SEO advice that fully blocked AI crawlers. If the merchant *wants* GEO visibility, these must be removed.
**Default recommendation (allow):**
```
# In templates/robots.txt.liquid
{%- for group in robots.default_groups -%}
  {{ group | newline_to_br | strip_html }}
{%- endfor -%}

User-agent: GPTBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Google-Extended
Allow: /

Sitemap: {{ shop.url }}/sitemap.xml
```

### GEO-C2 (Critical): No `llms.txt` or equivalent AI-readable site map
**Where to look:** Project root, `templates/page.llms.liquid`, or a static `llms.txt` served from the domain
**What to find:** A plain-text overview of the store's primary collections, top products, and brand description optimized for LLM ingestion
**Flag if:** No `llms.txt` exists at the store root
**Why it matters:** The emerging `llms.txt` convention lets LLMs grasp the store quickly without crawling every product page
**Note:** Shopify does not serve arbitrary root files, but a `page.llms` template at `/pages/llms-txt` is a reasonable substitute. Recommend the merchant publish this.

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
- AI crawlers (GPTBot, ClaudeBot, PerplexityBot) allowed in robots.txt
- `dateModified` on article schema and visible "last updated" in UI
- Product description opens with a clear, definition-style factual sentence
- Specifications rendered as semantic `<dl>` and PropertyValue in schema
- Heading hierarchy is clean and single-H1 per page
