---
name: app-analysis
description: How to analyze a web application to identify tech stack, routes, components, API contracts, and selector strategy. Supports two data sources — source code (default) or URL crawl results. Use when onboarding a new app or re-analyzing after major changes.
user-invocable: false
---

## Instructions

### Data Source

Before starting, determine the analysis mode:

- **Source-code mode:** The analyst has the app's source code in a local folder. Read files directly.
- **URL mode:** The analyst provided a live URL. The `url-crawler` skill has already run and produced `config/crawl-results.json`. Read that file instead of source code.

The app-profile.md you produce must include a `## Data Source` section at the top indicating which mode was used.

---

### Step 1: Identify the Tech Stack

**Source-code mode:**
Read `package.json`, framework config files, and entry points to determine:
- Frontend framework: React, Angular, Vue, Next.js, Svelte, etc.
- Language: TypeScript or JavaScript
- CSS approach: Tailwind, CSS modules, styled-components, SCSS
- State management: Redux, Zustand, Context API, etc.
- Routing: React Router, Next.js pages/app router, Angular Router
- Build tool: Vite, Webpack, Turbopack, etc.

**URL mode:**
Read `crawl-results.json` → `techStack` section:
- Framework detected from script sources, meta tags, and window globals
- CSS framework detected from class name patterns and stylesheet links
- Language is inferred (mark as "TypeScript (inferred)" or "unknown")
- State management and build tool are **unknown** — add to Known Unknowns

### Step 2: Map the Route Structure

**Source-code mode:**
Read the routing configuration to build a page map:
- For Next.js: read `app/` or `pages/` directory structure
- For React Router: read route definitions in the router config
- For Angular: read `app-routing.module.ts`

Produce a list of all routes with their associated components.

**URL mode:**
Read `crawl-results.json` → `routes` array:
- List all discovered routes with titles and status codes
- Generalize parameterized routes: if you see `/product/123` and `/product/456`, note the pattern as `/product/:id`
- Note: only routes reachable via links from the base URL are discovered. Dynamic routes or routes behind specific user flows may be missing. Add to Known Unknowns.

### Step 3: Identify Interactive Elements

**Source-code mode:**
For each page/component, identify:
- Forms and their fields (input types, validation rules, required fields)
- Buttons and their actions (submit, navigate, toggle, delete)
- Navigation elements (menus, links, breadcrumbs)
- Dynamic content (lists, tables, cards that load from APIs)
- Modals, drawers, and overlays
- Drag-and-drop interactions

**URL mode:**
Read `crawl-results.json` → `pages` object (per-page DOM inspection):
- Forms: fields, types, required attributes, testids, ARIA labels
- Buttons: text content, type, testids
- Links: navigation targets and text
- Note: elements hidden behind interactions (accordion expand, tab switch, modal triggers) may not have been captured during the crawl. Flag these as potential gaps.

### Step 4: Map API Contracts

**Source-code mode:**
Read API call patterns to identify:
- Endpoints used by the frontend (URLs, methods, request/response shapes)
- Authentication headers and token management
- Error handling patterns (how the UI responds to 4xx/5xx)

**URL mode:**
Read `crawl-results.json` → `apiEndpoints` array:
- Endpoints observed during page loads (URL, method, status)
- Note: only endpoints called during initial page loads are captured. Endpoints triggered by user actions (form submissions, button clicks) may be missing.
- Authentication patterns can be inferred from observed headers but request/response shapes are not captured. Add to Known Unknowns.

### Step 5: Assess Selector Strategy

**Source-code mode:**
Examine the codebase for testability:
- Does the app use `data-testid` attributes? Document the pattern.
- If not, identify the most stable selector strategy available (ARIA roles, semantic HTML, unique classes)
- Flag components that will be difficult to select (dynamic IDs, generic class names)

**URL mode:**
Read `crawl-results.json` → `selectorAudit` section:
- `testIdCoverage` percentage — how many elements have `data-testid`
- `testIdPattern` — naming convention (kebab-case, camelCase, etc.)
- `sampleTestIds` — examples of actual testid values
- If coverage is low (<20%), note that tests will rely more on ARIA roles and text selectors, which are less stable. Flag this in the app-profile.md.

### Step 6: Produce the app-profile.md

Generate the app profile document with all findings.

**Both modes include:**
- Tech stack summary
- Route map
- Interactive elements per page
- API endpoints
- Selector strategy and conventions
- Current version (if detectable)
- Environment details (filled in by analyst)
- Data seeding approach (filled in by analyst)

**URL mode additionally includes:**

```markdown
## Data Source
URL crawl — `{baseURL}`
Crawl date: {date} | Pages discovered: {N} | Depth: {N}

## Known Unknowns
Items the crawl could not determine. Ask the analyst to fill these in:
- [ ] State management approach
- [ ] Build tooling
- [ ] Internal component boundaries
- [ ] Validation rules (code-level)
- [ ] Error handling patterns not triggered during crawl
- [ ] Routes requiring specific state to access
- [ ] {any other gaps identified during analysis}

## Crawl Coverage
- Pages discovered: {N} of estimated {N}
- Selector audit: {testIdCoverage}% data-testid coverage
- API endpoints observed: {N} (page-load only; action-triggered endpoints may be missing)
- Authentication: {public only / authenticated crawl completed}

## Selector Audit
| Metric | Count |
|---|---|
| Total elements inspected | {N} |
| With data-testid | {N} ({%}) |
| With ARIA role | {N} |
| With unique ID | {N} |
| Naming pattern | {pattern} |
```
