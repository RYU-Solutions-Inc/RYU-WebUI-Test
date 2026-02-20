---
name: requirements-analysis
description: How to read requirements from ticketing systems (Jira, GitHub Issues), determine sprint vs. release scope, cross-reference with code changes and URL crawl results, and feed findings into impact analysis and test design. Use when a ticketing system is connected via .mcp.json.
user-invocable: false
---

## Instructions

### Overview

Requirements from a ticketing system are the third input source for test design, alongside URL crawl results and source code changes. Requirements provide the **business intent** — what SHOULD change and WHY.

Three input sources work together:
| Source | Provides | Available When |
|--------|----------|---------------|
| URL crawl | Live app structure, what ACTUALLY exists | Always (just need a URL) |
| Source code (git diff) | Code-level changes, WHAT changed | Source code in project folder |
| Requirements (tickets) | Business intent, what SHOULD change, acceptance criteria | Ticketing system connected via `.mcp.json` |

### Step 1: Detect Available Requirements Source

Check `.mcp.json` for configured ticketing connectors:
- **Jira:** Look for Jira MCP connector → access boards, sprints, tickets
- **GitHub Issues:** Look for GitHub MCP connector → access issues, milestones, labels
- **None connected:** Inform analyst: "No ticketing system connected. Add one via `.mcp.json` for requirements traceability."

### Step 2: Determine Sprint vs. Release Scope

**If Jira is connected:**
1. Read the ticket's `fixVersion` field → this is the release
2. Read the ticket's `sprint` field → this is the sprint
3. Compare against the manifest's `release` and `sprint` fields:
   - If `fixVersion` changed → **new release** (broader impact analysis)
   - If only `sprint` changed → **sprint within release** (incremental changes)
4. Store in manifest: `"releaseType": "sprint"` or `"releaseType": "new-release"`

**If GitHub Issues:**
1. Read milestone labels → milestones map to releases
2. Read issue labels for sprint tags (if used)

**If no ticketing system:**
1. Ask the analyst: "Is this a sprint update within the current release, or a new release?"
2. Store the answer in the manifest

### Step 3: Read Sprint/Release Tickets

Pull tickets for the current scope. For each ticket, extract:
- Ticket ID (for REQ-ID mapping in scenarios)
- Title and description (feature intent)
- Acceptance criteria (become test validation points)
- Type: feature, bug fix, enhancement, technical debt
- Priority: critical, high, medium, low
- Affected components (if tagged)
- Status: in progress, done, in review

**Organize by testing impact:**
| Ticket Type | Testing Impact |
|-------------|---------------|
| New Feature | NEW scenarios needed — design from acceptance criteria |
| Bug Fix | Specific regression test needed — verify the fix |
| Enhancement | Existing scenarios may need updating |
| Technical Debt | Check if refactoring changed behavior |

### Step 4: Cross-Reference with Code Changes

When both requirements AND source code are available:
1. **Requirements with code changes:** Normal — map requirement to changed files for traceability
2. **Requirements without code changes:** Flag as "potentially unimplemented"
3. **Code changes without requirements:** Flag as "unplanned change"

### Step 5: Cross-Reference with URL Re-Crawl

When both requirements AND URL crawl are available:
1. **Requirements reflected in crawl:** Good — UI matches requirements
2. **Requirements NOT reflected in crawl:** Flag — requirement may not be deployed or may be behind feature flag
3. **Crawl changes without requirements:** Flag — undocumented UI change

### Step 6: Feed into Impact Analysis

Produce a unified impact table combining all sources:

```markdown
## Unified Impact

| Item | Source | Category | Tickets | Changed Files |
|------|--------|----------|---------|---------------|
| User registration | Jira + git diff | NEW | PROJ-123 | register.tsx, auth.ts |
| Cart calculation | Jira + git diff | MUST UPDATE | PROJ-456 | cart.tsx |
| Password reset | Jira only, no code | UNIMPLEMENTED? | PROJ-789 | — |
| Footer update | git diff only, no ticket | UNPLANNED | — | footer.tsx |

## Gaps Detected
- PROJ-789 has no corresponding code changes
- Footer update has no ticket
```

### Step 7: Feed into Test Design

When the Scenario Designer designs tests:
- **Acceptance criteria** → become validation points in scenarios
- **Ticket IDs** → become REQ-IDs in scenario metadata (traceability)
- **Priority from tickets** → informs scenario priority (P0/P1/P2)
- **Bug fix descriptions** → become specific regression test scenarios
