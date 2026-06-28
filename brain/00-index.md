# Brain — Shopify Theme Audit Skill

This folder is the persistent memory layer for this product.
Each file covers one dimension of the product so any future session can pick up full context instantly.

## Files in this folder

| File | What it holds |
|---|---|
| `01-product-overview.md` | What this product is, what it does, why it exists |
| `02-architecture.md` | File-by-file map of how the skill is structured |
| `03-audit-methodology.md` | The exact audit process, step order, and scoring rules |
| `04-check-taxonomy.md` | Every check ID, its severity, and where it lives |
| `05-personas-and-usecases.md` | Who uses this, trigger prompts, and use-case patterns |
| `06-creator-and-positioning.md` | Creator background, agency context, distribution |
| `07-decisions-and-constraints.md` | Design decisions made and the reasoning behind them |
| `08-changelog-and-roadmap.md` | What changed in each version and what is planned next |

## How to use this brain

At the start of any work session on this repo, read `00-index.md` first, then read only the files relevant to the task at hand. You do not need to read all files every time.

- Fixing a check or adding a new check → read `03-audit-methodology.md` + `04-check-taxonomy.md`
- Updating documentation or README → read `01-product-overview.md` + `05-personas-and-usecases.md`
- Refactoring file structure → read `02-architecture.md`
- Deciding whether a new feature fits the product → read `07-decisions-and-constraints.md`
