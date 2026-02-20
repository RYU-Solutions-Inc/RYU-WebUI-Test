# Changelog

All notable changes to the Agentic AI E2E Testing Team.

---

## [0.5.0] — 2025-02-17

### Added — Daily Workflow Polish
- **`--test {TC-ID}` flag on `/qa/run`** — re-run specific test(s) by TC-ID. Resolves script path through version inheritance chain.
- **`--retry-flaky` flag on `/qa/run`** — re-run only flaky-flagged failures from the last run. Reads `test-history.json` to identify flaky tests.
- **`--edit {TC-ID}` flag on `/qa/design`** — edit an existing accepted scenario. Preserves TC-ID, generates review file for only the changed scenario.
- **Test annotations** — `@skip(reason, since)`, `@known-flaky(reason, since)`, `@blocked(ticket, since)`. Annotations control execution behavior: skip excludes from runs, known-flaky gets retries, blocked excludes until ticket resolved.
- **Annotation-to-Playwright mapping** — Script Engineer translates annotations to `test.skip()`, `test.describe.configure({ retries: 2 })`, etc.
- **Playwright retry configuration** — per-test retries for `@known-flaky` via `test.describe.configure()`. Global retries default to 0.
- **Annotation-aware results analysis** — `@known-flaky` failures excluded from pass-rate trends. "Passed on retry" flagged as flaky signal.
- **Manifest validation** — JSON structure validation on every read (catches merge conflicts, truncated writes, empty files)
- **Manifest backup** — `.manifest.json.bak` created before every manifest write. Automatic recovery offer on corruption.
- **Manifest integrity in pre-flight** — validates entire version inheritance chain before test execution
- **Single scenario re-review** — review file generated for only edited scenarios, preserving existing accepted scenarios
- **Annotation counts in `/qa/status`** — shows count of @skip, @known-flaky, @blocked tests

### Changed
- `/qa/run` command supports 6 modes (was 4): added `--test` and `--retry-flaky`
- `/qa/design` command supports `--edit` flag for modifying existing scenarios
- Pre-flight adds manifest integrity check to Infrastructure Checks
- All shared files (CLAUDE.md, README.md, ROADMAP.md) updated with new flags

---

## [0.4.0] — 2025-02-17

### Added — Test Run History & Project Catchup
- **`test-history.json`** — append-only cumulative run log per project. Results-analysis skill appends after every test run. Tracks run ID, version, mode, pass/fail counts, failure categories.
- **`/qa/catchup` command** — get up to speed on a project. Two modes: (1) familiar with tool, new to project → project briefing; (2) new to both → condensed tool intro + project briefing. Reads `handoff.md` for tribal knowledge.
- **`handoff.md`** — optional per-project file for outgoing analysts to document tribal knowledge (flaky tests, app quirks, stakeholder notes, manual steps)
- **Enhanced `/qa/status`** — version history table, insights (flaky detection, status escalation, stability streaks, last known pass, pass rate trends), suggested next action
- **Historical flaky detection** — results-analysis skill checks `test-history.json` before categorizing failures; flags tests that fail intermittently across recent runs
- **Trend data in reports** — reporting skill references `test-history.json` for pass rate trends and regression detection

### Fixed
- `impact-analysis` skill Step 8 referenced `CLAUDE.md` instead of `app-profile.md` (missed in 0.3.0)

---

## [0.3.0] — 2025-02-17

### Added — Project Workspace Restructuring
- **`projects/` subfolder model** — each onboarded app now lives in `projects/{app-name}/` instead of at the repository root
- **`app-profile.md`** — per-app context file (replaces per-app `CLAUDE.md` to avoid confusion with root Architect role)
- **`.active-project`** — file at repo root tracks which project is currently active
- **`/qa/switch` command** — switch between projects or list available ones
- **Active Project Resolution** — Architect auto-detects active project on session start (reads `.active-project`, auto-selects if only one, asks if multiple)
- **Path resolution protocol** — all relative paths in commands/skills/agents resolve under `projects/{active-project}/`

### Changed
- Root `CLAUDE.md` now contains only the Architect role (shared, permanent) — no app-specific data
- `/qa/onboard` creates folders under `projects/{project-name}/` and generates `app-profile.md`
- `/qa/status` shows active project name and lists available projects
- All shared files (OVERVIEW, AGENT-ARCHITECTURE, README, e2e-architect) updated for new structure

---

## [0.2.0] — 2025-02-17

### Added — 7 Enhancements from User Testing
- **QA Validator agent** — supervisor that validates sub-agent outputs before analyst presentation (scenario structure, script compilation, results accuracy). Max 2 retry rounds, then escalate.
- **Validation skill** — three modes: scenario validation (9 checks), script validation (9 checks), results validation (4 checks)
- **Requirements-analysis skill** — reads and maps requirements from Jira/GitHub for impact analysis and test design
- **`/qa/tour` command** — testing lifecycle overview with interactive demo offer. Runs at end of onboarding. Triggers on version updates.
- **Three input sources for `/qa/analyze`** — smart auto-detect of git changes, requirements tickets, and URL re-crawl. Cross-references for gap detection.
- **Sprint vs. release detection** — manifest tracks `releaseType` via Jira `fixVersion` or analyst input
- **URL-first onboarding** — `/qa/onboard` defaults to URL mode. Guided setup menu when no arguments. `--source` flag for source-code mode.
- **Review file lifecycle** — edit-in-place with Round numbers. Audit trail preserved in scenario files.
- **Background processing docs** — `Ctrl+B`, `Shift+Up/Down`, Agent Teams in README
- **Phase 2.5: Validation** in test design pipeline (between Design and Review)
- **Script Validation phase** in script generation pipeline (between generation and presentation)

### Changed
- **Chromium-only default** — playwright config defaults to single Chromium desktop project. Safari/Firefox optional.
- **WebKit** moved from Recommended to Optional in dependency table
- Design and build commands now include QA Validator step before presenting to analyst
- Impact analysis now supports three input sources with cross-referencing
- Onboard command ends with lifecycle tour
- All commands tables updated: 5 agents, 13 skills, 11 commands (was 4/9/9)

---

## [0.1.0] — 2025-02-16

### Added — Initial Build
- **4 agents** — E2E Architect (CLAUDE.md), Scenario Designer, Script Engineer, QA Reporter
- **11 skills** — app-analysis, impact-analysis, test-design, playwright-scripting, version-management, pre-flight, results-analysis, traceability, reporting, url-crawler, review-management
- **10 commands** — onboard, analyze, design, build, run, results, report, status, full-cycle, review
- **URL-only mode** — Playwright crawler for analyzing apps without source code access
- **Review system** — markdown review files with checkboxes, review gate on `/qa/build`
- **Version inheritance model** — each version stores only changed scripts, inherits rest
- **Multi-app architecture** — one team repo, multiple app contexts
- **Complete spec** — 39 specification files covering all features, agents, skills, and commands

---

## Version Numbering

- **0.x** — Pre-release development
- **1.0** — First production-ready release (pending real-world validation)
