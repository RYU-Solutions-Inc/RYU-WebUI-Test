---
name: reporting
description: How to produce quality reports including release readiness assessments, coverage reports, and version deltas. Use when generating reports for analysts or stakeholders.
user-invocable: false
---

## Instructions

### Report Principles

1. **Lead with the most important finding.** Don't bury critical information in tables.
2. **Be opinionated.** Reports should make recommendations, not just present data.
3. **Cite data sources.** Every number should reference where it came from.
4. **Be self-contained.** A reader should understand the report without other context.

### Release Readiness Quality Gates

Assess these gates and report pass/fail for each:

| Gate | Pass Criteria |
|---|---|
| NEW features covered | Every NEW item in the manifest has designed scenarios |
| MUST UPDATE completed | Every MUST UPDATE item has been updated |
| CHECK items verified | All CHECK items have been run and categorized |
| Review coverage | ≥80% of scenarios are reviewed (not draft) |
| Smoke tests passing | All @smoke tests pass on target environment |
| P0 tests passing | All @p0 tests pass |
| No open blockers | No real bugs categorized as P0 in the latest run |

### Recommendation Logic

- **GO:** All gates pass
- **CONDITIONAL GO:** Most gates pass; remaining issues are low-risk with documented workarounds
- **NO-GO:** Critical gates fail (P0 bugs, smoke failures, untested NEW features)

Always explain the reasoning. "CONDITIONAL GO because 2 P1 edge case tests are failing due to a known timing issue that does not affect production users."

### Report File Naming

Use version-stamped filenames: `{report-type}-v{X}.md`
- `rtm-v1.1.md`
- `coverage-v1.1.md`
- `release-readiness-v1.1.md`
- `version-delta-v1.1.md`

### Trend Data

When producing release readiness or coverage reports, reference `test-history.json` for:
- Pass rate trends across versions (improving or degrading)
- Flaky test identification (tests that fail intermittently)
- Regression detection (tests that recently started failing)
- Historical comparison data for version delta reports
