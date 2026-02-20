---
description: Design E2E test scenarios for a specific feature or flow
allowed-tools: Read, Write, Edit, Glob, Grep
argument-hint: "<feature-name> [--edit TC-ID]"
---

Design test scenarios for: $ARGUMENTS

The Architect delegates this to the **Scenario Designer** agent.

## Steps

1. E2E Architect reads CLAUDE.md and existing `test-cases/` for context
2. E2E Architect identifies the target feature from the argument
3. E2E Architect reads the application source code (or crawl results in URL mode) for the target feature
4. E2E Architect delegates to Scenario Designer with:
   - Feature name and scope
   - Relevant source code paths (or crawl results reference)
   - Requirements/tickets if available
   - Existing scenario IDs to avoid duplicates
5. Scenario Designer loads `test-design` and `review-management` skills
6. Scenario Designer runs Discover → Design → Trace phases
7. Scenario Designer writes scenarios to `test-cases/{component}.md`
8. Scenario Designer generates the review file: `test-cases/{component}.review.md`
9. E2E Architect delegates to **QA Validator** (scenario mode) to validate:
   - Four-layer structure completeness, TC-ID uniqueness, coverage categories, validation point references
   - If issues found → send back to Scenario Designer for fixes (max 2 rounds)
10. E2E Architect reviews the validated output
11. E2E Architect tells the analyst:
    - Summary: "{N} scenarios designed across {categories}"
    - "Open `test-cases/{component}.review.md` in your editor to review each scenario"
    - "Check ACCEPT, REJECT, or NEEDS CHANGES for each. Add feedback for any rejections."
    - "When done, save the file and tell me 'review done' or run `/qa/review {component}`"

## Edit Mode (`--edit`)

When the analyst runs `/qa/design {component} --edit {TC-ID}`:

1. Load the existing scenario from `test-cases/{component}.md`
2. Find the specific scenario by TC-ID, present it to the analyst
3. Ask: "What do you want to change?"
4. Delegate to Scenario Designer with the existing scenario + change request
5. Scenario Designer updates ONLY the specified scenario — preserves TC-ID
6. QA Validator re-validates the modified scenario
7. Review file updated with ONLY the changed scenario (status reset to `draft`, Round incremented)
8. After acceptance via `/qa/review`, run `/qa/build {component}` to regenerate the affected script

Multiple TC-IDs: `--edit TC-CART-008,TC-CART-012`

## Output

- `test-cases/{component}.md` — structured scenarios with review status `draft`
- `test-cases/{component}.review.md` — review file for analyst approval
- Coverage gap analysis presented to analyst

## What Happens Next

The analyst reviews scenarios in the `.review.md` file, then runs `/qa/review {component}` to process their decisions. Only accepted scenarios proceed to `/qa/build`.

## Review File Lifecycle

The review file is an edit-in-place artifact — the analyst always edits the same file at the same location. No renaming or manual versioning is needed. The system manages review rounds automatically via the `Round` field in the review file header. See the `review-management` skill for details.
