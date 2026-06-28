# Decisions and Constraints

## Core design decisions

### Decision: Markdown files only, no code
**Why:** Claude Code reads CLAUDE.md and instruction files natively. No build step, no runtime, no dependencies. The user copies files and it works. Maximum portability and zero friction.

**Constraint:** All "logic" must be expressed as instructions to Claude, not as executable code. Edge cases must be handled via rules in CLAUDE.md, not conditional branches in a script.

---

### Decision: Check IDs are stable and cross-referenced
**Why:** `before-after.md` is keyed by check ID (e.g. `C2`, `GEO-C1`). If a check ID changes, the fix lookup breaks. Check IDs must never be renumbered or reused.

**Constraint:** Adding a new check in the Critical tier does not renumber existing checks. New Critical checks get IDs like `C7`, `C8`, etc.

---

### Decision: Every finding must include file name, line number, and working code fix
**Why:** "Something is wrong with your images" is useless. A developer needs to know exactly where to go and exactly what to type. This is the core value proposition over generic audit tools.

**Constraint:** Checks that cannot produce a specific location (e.g. checking a live URL without source files) must be noted as "live-page analysis" in the report and explicitly flagged as less precise.

---

### Decision: Always include a "What This Theme Does Well" section
**Why:** A report that is only problems is not credible and creates adversarial dynamics with the client. Identifying genuine positives shows the auditor read the actual code. It also gives agency owners ammunition to deliver the report without it feeling punitive.

**Constraint:** Minimum 3 positive findings per report. Claude must find real positives, not invent them. If fewer than 3 genuine positives exist, that fact itself is noted.

---

### Decision: AEO and GEO checks are separate from classic SEO
**Why:** Classic SEO (title tags, canonical, sitemap) is well understood and covered by existing tools. AEO (ChatGPT/Claude/Perplexity citations) and GEO (Google AI Overviews, LLM shopping) are newer and not covered by any Shopify-specific tool. This is where the skill adds unique value.

**Constraint:** AEO and GEO checks must not overlap with classic SEO checks. If a check applies to multiple categories, assign it to the most specific one and note the overlap.

---

### Decision: Quick-wins mode reorders, does not filter
**Why:** A merchant in quick-wins mode still needs to know about every issue, including the ones that take a week to fix. Hiding items would be dishonest. The mode changes the presentation order, not the completeness.

**Constraint:** In quick-wins mode, the full Critical → High → Medium → Low sections must appear after the Quick Wins table. Nothing is omitted.

---

### Decision: N+1 loops under 5 items are not flagged
**Why:** Flagging a loop over 3 nav items as a performance issue would be noise. The real N+1 problem is loops over product collections, search results, or metafield queries with 20+ items.

**Constraint:** Threshold is 5 items. Loops iterating a known-small set (nav items, 2-3 featured products hardcoded) are explicitly exempt.

---

### Decision: Design recommendations are out of scope
**Why:** This is a technical audit. Recommending font changes, color palettes, or layout redesigns is subjective and outside the audit's mandate. Including them would dilute the technical findings and create scope creep.

**Constraint:** Any finding that would require a design decision (not a code fix) is excluded from the report.

---

## Known limitations

- **Apps-audit.md relies on script fingerprints.** If an app changes its script URL pattern, the detection logic breaks until `apps-audit.md` is updated.
- **AEO/GEO outcomes are probabilistic.** The skill can verify that the technical signals are correct, but cannot guarantee that ChatGPT or Google AI Overviews will actually cite the store.
- **Large themes may hit Claude's context window.** The two-pass workaround (audit layout+sections in pass 1, snippets+templates in pass 2) is documented in the README and CLAUDE.md.
- **MCP-only audits skip source-level checks.** Without local theme files, checks that require reading Liquid source (C2, C4, H1, liquid-patterns checks) cannot run. The report must note which checks were skipped.
