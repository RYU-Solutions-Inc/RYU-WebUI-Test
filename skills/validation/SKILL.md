---
name: validation
description: Quality validation checklists for scenario design, script generation, and test results. Used by the QA Validator agent to catch errors before outputs reach the analyst.
user-invocable: false
---

## Instructions

### Overview

This skill provides structured validation checklists for three pipeline phases. The QA Validator loads this skill and runs the appropriate checklist based on the validation mode requested by the E2E Architect.

### Scenario Validation Checklist

After the Scenario Designer writes `test-cases/{component}.md`, validate:

1. **Four-layer structure:** Every scenario must have all four sections: Flow, Validation Points, Expected Results, Instrumentation. Read the file and check each `## TC-` section.

2. **Flow completeness:** Each flow must have numbered steps (1, 2, 3...) forming a complete user journey from entry to exit. Flag flows that end abruptly or skip steps.

3. **Expected results specificity:** Every expected result must reference a specific, observable UI element or condition. Reject:
   - "Page loads correctly" → should specify what element appears
   - "Form submits successfully" → should specify redirect URL, success message element, or API response
   - "Error is shown" → should specify error message text or element selector

4. **Validation point step references:** Each row in the Validation Points table has a "Step" column. Verify that every step number references an actual step in the Flow section. Flag step numbers that don't exist.

5. **TC-ID format:** Must follow `TC-{COMPONENT}-{NNN}` (e.g., `TC-LOGIN-001`). Component name must be uppercase letters/hyphens. Number must be 3 digits, zero-padded.

6. **TC-ID uniqueness:** Read ALL files in `test-cases/` and collect every TC-ID. Flag any duplicates.

7. **Coverage categories:** For each component, verify at least one scenario in each category:
   - Happy Path (standard successful flow)
   - Edge Cases (boundary conditions)
   - Negative Scenarios (invalid inputs, unauthorized actions)
   - Error Conditions & Recovery (system failures, graceful degradation)

8. **Instrumentation:** Each scenario must specify:
   - Screenshots at key steps (at minimum: before action and after action)
   - Network capture for relevant API calls
   - Console log/error capture

9. **Metadata completeness:** Each scenario must have: Category, Requirement (or "N/A"), Review Status, Priority (P0/P1/P2).

### Script Validation Checklist

After the Script Engineer writes scripts, validate:

1. **TypeScript compilation:** Run `npx tsc --noEmit` targeting the generated files. If a `tsconfig.json` exists, use it. If not, run with `--strict --noEmit --moduleResolution node` flags. Report any compilation errors with file paths and line numbers.

2. **Import resolution:** For every `import` or `require` statement in generated files, verify the referenced file exists at the specified path. Check both relative imports and imports from `@playwright/test`.

3. **Selector cross-check:** Extract all selectors used in test files and Page Objects. Cross-reference against:
   - Source code: search for matching `data-testid`, class names, IDs in app source files
   - Crawl results: search `config/crawl-results.json` for matching testids, ARIA labels, and elements
   - Flag selectors that don't match any known element

4. **Page Object coverage:** For each test file, identify which Page Objects it imports. Verify that every `page.locator()`, `page.getByRole()`, `page.getByTestId()` call in the test either:
   - Uses a method/property from the imported Page Object, OR
   - Is a direct Playwright call that should be in the Page Object instead (flag as POM violation)

5. **TC-ID traceability:** Every `.spec.ts` file must have a comment referencing its source TC-ID (e.g., `// TC-LOGIN-001`). The referenced TC-ID must exist in `test-cases/`.

6. **Fixture dynamic data:** Read fixture factory functions. Flag any that return hardcoded values (static strings, fixed numbers). Factories must use `Date.now()`, `crypto.randomUUID()`, or similar dynamic generation.

7. **Test isolation:** Check for patterns that indicate inter-test dependencies:
   - Variables declared outside `test()` blocks that are mutated inside them
   - Sequential test assumptions (comments like "run after test X")
   - Missing `beforeEach`/`afterEach` hooks
   - Shared state between test files

8. **Tagging compliance:** Check that each test has appropriate tags:
   - P0 scenarios must have `@smoke` tag
   - All scenarios must have priority tag (`@p0`, `@p1`, or `@p2`)
   - New scenarios in the current version must have `@new` tag

9. **Setup/teardown hooks:** Every `test.describe()` block must have:
   - `test.beforeEach()` with data seeding logic
   - `test.afterEach()` with data reset logic

### Results Validation Checklist

After the E2E Architect interprets test execution results, validate:

1. **Failure categorization cross-reference:** Read the version manifest to get impact categories. For each test failure:
   - Failure in UNAFFECTED area → must be flagged as high priority (unexpected regression)
   - Failure in MUST UPDATE area → should be flagged as expected (maintenance needed)
   - Failure in NEW area → should be flagged as expected (new feature under development)

2. **Flaky test detection:** If previous run results exist, compare outcomes. A test that passed previously but fails now (or vice versa) without corresponding code changes is a potential flaky test.

3. **Coverage accuracy:** Count actual `.spec.ts` files in `versions/` and compare against the numbers reported in the test results summary. Flag any mismatches.

4. **Missing results:** Read the manifest to identify all tests that should have run. Cross-reference with actual test results. Flag any tests in the manifest with no results.

### Validation Report Format

Return a structured markdown report:

```markdown
# Validation Report: {Scenario|Script|Results} — {component}

**Status:** PASS | FAIL ({N} issues)
**Validated:** {ISO timestamp}
**Mode:** {scenario|script|results}

## Issues

### Critical (blocks delivery)
1. [{TC-ID or file:line}] {Description of the issue}
   **Fix:** {Specific action needed}

### Warnings (should fix)
1. [{TC-ID or file:line}] {Description of the issue}
   **Fix:** {Specific action needed}

## Summary
- Checks run: {N}
- Passed: {N}
- Critical failures: {N}
- Warnings: {N}
```

### Retry Protocol

When the Architect routes issues back to the originating sub-agent:
1. The sub-agent receives the validation report with specific issues
2. The sub-agent fixes the issues and writes updated files
3. The Architect re-invokes the Validator on the same files
4. If issues remain after 2 retry rounds, the Validator's report is escalated to the analyst with a note: "These issues could not be auto-resolved. Please review."
