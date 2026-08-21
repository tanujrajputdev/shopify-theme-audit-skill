# Third-Party App Overhead Checklist

Most Shopify stores die from app bloat, not theme bloat. A well-built theme can still load 1.5MB of JavaScript and hit 8+ seconds on mobile because of how the merchant's apps inject themselves.

This checklist covers detection of common app signatures in compiled HTML/JS, scoring of their loading strategy, and recommended fixes.

Use this checklist whenever:
- The user asks for a "full audit" and you can read theme files or live HTML
- Lighthouse Performance is below 60 and theme files alone don't explain it
- The user mentions a specific app slowing the store

Severity model is the same as `audit-checklist.md`: Critical -10, High -5, Medium -2, Low -1. Apps that are render-blocking and globally loaded score Critical. Apps that are deferred and conditionally loaded score zero or near-zero.

---

## How to detect apps

Without theme access (live HTML only), search the rendered source for these signatures:

| App | Signature to grep | Typical category |
|---|---|---|
| Klaviyo | `klaviyo.com/onsite`, `_learnq`, `klaviyo_subscribe` | Email + popups |
| Judge.me | `judge.me/widget`, `jdgm-widget`, `jdgm.js` | Reviews |
| Loox | `loox.io`, `loox-reviews`, `LooxReviews` | Reviews |
| Yotpo | `staticw2.yotpo.com`, `yotpo-widget`, `yotpoTrackEvent` | Reviews + loyalty |
| Stamped.io | `stamped.io/widget`, `stamped-widget` | Reviews |
| Rebuy | `rebuyengine.com`, `data-rebuy-id`, `Rebuy.SmartCart` | Upsell / cart |
| ReConvert | `reconvert.io`, `data-reconvert` | Post-purchase upsell |
| Gorgias | `client.gorgias.chat`, `gorgias-chat` | Helpdesk + chat |
| Tidio | `code.tidio.co`, `tidioChatApi` | Chat |
| Zendesk | `static.zdassets.com`, `zEACLoader` | Chat |
| Smile.io | `smile.io/v1`, `smile-launcher` | Loyalty |
| LoyaltyLion | `loyaltylion.net`, `loyaltylion-` | Loyalty |
| Recharge | `static.rechargecdn.com`, `Recharge.Checkout` | Subscriptions |
| Bold Subscriptions | `cdn.boldcommerce.com/sub` | Subscriptions |
| Privy | `widget.privy.com`, `_privy` | Popups |
| Justuno | `justuno.com/vck.php` | Popups |
| Nosto | `connect.nosto.com`, `nostojs` | Recommendations |
| LimeSpot | `limespot.com/scripts` | Recommendations |
| Searchanise | `searchanise.com`, `snize_` | Search |
| Boost AI Search | `boost-commerce.com`, `boostPFSFilterApp` | Search + filter |
| Tapcart | `tapcart.com` | Mobile app |
| Hotjar | `static.hotjar.com`, `hjBootstrap` | Analytics |
| Microsoft Clarity | `clarity.ms`, `clarityScript` | Session recording |
| Vimeo / Wistia | `player.vimeo.com`, `fast.wistia.com` | Video |
| PageFly | `pagefly.io`, `__pagefly`, `pf-` class prefix | Page builder |
| Shogun | `shogun.site`, `shogun-landing`, `getshogun.com` | Page builder |
| GemPages | `gempages.net`, `gem-`, `gemPagesConfig` | Page builder |
| Zipify Pages | `zipify.com`, `zpages-` | Page builder |
| LayoutHub | `layouthub.com`, `lh-section` | Page builder |
| Replo | `replo.app`, `data-replo-id` | Page builder |
| accessiBe | `acsbapp.com`, `acsb.js`, `accessiBe` | Accessibility overlay |
| UserWay | `userway.org`, `userwayWidgetApp` | Accessibility overlay |
| AudioEye | `audioeye.com`, `ae-toolbar` | Accessibility overlay |
| EqualWeb | `equalweb.com`, `nagich` | Accessibility overlay |
| Shopify web pixels | `web-pixels-manager`, `sandbox/worker`, `Shopify.analytics` | Platform pixel runtime |

With theme file access, also check:
- `layout/theme.liquid` for `{{ content_for_header }}` placement (apps inject here — must be in `<head>`, not body)
- `sections/` and `snippets/` for hardcoded app blocks (`{% render 'klaviyo-form' %}`, `{% render 'judgeme-widget' %}`)
- `config/settings_data.json` for app block IDs
- Theme app extensions in `extensions/` if running on a Shopify CLI theme

---

## CRITICAL — Each deducts 10 points

### APP-C1: App script loads synchronously in head
**Where to look:** Rendered HTML `<head>`
**What to find:** `<script src="https://...app-domain..."></script>` without `defer` or `async`
**Why it matters:** Every render-blocking app script pauses HTML parsing. A single 80KB Klaviyo script adds 300-600ms to FCP on mobile. Stack 3-4 apps and you've added 2+ seconds.
**Flag if:** Any third-party app script in `<head>` lacks `defer`/`async` and isn't a tag manager / consent gate
**Fix:** Most apps support deferred loading via the merchant's app settings. For apps that inject via Script Tags API (uncontrollable from theme), wrap the integration in a custom script that loads on `requestIdleCallback`:
```html
<script>
  (window.requestIdleCallback || setTimeout)(() => {
    const s = document.createElement('script');
    s.src = 'https://app.example.com/widget.js';
    s.async = true;
    document.head.appendChild(s);
  }, { timeout: 3000 });
</script>
```

---

### APP-C2: Chat widget loads above the fold on mobile
**Where to look:** Initial mobile page weight + LCP filmstrip
**What to find:** Gorgias / Tidio / Zendesk loading immediately on mobile, blocking LCP
**Why it matters:** Chat is the single largest performance offender. A typical chat bundle is 200-400KB and runs immediately. On mobile this single addition can drop Lighthouse Performance from 70 to 30.
**Flag if:** Chat widget script is in the rendered head and not deferred behind user intent (scroll, idle, or button click)
**Fix:** Lazy-mount the chat. Replace the chat snippet's direct `<script>` injection with a deferred loader bound to user intent:
```html
<button id="open-chat" class="chat-launcher" aria-label="Open chat">
  Need help?
</button>
<script>
  const loadChat = () => {
    if (window.__chatLoaded) return;
    window.__chatLoaded = true;
    const s = document.createElement('script');
    s.src = 'https://client.gorgias.chat/...';
    s.async = true;
    document.head.appendChild(s);
  };
  document.getElementById('open-chat').addEventListener('click', loadChat);
  addEventListener('scroll', () => { if (scrollY > 600) loadChat(); }, { once: true });
</script>
```

---

### APP-C3: Popup / email-capture loads on every page including PDP
**Where to look:** Klaviyo / Privy / Justuno script tags
**What to find:** Popup app firing on all pages and all sessions, not gated by page type or scroll depth
**Why it matters:** Popup scripts are large (Klaviyo onsite is ~150KB compressed) and run on first paint. They also degrade INP because the modal logic listens on scroll, mouse-leave, and idle events globally.
**Flag if:** Popup loads on PDP/cart (high-intent pages where popups hurt conversion AND performance)
**Fix:** Configure the app's targeting rules to exclude PDP, cart, and checkout. For Klaviyo specifically, use form targeting → URL exclude rules. Theme-side guard:
```liquid
{% comment %} sections/klaviyo-form.liquid {% endcomment %}
{% unless template contains 'product' or template contains 'cart' %}
  {%- render 'klaviyo-form' -%}
{% endunless %}
```

---

### APP-C4: More than 5 third-party apps loading globally
**Where to look:** Network panel `Initiator: Other` requests on initial pageload
**What to find:** Count of unique third-party origins fetched on initial pageload
**Why it matters:** Each new origin requires DNS + TCP + TLS handshakes (200-400ms on mobile). 5 apps × 300ms = 1.5s of pure connection overhead.
**Flag if:** 6 or more distinct app origins loaded on the homepage or PDP
**Fix:** Audit each app's actual business value. Most stores can eliminate 2-3 apps with minor workflow changes. For apps that must stay, use `<link rel="preconnect">` for the top 3 by weight:
```liquid
<link rel="preconnect" href="https://static.klaviyo.com" crossorigin>
<link rel="preconnect" href="https://staticw2.yotpo.com" crossorigin>
<link rel="preconnect" href="https://client.gorgias.chat" crossorigin>
```

---

### APP-C5: Page builder injects more script weight than Shopify's own App Store limit allows

**Where to look:** Rendered HTML of any page built with the app; network panel filtered to the builder's origin
**What to find:** Total JS + CSS the builder injects, and which templates it loads on
**Detect via:** `pagefly.io`, `shogun.site`, `gempages.net`, `zipify.com`, `layouthub.com`, `replo.app` (see detection table)

**The citable limit — Shopify's own:**
> "To be published in the Shopify App Store, your app must not reduce storefront Lighthouse performance scores by more than 10 points."
> — [About performance optimization](https://shopify.dev/docs/apps/build/performance), restated as [Built for Shopify requirement 2.2.1](https://shopify.dev/docs/apps/launch/built-for-shopify/requirements)

Shopify measures this as a weighted average across three templates — **Home 17%, Product 40%, Collection 43%** — comparing Lighthouse before and after install ([Storefront performance](https://shopify.dev/docs/apps/build/performance/storefront)). A theme also cannot enter the Theme Store below an **average Lighthouse performance score of 60** across those pages ([Theme store requirements](https://shopify.dev/docs/storefronts/themes/store/requirements)).

**Flag if:** The builder's injected JS exceeds **~250KB** on any template, loads on templates that contain no builder-authored content, or independent measurement shows a Lighthouse delta beyond 10 points.

**Why it matters:** Ten points is the published ceiling. Published third-party benchmarks put the major page builders at **260–340KB of injected JavaScript** and mobile Lighthouse deltas of **16–35 points** — two to three and a half times Shopify's own limit. Treat any measured delta over 10 points as Critical regardless of which app it is.

**How to measure it yourself (do this rather than trusting a vendor number):**
1. Duplicate the theme, remove the builder's content from one copy, publish both to a dev store.
2. Run Lighthouse three times per template on each, take the median.
3. Weighted delta = `(home_delta × 0.17) + (product_delta × 0.40) + (collection_delta × 0.43)`.
4. Report the number. A measured delta beats a benchmark blog every time.

**Fix — in priority order:**
1. **Scope the script to the templates that need it.** The most common failure is a builder loading globally to serve three landing pages.
2. **Rebuild high-traffic templates as native Online Store 2.0 sections.** PDP and collection carry 83% of Shopify's weighting; those are the pages worth moving off the builder first.
3. **Keep the builder for genuinely low-traffic marketing pages** where the delta does not touch product or collection.

**Do not flag** a builder used only on `/pages/*` marketing templates whose script does not load on product, collection, or home. That is the correct way to use one.

---

### APP-C6: Accessibility overlay installed

**Where to look:** Rendered HTML `<head>` and end of `<body>`; `layout/theme.liquid`
**Detect via:** `acsbapp.com` / `acsb.js` (accessiBe), `userway.org` (UserWay), `audioeye.com` (AudioEye), `equalweb.com` / `nagich` (EqualWeb)

**The citable record:**
- The **FTC ordered accessiBe to pay $1,000,000** for deceptive claims that its product could make websites WCAG-compliant. The order was announced January 2025 and [approved as final in April 2025](https://www.ftc.gov/news-events/news/press-releases/2025/04/ftc-approves-final-order-requiring-accessibe-pay-1-million). It **bars the company from representing that its automated product can make any website WCAG-compliant, or keep it compliant over time, without evidence** ([case file](https://www.ftc.gov/legal-library/browse/cases-proceedings/2223156-accessibe-inc)).
- The FTC also alleged accessiBe **formatted third-party articles and reviews to appear independent** when they were not.
- **WebAIM's screen reader user survey found 67% of respondents rated accessibility overlays, plugins, and widgets "not effective" — rising to 72% among respondents who have a disability** ([WebAIM](https://webaim.org/projects/screenreadersurvey/)).
- Businesses running overlays **have still been sued in the hundreds.** Courts assess actual accessibility, not installed software.
- Shopify's own bar is structural, not bolt-on: themes need a **minimum average Lighthouse accessibility score of 90** to enter the Theme Store ([requirements](https://shopify.dev/docs/storefronts/themes/store/requirements)). An overlay does not raise that score, because it does not change the markup Lighthouse parses.

**Flag if:** Any overlay script is present. Severity Critical.

**Why it matters — three separate costs:**
1. **Legal.** The overlay is frequently sold as litigation protection. It is not, and the vendor is now legally barred from claiming it is. A merchant who believes they are covered and is not has bought risk, not insurance.
2. **Accessibility.** Overlays run after the DOM is parsed. Assistive tech reads the source. Overlays routinely break keyboard navigation and fight the user's own screen reader settings.
3. **Performance.** It is another render-blocking third-party origin on every pageview.

**Fix:** Remove the overlay and fix the underlying markup. The overlay is almost always masking findings this audit already reports:
- `C5` — skip-to-content link
- `H5` — focus trap on cart drawer / modals
- `M2` / `S-H5` — image alt text
- Heading hierarchy (`S-H1`, `S-H2`)
- Colour contrast and visible focus states in theme CSS

```liquid
{% comment %}
  layout/theme.liquid — REMOVE overlay injections like these:
  <script src="https://acsbapp.com/apps/app/dist/js/app.js"></script>
  <script src="https://cdn.userway.org/widget.js" data-account="..."></script>
{% endcomment %}
```

**How to report this one:** State it factually and without moralising. The merchant was very likely sold the overlay as compliance protection by a vendor the FTC has since fined for exactly that claim. Cite the order, list the underlying issues the audit found, and let the record speak.

**Note:** If the merchant has a contractual or procurement reason to keep the overlay, still fix the underlying markup. The two are not alternatives — one is real remediation and the other is a widget.

---

## HIGH — Each deducts 5 points

### APP-H1: Review widget renders empty container above the fold
**Where to look:** Judge.me / Loox / Yotpo / Stamped review widgets on PDP
**What to find:** Review widget DOM container in viewport on page load with no `min-height` or `aspect-ratio` reserved
**Why it matters:** When the async review JS runs, the widget injects 200-800px of content, pushing the buy box down. This is a CLS event and a conversion event — buy-box shift on mobile is one of the highest-impact CLS sources.
**Fix:**
```liquid
<div class="reviews-container" style="min-height: 180px;">
  {%- render 'judgeme-product-widget' -%}
</div>
```
Better: lazy-mount the widget below the fold via IntersectionObserver.

---

### APP-H2: Multiple review apps installed
**Where to look:** Rendered HTML
**What to find:** Two or more review platforms loading scripts (e.g. Judge.me AND Yotpo, or Loox AND Stamped)
**Why it matters:** Migrations are often half-finished. Old app stays loading while new one is set up. Doubles the review JS payload and creates conflicting widgets.
**Fix:** Identify the merchant's current review source of truth and remove the others entirely from the theme.

---

### APP-H3: Loyalty widget loads on every pageview for non-logged-in users
**Where to look:** Smile.io / LoyaltyLion launcher script
**What to find:** Loyalty launcher script loading globally regardless of customer login state
**Why it matters:** Loyalty widgets are useless to anonymous users but still cost 80-150KB. Most stores have <20% of sessions with a logged-in customer.
**Fix:**
```liquid
{% if customer %}
  {%- render 'smile-launcher' -%}
{% endif %}
```

---

### APP-H4: Recommendations widget makes a synchronous API call on render
**Where to look:** Nosto / LimeSpot / Rebuy recommendation rails
**What to find:** Network waterfall showing the recommendations API request blocking other resources
**Why it matters:** Recommendation API calls (200-800ms) compete with critical rendering resources.
**Fix:** Render recommendations from server-side Shopify data first (`{% render 'product-recommendations' %}` using Shopify's native endpoint), then hydrate with personalized data after `load`.

---

### APP-H5: Search filter app replaces native collection page
**Where to look:** Boost AI Search / Searchanise on collection pages
**What to find:** Native Shopify collection grid hidden and replaced with app-rendered grid
**Why it matters:** Doubles the work — Shopify renders products server-side, then the app blanks them and re-renders client-side. Causes CLS, INP, and a flash of empty grid.
**Fix:** Most search/filter apps offer a "progressive enhancement" mode that augments the native grid instead of replacing it. Enable it in the app settings.

---

### APP-H6: Subscription app hijacks add-to-cart button
**Where to look:** Recharge / Bold Subscriptions integration on PDP
**What to find:** Subscription app injects its own ATC handler, replacing or wrapping the theme's ATC. Adds 200-400ms to first-click responsiveness.
**Fix:** Use the app's native theme integration (not the legacy Script Tag injection). For Recharge, use Recharge's checkout integration block instead of `recharge.js`.

---

### APP-H7: Web pixel sprawl and oversized custom pixel payloads

**Where to look:** Shopify admin → **Settings → Customer events**; rendered HTML for `web-pixels-manager`
**What to find:** How many pixels are registered, whether each is an **app pixel** (strict sandbox) or a **custom pixel** (lax sandbox), and how much code each custom pixel carries

**Important — read before writing a threshold into a report.** Shopify publishes **no maximum payload size for web pixels.** Do not cite one. The 128KB figures in Shopify's documentation belong to two other features and are frequently misattributed to pixels:

| Limit | Actual subject | Source |
|---|---|---|
| **128KB** | JSON **metafield** writes, API 2026-04+ | [changelog](https://shopify.dev/changelog/reduced-metafield-value-sizes) |
| **128KB** | **Shopify Functions** input size (raised from 64KB) | [changelog](https://shopify.dev/changelog/shopify-functions-input-size-limit-increased-to-128kb) |
| **64KB compressed** | **UI extensions** bundle size | [App extensions](https://shopify.dev/docs/apps/build/app-extensions) |
| *(none published)* | **Web pixels** | — |

If you flag pixel weight, say plainly that the threshold is this audit's, not Shopify's. Never present a self-chosen number as a platform limit.

**The citable limits that do apply to pixels** are the performance ceilings every storefront script is measured against — [Built for Shopify requirements](https://shopify.dev/docs/apps/launch/built-for-shopify/requirements) §2.1–2.2: **LCP ≤ 2.5s, CLS ≤ 0.1, INP ≤ 200ms**, and **no more than a 10-point storefront Lighthouse reduction**. A pixel that pushes a store past those is a finding with a real number behind it.

**The documented mechanic that actually matters — sandbox type:**

| Pixel type | Sandbox | Runtime | Main-thread cost |
|---|---|---|---|
| **App pixel** (from an installed app) | **Strict** — web worker | Off the main thread | Low |
| **Custom pixel** (merchant-written, admin UI) | **Lax** | Not worker-isolated | Higher — can contend with rendering |

Source: [About web pixels](https://shopify.dev/docs/apps/build/marketing/pixels) and [Web Pixels API](https://shopify.dev/docs/api/web-pixels-api).

**Flag if:**
- **(High)** More than one pixel sends to the same destination — duplicate GA4, or a GTM container plus a standalone GA4 pixel. Double-counted events corrupt the merchant's own reporting, which is a data-integrity problem before it is a performance one.
- **(High)** A custom pixel carries heavy logic — large lookup objects, retry loops, polling, or an inlined vendor SDK. This is the one worth a size heuristic: **flag custom pixels over ~15KB of code** and state the threshold is ours.
- **(Medium)** A pixel duplicates a script also hardcoded in `theme.liquid`. Migrating to a pixel is correct; leaving both is not.
- **(Medium)** Pixels fire on every event when the destination needs three or four. Subscribe narrowly.

**Fix — subscribe to specific events instead of everything:**
```javascript
// WRONG — custom pixel subscribing to all events, then filtering client-side
analytics.subscribe('all_events', (event) => {
  if (['product_viewed','checkout_completed'].includes(event.name)) {
    fetch('https://example.com/collect', {
      method: 'POST',
      body: JSON.stringify(event)
    });
  }
});

// RIGHT — subscribe only to what the destination consumes
analytics.subscribe('product_viewed', (event) => {
  navigator.sendBeacon('https://example.com/collect', JSON.stringify({
    name: event.name,
    id: event.id,
    productId: event.data.productVariant.product.id
  }));
});

analytics.subscribe('checkout_completed', (event) => {
  navigator.sendBeacon('https://example.com/collect', JSON.stringify({
    name: event.name,
    id: event.id,
    total: event.data.checkout.totalPrice.amount
  }));
});
```
`sendBeacon` does not block unload and does not need a retry loop. Most oversized custom pixels are oversized because they hand-roll delivery that the platform already handles.

**Do not flag** app pixels simply for existing. Strict-sandbox app pixels running in a web worker are the *correct*, Shopify-recommended replacement for hardcoded tracking script in `theme.liquid`. Migrating tracking into a pixel is an improvement — note it as a positive finding.

---

## MEDIUM — Each deducts 2 points

### APP-M1: Hotjar / Clarity session recording on all sessions
**Where to look:** Hotjar or Microsoft Clarity script tags
**What to find:** Session recording firing on 100% of sessions in production
**Why it matters:** Session recording scripts continuously record DOM mutations. Cost: 80-150KB JS, plus persistent CPU work. Useful for debugging, but most stores don't need 100% sample rate forever.
**Fix:** Reduce sample rate in app settings, or gate behind a query param when actively debugging.

---

### APP-M2: Analytics script loads twice (GTM + GA4 standalone)
**Where to look:** Rendered HTML
**What to find:** Both Google Tag Manager AND a standalone gtag.js script tag for GA4
**Why it matters:** GTM already loads GA4. Double-load = duplicate events + wasted bandwidth.
**Fix:** Remove the standalone gtag.js. Configure GA4 through GTM only.

---

### APP-M3: App CSS injected globally for a single section
**Where to look:** App-injected `<link rel="stylesheet">` tags
**What to find:** A 30-80KB app CSS loaded on every page when the widget only appears on one template
**Fix:** If the app injects via Script Tag, request that the app team conditionally inject (most refuse). Otherwise gate the widget block conditionally in the theme.

---

### APP-M4: Video player script loaded but no video on page
**Where to look:** Vimeo / Wistia / YouTube iframe API scripts
**What to find:** Player API script loading on pages that have no video embed
**Fix:** Move the script tag from `theme.liquid` to the section that embeds the video.

---

## LOW — Each deducts 1 point

### APP-L1: App fonts loaded redundantly
**Where to look:** Network panel for `fonts.googleapis.com` or `use.typekit.net`
**What to find:** Same font family loaded by both the theme and an app
**Fix:** Configure the app to use the system or theme font.

---

### APP-L2: Old app remnants left in theme
**Where to look:** `assets/` directory and `theme.liquid`
**What to find:** Snippet references to apps that are no longer installed, or asset files prefixed with `app-` for uninstalled apps
**Fix:** Remove the dead references and asset files.

---

### APP-L3: A/B testing tool with anti-flicker snippet > 2s timeout
**Where to look:** Google Optimize / VWO / Convert anti-flicker code
**What to find:** Hide-body snippet with timeout > 2000ms
**Why it matters:** Anti-flicker hides the page until the test JS loads. A 4s timeout means the user can see a blank page for 4 full seconds if the script is slow.
**Fix:** Drop timeout to 1500ms maximum.

---

## How to present app findings in the report

Add a dedicated section between the technical critical issues and the SEO findings:

```
### Third-Party App Overhead

The store loads [N] third-party app origins on initial pageview. The top offenders by impact:

1. [App name] — [Critical / High / Medium / Low]
   Bundle: [size] | Loading: [sync / async / deferred] | Visibility: [above-fold / below-fold]
   Cost: [estimated ms added to LCP/FCP/INP]
   Recommendation: [specific action]

2. [App name] — ...

Combined estimated savings if app loading is fixed: [X] Lighthouse Perf points, [Y]ms LCP improvement.
```

Always tell the merchant which apps are **earning their weight** and which are not. Apps that drive >5% of revenue are worth optimizing around; apps that don't may be worth uninstalling.

---

## What NOT to flag

- The Shopify Inbox script — it's first-party and ships with Shopify
- Shopify Pay / Shop App scripts — same
- The `_shopify_y` analytics cookie script — required for Shopify reporting
- App scripts inside the checkout (you cannot audit checkout from the theme)
