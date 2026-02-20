# Projects

This folder contains one subfolder per application you are testing.

## Getting Started

Run the onboard command to set up your first app:

```
/ryu-webui-testing:onboard https://your-app.com
```

The team will create a project folder here automatically with everything needed to start testing.

## Structure

Each project folder follows this layout:

```
projects/
└── your-app-name/
    ├── app-profile.md        ← App profile, tech stack, conventions
    ├── config/               ← Playwright config and crawler
    ├── pages/                ← Page Object Models
    ├── fixtures/             ← Test data
    ├── helpers/              ← Shared utilities
    ├── preflight/            ← Environment checks
    ├── test-cases/           ← Designed scenarios
    ├── versions/             ← Test scripts per version
    └── reports/              ← Generated reports
```
