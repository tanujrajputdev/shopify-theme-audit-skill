# Shopify Liquid Patterns — Correct vs Incorrect

This file contains before/after code examples for every common Liquid pattern issue found in Shopify theme audits. When you find an incorrect pattern during an audit, use the corresponding correct version as the fix recommendation.

---

## IMAGE-01: Image URL Filter

**INCORRECT — deprecated, will be removed**
```liquid
<img src="{{ product.featured_image | img_url: '800x' }}" alt="{{ product.featured_image.alt }}">
```

**CORRECT — current, CDN-optimized, WebP-automatic**
```liquid
<img
  src="{{ product.featured_image | image_url: width: 800 }}"
  alt="{{ product.featured_image.alt | escape }}"
  width="800"
  height="{{ 800 | divided_by: product.featured_image.aspect_ratio | round }}"
  loading="lazy">
```

**CORRECT — responsive with srcset**
```liquid
<img
  src="{{ product.featured_image | image_url: width: 800 }}"
  srcset="
    {{ product.featured_image | image_url: width: 400 }} 400w,
    {{ product.featured_image | image_url: width: 800 }} 800w,
    {{ product.featured_image | image_url: width: 1200 }} 1200w
  "
  sizes="(min-width: 768px) 50vw, 100vw"
  alt="{{ product.featured_image.alt | escape }}"
  width="800"
  height="{{ 800 | divided_by: product.featured_image.aspect_ratio | round }}"
  loading="lazy">
```

---

## IMAGE-02: Above-the-fold hero image (do NOT lazy load)

**INCORRECT — lazy loading the LCP image delays it**
```liquid
<img src="{{ section.settings.hero_image | image_url: width: 1440 }}" loading="lazy">
```

**CORRECT — no lazy load, add fetchpriority**
```liquid
<img
  src="{{ section.settings.hero_image | image_url: width: 1440 }}"
  srcset="
    {{ section.settings.hero_image | image_url: width: 768 }} 768w,
    {{ section.settings.hero_image | image_url: width: 1440 }} 1440w
  "
  sizes="100vw"
  alt="{{ section.settings.hero_image.alt | escape }}"
  width="1440"
  height="{{ 1440 | divided_by: section.settings.hero_image.aspect_ratio | round }}"
  fetchpriority="high">
```

---

## METAFIELD-01: Accessing metafield values safely

**INCORRECT — no null check, will error if metafield does not exist**
```liquid
<p>Material: {{ product.metafields.custom.material.value }}</p>
```

**INCORRECT — overly complex null check**
```liquid
{% if product.metafields.custom.material != nil and product.metafields.custom.material != blank %}
  <p>Material: {{ product.metafields.custom.material.value }}</p>
{% endif %}
```

**CORRECT — clean nil check, assign once**
```liquid
{% assign material = product.metafields.custom.material %}
{% if material != blank %}
  <p>Material: {{ material.value }}</p>
{% endif %}
```

---

## METAFIELD-02: Accessing JSON metafield values

**INCORRECT — treating JSON metafield as plain string**
```liquid
{{ product.metafields.custom.size_chart.value }}
```

**CORRECT — parse JSON and iterate**
```liquid
{% assign size_chart = product.metafields.custom.size_chart.value %}
{% if size_chart != blank %}
  <table>
    {% for row in size_chart %}
      <tr>
        <td>{{ row.size }}</td>
        <td>{{ row.chest }}</td>
        <td>{{ row.waist }}</td>
      </tr>
    {% endfor %}
  </table>
{% endif %}
```

---

## METAFIELD-03: Accessing list metafields

**INCORRECT — accessing list metafield as single value**
```liquid
{{ product.metafields.custom.features.value }}
```

**CORRECT — list metafields return an array**
```liquid
{% assign features = product.metafields.custom.features.value %}
{% if features != blank %}
  <ul>
    {% for feature in features %}
      <li>{{ feature }}</li>
    {% endfor %}
  </ul>
{% endif %}
```

---

## LOOP-01: Avoiding N+1 queries in collection loops

**INCORRECT — metafield access inside loop causes N+1 queries**
```liquid
{% for product in collection.products %}
  <div class="product-card">
    <p>{{ product.title }}</p>
    {% comment %} This accesses the database for EVERY product in the loop {% endcomment %}
    {% assign badge = product.metafields.custom.badge.value %}
    {% if badge != blank %}
      <span class="badge">{{ badge }}</span>
    {% endif %}
  </div>
{% endfor %}
```

**CORRECT — use paginate to limit loop size, or accept the trade-off consciously**
```liquid
{% comment %}
  If metafields are essential per product card, document this as a known trade-off.
  Keep paginate to 12-24 to limit the query count.
  Never loop more than 50 products with per-product metafield access.
{% endcomment %}
{% paginate collection.products by 12 %}
  {% for product in collection.products %}
    <div class="product-card">
      {% assign badge = product.metafields.custom.badge.value %}
      {% if badge != blank %}
        <span class="badge">{{ badge }}</span>
      {% endif %}
    </div>
  {% endfor %}
{% endpaginate %}
```

---

## SECTION-01: Section schema best practices

**INCORRECT — missing type and name, settings have no defaults**
```json
{
  "settings": [
    {
      "id": "heading",
      "type": "text"
    }
  ]
}
```

**CORRECT — full schema with name, type, label, defaults, and info**
```json
{
  "name": "Featured Collection",
  "class": "section-featured-collection",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Heading",
      "default": "Featured products",
      "info": "Displayed above the product grid"
    },
    {
      "type": "collection",
      "id": "collection",
      "label": "Collection"
    },
    {
      "type": "range",
      "id": "products_to_show",
      "label": "Number of products",
      "min": 2,
      "max": 12,
      "step": 2,
      "default": 4
    }
  ],
  "presets": [
    {
      "name": "Featured Collection"
    }
  ]
}
```

---

## SECTION-02: Preload hint for above-fold section images

**INCORRECT — no preload hint in theme.liquid for hero section image**
```liquid
{% comment %} In layout/theme.liquid, no preload hint exists {% endcomment %}
```

**CORRECT — add preload in theme.liquid head for sections that show above the fold**
```liquid
{% comment %} In layout/theme.liquid, inside <head> {% endcomment %}
{% if template == 'index' %}
  {% for section in template.sections %}
    {% if section.type == 'slideshow' or section.type == 'hero' %}
      {% if section.settings.image != blank %}
        <link
          rel="preload"
          as="image"
          href="{{ section.settings.image | image_url: width: 1440 }}"
          imagesrcset="{{ section.settings.image | image_url: width: 768 }} 768w, {{ section.settings.image | image_url: width: 1440 }} 1440w">
      {% endif %}
    {% endif %}
  {% endfor %}
{% endif %}
```

---

## SCRIPT-01: JavaScript loading patterns

**INCORRECT — synchronous script blocks page parsing**
```html
<script src="{{ 'theme.js' | asset_url }}"></script>
```

**CORRECT — deferred, non-render-blocking**
```html
<script src="{{ 'theme.js' | asset_url }}" defer></script>
```

**CORRECT — async for analytics that do not need DOM ready**
```html
<script src="{{ 'analytics.js' | asset_url }}" async></script>
```

**CORRECT — modern ES module (automatically deferred)**
```html
<script src="{{ 'theme.js' | asset_url }}" type="module"></script>
```

---

## SCRIPT-02: Passing Liquid variables to JavaScript

**INCORRECT — unsafe, breaks if value contains quotes**
```liquid
<script>
  var productId = {{ product.id }};
  var productTitle = "{{ product.title }}";  ← breaks if title has quotes
</script>
```

**CORRECT — use data attributes or JSON serialization**
```liquid
<div
  data-product-id="{{ product.id }}"
  data-product-title="{{ product.title | json }}"
  id="product-{{ product.id }}">
</div>

<script>
  const el = document.getElementById('product-{{ product.id }}');
  const productId = el.dataset.productId;
  const productTitle = JSON.parse(el.dataset.productTitle);
</script>
```

---

## ACCESSIBILITY-01: Skip to content link

**CORRECT — must be the first focusable element in body**
```liquid
{% comment %} In layout/theme.liquid, first element inside <body> {% endcomment %}
<a class="skip-to-content-link button visually-hidden" href="#MainContent">
  {{ 'accessibility.skip_to_text' | t }}
</a>
```

**CSS for visually hidden but focusable:**
```css
.visually-hidden:not(:focus):not(:active) {
  clip: rect(0 0 0 0);
  clip-path: inset(50%);
  height: 1px;
  overflow: hidden;
  position: absolute;
  white-space: nowrap;
  width: 1px;
}
```

---

## ACCESSIBILITY-02: Icon buttons without visible text

**INCORRECT — icon button with no accessible label**
```liquid
<button class="cart-icon-button">
  {% render 'icon-cart' %}
</button>
```

**CORRECT — icon button with aria-label**
```liquid
<button
  class="cart-icon-button"
  aria-label="{{ 'sections.header.cart' | t }}"
  aria-expanded="false"
  aria-controls="cart-drawer">
  {% render 'icon-cart' %}
  <span class="visually-hidden">{{ 'sections.header.cart' | t }}</span>
</button>
```

---

## SEO-01: Product structured data

**CORRECT — JSON-LD Product schema on product pages**
```liquid
{% comment %} In sections/main-product.liquid or templates/product.liquid {% endcomment %}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": {{ product.title | json }},
  "description": {{ product.description | strip_html | truncate: 500 | json }},
  "url": {{ canonical_url | json }},
  "image": {{ product.featured_image | image_url: width: 800 | prepend: 'https:' | json }},
  "brand": {
    "@type": "Brand",
    "name": {{ shop.name | json }}
  },
  "offers": {
    "@type": "Offer",
    "price": {{ product.price | money_without_currency | json }},
    "priceCurrency": {{ shop.currency | json }},
    "availability": {% if product.available %}"https://schema.org/InStock"{% else %}"https://schema.org/OutOfStock"{% endif %},
    "url": {{ canonical_url | json }}
  }
  {% if product.metafields.reviews.rating.value != blank %}
  ,"aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": {{ product.metafields.reviews.rating.value }},
    "reviewCount": {{ product.metafields.reviews.rating_count.value }}
  }
  {% endif %}
}
</script>
```

---

## MONEY-01: Correct money formatting

**INCORRECT — deprecated filter**
```liquid
{{ product.price | money_format }}
```

**CORRECT**
```liquid
{{ product.price | money }}
{% comment %} Or with currency code {% endcomment %}
{{ product.price | money_with_currency }}
```

---

## COMPARE-01: Price comparison display

**INCORRECT — shows compare price even when not on sale**
```liquid
<span class="price">{{ product.price | money }}</span>
<span class="compare-price">{{ product.compare_at_price | money }}</span>
```

**CORRECT — only show compare price when it is actually higher**
```liquid
<span class="price">{{ product.selected_or_first_available_variant.price | money }}</span>
{% assign variant = product.selected_or_first_available_variant %}
{% if variant.compare_at_price > variant.price %}
  <span class="compare-price">{{ variant.compare_at_price | money }}</span>
  <span class="visually-hidden">{{ 'products.product.price.sale_price' | t }}</span>
{% endif %}
```
