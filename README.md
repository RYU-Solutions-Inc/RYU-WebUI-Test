# RYU WebUI Testing — Agentic AI QA Team

A Claude Code plugin that gives you a full virtual QA team for end-to-end web UI testing with Playwright. One command to onboard any web app, then design, build, and run tests — entirely agent-driven.

---

## What It Does

This plugin installs a 4-agent QA team into Claude Code:

| Agent | Role |
|---|---|
| **E2E Test Architect** | Lead — the only agent you talk to directly |
| **Scenario Designer** | Designs structured test scenarios from requirements or UI crawl |
| **Script Engineer** | Generates Playwright scripts, Page Objects, fixtures, and helpers |
| **QA Validator** | Validates agent outputs before they reach you |
| **QA Reporter** | Produces RTMs, coverage reports, and release readiness assessments |

---

## Prerequisites

- **Claude Code** v1.0.33 or later
- **Node.js** v18 or later
- **npm** (comes with Node.js)

---

## Installation

### Step 1 — Add the marketplace

```
/plugin marketplace add RYU-Solutions-Inc/RYU-WebUI-Test
```

### Step 2 — Install the plugin

```
/plugin install ryu-webui-testing@ryu-webui-testing
```

### Step 3 — Install Playwright (in your project folder)

```bash
npm install
npx playwright install chromium
```

---

## Quick Start

### Onboard a new app (URL mode — no source code needed)

```
/ryu-webui-testing:onboard https://your-app.com
```

The team will crawl the live app, identify its structure, and set up the project scaffold automatically.

### Or onboard from source code

```
/ryu-webui-testing:onboard --source
```

---

## Command Reference

| Command | What It Does |
|---|---|
| `/ryu-webui-testing:onboard [url \| --source]` | Set up a new app for testing |
| `/ryu-webui-testing:analyze` | Run impact analysis when the app changes |
| `/ryu-webui-testing:design {feature}` | Design test scenarios for a feature |
| `/ryu-webui-testing:review {component}` | Review and approve/reject designed scenarios |
| `/ryu-webui-testing:build {feature}` | Generate Playwright scripts from approved scenarios |
| `/ryu-webui-testing:run --smoke` | Run smoke suite (P0 tests only) |
| `/ryu-webui-testing:run --regression` | Run full regression suite |
| `/ryu-webui-testing:run --new` | Run only new tests for the current version |
| `/ryu-webui-testing:run --test {TC-ID}` | Re-run a specific test by ID |
| `/ryu-webui-testing:results` | Interpret the most recent test run |
| `/ryu-webui-testing:report` | Generate RTM, coverage, or release readiness report |
| `/ryu-webui-testing:status` | Show current project state |
| `/ryu-webui-testing:switch {project}` | Switch between app projects |
| `/ryu-webui-testing:full-cycle` | Run the entire pipeline end-to-end |
| `/ryu-webui-testing:tour` | Show the testing lifecycle overview |
| `/ryu-webui-testing:catchup` | Get a full briefing on the current project state |

---

## How Projects Are Organized

Each app you test gets its own folder under `projects/`:

```
projects/
└── my-app/
    ├── app-profile.md        ← App details, tech stack, conventions
    ├── config/               ← Playwright config and crawler script
    ├── pages/                ← Page Object Models
    ├── fixtures/             ← Test data factories
    ├── helpers/              ← Shared utilities
    ├── preflight/            ← Environment validation tests
    ├── test-cases/           ← Scenario markdown files
    ├── versions/             ← Versioned test specs
    │   └── v1.0.0/
    │       ├── manifest.json
    │       └── *.spec.ts
    └── reports/              ← Generated reports
```

Multiple apps are supported — each gets its own isolated folder.

---

## Typical Workflow

```
1. /ryu-webui-testing:onboard https://my-app.com     ← First time setup
2. /ryu-webui-testing:design login                   ← Design scenarios
3. /ryu-webui-testing:review login                   ← Approve/reject
4. /ryu-webui-testing:build login                    ← Generate scripts
5. /ryu-webui-testing:run --smoke                    ← Run smoke tests
6. /ryu-webui-testing:report                         ← Generate report
```

When the app is updated:

```
1. /ryu-webui-testing:analyze                        ← What changed?
2. /ryu-webui-testing:design {new-feature}           ← Design new scenarios
3. /ryu-webui-testing:build {new-feature}            ← Generate new scripts
4. /ryu-webui-testing:run --regression               ← Full regression
5. /ryu-webui-testing:report                         ← Release readiness
```

---

## Multi-App Support

Work across multiple applications by switching projects:

```
/ryu-webui-testing:switch my-other-app
```

Or list all your projects:

```
/ryu-webui-testing:switch
```

---

## Connecting External Tools

The `.mcp.json` file contains example connector configs for Jira, GitHub Issues, and more. Fill in your credentials to enable requirement pulling and ticket traceability.

---

## License

MIT — see [LICENSE](LICENSE) for details.

Built by [RYU Solutions Inc.](https://github.com/RYU-Solutions-Inc)
