# Personas and Use Cases

## Who uses this skill

### 1. Shopify developer (primary user)
Works on client themes. Uses the audit before handing a project to a client ("pre-launch review"), after migrating a theme to OS 2.0, or as a QA step before peak-traffic events like Black Friday.

Pain: manual audits take hours and miss items under time pressure.
Gain: consistent, sourced audit in under five minutes that catches everything.

### 2. Agency owner / account manager
Sells audits as a service or uses them as a discovery tool at the start of a new client engagement. Needs a professional-looking, credible deliverable — not a rough checklist.

Pain: writing an audit report from scratch takes a full day.
Gain: a structured, scored, cite-able report in minutes that looks like an agency deliverable.

### 3. Shopify merchant (direct)
Runs their own store. May have a developer who built the theme but doesn't know what to ask them to fix.

Pain: doesn't know what's wrong or how to prioritize developer time.
Gain: a ranked fix list with effort estimates so they can have an informed conversation with their developer.

### 4. SEO / AEO / GEO specialist
Not a theme developer, but responsible for the store's search visibility. Specifically needs the SEO/AEO/GEO portion of the audit.

Pain: doesn't have Liquid knowledge to read theme files.
Gain: Claude reads the files and surfaces the schema issues, crawler blocks, and structured data gaps.

---

## Trigger prompts (what users actually type)

**Full audit:**
```
Audit this Shopify theme using the audit skill. Cover performance,
accessibility, conversion, SEO, AEO, and GEO.
```

**SEO/AEO/GEO only (split-score mode):**
```
Run an SEO, AEO, and GEO audit on this theme using the audit skill.
Use split-score mode. Skip the performance-only sections.
```

**Quick wins (quick-wins mode):**
```
Run the audit skill in quick-wins mode. Show me the top 10 fixes ranked by
impact divided by effort, then the full report.
```

**App overhead only:**
```
Audit only third-party app overhead using the audit skill. Use apps-audit.md.
List every app detected, its loading strategy, and its measurable cost.
```

**Focused single area:**
```
Audit only the product page for AEO — focus on Product schema, FAQ schema,
factual summary opening, and specification structure.
```

**Large theme (two-pass):**
```
Pass 1: Audit layout/ and sections/ using the audit skill, full coverage.
Pass 2: Continue — check snippets/ and templates/. Add to the previous report.
```

**Live store via MCP:**
```
Connect to my Shopify store via the MCP. Pull shop info and a sample product
page. Run the full audit skill against what you see. Note which checks
required theme file access and were skipped.
```

---

## Use-case patterns

| Use case | Mode | Notes |
|---|---|---|
| Pre-launch review | Full audit | Run before going live to catch regressions |
| Agency client deliverable | Full audit or split-score | Report looks professional; include score and grade in client email |
| Migration QA (OS 1.0 → 2.0) | Full audit | Focus on deprecated API checks (C2, deprecated-apis.md) |
| Pre-Black Friday hardening | Quick-wins mode | Time-boxed; merchant wants the 5-minute fixes, not multi-day rewrites |
| AI-search readiness | Split-score, SEO/AEO/GEO | Confirm store can be cited by ChatGPT, Claude, Perplexity, Google AI Overviews |
| Structured data sweep | Split-score, SEO/AEO/GEO | Verify Product, FAQ, BreadcrumbList, Organization, Article, HowTo schemas |
| Robots.txt review | Focused: GEO-C1 | Confirm AI crawler intent is reflected in `templates/robots.txt.liquid` |
| Section-by-section review | Focused | Point Claude at one section file for a targeted audit |

---

## What the output format signals to each persona

- **Score + grade at the top** → immediately answers "how bad is it?" for the merchant and agency owner
- **File name + line number on every finding** → gives the developer exactly where to go; no guessing
- **Working code fix per issue** → developer can implement without additional research
- **What This Theme Does Well section** → gives the agency owner credibility points; the report is balanced, not just a complaint list
- **Estimated Impact section** → gives the merchant and agency a "before/after" to justify the work to stakeholders
- **Recommended Fix Order** → gives the time-constrained developer a ranked to-do list instead of an undifferentiated pile
