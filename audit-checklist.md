# Shopify Theme Audit Checklist

Check every item in this list when auditing a theme. Each item includes what to look for, where to look, and why it matters.

This file covers performance, accessibility, conversion, and core Liquid quality issues. For search-related checks (classic SEO, Answer Engine Optimization for ChatGPT/Claude/Perplexity, Generative Engine Optimization for AI Overviews), see `seo-aeo-geo-checklist.md` — it is part of the same audit and uses the same severity model.

---

## CRITICAL SEVERITY — Each deducts 10 points

### C1: Render-blocking scripts in `<head>`
**Where to look:** `layout/theme.liquid`
**What to find:** `<script src="...">` tags without `defer` or `async` inside `<head>` before `</head>`
**Why it matters:** Every synchronous script in `<head>` pauses HTML parsing. A 50KB script adds 200–400ms to Time to First Byte on mobile.
**Flag if:** Any `<script src>` tag exists in head without `defer` or `async`, except for inline scripts
**Do not flag:** Inline `<script>` blocks (they are evaluated differently), scripts with `type="application/ld+json"`

---

### C2: Deprecated `img_url` filter in use
**Where to look:** All Liquid files, especially `sections/main-product.liquid`, `snippets/product-card.liquid`
**What to find:** `| img_url:` anywhere in Liquid templates
**Why it matters:** Shopify deprecated `img_url` in September 2023. It will be removed. It also does not support modern CDN optimizations that `image_url` provides.
**Flag if:** `img_url` appears in any Liquid file
**Correct replacement:** See `liquid-patterns.md` section IMAGE-01

---

### C3: Images without lazy loading
**Where to look:** All section and snippet files containing `<img>` tags
**What to find:** `<img>` tags without `loading="lazy"` attribute
**Why it matters:** Product listing pages with 24 products load all images immediately. This adds 2–5 seconds to page load on mobile.
**Flag if:** Any `<img>` tag below the fold does not have `loading="lazy"`
**Do not flag:** The first image in the page (hero image, first product image) — lazy loading the LCP image delays it

---

### C4: N+1 Liquid database queries inside loops
**Where to look:** `sections/main-collection-product-grid.liquid`, any section with `{% for product in collection.products %}`
**What to find:** Accessing metafields, variant details, or media inside a for loop that iterates more than 10 items
**Why it matters:** Each metafield access inside a loop is a separate database query. 50 products × 1 metafield = 50 queries. This multiplies page generation time.
**What qualifies:**
```liquid
{% for product in collection.products %}
  {{ product.metafields.custom.material.value }}  ← THIS IS AN N+1 QUERY
{% endfor %}
```
**Do not flag:** Accessing `.price`, `.title`, `.handle`, `.url`, `.featured_image` — these are cached per product

---

### C5: No skip-to-content link (accessibility)
**Where to look:** `layout/theme.liquid`, first element inside `<body>`
**What to find:** Presence of `<a href="#MainContent"` or similar skip link as the first focusable element
**Why it matters:** Required for WCAG 2.1 AA compliance. Keyboard users and screen reader users cannot navigate to content without tabbing through the entire navigation. ADA lawsuits are increasing against ecommerce stores.
**Flag if:** No skip link exists as the first or second element after `<body>`

---

### C6: Missing image dimensions (CLS)
**Where to look:** All `<img>` tags in sections and snippets
**What to find:** `<img>` tags without explicit `width` and `height` attributes
**Why it matters:** Images without dimensions cause Cumulative Layout Shift — the page visibly jumps as images load. Google penalizes CLS in Core Web Vitals.
**Flag if:** Product images, collection images, or banner images lack `width` and `height`
**Correct approach:** Always set width and height even for responsive images. The browser uses the ratio to reserve space.

---

## HIGH SEVERITY — Each deducts 5 points

### H1: JavaScript files without defer or async
**Where to look:** `layout/theme.liquid` and any section files with `<script src>`
**What to find:** `<script src="...">` without `defer` or `async`
**Flag if:** Any external script file loads synchronously
**Exceptions:** Critical above-fold functionality that genuinely cannot be deferred

---

### H2: No WebP format for product images
**Where to look:** `image_url` filter usage throughout Liquid files
**What to find:** Whether the theme uses format-agnostic image delivery or forces JPEG/PNG
**How to check:** Look for `| image_url:` filter. The `image_url` filter automatically serves WebP to supporting browsers.
**Flag if:** Images are served via hardcoded `.jpg` URLs or the old `img_url` filter which does not auto-convert

---

### H3: No preload hint for LCP image
**Where to look:** `layout/theme.liquid` `<head>` section and `sections/slideshow.liquid` or hero section
**What to find:** Whether the above-fold hero image has a `<link rel="preload" as="image">` hint
**Why it matters:** The hero image is usually the Largest Contentful Paint element. Preloading it improves LCP by 300–800ms.
**Flag if:** No preload link exists for the primary hero/banner image

---

### H4: Global CSS loading unused styles on all pages
**Where to look:** `layout/theme.liquid` and `assets/` directory
**What to find:** A single large CSS file (over 100KB) loaded on every page
**Flag if:** A CSS file over 80KB loads globally with no page-specific splitting
**Why it matters:** A 200KB CSS file adds 150–300ms on mobile 4G connections

---

### H5: Cart drawer or modal without focus trap
**Where to look:** `sections/cart-drawer.liquid`, `assets/cart-drawer.js`
**What to find:** When the cart drawer opens, does keyboard focus get trapped inside it?
**Flag if:** No focus trap is implemented — keyboard users can tab behind the drawer to hidden page elements

---

### H6: Missing canonical tag on paginated collection pages
**Where to look:** `layout/theme.liquid` or `templates/collection.liquid`
**What to find:** `<link rel="canonical">` that handles `?page=2` pagination correctly
**Flag if:** Canonical points to the paginated URL instead of the base collection URL, or is missing entirely

---

### H7: Theme app extension blocks not using lazy loading
**Where to look:** Any block schema definitions in sections
**What to find:** App blocks that load heavy JavaScript for features not visible above the fold
**Flag if:** An app block loads more than 30KB of JavaScript that is not deferred

---

## MEDIUM SEVERITY — Each deducts 2 points

### M1: Liquid variables assigned multiple times in the same template
**Where to look:** Complex section files
**What to find:** The same expression computed more than once instead of assigned to a variable once and reused
```liquid
{% comment %} WRONG — computes the same thing twice {% endcomment %}
{% if product.metafields.custom.badge.value != blank %}
  <span>{{ product.metafields.custom.badge.value }}</span>
{% endif %}

{% comment %} RIGHT — assign once, reference twice {% endcomment %}
{% assign badge = product.metafields.custom.badge.value %}
{% if badge != blank %}
  <span>{{ badge }}</span>
{% endif %}
```

---

### M2: Missing alt text in Liquid image tags
**Where to look:** All `<img>` tags and `image_url` filter usages
**What to find:** `<img>` tags with empty or missing `alt` attributes
**Flag if:** `alt=""` is used for informational images, or `alt` is missing entirely
**Do not flag:** `alt=""` on purely decorative images (correct for decorative)

---

### M3: Inline styles overriding theme customizer settings
**Where to look:** Section Liquid files
**What to find:** Hardcoded colors, fonts, or spacing in `style=""` attributes that prevent merchants from customizing via the theme editor
**Flag if:** Color values like `style="color: #000000"` exist in places that should be theme-customizer-controlled

---

### M4: No structured data for products
**Where to look:** `templates/product.liquid` or `sections/main-product.liquid`
**What to find:** JSON-LD `Product` schema with name, image, description, offers, aggregateRating
**Flag if:** No `<script type="application/ld+json">` exists on product pages with Product schema

---

### M5: Breadcrumb navigation missing or not schema-marked
**Where to look:** Product and collection page templates
**What to find:** Breadcrumb nav with corresponding JSON-LD `BreadcrumbList` schema
**Why it matters:** Breadcrumbs appear in Google search results as rich snippets. They also improve user navigation and are an accessibility best practice.

---

### M6: Search page returns no results without suggestions
**Where to look:** `templates/search.liquid`
**What to find:** What happens when search returns 0 results
**Flag if:** The zero-results state shows only "No results found" with no recommended products or search suggestions

---

### M7: Password page does not match brand
**Where to look:** `templates/password.liquid`
**Flag if:** The password page uses default Shopify branding with no logo, brand colors, or launch message

---

## LOW SEVERITY — Each deducts 1 point

### L1: Console errors in JavaScript
**What to look for:** `console.log`, `console.warn` statements left in production JavaScript files in `assets/`

### L2: Commented-out code blocks in Liquid files
**What to look for:** Large blocks of commented-out Liquid or HTML that add to file size

### L3: CSS and JS files not minified
**What to look for:** `assets/*.css` and `assets/*.js` files that are not minified (easy to read, lots of whitespace)

### L4: Theme does not use CSS custom properties for brand colors
**What to look for:** Hardcoded hex values repeated throughout CSS files instead of CSS custom properties like `var(--color-primary)`

### L5: Missing 404 page customization
**What to look for:** `templates/404.liquid` — whether it has a helpful message, navigation links, and search

### L6: Footer missing basic trust elements
**What to look for:** Whether footer includes payment icons, trust badges, or social proof elements

---

## POSITIVE FINDINGS — Add these to the "What This Theme Does Well" section

Note when you find any of the following already implemented correctly:
- Hero image with preload hint
- All images using `image_url` with correct dimensions
- Skip-to-content link present
- Product structured data with all required fields
- Cart drawer with focus trap
- JavaScript using defer/async throughout
- CSS custom properties for all brand colors
- Zero render-blocking resources in head
- All collection images with lazy loading
