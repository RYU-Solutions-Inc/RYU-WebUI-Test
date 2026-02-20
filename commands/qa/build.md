---
description: Generate Playwright scripts from designed test scenarios
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
argument-hint: "<feature-name> [--force]"
---

Generate Playwright scripts for: $ARGUMENTS

The Architect delegates this to the **Script Engineer** agent.

## Review Gate

Before generating scripts, check scenario review status in `test-cases/{component}.md`:

- **All accepted** → proceed with script generation
- **Some not reviewed (draft)** → warn the analyst, list the unreviewed TC-IDs, and stop
- **Some rejected or needs_changes** → warn the analyst, list them, and stop
- **`--force` flag provided** → skip the review gate and build all scenarios regardless (for rapid prototyping)

If stopped, tell the analyst: "Run `/qa/review {component}` to process your review, or re-run with `--force` to skip the review gate."

## Steps

1. E2E Architect reads `test-cases/{component}.md` for the target feature
2. E2E Architect checks review status (see Review Gate above)
3. E2E Architect reads the current version manifest to know what version folder to target
4. E2E Architect delegates to Script Engineer with:
   - Only scenarios with `Review Status: accepted` (unless `--force`)
   - App source code paths (or crawl results) for selector extraction
   - Existing Page Objects and fixtures to reuse
   - Version folder path for output
5. Script Engineer loads `playwright-scripting` skill
6. Script Engineer generates test specs, Page Objects, fixtures, and helpers
7. Script Engineer writes output to appropriate locations
8. E2E Architect delegates to **QA Validator** (script mode) to validate:
   - TypeScript compilation (`npx tsc --noEmit`), import resolution, selector references, fixture validity, test isolation
   - If issues found → send back to Script Engineer for fixes (max 2 rounds)
9. E2E Architect reviews the validated output for standard compliance
10. E2E Architect updates the manifest with script generation status
11. E2E Architect presents results to analyst

## Output

- `versions/v{X}/{component}/*.spec.ts` — Playwright test files
- `pages/*.page.ts` — new or updated Page Objects
- `fixtures/*.ts` — new or updated fixture factories
- `helpers/*.ts` — new or updated shared utilities
- Updated `manifest.json` with script status
