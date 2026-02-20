---
name: pre-flight
description: Pre-flight validation checks for environment, data, and infrastructure readiness before test execution. Use before every test run.
user-invocable: false
---

## Instructions

### Environment Checks

1. **App reachable:** HTTP GET the base URL from CLAUDE.md. Expect 200.
2. **Correct version:** Check the app's version endpoint or meta tag. Compare against the manifest's version.
3. **Services up:** HTTP GET each dependent service URL listed in CLAUDE.md (auth, payment, etc.).
4. **Feature flags:** If applicable, verify required feature flags are enabled via the flag service API.

### Data Checks

1. **Seed endpoint:** POST to the seed endpoint. Expect success response.
2. **Test accounts:** Attempt a login API call with test credentials. Expect success.
3. **Key data exists:** Query critical data endpoints (product list, user list) to verify data was seeded.

### Infrastructure Checks

1. **Browsers installed:** Run `npx playwright install --check` or check browser binary paths.
2. **Config valid:** Parse `playwright.config.ts` — verify baseURL is set, projects are defined.
3. **Disk space:** Check available disk space in the screenshots/results directory.
4. **Manifest integrity:** Validate all manifests in the version inheritance chain:
   - Read the current version's `manifest.json` — verify it parses as valid JSON
   - Check required fields: `version`, `base` (except first version), `regression_scope`
   - Walk the `base` chain — for each ancestor, verify its `manifest.json` exists and parses correctly
   - If any manifest in the chain is corrupted, report which one and stop — do not run tests against a broken inheritance chain
   - Check for `.manifest.json.bak` files and suggest recovery if the primary is corrupted

### Failure Handling

If ANY check fails:
- Stop immediately — do NOT proceed to test execution
- Report exactly which check failed with specific error details
- Suggest remediation (e.g., "Run `npx playwright install` to install missing browsers")
- Offer to attempt auto-fix for safe operations (re-seed data, retry endpoint)
- Wait for analyst decision before proceeding

### Pre-Flight Scripts

Generate pre-flight as Playwright test files in `preflight/`:
- `preflight/environment.spec.ts`
- `preflight/data.spec.ts`
- `preflight/infrastructure.spec.ts`

These run with: `npx playwright test preflight/ --reporter=list`

Target total execution time: under 30 seconds.
