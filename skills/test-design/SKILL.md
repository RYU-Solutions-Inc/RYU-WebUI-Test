---
name: test-design
description: Test design methodology including coverage categories, scenario structure, TC-ID format, priority levels, and review statuses. Use when designing E2E test scenarios.
user-invocable: false
---

## Instructions

### Coverage Category Methodology

For every feature, design scenarios across ALL categories. Do not skip any category.

**Happy Path** — Standard successful user journey
- The most common way a user would complete this task
- Multiple happy paths if the feature has multiple valid flows (e.g., login with email vs. SSO)
- One happy path per feature should be tagged `@smoke` as the critical sanity check

**Edge Cases** — Boundary conditions and limits
- Minimum and maximum valid values (form field limits, quantity limits, text length)
- Boundary values (exactly at the limit, one below, one above)
- Empty states (what does the page look like with no data?)
- Pagination boundaries (first page, last page, single-page result)
- Unicode, special characters, very long strings in text fields

**Negative Scenarios** — Invalid inputs and unauthorized actions
- Required field left empty
- Invalid format (wrong email, phone, date format)
- Duplicate submissions (same form submitted twice)
- Unauthorized access (accessing a page without login, accessing another user's data)
- Disabled features (clicking a disabled button, submitting during loading state)

**Error Conditions & Recovery** — System failures and graceful degradation
- API timeout during a critical action
- API returns 500 error
- Network loss during form submission
- Session expiry during a multi-step flow
- Graceful error messages shown to user
- Recovery: can the user retry after an error?

### Scenario Structure

Every scenario MUST have all four layers:

**Flow** — Numbered steps describing user actions
- Start from a known state (which page, what data exists)
- Each step is one user action (click, fill, navigate, wait)
- Be specific: "Click the 'Apply' button" not "Apply the coupon"

**Validation Points** — What to check and where
- Place VPs immediately after the step that should trigger the observable result
- Each VP checks ONE thing
- Reference specific UI elements by selector or text content

**Expected Results** — How to judge pass/fail
- Must be specific and observable: "Element `.total` contains '$45.00'" not "Total is correct"
- Include the exact value, text, or state expected
- For dynamic values, describe the relationship: "Total decreased by the coupon percentage"

**Instrumentation** — Debug aids
- Screenshots: capture before and after key actions
- Network: capture relevant API calls with expected status codes
- Console: always capture console errors
- Trace ID: if the app includes backend trace IDs, capture them

### TC-ID Assignment

Format: `TC-{COMPONENT}-{NNN}`
- COMPONENT is uppercase, matches the folder name (e.g., CART, AUTH, CHECKOUT)
- NNN is zero-padded sequential within the component
- Check existing scenarios to avoid duplicate IDs
- IDs are permanent — do not reuse a deleted scenario's ID

### Priority Assignment

- **P0** — Critical business flow that, if broken, blocks the user from completing a core task. Smoke test candidate.
- **P1** — Important flow that affects significant user segments but has workarounds.
- **P2** — Standard coverage for completeness. Edge cases and low-risk negative scenarios.

### Review Status

All AI-generated scenarios start as `draft`. Only the analyst can change the status to `reviewed`, `revised`, or `rejected`.

### Test Annotations

When designing or updating scenarios, handle annotations as follows:

- **New scenarios:** No annotations by default. Annotations are added later based on execution results.
- **Editing scenarios:** Preserve existing annotations unless the analyst explicitly asks to change them.
- **Flaky detection:** If results-analysis flags a test as flaky, suggest adding `@known-flaky` annotation to the analyst.
- **Blocked tests:** If a test failure is categorized as a real bug and a ticket is filed, add `@blocked(ticket: "...", since: "vX")`.
- **Removing annotations:** When the underlying issue is resolved, remind the analyst to remove the annotation.

Annotation format: `**Annotations:** @annotation(param: "value", since: "vX")`

Place the Annotations field after Priority in the scenario header. Omit the field entirely if no annotations exist.

Supported annotations:
- `@skip(reason: "...", since: "vX")` — excluded from all runs and pass-rate calculations
- `@known-flaky(reason: "...", since: "vX")` — included in runs with automatic retries; failures don't count as regressions
- `@blocked(ticket: "JIRA-456", since: "vX")` — excluded from runs until ticket resolved
