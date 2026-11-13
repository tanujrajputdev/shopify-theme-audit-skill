---
name: shopify-theme-audit
description: Audit a Shopify theme (Online Store 2.0 or 1.0) for performance, Core Web Vitals, accessibility, conversion, third-party app overhead, SEO, AEO and GEO, and return a scored report with file paths, line numbers and working Liquid fixes. Use when the user asks to audit or review a Shopify theme or its Liquid sections and snippets, find out why a store is slow, check for deprecated Shopify APIs, check structured data, robots.txt or AI-crawler access, or asks whether ChatGPT, Claude, Perplexity or Google AI Overviews can cite the store.
---

# Shopify Theme Audit

The full audit procedure is in `CLAUDE.md` in this skill's folder. Read it first and follow it exactly.

Where things are when this runs as an installed skill:

- Every reference file that `CLAUDE.md` names (`audit-checklist.md`, `seo-aeo-geo-checklist.md`, `apps-audit.md`, `liquid-patterns.md`, `before-after.md`, `deprecated-apis.md`, `scoring.md`) sits in this skill's folder, next to this file. Read them from here, not from the user's project.
- The theme to audit is the user's current working directory, unless they give you another path, a store URL or a Shopify MCP connection.
- Sample reports showing the expected output are in `examples/`.
