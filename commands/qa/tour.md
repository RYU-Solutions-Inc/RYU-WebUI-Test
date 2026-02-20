---
description: Show the testing lifecycle overview, review workflow, and quick commands
allowed-tools: Read, Glob, Grep
argument-hint: ""
---

Present the testing lifecycle tour to the analyst.

This is direct Architect work — no delegation needed.

## Steps

1. Present the lifecycle overview (see content below)
2. Check for `.version` file — if it exists and is older than current team version, show "What's New" section
3. Ask the analyst if they want an interactive demo
4. If yes, pick the simplest feature from CLAUDE.md and run a mini design → review → build cycle
5. If no, continue to normal operation

## Tour Content

Present the following:

```
## Your Testing Lifecycle

Here's how your AI testing team works. There are 6 phases:

1. ANALYZE  — AI reads code changes + requirements + URL, categorizes impact
2. DESIGN   — AI designs test scenarios → validated by QA Validator → you review ← REVIEW REQUIRED
3. REVIEW   — You accept/reject scenarios in .review.md files ← EDIT OUTSIDE CLI
4. BUILD    — AI generates Playwright scripts → validated by QA Validator
5. RUN      — Playwright executes tests, AI interprets results
6. REPORT   — AI generates traceability + release readiness reports

## What Needs Your Review

After DESIGN, open test-cases/{component}.review.md in your text editor:
• Check ACCEPT, REJECT, or NEEDS CHANGES for each scenario
• Add feedback for any rejections
• Save the file and tell me "review done" or run /qa/review

## Skipping Reviews

Add --force to /qa/build to skip the review gate (rapid prototyping).
Not recommended for production test suites.

## Three Input Sources

Your analysis gets smarter with more context:
• URL crawl — always available, shows what your app looks like live
• Source code — add to this folder for deeper analysis
• Requirements — connect Jira/GitHub via .mcp.json for acceptance criteria

## Browser Configuration

Starting with Chromium desktop. Add more browsers anytime:
• Safari (WebKit), Firefox, mobile emulation, tablet emulation

## Quick Commands

/qa/onboard              → set up testing for a new app
/qa/design {feature}     → design tests for a feature
/qa/review               → process your review decisions
/qa/build {feature}      → generate Playwright scripts
/qa/run --smoke          → quick smoke test
/qa/analyze {version}    → analyze changes for a new version
/qa/full-cycle {version} → run the entire pipeline with checkpoints
/qa/status               → see where things stand
/qa/tour                 → see this guide again
/qa/switch {project}     → switch to a different app project
/qa/catchup              → get up to speed on the current project
```

## Interactive Demo

After the tour, ask:

"Would you like a quick demo? I can design a few test scenarios for one feature so you can experience the full design → review → build workflow hands-on."

If yes → pick the simplest feature from CLAUDE.md, run mini design → review → build cycle.
If no → done.
