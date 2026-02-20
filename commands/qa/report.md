---
description: Generate quality reports (RTM, coverage, readiness, version delta)
allowed-tools: Read, Write, Edit, Glob, Grep
argument-hint: "[rtm|coverage|readiness|delta]"
---

Generate quality reports. Report type requested: $ARGUMENTS

The Architect delegates this to the **QA Reporter** agent.

## Steps

1. E2E Architect gathers all project artifacts (manifests, scenarios, scripts, results)
2. E2E Architect delegates to QA Reporter with:
   - Current version and manifest
   - All test-case files with review statuses
   - Most recent test results
   - Previous reports for trend comparison
3. QA Reporter loads `traceability` and `reporting` skills
4. QA Reporter produces requested reports
5. QA Reporter writes output to `reports/`
6. E2E Architect reviews reports for accuracy
7. E2E Architect presents summary to analyst

## Report Types

- `rtm` — requirements traceability matrix
- `coverage` — coverage report by feature and category
- `readiness` — release readiness assessment with quality gates
- `delta` — version comparison showing what changed
- _(blank)_ — generate all reports

## Output

- `reports/rtm-v{X}.md` — requirements traceability matrix
- `reports/coverage-v{X}.md` — coverage report
- `reports/release-readiness-v{X}.md` — release readiness assessment
- `reports/version-delta-v{X}.md` — version comparison
