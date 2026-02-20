---
description: Run impact analysis for a new application version (smart auto-detect of git, tickets, and URL)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
argument-hint: "<version-identifier>"
---

Run impact analysis for version: $ARGUMENTS

This is direct Architect work using the `impact-analysis`, `version-management`, and `requirements-analysis` skills.

## Smart Auto-Detect

Auto-detect what input sources are available and combine them:

1. **Git changes:** Run `git diff` and `git log` — if changes found, read diffs and categorize impact
2. **Requirements tickets:** Check `.mcp.json` for Jira/GitHub connector — if connected, read sprint/release tickets
3. **URL re-crawl:** Check `app-profile.md` for `baseURL` — if configured, offer to re-crawl for UI changes

Report to the analyst:
```
Sources detected:
✓ Git changes: {N} files changed since {previous version}
✓ Jira Sprint {N}: {N} tickets ({breakdown})
○ URL re-crawl: available — would you like me to re-crawl?
```

## Sprint vs. Release

- If Jira connected: read `fixVersion` and `sprint` from tickets
  - `fixVersion` changed → `"releaseType": "new-release"` (broader analysis)
  - Only `sprint` changed → `"releaseType": "sprint"` (incremental)
- If no Jira: ask the analyst
- Store in manifest

## Steps

1. If no version identifier was provided in the arguments, ask the analyst for it
2. Load `impact-analysis`, `version-management`, and `requirements-analysis` skills
3. **Auto-detect available sources** (git, tickets, URL)
4. **Git analysis** (if available): Run git diff, map changes to components, pages, and flows
5. **Requirements analysis** (if available): Read sprint/release tickets, extract acceptance criteria, determine sprint vs. release scope
6. **URL re-crawl** (if analyst confirms): Re-run crawler, compare with previous crawl results
7. **Cross-reference all sources:**
   - Requirements with code changes → normal, map for traceability
   - Requirements without code changes → flag "potentially unimplemented"
   - Code changes without requirements → flag "unplanned change"
   - URL changes without code or requirements → flag "undocumented UI change"
8. Cross-reference against existing test scenarios and Page Objects
9. Categorize all items: NEW, MUST UPDATE, CHECK, UNAFFECTED
10. Create `versions/v{X}/` directory
11. Write `versions/v{X}/manifest.json` (including release, sprint, releaseType fields)
12. Write `versions/v{X}/impact-report.md` with unified impact table and gap analysis
13. Present summary to analyst with action items

## Output

- `versions/v{X}/manifest.json` — version manifest with release/sprint metadata
- `versions/v{X}/impact-report.md` — human-readable impact report with all sources
