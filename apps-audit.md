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
