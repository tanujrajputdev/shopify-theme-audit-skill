# Shopify Deprecated APIs and Current Replacements

Flag every deprecated API during audits. Always show the exact replacement, not just the filter name.

---

## Image Filters

| Deprecated | Current | Status |
|---|---|---|
| `img_url` | `image_url` | Deprecated Sept 2023 — will be removed |
| `img_tag` | Use `<img>` with `image_url` manually | Deprecated |
| `product_img_url` | `image_url` | Deprecated |

---

## Money Filters

| Deprecated | Current |
|---|---|
| `money_format` | `money` |
| Use `shop.money_format` in JS | Use `Shopify.money_format` — still valid |

---

## URL Filters

| Deprecated | Current |
|---|---|
| `product_url` (shorthand) | `product.url` property |
| Hardcoded `/products/{{ product.handle }}` | `{{ product.url }}` |

---

## AJAX API (JavaScript)

| Deprecated | Current |
|---|---|
| `jQuery.getJSON('/cart.js')` | `fetch('/cart.js').then(r => r.json())` |
| `$.ajax` for cart operations | `fetch` with appropriate headers |
| `Shopify.formatMoney()` from old CDN | Include `shopify_currency.js` from theme or use Intl.NumberFormat |

---

## Theme Architecture

| Old (1.0) | Current (2.0) |
|---|---|
| `{% section 'header' %}` in templates | `{% sections 'header-group' %}` or JSON templates |
| `.liquid` templates only | JSON templates (e.g. `product.json`) with section references |
| Global sections in `layout/theme.liquid` | Section groups in JSON templates |
| `settings_data.json` editing for customization | Theme editor via sections and blocks |

**How to detect theme version:**
- Online Store 2.0: `templates/product.json` exists (not just `product.liquid`)
- 1.0: Only `.liquid` files in `templates/`, no JSON templates

---

## Liquid Tags and Objects

| Deprecated | Current |
|---|---|
| `{% include 'snippet-name' %}` | `{% render 'snippet-name' %}` |
| `linklists` object | `link_lists` object (both still work, `linklists` is the old form) |
| `all_products['handle'].price` | Still works but expensive — avoid in loops |

**Important note on `include` vs `render`:**
`{% include %}` gives the snippet full access to the parent template's variables. `{% render %}` creates an isolated scope — variables must be explicitly passed. `{% render %}` is the correct current pattern and is required for section blocks.

```liquid
{% comment %} INCORRECT (old pattern) {% endcomment %}
{% include 'product-card' %}

{% comment %} CORRECT (current pattern) {% endcomment %}
{% render 'product-card', product: product, show_vendor: section.settings.show_vendor %}
```

---

## Cart API

| Deprecated | Current |
|---|---|
| `CartJS` library | Native Shopify Fetch API + `sections` parameter |
| Re-rendering cart via full page reload | Section rendering API: `fetch('/cart/add.js', { body: formData })` + `sections: ['cart-items']` |

**Correct cart add with section rendering:**
```javascript
fetch('/cart/add.js', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    id: variantId,
    quantity: 1,
    sections: ['cart-drawer', 'cart-icon-bubble'],
  }),
})
.then(response => response.json())
.then(data => {
  // data.sections contains pre-rendered HTML for each section
  document.getElementById('cart-drawer').innerHTML = data.sections['cart-drawer'];
});
```

---

## Predictive Search

| Deprecated | Current |
|---|---|
| Custom search implementation via `/search?type=product&q=` | Shopify Predictive Search API: `GET /search/suggest.json?q=query&resources[type]=product` |

---

## Theme Check (run before submitting to Shopify Theme Store)

If the theme is being submitted to the Shopify Theme Store or given to a client who will submit it, flag any patterns that fail Shopify Theme Check:

1. `{% include %}` instead of `{% render %}` — fails Theme Check
2. Direct access to `all_products` in loops — flagged as performance issue
3. JavaScript in Liquid files (should be in `assets/`) — flagged
4. Hardcoded store-specific content — flagged
5. Missing required sections: `header`, `footer`, `password`, `gift-card`, `404`
