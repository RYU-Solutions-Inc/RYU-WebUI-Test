---
name: review-management
description: How to generate markdown review files for analyst approval, parse completed reviews, and update scenario statuses. Use after test scenarios are designed and before scripts are generated.
user-invocable: false
---

## Instructions

### Overview

Test scenarios must be reviewed and approved by the analyst before scripts can be generated. The review process uses markdown files that the analyst edits in any text editor (VS Code, Notepad++, etc.). This keeps the process simple, portable, and works entirely within the file system.

### Review File Location

For each `test-cases/{component}.md`, generate a corresponding `test-cases/{component}.review.md`.

### Generating a Review File

After the Scenario Designer writes `test-cases/{component}.md`, generate the review file with this format:

```markdown
# Review: {Component Name} Scenarios

**Generated:** {date} | **Component:** {component} | **Total Scenarios:** {N} | **Round:** 1
**File:** `test-cases/{component}.md`

> Open this file in your editor. For each scenario, check ONE box (ACCEPT / REJECT / NEEDS CHANGES).
> Add feedback next to any rejected or needs-changes items.
> Save the file when done, then tell the Architect: "review done" or run `/qa/review {component}`

---

## Quick Actions

- [ ] **ACCEPT ALL** — accept every scenario below (overrides individual selections)

---

## TC-{COMP}-001: {Scenario Title}

**Category:** {category} | **Priority:** {P0/P1/P2} | **Requirement:** {REQ-ID or N/A}
**Flow:** {one-line summary of the user flow}
**Validation Points:** {count} | **Coverage:** {happy_path / edge_case / negative / error_recovery}

- [ ] ACCEPT
- [ ] REJECT
- [ ] NEEDS CHANGES
- **Feedback:** _write here_

---

## TC-{COMP}-002: {Scenario Title}
...

---

## Overall Feedback

_Write any general feedback about coverage gaps, missing scenarios, or priority adjustments here:_


---

## Review Summary (auto-filled by Architect after processing)

| Status | Count |
|---|---|
| Accepted | — |
| Rejected | — |
| Needs Changes | — |
| Not Reviewed | — |
```

### Review File Rules

1. **One checkbox per scenario.** The analyst checks exactly one of ACCEPT / REJECT / NEEDS CHANGES.
2. **ACCEPT ALL** overrides individual selections — if checked, all scenarios are accepted regardless of individual checkboxes.
3. **Feedback is required** for REJECT and NEEDS CHANGES — if feedback is empty, treat as "not reviewed" and ask the analyst to provide feedback.
4. **Overall Feedback** is optional but encouraged for coverage gap observations.
5. **Compact summaries** — each scenario gets a 3-line summary, not the full spec. The full spec stays in `test-cases/{component}.md`.

### Parsing a Completed Review

When the analyst says "review done" or runs `/qa/review`:

1. Read `test-cases/{component}.review.md`
2. Check if **ACCEPT ALL** is checked (`[x]`) — if so, accept everything
3. Otherwise, parse each scenario section:
   - `[x] ACCEPT` → status = `accepted`
   - `[x] REJECT` → status = `rejected`, extract feedback
   - `[x] NEEDS CHANGES` → status = `needs_changes`, extract feedback
   - No box checked → status = `not_reviewed`
4. Feedback is the text after `**Feedback:**` on the same line or next line
5. Read "Overall Feedback" section for general comments

### Processing Review Decisions

After parsing:

1. **Update `test-cases/{component}.md`** — change `Review Status:` for each scenario:
   - `accepted` → `Review Status: accepted`
   - `rejected` → `Review Status: rejected` (add rejection reason as a comment)
   - `needs_changes` → `Review Status: needs_changes` (add feedback as a comment)
   - `not_reviewed` → remains `Review Status: draft`

2. **Update the Review Summary table** at the bottom of the review file with counts.

3. **Report to the analyst:**
   - "X accepted, Y rejected, Z need changes, W not reviewed"
   - For rejected items: list the TC-IDs and summarize the feedback
   - For needs-changes items: list the TC-IDs and the requested changes
   - If there is overall feedback, present it

4. **If any items were rejected or need changes:**
   - Ask the analyst: "Should I have the Scenario Designer revise the flagged scenarios?"
   - If yes, delegate back to Scenario Designer with the specific feedback per TC-ID
   - After revision, generate a new review file for the revised scenarios only

5. **If all items are accepted:**
   - Report: "All scenarios accepted. Ready for script generation via `/qa/build {component}`."

### Review Status Flow

```
draft → [review] → accepted → [build scripts]
                  → rejected → [redesign] → draft → [review again]
                  → needs_changes → [revise] → draft → [review again]
```

### Review Gate Enforcement

The `/qa/build` command MUST check review status before generating scripts:
- Only scenarios with `Review Status: accepted` can be scripted
- If any scenarios are `draft`, `rejected`, or `needs_changes`, the build command warns and lists them
- The analyst can override with `--force` to build despite unreviewed scenarios (for rapid prototyping)

### Single Scenario Re-review

When a previously accepted scenario is edited via `/qa/design {component} --edit {TC-ID}`:

1. The edited scenario's review status is reset to `draft` in `test-cases/{component}.md`
2. A new review file is generated containing ONLY the edited scenario(s), not the entire component
3. The Round number is incremented from the last review round for this component
4. All other scenarios in the component retain their existing review status (accepted, etc.)
5. After the analyst re-accepts, the scenario status returns to `accepted` and `/qa/build` can regenerate its script

This allows surgical edits without re-reviewing the entire component.

### Multi-Round Reviews

Reviews can go through multiple rounds:
- Round 1: Initial design → review → some rejected
- Round 2: Redesigned scenarios → review → all accepted
- Each round appends to the review history, it doesn't overwrite
- The review file is an audit trail of what was approved and when

### Review File Lifecycle

The review file is an **edit-in-place** artifact. The analyst always edits the same file at the same location — no renaming, no version suffixes, no manual versioning.

**How versioning works internally:**

1. **Initial review:** Review file is generated with `**Round:** 1` in the header
2. **Analyst edits:** The analyst checks boxes and adds feedback in the same file
3. **Processing:** The Architect reads the review file, updates `test-cases/{component}.md` with statuses and feedback as comments, and fills in the Review Summary table
4. **Re-review (if needed):** If scenarios were rejected or need changes, the Scenario Designer revises them. A NEW review file replaces the old one with `**Round:** 2`, covering only the revised scenarios
5. **Audit trail:** Previous review decisions are preserved as comments in `test-cases/{component}.md` — the scenario file is the audit trail, not the review file

**Rules:**
- The review file name never changes: always `test-cases/{component}.review.md`
- The analyst never needs to rename, copy, or version the file
- Round numbers are managed automatically by the Architect
- The `.review.md` file represents the CURRENT review round; historical decisions live in the scenario file
