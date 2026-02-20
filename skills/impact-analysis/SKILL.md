---
name: impact-analysis
description: How to read git diffs, classify changes by impact level, map to existing tests, and produce version manifests. Use when a new application version is delivered.
user-invocable: false
---

## Instructions

### Step 1: Get the Diff

Run `git diff {previous-version-tag}..{current-version-tag}` to get the full list of changed files.

If tags aren't available, use `git log --oneline {previous-commit}..{current-commit}` to identify the range, then diff.

### Step 2: Classify Changed Files

For each changed file, classify it:

| File Type | Impact Level | What to Check |
|---|---|---|
| Component file (.tsx, .vue, .svelte) | HIGH | Changed selectors, altered flow, new/removed elements |
| CSS/style file | MEDIUM | Layout changes that might affect element visibility or position |
| API/service file | HIGH | Changed request/response shapes, new endpoints, removed endpoints |
| Route/config file | HIGH | New pages, changed URLs, altered navigation structure |
| Utility/helper | LOW-MEDIUM | Shared logic changes that might ripple |
| Test/spec file | NONE | App's own tests changing — irrelevant to our E2E tests |
| Docs/README | NONE | No impact |
| Package.json / config | LOW | Dependency changes — usually no direct test impact |

### Step 3: Map Changes to Existing Tests

For each HIGH or MEDIUM impact file:

1. Identify which component/page it belongs to
2. Look up existing test scenarios in `test-cases/{component}.md`
3. Look up existing scripts in `versions/` and Page Objects in `pages/`
4. Determine the specific impact:
   - **Selector renamed** → Page Object needs updating → MUST UPDATE
   - **New element added** → New validation points possible → CHECK or NEW
   - **Element removed** → Tests referencing it will fail → MUST UPDATE
   - **Flow changed** (steps added/removed/reordered) → Script flow needs updating → MUST UPDATE
   - **API contract changed** → Network interception assertions need updating → MUST UPDATE
   - **Shared component changed** → All pages using it might be affected → CHECK

### Step 4: Categorize

Place every existing test into one of four buckets:

- **NEW** — Net new feature needing full design cycle
- **MUST UPDATE** — Existing tests that WILL fail; update before running
- **CHECK** — Existing tests that MIGHT be affected; run and verify
- **UNAFFECTED** — No relation to any change; inherit as-is

When in doubt between CHECK and UNAFFECTED, choose CHECK. It's better to run a test unnecessarily than to miss a regression.

### Step 5: Produce the Manifest

Write `versions/v{X}/manifest.json` with the categorization results.

### Step 6: Produce the Impact Report

Write `versions/v{X}/impact-report.md` with:
- Summary of changes (what changed, how many files, which components)
- Detailed breakdown per component with specific change details
- Action items: what needs to be designed, updated, or verified
- Risk assessment: which areas have the highest regression risk

### Pulling Requirements Context (Optional)

If Jira is connected via MCP:
1. Query for tickets in the release (by fix version, sprint, or label)
2. Read acceptance criteria from each ticket
3. Cross-reference ticket descriptions against code changes
4. Add requirement IDs to the manifest entries for traceability
