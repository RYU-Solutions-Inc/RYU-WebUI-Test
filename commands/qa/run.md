---
description: Execute tests in the specified mode (--new, --impact, --regression, --smoke)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
argument-hint: "--mode [--test TC-ID] [--retry-flaky] [--browsers list] [--device profile]"
---

Execute tests with arguments: $ARGUMENTS

This is direct Architect work using the `pre-flight`, `version-management`, and `results-analysis` skills.

## Steps

1. Parse the mode argument: `--new`, `--impact {component}`, `--regression`, `--smoke`, `--test {TC-ID}`, or `--retry-flaky`
2. Parse optional arguments: `--browsers {list}`, `--device {profile}`
3. Load `pre-flight` skill → run all pre-flight checks
4. If pre-flight fails → STOP, report failures, offer remediation
5. If pre-flight passes → load `version-management` skill
6. Resolve the test file list based on mode:
   - `--new`: files tagged @new in current version folder
   - `--impact {component}`: all files under {component}/ across version chain
   - `--regression`: complete resolved set from manifest inheritance
   - `--smoke`: files tagged @smoke across all components
   - `--test {TC-ID}`: resolve specific test(s) — see step 6a
   - `--retry-flaky`: resolve flaky tests from last run — see step 6b
6a. **Single test resolution (`--test`):**
   - Parse TC-IDs (comma-separated if multiple: `--test TC-CART-008,TC-AUTH-005`)
   - For each TC-ID, search `test-cases/` to find the component
   - Use version-management skill's "Resolving Individual Tests" to find the script file path
   - Build a Playwright command with `--grep` targeting the TC-ID test names
6b. **Retry-flaky resolution (`--retry-flaky`):**
   - Read the last run entry from `test-history.json`
   - Identify TC-IDs that failed AND are either annotated `@known-flaky` or flagged flaky in test-history.json
   - Resolve each to its script file path using individual test resolution
   - If no flaky failures found, report "No flaky failures to retry" and stop
7. Check for test annotations — exclude `@skip` and `@blocked` tests from the run (unless `--test` explicitly targets them)
8. Build the Playwright command with resolved file paths and browser/device config
9. Execute: `npx playwright test {file-list} --config=config/playwright.config.ts`
10. Wait for completion
11. Load `results-analysis` skill → interpret results
12. Present categorized results to analyst

## Run Modes

- `--new` — run only scripts added/changed in the current version
- `--impact {component}` — run all scripts under the specified component
- `--regression` — run the complete resolved test set
- `--smoke` — run @smoke-tagged tests only
- `--test {TC-ID}` — run specific test(s) by TC-ID (comma-separated for multiple)
- `--retry-flaky` — re-run only flaky-flagged failures from the last run

## Optional Flags

- `--browsers {list}` — comma-separated browser list (e.g., "chrome,safari")
- `--device {profile}` — device profile (e.g., "mobile", "tablet", "desktop")

## Output

- Pre-flight results (pass/fail per check)
- `test-results/` — JUnit XML, HTML report, screenshots, videos, traces
- Interpreted results summary with categorized failures and fix suggestions
