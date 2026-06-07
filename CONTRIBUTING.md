# Contributing

## Adding a new check

1. Open `audit-checklist.md`
2. Add your check under the correct severity section
3. Follow this format exactly:

```
### H8: [Short name]
**Where to look:** [file/directory]
**What to find:** [exact thing to search for]
**Why it matters:** [performance/SEO/accessibility impact]
**Flag if:** [specific condition]
**Do not flag:** [edge cases to skip]
```

## Adding a Liquid pattern

1. Open `liquid-patterns.md`
2. Add a new section with a unique ID (e.g. `SECTION-03`)
3. Always include both INCORRECT and CORRECT versions
4. Add a comment explaining WHY the incorrect version is wrong

## Submitting

- Open an issue first for large additions
- PRs for individual checks can go straight to review
- Keep additions focused — one check per PR is ideal
