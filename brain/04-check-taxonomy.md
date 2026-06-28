# Check Taxonomy

All check IDs, their severity, what they detect, and which file defines them.

## Technical & CRO Checks (`audit-checklist.md`)

### Critical — −10 points each

| ID | What it checks |
|---|---|
| C1 | Render-blocking scripts in `<head>` without defer/async |
| C2 | Deprecated `img_url` filter still in use (should be `image_url`) |
| C3 | Images missing `loading="lazy"` (below-fold images) |
| C4 | N+1 Liquid queries in loops iterating 5+ items |
| C5 | No skip-to-content link as first focusable element |
| C6 | Images missing explicit `width` and `height` attributes (causes CLS) |

### High — −5 points each

| ID | What it checks |
|---|---|
| H1 | JavaScript files loaded without `defer` attribute |
| H2 | No WebP image format in use (JPEG/PNG only) |
| H3 | No `<link rel="preload">` hint for LCP hero image |
| H4 | Global CSS file (e.g. `base.css`) over 50KB loaded on every page |
| H5 | Modal/drawer has no `trapFocus()` keyboard trap |
| H6 | Canonical tag issues (`?variant=` params not canonicalized, pagination not handled) |
| H7 | Heavy third-party app blocks loaded above the fold |

### Medium — −2 points each

| ID | What it checks |
|---|---|
| M1 | Same Liquid expression repeated 3+ times (should be assigned to variable) |
| M2 | `<img>` tags missing `alt` attribute |
| M3 | Inline styles used for layout (should be CSS classes) |
| M4 | No product structured data (`{{ product \| structured_data }}` missing) |
| M5 | No `BreadcrumbList` schema on product/collection/article pages |
| M6 | Zero-results search state shows blank page or generic message |
| M7 | Password page uses Shopify default branding (no store customization) |

### Low — −1 point each

| ID | What it checks |
|---|---|
| L1 | `console.log()` statements left in production JavaScript |
| L2 | Commented-out dead code blocks |
| L3 | Unminified JS or CSS assets in `assets/` folder |
| L4 | No CSS custom properties (hardcoded hex values throughout) |
| L5 | 404 page is the bare Shopify default with no helpful navigation |
| L6 | Footer missing trust elements (payment icons, security badge, return policy link) |

---

## SEO / AEO / GEO Checks (`seo-aeo-geo-checklist.md`)

### SEO Critical — −10 points each

| ID | What it checks |
|---|---|
| S-C1 | `<title>` tag missing or empty on any page type |
| S-C2 | Meta description missing on collection and product pages |
| S-C3 | Canonical tag missing or pointing to wrong URL |
| S-C4 | Sitemap not accessible at `/sitemap.xml` |
| S-C5 | Product JSON-LD structured data missing on product pages |

### SEO High — −5 points each

| ID | What it checks |
|---|---|
| S-H1 | Multiple `<h1>` elements on a page (header logo wrapped in h1 + content h1) |
| S-H2 | Heading hierarchy broken (h3 following h1, skipping h2) |
| S-H3 | Open Graph tags missing (`og:title`, `og:description`, `og:image`) |
| S-H4 | Twitter Card tags missing |
| S-H5 | Image alt text not descriptive (generic "image" or empty) |
| S-H6 | `hreflang` missing or incorrect for multi-market/multi-locale stores |
| S-H7 | Paginated collection pages not handled (no canonical for page 2+) |

### SEO Medium — −2 points each

| ID | What it checks |
|---|---|
| S-M1 | Title tag too long (over 60 chars) or too short (under 30 chars) |
| S-M2 | Meta description too long (over 160 chars) or too short (under 70 chars) |
| S-M3 | URL handles contain stop words or are not descriptive |
| S-M4 | Product pages have no internal links to related products or collections |
| S-M5 | Non-indexable pages (thank-you, account) missing `<meta name="robots" content="noindex">` |
| S-M6 | Logo not wrapped in semantic markup (`<p class="logo">` instead of site name text alternative) |

### AEO Critical — −10 points each

| ID | What it checks |
|---|---|
| AEO-C1 | FAQ accordion/section present in the DOM but no `FAQPage` JSON-LD schema |
| AEO-C2 | Product page description opens with marketing language instead of a factual product definition |

### AEO High — −5 points each

| ID | What it checks |
|---|---|
| AEO-H1 | Product specifications listed as prose paragraph, not `PropertyValue` structured data |
| AEO-H2 | How-to or instructional content present but no `HowTo` schema |
| AEO-H3 | No `Speakable` schema hints on key content (product name, description, price) |
| AEO-H4 | No author or publisher E-E-A-T signals (no `Person` or `Organization` schema) |

### AEO Medium — −2 points each

| ID | What it checks |
|---|---|
| AEO-M1 | Article or blog post pages missing `dateModified` in schema |
| AEO-M2 | Product reviews present in DOM but no individual `Review` schema items |
| AEO-M3 | No comparison content ("vs" pages, feature comparison tables) anywhere in the store |

### GEO Critical — −10 points each

| ID | What it checks |
|---|---|
| GEO-C1 | `templates/robots.txt.liquid` blocks `GPTBot`, `ClaudeBot`, or `PerplexityBot` |
| GEO-C2 | No `llms.txt` file at root providing a plain-language store overview for LLMs |

### GEO High — −5 points each

| ID | What it checks |
|---|---|
| GEO-H1 | No `Organization` schema with `sameAs` social profile links |
| GEO-H2 | No `BreadcrumbList` schema on product and collection pages |
| GEO-H3 | Key product content (price, specs, availability) rendered via JavaScript only (not in HTML source) |
| GEO-H4 | Brand description is thin (under 50 words) or contains no factual differentiators |

### GEO Medium — −2 points each

| ID | What it checks |
|---|---|
| GEO-M1 | `Offer` schema missing `shippingDetails` and `hasMerchantReturnPolicy` |
| GEO-M2 | Product images use generic filenames (`image1.jpg`) instead of descriptive names |
| GEO-M3 | No machine-readable material or country-of-origin data in product schema |

---

## Score modifiers (applied after base calculation)

| Condition | Modifier |
|---|---|
| Zero Critical issues across both checklists | +5 bonus |
| More than 4 Critical issues across both checklists | −5 additional penalty |
| Theme from the official Shopify Theme Store | Note in report — issues are likely merchant customizations, not original theme |

## Grade thresholds

| Score | Grade | Meaning |
|---|---|---|
| 90–100 | A | Production-ready. Minor polish only. |
| 75–89 | B | Good theme. A few meaningful improvements. |
| 60–74 | C | Average. Several issues affecting performance or conversions. |
| 40–59 | D | Below standard. Multiple high-impact issues needing immediate attention. |
| 0–39 | F | Significant problems. Performance or accessibility issues harming the store. |
