---
description: Process a completed review file and update scenario statuses
allowed-tools: Read, Write, Edit, Glob, Grep
argument-hint: "[component-name]"
---

Process the completed review for: $ARGUMENTS

This is direct Architect work using the `review-management` skill.

## Steps

1. Load `review-management` skill
2. If a component name was provided, read `test-cases/{component}.review.md`
3. If no component name was provided, scan `test-cases/` for any `.review.md` files with unchecked items — process all of them
4. Parse the review file:
   - Check if **ACCEPT ALL** is selected
   - Otherwise, parse each scenario's checkbox (ACCEPT / REJECT / NEEDS CHANGES)
   - Extract feedback for rejected and needs-changes items
   - Read the Overall Feedback section
5. Update `test-cases/{component}.md` — change `Review Status:` for each scenario based on the review decisions
6. Update the Review Summary table at the bottom of the review file
7. Present results to the analyst:
   - Count of accepted, rejected, needs-changes, not-reviewed
   - List rejected items with feedback
   - List needs-changes items with feedback
   - Overall feedback if provided
8. If any items were rejected or need changes:
   - Ask: "Should I have the Scenario Designer revise the flagged scenarios?"
   - If yes, delegate to Scenario Designer with the specific feedback per TC-ID
   - After revision, generate a new review file for the revised scenarios only
9. If all items are accepted:
   - Report: "All scenarios accepted. Ready for `/qa/build {component}`."

## Output

- Updated `test-cases/{component}.md` with review statuses
- Updated `test-cases/{component}.review.md` with summary table
- Inline review summary for the analyst

## Review File Lifecycle

The `.review.md` file is an edit-in-place artifact. After processing, the Architect records review decisions as comments in `test-cases/{component}.md` (the audit trail). If re-review is needed, a new review file replaces the old one with an incremented `Round` number, covering only revised scenarios.
