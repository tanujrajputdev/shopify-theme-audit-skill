# Before / After — Working Fix Gallery

This file is a reference of exact code replacements. When you flag an issue from `audit-checklist.md` or `seo-aeo-geo-checklist.md`, paste the matching RIGHT block into the report — do not paraphrase, do not invent. Every pair below is production-tested against Online Store 2.0.

Each entry is paired by check ID. Find the issue in the audit checklist, then find the matching ID here.

---

## C1 — Render-blocking script in `<head>`

### WRONG
```liquid
{% comment %} layout/theme.liquid — head {% endcomment %}
<script src="{{ 'vendor.js' | asset_url }}"></script>
<script src="{{ 'theme.js' | asset_url }}"></script>
```

### RIGHT
```liquid
{% comment %} layout/theme.liquid — head {% endcomment %}
<script src="{{ 'vendor.js' | asset_url }}" defer></script>
<script src="{{ 'theme.js' | asset_url }}" defer></script>

{% comment %} Only inline tracking / consent scripts stay synchronous {% endcomment %}
```

---

## C2 — Deprecated `img_url` filter

### WRONG
```liquid
<img src="{{ product.featured_image | img_url: '600x600' }}"
     alt="{{ product.title }}">
```

### RIGHT
```liquid
<img src="{{ product.featured_image | image_url: width: 600 }}"
     srcset="{{ product.featured_image | image_url: width: 300 }} 300w,
             {{ product.featured_image | image_url: width: 600 }} 600w,
             {{ product.featured_image | image_url: width: 1200 }} 1200w"
     sizes="(min-width: 750px) 600px, 100vw"
     width="{{ product.featured_image.width }}"
     height="{{ product.featured_image.height }}"
     alt="{{ product.featured_image.alt | default: product.title | escape }}"
     loading="lazy">
```

---

## C3 — Images without lazy loading

### WRONG
```liquid
{% for product in collection.products %}
  <img src="{{ product.featured_image | image_url: width: 400 }}"
       alt="{{ product.title }}">
{% endfor %}
```

### RIGHT
```liquid
{% for product in collection.products %}
  <img src="{{ product.featured_image | image_url: width: 400 }}"
       width="{{ product.featured_image.width }}"
       height="{{ product.featured_image.height }}"
       alt="{{ product.featured_image.alt | default: product.title | escape }}"
       loading="{% if forloop.first %}eager{% else %}lazy{% endif %}"
       fetchpriority="{% if forloop.first %}high{% else %}auto{% endif %}">
{% endfor %}
```

---

## C4 — N+1 metafield access inside a loop

### WRONG
```liquid
{% for product in collection.products %}
  <span>{{ product.metafields.custom.material.value }}</span>
  <span>{{ product.metafields.custom.origin.value }}</span>
{% endfor %}
```

### RIGHT
```liquid
{% comment %}
  Resolve metafields once per product up front, then read locally.
  For collections with >50 products, batch metafield reads via the Storefront API
  or move secondary-detail rendering to a deferred fetch.
{% endcomment %}
{% for product in collection.products %}
  {% assign material = product.metafields.custom.material.value %}
  {% assign origin = product.metafields.custom.origin.value %}
  {% if material != blank %}<span>{{ material }}</span>{% endif %}
  {% if origin != blank %}<span>{{ origin }}</span>{% endif %}
{% endfor %}
```

---

## C5 — Missing skip-to-content link

### WRONG
```liquid
{% comment %} layout/theme.liquid {% endcomment %}
<body>
  {% section 'header' %}
  <main>{{ content_for_layout }}</main>
```

### RIGHT
```liquid
<body>
  <a href="#MainContent" class="skip-to-content-link">
    {{ 'accessibility.skip_to_text' | t }}
  </a>
  {% section 'header' %}
  <main id="MainContent" tabindex="-1">
    {{ content_for_layout }}
  </main>
```

```css
/* assets/base.css */
.skip-to-content-link {
  position: absolute;
  left: 0;
  top: 0;
  padding: 0.75rem 1rem;
  background: #000;
  color: #fff;
  transform: translateY(-100%);
  z-index: 9999;
}
.skip-to-content-link:focus { transform: translateY(0); }
```

---

## C6 — Image missing width or height (CLS)

### WRONG
```liquid
<img src="{{ 'logo.png' | asset_url }}" alt="{{ shop.name }}">
```

### RIGHT
```liquid
<img src="{{ 'logo.png' | asset_url }}"
     alt="{{ shop.name }}"
     width="180"
     height="60"
     fetchpriority="high">
```

For responsive images, always set the **intrinsic** dimensions — the browser reads the ratio:

```liquid
<img src="{{ image | image_url: width: 1200 }}"
     width="{{ image.width }}"
     height="{{ image.height }}"
     style="aspect-ratio: {{ image.aspect_ratio }};"
     alt="{{ image.alt | escape }}">
```

---

## C7 — Oversized `json` metafield (2026-04 128KB cap)

Limit: **128KB** for `json` metafields on API 2026-04+ ([changelog](https://shopify.dev/changelog/reduced-metafield-value-sizes)). Values written before 1 April 2026 are grandfathered at 2MB and still *read* fine — they fail on the next *write*. Shopify Functions receive `null` for any metafield over **10,000 bytes**.

### WRONG
```liquid
{% comment %} One blob holding every spec, parsed on every product card {% endcomment %}
{%- assign specs = product.metafields.custom.spec_table.value -%}
<span>{{ specs.rows[0].value }}</span>
```

### RIGHT
```liquid
{% comment %}
  snippets/product-card.liquid
  Small metafield for the card. The heavy one never loads here.
{% endcomment %}
{%- assign summary = product.metafields.custom.spec_summary.value -%}
{%- if summary != blank -%}
  <span class="card-spec">{{ summary.headline }}</span>
{%- endif -%}
```

```liquid
{% comment %}
  sections/main-product.liquid
  Full detail only on the PDP, and only the fields rendered.
{% endcomment %}
{%- assign detail = product.metafields.custom.spec_detail.value -%}
{%- if detail != blank -%}
  <dl class="product-specs">
    {%- for row in detail.rows -%}
      <dt>{{ row.label }}</dt>
      <dd>{{ row.value }}</dd>
    {%- endfor -%}
  </dl>
{%- endif -%}
```

Prefer metaobjects when the data is genuinely relational — a list of metaobject references holds up to 1024 items and loads only the entries referenced.

---

## H1 — JS without defer

### WRONG
```liquid
<script src="{{ 'cart.js' | asset_url }}"></script>
```

### RIGHT
```liquid
<script src="{{ 'cart.js' | asset_url }}" defer></script>

{% comment %} If you need execution order independent of DOM, use type="module" {% endcomment %}
<script src="{{ 'cart.js' | asset_url }}" type="module"></script>
```

---

## H3 — No preload hint for LCP image

### WRONG
```liquid
{% comment %} layout/theme.liquid — no preload {% endcomment %}
```

### RIGHT
```liquid
{% comment %} layout/theme.liquid — inside <head>, after meta tags {% endcomment %}
{% if template == 'index' and settings.hero_image %}
  <link rel="preload"
        as="image"
        href="{{ settings.hero_image | image_url: width: 1600 }}"
        imagesrcset="{{ settings.hero_image | image_url: width: 800 }} 800w,
                     {{ settings.hero_image | image_url: width: 1600 }} 1600w,
                     {{ settings.hero_image | image_url: width: 2400 }} 2400w"
        imagesizes="100vw"
        fetchpriority="high">
{% endif %}
```

---

## H6 — Canonical wrong on paginated collection

### WRONG
```liquid
<link rel="canonical" href="{{ canonical_url }}">
```
(emits `/collections/all?page=2` as canonical of itself — splits ranking signals)

### RIGHT
```liquid
{% if template contains 'collection' and current_page > 1 %}
  <link rel="canonical" href="{{ shop.url }}{{ collection.url }}">
{% else %}
  <link rel="canonical" href="{{ canonical_url }}">
{% endif %}
```

---

## M2 — Missing or empty alt text

### WRONG
```liquid
<img src="{{ image | image_url: width: 800 }}" alt="">
<img src="{{ image | image_url: width: 800 }}">
```

### RIGHT
```liquid
{% comment %} Informational image — derive alt with a safe fallback {% endcomment %}
<img src="{{ image | image_url: width: 800 }}"
     alt="{{ image.alt | default: product.title | escape }}">

{% comment %} Truly decorative — explicit empty alt + role {% endcomment %}
<img src="{{ image | image_url: width: 800 }}" alt="" role="presentation">
```

---

## M4 — No product structured data

### WRONG
```liquid
{% comment %} sections/main-product.liquid — no JSON-LD {% endcomment %}
```

### RIGHT
```liquid
{% comment %} sections/main-product.liquid — near the closing tag {% endcomment %}
<script type="application/ld+json">
{{ product | structured_data }}
</script>
```

For full control, build it manually:

```liquid
<script type="application/ld+json">
{
  "@context": "https://schema.org/",
  "@type": "Product",
  "name": {{ product.title | json }},
  "image": [
    {% for img in product.images limit: 5 %}
      {{ img | image_url: width: 1600 | prepend: 'https:' | json }}{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ],
  "description": {{ product.description | strip_html | json }},
  "sku": {{ product.selected_or_first_available_variant.sku | json }},
  "brand": { "@type": "Brand", "name": {{ product.vendor | json }} },
  "offers": {
    "@type": "Offer",
    "url": {{ shop.url | append: product.url | json }},
    "priceCurrency": {{ cart.currency.iso_code | json }},
    "price": {{ product.selected_or_first_available_variant.price | money_without_currency | strip | json }},
    "availability": "{% if product.available %}https://schema.org/InStock{% else %}https://schema.org/OutOfStock{% endif %}",
    "itemCondition": "https://schema.org/NewCondition"
  }
}
</script>
```

---

## M5 — Breadcrumbs without schema

### WRONG
```liquid
<nav class="breadcrumbs">
  <a href="/">Home</a> / <a href="{{ collection.url }}">{{ collection.title }}</a> / {{ product.title }}
</nav>
```

### RIGHT
```liquid
<nav class="breadcrumbs" aria-label="Breadcrumb">
  <ol>
    <li><a href="{{ shop.url }}">Home</a></li>
    {% if collection %}
      <li><a href="{{ collection.url }}">{{ collection.title }}</a></li>
    {% endif %}
    <li aria-current="page">{{ product.title }}</li>
  </ol>
</nav>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": {{ shop.url | json }} }
    {%- if collection -%},
    { "@type": "ListItem", "position": 2, "name": {{ collection.title | json }},
      "item": {{ shop.url | append: collection.url | json }} }
    {%- endif -%},
    { "@type": "ListItem",
      "position": {% if collection %}3{% else %}2{% endif %},
      "name": {{ product.title | json }},
      "item": {{ shop.url | append: product.url | json }} }
  ]
}
</script>
```

---

## S-C1 — Title tag missing or duplicated

### WRONG
```liquid
<title>{{ shop.name }}</title>
```

### RIGHT
```liquid
<title>
  {%- if page_title -%}
    {{ page_title }}{% if current_tags %} – tagged "{{ current_tags | join: ', ' }}"{% endif %}{% if current_page != 1 %} – Page {{ current_page }}{% endif %}
    {%- unless page_title contains shop.name %} – {{ shop.name }}{% endunless -%}
  {%- else -%}
    {{ shop.name }}
  {%- endif -%}
</title>
```

---

## S-C2 — Meta description missing

### WRONG
```liquid
{% comment %} no meta description emitted {% endcomment %}
```

### RIGHT
```liquid
{%- if page_description -%}
  <meta name="description" content="{{ page_description | escape }}">
{%- elsif template contains 'product' -%}
  <meta name="description" content="{{ product.description | strip_html | truncate: 155 | escape }}">
{%- elsif template contains 'collection' -%}
  <meta name="description" content="{{ collection.description | default: collection.title | strip_html | truncate: 155 | escape }}">
{%- else -%}
  <meta name="description" content="{{ shop.description | default: shop.name | escape }}">
{%- endif -%}
```

---

## S-H3 — Open Graph tags missing

### WRONG
```liquid
{% comment %} no OG tags — social shares show a broken card {% endcomment %}
```

### RIGHT
```liquid
<meta property="og:site_name" content="{{ shop.name }}">
<meta property="og:url" content="{{ canonical_url }}">
<meta property="og:title" content="{{ page_title | default: shop.name | escape }}">
<meta property="og:type" content="{% if template contains 'product' %}product{% elsif template contains 'article' %}article{% else %}website{% endif %}">
<meta property="og:description" content="{{ page_description | default: shop.description | escape }}">

{%- if template contains 'product' and product.featured_image -%}
  <meta property="og:image" content="https:{{ product.featured_image | image_url: width: 1200 }}">
  <meta property="og:image:width" content="1200">
  <meta property="og:image:height" content="{{ 1200 | divided_by: product.featured_image.aspect_ratio | round }}">
{%- elsif settings.share_image -%}
  <meta property="og:image" content="https:{{ settings.share_image | image_url: width: 1200 }}">
{%- endif -%}

<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="{{ page_title | default: shop.name | escape }}">
<meta name="twitter:description" content="{{ page_description | default: shop.description | escape }}">
```

---

## AEO-C1 — FAQ accordion without `FAQPage` schema

### WRONG
```liquid
{% for block in section.blocks %}
  <details>
    <summary>{{ block.settings.question }}</summary>
    <div>{{ block.settings.answer }}</div>
  </details>
{% endfor %}
```

### RIGHT
```liquid
{% for block in section.blocks %}
  <details>
    <summary>{{ block.settings.question }}</summary>
    <div>{{ block.settings.answer }}</div>
  </details>
{% endfor %}

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {% for block in section.blocks %}
    {
      "@type": "Question",
      "name": {{ block.settings.question | json }},
      "acceptedAnswer": {
        "@type": "Answer",
        "text": {{ block.settings.answer | strip_html | json }}
      }
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ]
}
</script>
```

---

## AEO-C2 — PDP opens with marketing fluff, not a factual definition

### WRONG
```html
<h1>Experience the comfort you deserve.</h1>
<p>You're going to love how this feels every single morning…</p>
```

### RIGHT
```liquid
<h1>{{ product.title }}</h1>
<p class="product-summary">
  {%- comment -%}
    First sentence: what it is + key fact, factual tone.
    AI engines quote the first 1–2 sentences when answering "what is X".
  {%- endcomment -%}
  {{ product.metafields.custom.factual_summary.value | default: product.description | strip_html | truncate: 240 }}
</p>
```

Or hardcode in a metaobject and reference it:

```liquid
<p class="product-summary">
  The {{ product.title }} is a {{ product.type | downcase }} made of
  {{ product.metafields.custom.material.value }}, designed for
  {{ product.metafields.custom.use_case.value }}.
</p>
```

---

## AEO-H1 — Specs in prose instead of `PropertyValue`

### WRONG
```liquid
<p>Made of organic cotton, 220 GSM, weighs 320 grams, machine washable at 30°C.</p>
```

### RIGHT
```liquid
<dl class="product-specs">
  <dt>Material</dt><dd>{{ product.metafields.custom.material.value }}</dd>
  <dt>Weight</dt><dd>{{ product.metafields.custom.weight_grams.value }} g</dd>
  <dt>Fabric weight</dt><dd>{{ product.metafields.custom.gsm.value }} GSM</dd>
  <dt>Care</dt><dd>{{ product.metafields.custom.care_instructions.value }}</dd>
</dl>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": {{ product.title | json }},
  "additionalProperty": [
    { "@type": "PropertyValue", "name": "Material", "value": {{ product.metafields.custom.material.value | json }} },
    { "@type": "PropertyValue", "name": "Weight", "value": {{ product.metafields.custom.weight_grams.value | json }}, "unitText": "g" },
    { "@type": "PropertyValue", "name": "Fabric weight", "value": {{ product.metafields.custom.gsm.value | json }}, "unitText": "GSM" }
  ]
}
</script>
```

---

## AEO-H4 — No author / publisher on articles (E-E-A-T)

### WRONG
```liquid
<h1>{{ article.title }}</h1>
<p>By {{ article.author }}</p>
```

### RIGHT
```liquid
<h1>{{ article.title }}</h1>
<p>By <a href="/pages/author-{{ article.author | handleize }}">{{ article.author }}</a></p>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": {{ article.title | json }},
  "datePublished": {{ article.published_at | date: '%Y-%m-%dT%H:%M:%S%z' | json }},
  "dateModified": {{ article.updated_at | date: '%Y-%m-%dT%H:%M:%S%z' | json }},
  "author": {
    "@type": "Person",
    "name": {{ article.author | json }},
    "url": {{ shop.url | append: '/pages/author-' | append: article.author | handleize | json }}
  },
  "publisher": {
    "@type": "Organization",
    "name": {{ shop.name | json }},
    "logo": {
      "@type": "ImageObject",
      "url": "https:{{ settings.logo | image_url: width: 600 }}"
    }
  },
  "image": "https:{{ article.image | image_url: width: 1600 }}",
  "mainEntityOfPage": {{ shop.url | append: article.url | json }}
}
</script>
```

---

## R-C2 — Answer content exists only as an image or PDF

A `site:`-scoped fanout retrieves **text**. A size chart shipped as a JPG is invisible to it.

### WRONG
```liquid
{% comment %} sections/size-guide.liquid {% endcomment %}
<h2>Size Guide</h2>
<img src="{{ 'size-chart.jpg' | asset_url }}" alt="Size chart">
<a href="{{ 'sizing.pdf' | asset_url }}">Download sizing PDF</a>
```

### RIGHT
```liquid
{% comment %} sections/size-guide.liquid — real text, server-rendered {% endcomment %}
{%- assign sizes = product.metafields.custom.size_chart.value -%}
<h2>Size guide</h2>
{%- if sizes != blank -%}
  <table class="size-guide">
    <caption>{{ product.title }} measurements in centimetres</caption>
    <thead>
      <tr>
        <th scope="col">Size</th>
        <th scope="col">Chest</th>
        <th scope="col">Waist</th>
        <th scope="col">Length</th>
      </tr>
    </thead>
    <tbody>
      {%- for row in sizes -%}
        <tr>
          <th scope="row">{{ row.size }}</th>
          <td>{{ row.chest }} cm</td>
          <td>{{ row.waist }} cm</td>
          <td>{{ row.length }} cm</td>
        </tr>
      {%- endfor -%}
    </tbody>
  </table>
  <p>Between sizes? {{ shop.name }} recommends sizing up for a relaxed fit.</p>
{%- endif -%}
```

Keep the image as a visual aid if you like — but the measurements must also exist as text in the HTML. Apply the same rule to shipping, returns, materials, and warranty pages.

---

## R-C3 / R-H2 — Reviews and specs injected by JavaScript

Content a third-party widget renders client-side is not in the HTML, so retrieval crawlers cannot read it and a `site:`-scoped search cannot find it. It is also off-domain content the merchant does not control.

### WRONG
```liquid
{% comment %} sections/main-product.liquid {% endcomment %}
<div id="reviews-widget" data-product="{{ product.id }}"></div>
<script src="https://cdn.reviews-app.example/widget.js" async></script>
```

### RIGHT
```liquid
{% comment %} sections/main-product.liquid — server-rendered reviews from metafields {% endcomment %}
{%- assign reviews = product.metafields.reviews.list.value -%}
{%- if reviews != blank -%}
<section class="product-reviews" id="reviews">
  <h2>Customer reviews for {{ product.title }}</h2>
  {%- for review in reviews -%}
    <article class="review">
      <h3>{{ review.title }}</h3>
      <p class="review-rating">Rated {{ review.rating }} out of 5</p>
      <p class="review-body">{{ review.body }}</p>
      <p class="review-meta">{{ review.author }} — {{ review.date }}</p>
    </article>
  {%- endfor -%}
</section>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": {{ product.title | json }},
  "review": [
    {%- for review in reviews -%}
      {
        "@type": "Review",
        "author": { "@type": "Person", "name": {{ review.author | json }} },
        "datePublished": {{ review.date | json }},
        "name": {{ review.title | json }},
        "reviewBody": {{ review.body | json }},
        "reviewRating": {
          "@type": "Rating",
          "ratingValue": {{ review.rating | json }},
          "bestRating": "5"
        }
      }{%- unless forloop.last -%},{%- endunless -%}
    {%- endfor -%}
  ]
}
</script>
</section>
{%- endif -%}
```

**How to verify the fix:** open view-source (not the inspector — the inspector shows the post-JavaScript DOM) and search for the review text. If it is not in the raw HTML, it is not retrievable.

**Note on app-hosted reviews:** many review apps can sync reviews into Shopify metafields, which is what makes the block above possible. If the app cannot, that is a genuine reason to reconsider the app — the reviews are trust content the merchant is renting rather than owning.

---

## GEO-C1 — `robots.txt.liquid` blocks AI crawlers

### WRONG
```liquid
{% comment %} templates/robots.txt.liquid {% endcomment %}
User-agent: GPTBot
Disallow: /

User-agent: ClaudeBot
Disallow: /

User-agent: PerplexityBot
Disallow: /
```

### RIGHT
```liquid
{% comment %} templates/robots.txt.liquid — allow AI bots, keep checkout disallowed {% endcomment %}
{% for group in robots.default_groups %}
  {{- group.user_agent }}
  {% for rule in group.rules %}
    {{- rule }}
  {% endfor %}
  {% if group.sitemap %}
    {{- group.sitemap }}
  {% endif %}
{% endfor %}

# --- RETRIEVAL BOTS: these gate whether the store can be CITED ---

User-agent: OAI-SearchBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Google-Extended
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Claude-Web
Allow: /

# --- TRAINING BOTS: separate decision, safe to omit if opting out of training ---

User-agent: GPTBot
Allow: /

User-agent: CCBot
Allow: /
```

> **Corrected in v2.1 — check your theme if you applied the earlier version of this block.**
> The v2.0 RIGHT block allowed `GPTBot` but omitted `OAI-SearchBot`. Those are different crawlers with different jobs: `GPTBot` collects **training** data, while `OAI-SearchBot` builds the index behind **ChatGPT Search citations**. A store running the old block opted into training and out of citations — the inverse of what merchants almost always want.
>
> This matters more since ChatGPT Search began issuing `site:`-scoped fanout queries at scale on August 8, 2026 (0.37% → 16.8% of all fanouts in a single day). Those scoped searches have to reach your domain to return anything.
>
> Allowing the retrieval bots while disallowing `GPTBot` is a coherent position — cited but not trained on. Do not flag that combination. The incoherent one is the reverse.

---

## GEO-L3 — `llms.txt` — RETIRED CHECK, do not flag

**Removed as a scored check in v2.1. Do not emit a finding for a missing `llms.txt`, and do not paste a RIGHT block for it.**

v2.0 scored a missing `llms.txt` as Critical (−10) and listed creating one as a quick win. The 2026 evidence supports neither:

- No major AI provider reads `llms.txt` in production. `GPTBot`, `ClaudeBot`, `PerplexityBot`, `OAI-SearchBot`, and `Google-Extended` overwhelmingly skip it and crawl HTML directly.
- An Ahrefs study of ~137,000 sites found **97% of `llms.txt` files received zero traffic.** One instrumented domain logged 84 requests to `/llms.txt` out of 62,100 total AI-bot visits — **0.1%**.
- Google's June 2026 documentation states `llms.txt` has **no effect, positive or negative,** on Search rankings or AI Overviews.
- Large-scale studies find no relationship between having the file and being cited.

The genuine adopters are documentation sites feeding coding assistants (Cursor, Continue, Cline, MCP integrations). That is not a Shopify storefront use case.

**Where the effort belongs instead — `R-C2`:** a real, crawlable, text-based page for every question a buyer asks before purchasing (shipping, returns, sizing, materials, warranty). That is what ChatGPT's `site:`-scoped fanouts actually retrieve. A store with a polished `llms.txt` and no sizing page has optimized the file nobody reads and skipped the one that gets cited.

---

## GEO-H1 — `Organization` schema missing

### WRONG
```liquid
{% comment %} no Organization schema anywhere {% endcomment %}
```

### RIGHT
```liquid
{% comment %} layout/theme.liquid — emit once, on homepage only {% endcomment %}
{% if template == 'index' %}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": {{ shop.name | json }},
  "url": {{ shop.url | json }},
  "logo": "https:{{ settings.logo | image_url: width: 600 }}",
  "description": {{ shop.description | json }},
  "sameAs": [
    {{ settings.social_instagram_link | json }},
    {{ settings.social_tiktok_link | json }},
    {{ settings.social_facebook_link | json }},
    {{ settings.social_youtube_link | json }},
    {{ settings.social_twitter_link | json }}
  ]
}
</script>
{% endif %}
```

---

## GEO-M1 — No shipping / returns schema inside `offers`

### WRONG
Plain `offers` block without shipping or return details.

### RIGHT
```liquid
"offers": {
  "@type": "Offer",
  "url": {{ shop.url | append: product.url | json }},
  "priceCurrency": {{ cart.currency.iso_code | json }},
  "price": {{ product.selected_or_first_available_variant.price | money_without_currency | strip | json }},
  "availability": "https://schema.org/InStock",
  "shippingDetails": {
    "@type": "OfferShippingDetails",
    "shippingRate": {
      "@type": "MonetaryAmount",
      "value": "0",
      "currency": {{ cart.currency.iso_code | json }}
    },
    "shippingDestination": {
      "@type": "DefinedRegion",
      "addressCountry": "{{ localization.country.iso_code }}"
    },
    "deliveryTime": {
      "@type": "ShippingDeliveryTime",
      "handlingTime": { "@type": "QuantitativeValue", "minValue": 0, "maxValue": 1, "unitCode": "DAY" },
      "transitTime":  { "@type": "QuantitativeValue", "minValue": 2, "maxValue": 5, "unitCode": "DAY" }
    }
  },
  "hasMerchantReturnPolicy": {
    "@type": "MerchantReturnPolicy",
    "applicableCountry": "{{ localization.country.iso_code }}",
    "returnPolicyCategory": "https://schema.org/MerchantReturnFiniteReturnWindow",
    "merchantReturnDays": 30,
    "returnMethod": "https://schema.org/ReturnByMail",
    "returnFees": "https://schema.org/FreeReturn"
  }
}
```

---

## Liquid pattern — assign once, reference many (M1)

### WRONG
```liquid
{% if product.metafields.custom.badge.value != blank %}
  <span class="badge">{{ product.metafields.custom.badge.value }}</span>
  <meta itemprop="badge" content="{{ product.metafields.custom.badge.value }}">
{% endif %}
```

### RIGHT
```liquid
{% assign badge = product.metafields.custom.badge.value %}
{% if badge != blank %}
  <span class="badge">{{ badge }}</span>
  <meta itemprop="badge" content="{{ badge }}">
{% endif %}
```

---

## Inline style override (M3)

### WRONG
```liquid
<div style="background-color: #FF6B35; color: #ffffff; padding: 24px;">
```

### RIGHT
```liquid
{% comment %} sections/promo.liquid {% endcomment %}
<div class="promo-block"
     style="--promo-bg: {{ section.settings.bg_color }}; --promo-fg: {{ section.settings.text_color }};">
  …
</div>

{% schema %}
{
  "name": "Promo",
  "settings": [
    { "type": "color", "id": "bg_color", "label": "Background", "default": "#FF6B35" },
    { "type": "color", "id": "text_color", "label": "Text", "default": "#FFFFFF" }
  ]
}
{% endschema %}
```

```css
.promo-block {
  background: var(--promo-bg);
  color: var(--promo-fg);
  padding: 1.5rem;
}
```

---

## L1 — `console.log` left in production JS

### WRONG
```js
// assets/cart.js
console.log('cart updated', cart);
console.warn('variant', variantId);
```

### RIGHT
```js
// assets/cart.js
{% comment %} Strip from theme.js or wrap behind a build-time flag {% endcomment %}
const DEBUG = false;
const log = (...args) => { if (DEBUG) console.log(...args); };

log('cart updated', cart);
```

---

## APP-C5 — Page builder loading globally

Limit: an app **must not reduce storefront Lighthouse performance by more than 10 points** to be listed in the Shopify App Store ([performance docs](https://shopify.dev/docs/apps/build/performance)). Weighted Home 17% / Product 40% / Collection 43%.

### WRONG
```liquid
{% comment %} layout/theme.liquid — builder script on every template {% endcomment %}
<script src="https://cdn.pagefly.io/pagefly-runtime.js"></script>
<link rel="stylesheet" href="https://cdn.pagefly.io/pagefly.css">
```

### RIGHT
```liquid
{% comment %}
  layout/theme.liquid
  Load the builder only on templates it actually renders.
{% endcomment %}
{%- assign builder_templates = 'page.landing,page.promo,page.lookbook' | split: ',' -%}
{%- assign current = template.name -%}
{%- if template.suffix != blank -%}
  {%- assign current = template.name | append: '.' | append: template.suffix -%}
{%- endif -%}

{%- if builder_templates contains current -%}
  <link rel="preconnect" href="https://cdn.pagefly.io" crossorigin>
  <link rel="stylesheet" href="https://cdn.pagefly.io/pagefly.css">
  <script src="https://cdn.pagefly.io/pagefly-runtime.js" defer></script>
{%- endif -%}
```

Product and collection carry 83% of Shopify's weighting. If the builder must stay somewhere, keep it off those two first.

---

## APP-C6 — Accessibility overlay

The FTC [ordered accessiBe to pay $1,000,000](https://www.ftc.gov/news-events/news/press-releases/2025/04/ftc-approves-final-order-requiring-accessibe-pay-1-million) and barred it from claiming its automated product makes sites WCAG-compliant. WebAIM found **67% of screen reader users rate overlays "not effective"** (72% among respondents with a disability).

### WRONG
```liquid
{% comment %} layout/theme.liquid — a widget standing in for remediation {% endcomment %}
<script>
  window.interdeal = { sitekey: "abc123", Position: "right" };
</script>
<script src="https://cdn.userway.org/widget.js" data-account="XXXXXX"></script>
<script src="https://acsbapp.com/apps/app/dist/js/app.js" defer></script>
```

### RIGHT
```liquid
{% comment %}
  layout/theme.liquid — overlay removed. Fix the markup instead.
  These are the structural fixes the overlay was covering for:
{% endcomment %}
<a class="skip-to-content" href="#MainContent">
  {{ 'accessibility.skip_to_text' | t }}
</a>

<main id="MainContent" role="main" tabindex="-1">
  {{ content_for_layout }}
</main>
```

```css
/* assets/theme.css — visible focus, real contrast, honoured motion preference */
.skip-to-content {
  position: absolute;
  left: -9999px;
}
.skip-to-content:focus {
  left: 1rem;
  top: 1rem;
  z-index: 9999;
  padding: 0.75rem 1rem;
  background: #fff;
  color: #111;
}

*:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

Pair this with `C5` (skip link), `H5` (focus trap), `M2` (alt text), and the heading-hierarchy checks. Those are the actual remediation.

---

## APP-H7 — Custom web pixel doing too much

**No Shopify-published payload cap exists for web pixels.** Do not cite one. The applicable published limits are the Built for Shopify performance ceilings: **LCP ≤ 2.5s, CLS ≤ 0.1, INP ≤ 200ms**, and no more than a 10-point Lighthouse reduction.

### WRONG
```javascript
// Custom pixel: subscribes to everything, filters client-side, hand-rolls retries
analytics.subscribe('all_events', (event) => {
  const RETRY = 3;
  let attempt = 0;
  const send = () => {
    fetch('https://example.com/collect', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(event)
    }).catch(() => {
      if (attempt++ < RETRY) setTimeout(send, 1000 * attempt);
    });
  };
  send();
});
```

### RIGHT
```javascript
// Subscribe only to the events the destination consumes, send the fields it needs.
const ENDPOINT = 'https://example.com/collect';

const send = (payload) => {
  navigator.sendBeacon(ENDPOINT, JSON.stringify(payload));
};

analytics.subscribe('product_viewed', (event) => {
  send({
    name: event.name,
    id: event.id,
    productId: event.data.productVariant.product.id,
    price: event.data.productVariant.price.amount
  });
});

analytics.subscribe('checkout_completed', (event) => {
  send({
    name: event.name,
    id: event.id,
    orderId: event.data.checkout.order.id,
    total: event.data.checkout.totalPrice.amount,
    currency: event.data.checkout.totalPrice.currencyCode
  });
});
```

`sendBeacon` survives page unload and needs no retry loop — which is what most oversized custom pixels are actually reimplementing.

---

## Notes on using this file

- When you find an issue, **copy the RIGHT block verbatim** into the report under `Fix:`. Do not summarise.
- If the user's theme uses a different variable name (e.g. `settings.hero_img` instead of `settings.hero_image`), substitute the name but keep the structure.
- Never invent filter names. If a fix needs a filter not shown here, check `liquid-patterns.md` and `deprecated-apis.md` first.
- These snippets are **examples** — confirm each one compiles in the merchant's actual context before committing.
