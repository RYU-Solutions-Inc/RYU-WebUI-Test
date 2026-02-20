---
description: Get up to speed on the active project — tool intro + comprehensive project briefing
allowed-tools: Read, Glob, Grep
---

Get the analyst up to speed on the active project. Designed for someone inheriting a project or returning after an absence.

This is direct Architect work — no delegation needed.

## Steps

### Detect Familiarity
1. Ask the analyst: "Are you familiar with the QA E2E testing team, or is this your first time using it?"
2. If **first time** → present condensed tool overview (step 3), then continue to project briefing (step 4)
3. If **familiar** → skip to project briefing (step 4)

### Condensed Tool Overview (first-timers only)
3. Present a 60-second version of the testing lifecycle:
   - 6 phases: ANALYZE → DESIGN → REVIEW → BUILD → RUN → REPORT
   - What requires human review (scenario review files)
   - Key commands: `/qa/design`, `/qa/build`, `/qa/run`, `/qa/status`
   - Do NOT offer interactive demo (that's `/qa/tour`'s job)

### Project Briefing (everyone)
4. Read `app-profile.md` and present **App Summary**: app name, URL, tech stack, key flows, environments, conventions
5. Read current version manifest and `test-cases/` to present **Testing Status**: version, scenarios, scripts, latest run results
6. Read `test-history.json` (if exists) or scan version manifests for **Version History**: timeline with pass rates
7. Scan `test-cases/` and `versions/` for **Coverage Map**: components with scenario/script counts, gaps flagged
8. Identify **Open Items**: pending reviews, failing tests, scenarios without scripts, unverified CHECK items
9. Read `handoff.md` (if exists) for **Handoff Notes**: flaky tests, app quirks, stakeholder preferences, manual steps
10. Scan recent `versions/v{X}/impact-report.md` for **Recent Activity**: last 2-3 versions summarized
11. Determine **Suggested Next Action** based on project state:
    - Pending reviews → suggest `/qa/review`
    - Recent failures → suggest `/qa/results`
    - Coverage gaps → suggest `/qa/design {component}`
    - All green → "Project is in good shape"
12. Ask: "Any questions about the project, or ready to get started?"

## Output

Inline conversational briefing — not saved to file. Present sections in order:
1. Tool Overview (first-timers only)
2. App Summary
3. Testing Status
4. Version History
5. Coverage Map
6. Open Items
7. Handoff Notes (if handoff.md exists)
8. Recent Activity
9. Suggested Next Action

## Handoff Notes

`handoff.md` is an optional file in the project folder. The outgoing analyst creates it to pass tribal knowledge. Suggested sections:
- **Known Flaky Tests** — tests that fail intermittently with workarounds
- **App Quirks** — feature flags, sandbox resets, environment-specific behaviors
- **Stakeholder Notes** — preferences, priorities, release cadence
- **Manual Steps Not Yet Automated** — things the analyst does manually

If `handoff.md` doesn't exist, skip this section silently.
