---
description: Show the current project state summary
allowed-tools: Read, Glob, Grep
---

Show the current project status.

This is direct Architect work — no delegation needed.

## Steps

1. Read `.active-project` to identify the current project
2. Display: "Active project: {name} (projects/{name}/)"
3. List all available projects in `projects/`
4. Read CLAUDE.md for current app and version
5. Read the current version's manifest for categorization status
6. Count scenarios in `test-cases/` by review status
7. Count scripts in the current version folder
7a. Scan `test-cases/` for annotation counts (@skip, @known-flaky, @blocked)
8. Read most recent test results if available
9. Read `test-history.json` for historical run data
10. Build version history table from test-history.json entries (version, date, total tests, pass rate)
11. Generate insights from test-history.json:
    - **Flaky test detection** — TC-ID failed in ≥2 of last 5 runs but not consistently
    - **Status escalation** — TC-ID category changed from flaky/test_maintenance to real_bug
    - **Stability streaks** — component with 100% pass rate across last N runs
    - **Last known pass** — for currently failing TC-IDs, find the last run where they passed
    - **Pass rate trend** — compare current vs previous version pass rates
12. Determine suggested next action based on project state:
    - Pending reviews → suggest `/qa/review`
    - Recent failures → suggest `/qa/results`
    - Coverage gaps → suggest `/qa/build`
    - All green → "Project is in good shape"
13. Present a concise status summary inline (not saved to file)

## Output Format

Respond conversationally with a summary like:

```
Active Project: canadian-retirement-planner (projects/canadian-retirement-planner/)
Available projects: canadian-retirement-planner

Current State: App v1.1
  Version: v1.1 (base: v1.0)
  Scenarios: 97 total | 86 reviewed | 11 draft
  Scripts: 87 generated | 10 pending

Last Run: Feb 17 — v1.1 — 78/84 passed (93%)
  3 real bugs | 2 test maintenance | 1 environment
Annotations: 3 @skip | 2 @known-flaky | 1 @blocked

Version History:
  v1.0  Feb 15  64 tests  96.9% pass  (2 failures)
  v1.1  Feb 17  84 tests  92.8% pass  (5 failures) ← current

Insights:
  ⚠ TC-CHECKOUT-014 has failed 3 of last 10 runs — likely flaky
  ⚠ TC-AUTH-005 escalated from flaky (v1.0) to real bug (v1.1)
  ✓ auth/: 100% pass since v1.0 (28 tests, all stable)
  ✓ dashboard/: inherited unchanged for 2 versions, always passing

Coverage Gaps: 6 scenarios identified but not yet designed
Open Items: 3 pending reviews, 2 test maintenance fixes

Suggested Next Action: You have 3 real bugs from the last run. Run /qa/results to review them.
```

Do NOT save this to a file — present inline only.
