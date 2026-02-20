---
description: Analyze a new application and set up the project structure for testing (URL-first)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
argument-hint: "[<app-URL> | --source]"
---

Onboard a new application for E2E testing.

This is direct Architect work — no delegation needed.

## Getting Started — Guided Setup

Parse $ARGUMENTS to determine mode:

**If a URL was provided** (argument starts with `http`):
→ Proceed directly to URL mode with that URL

**If `--source` flag was provided:**
→ Proceed directly to source-code mode

**If no arguments provided:**
→ Present the guided setup menu:

```
Welcome! Let's set up testing for your application.

How would you like to get started?

1. Paste a URL to your app (recommended — fastest way to start)
2. I have source code in this folder
3. I have both — URL + source code

Tip: You can always enrich your analysis later by:
• Adding source code for deeper analysis (component boundaries, validation rules)
• Connecting Jira or GitHub via .mcp.json for requirements traceability
```

**Option 1 (URL):** Ask for the URL, proceed to URL mode
**Option 2 (Source):** Proceed to source-code mode
**Option 3 (Both):** Ask for the URL first, run URL mode, then enrich with source code analysis

---

## Steps (URL Mode — Default)

1. Validate the provided URL is reachable: `curl -sI <baseURL>` — expect a 2xx or 3xx response
2. If the URL is unreachable, report the error and stop
3. Ensure Playwright is available: `npx playwright install chromium`
4. Load the `url-crawler` skill
5. Generate the crawler script at `config/crawler.js` following the skill's specification
6. Run the crawler: `node config/crawler.js`
7. Read the output from `config/crawl-results.json`
8. Load the `app-analysis` skill (URL mode)
9. Follow the app-analysis URL-mode methodology using crawl results
10. Generate `app-profile.md` with all findings, including:
    - `Data Source` section (URL crawl)
    - `Known Unknowns` section (what the crawl couldn't determine)
    - `Crawl Coverage` section (pages found, depth, selector audit)
11. Determine the project name:
    - From URL: extract domain name (e.g., `my-app.example.com` → `my-app`)
    - Ask the analyst to confirm or provide a custom name
12. Create the project folder structure under `projects/{project-name}/`:
    - `versions/`, `pages/`, `fixtures/`, `helpers/`, `config/`, `ci-cd/`, `test-cases/`, `reports/`, `preflight/`
13. Write `.active-project` at the repository root with the project name
14. Generate `projects/{project-name}/config/playwright.config.ts` with `baseURL` set to the provided URL (Chromium desktop only — other browsers can be added later)
15. Generate pre-flight test suite in `projects/{project-name}/preflight/`
16. Present the app-profile.md to the analyst for review
17. Ask the analyst:
    - Does the app require authentication? If yes, provide test credentials for an authenticated re-crawl
    - Are there areas of the app the crawl might have missed?
    - Can you fill in any of the Known Unknowns?
18. If test credentials are provided, regenerate `config/crawler.js` with authentication logic, re-crawl, and update app-profile.md
19. Inform the analyst about enrichment options:
    - "Add source code to this folder for deeper analysis"
    - "Connect Jira/GitHub via `.mcp.json` for requirements traceability"
    - "Starting with Chromium. Add Safari, Firefox, or mobile emulation anytime."
20. Run the lifecycle tour (see `/qa/tour`)

---

## Steps (Source-Code Mode)

1. Read the application source code in the current working directory
2. Load the `app-analysis` skill
3. Follow the app-analysis methodology to identify tech stack, routes, components, APIs, and selector patterns
4. Generate `app-profile.md` with all findings
5. Create the project folder structure (same as URL mode)
6. Generate `config/playwright.config.ts` with Chromium desktop only
7. Generate pre-flight test suite in `preflight/`
8. Present the app-profile.md to the analyst for review and refinement
9. Ask the analyst to fill in details the code didn't reveal (environment URLs, data seeding, credentials)
10. Inform the analyst about enrichment options (URL crawl, ticketing system, more browsers)
11. Run the lifecycle tour (see `/qa/tour`)

---

## Output

- `app-profile.md` — application profile
- `projects/{project-name}/config/playwright.config.ts` — Playwright configuration (Chromium desktop)
- `projects/{project-name}/config/crawler.js` — generated crawler script (URL mode only)
- `projects/{project-name}/config/crawl-results.json` — crawl output data (URL mode only)
- `projects/{project-name}/preflight/*.spec.ts` — pre-flight validation suite
- `.active-project` — active project marker
- Project folder structure created
- Lifecycle tour presented to analyst
