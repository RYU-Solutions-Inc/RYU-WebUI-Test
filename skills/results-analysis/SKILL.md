---
name: results-analysis
description: How to interpret Playwright test results, categorize failures, cross-reference manifests, and suggest fixes. Use after test execution completes.
user-invocable: false
---

## Instructions

### Step 1: Parse Results

Read the JUnit XML and/or HTML report from `test-results/`. Extract:
- Total tests, passed, failed, skipped
- For each failure: test name, error message, stack trace, duration

### Step 2: Gather Artifacts

For each failed test, locate:
- Screenshot at the failure point (`test-results/screenshots/`)
- Playwright trace file (`test-results/traces/`)
- Console logs captured during the test
- Network request/response logs

### Step 3: Categorize Each Failure

| Category | Signals |
|---|---|
| **Real Bug** | The app rendered unexpected content, returned wrong data, or behaved differently than specified. The test logic is correct but the app is wrong. |
| **Flaky Test** | The test has passed before on the same version. Error is timing-related (timeout, element not yet visible). Check if this test has failed intermittently. |
| **Environment Issue** | Error references connectivity (ECONNREFUSED, timeout reaching endpoint), service unavailability (503), or missing data. |
| **Test Maintenance** | Error references selectors that no longer exist, steps that don't match the current flow, or expected values that have changed. Often correlates with MUST UPDATE items in the manifest. |

**Historical flaky detection:** Before categorizing, check `test-history.json` for this TC-ID's history. If the test has failed in ≥2 of the last 5 runs but not consistently (i.e., it has also passed in that window), flag it as likely flaky regardless of the current error signal.

**Annotation-aware analysis:** When categorizing failures:
- Tests annotated `@known-flaky` that fail: categorize as "Flaky Test" regardless of other signals. Do not count these in pass-rate trend calculations.
- Tests annotated `@skip` or `@blocked` should not appear in results (they shouldn't have run). If they do appear, flag this as a configuration issue.
- Tests that "passed on retry" (Playwright reports these with a retry count in JUnit XML): flag as flaky signal even if not yet annotated. Suggest adding `@known-flaky` annotation.

### Step 4: Cross-Reference Manifest

Check the current version's manifest to contextualize failures:
- Failure in a **MUST UPDATE** area that wasn't updated → expected test maintenance
- Failure in a **CHECK** area → move to MUST UPDATE, investigate for regression
- Failure in an **UNAFFECTED** area → unexpected regression, HIGH priority
- Failure in a **NEW** area → expected during development, normal iteration

### Step 5: Suggest Fixes

For each failure, provide a specific, actionable fix suggestion:
- Selector issues: name the old selector and the new one
- Timing issues: specify which wait to add and where
- Data issues: identify what data is missing or wrong
- Real bugs: describe the expected vs. actual behavior with evidence

### Step 6: Summarize

Produce a summary with:
- Overall pass rate
- Failure count by category
- Most critical failures first (real bugs, then unexpected regressions)
- Action items for the analyst

### Step 7: Update Test History

After producing the summary, append a run entry to `test-history.json` (create the file if it doesn't exist):

1. Read the existing `test-history.json` file (or initialize with `{ "runs": [] }` if missing)
2. Generate a run ID: `run-{YYYY-MM-DD}-{NNN}` where NNN increments per day
3. Append a new entry to the `runs` array:

```json
{
  "id": "run-2026-02-16-001",
  "date": "2026-02-16T14:30:00Z",
  "version": "1.0",
  "mode": "regression",
  "total": 64,
  "passed": 62,
  "failed": 2,
  "skipped": 0,
  "pass_rate": 96.9,
  "duration_seconds": 187,
  "report": "reports/run-results-2026-02-16.md",
  "failures": {
    "TC-AUTH-005": { "category": "flaky" },
    "TC-CART-008": { "category": "test_maintenance" }
  }
}
```

4. Write the updated file back

This file is **append-only** — never delete or overwrite existing entries.
