---
description: Interpret the most recent test results and categorize failures
allowed-tools: Read, Glob, Grep
---

Interpret the most recent test results.

This is direct Architect work using the `results-analysis` skill.

## Steps

1. Load `results-analysis` skill
2. Find the most recent test results in `test-results/`
3. Parse JUnit XML and/or HTML report
4. For each failure, gather artifacts (traces, screenshots, console logs)
5. Categorize each failure: real bug, flaky test, environment issue, or test maintenance
6. Cross-reference against the version manifest for context
7. Suggest specific fixes for each failure
8. Present categorized summary to analyst
9. Append run entry to `test-history.json` (see results-analysis skill, Step 7)

## Output

- Categorized failure summary with fix suggestions (inline)
- `reports/run-results-{date}.md` — persistent results report
- `test-history.json` — updated with this run's entry (appended)
