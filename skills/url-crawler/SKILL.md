---
name: url-crawler
description: How to generate and execute a Playwright Node.js crawler script that discovers routes, interactive elements, API endpoints, tech stack, and selector patterns from a live URL. Use during URL-mode onboarding when source code is not available.
user-invocable: false
---

## Instructions

### Overview

When onboarding an app via URL (no source code), you generate a self-contained Node.js script that uses Playwright to crawl the live application. The script does all the browser work; you analyze its structured JSON output.

**Two-phase approach:**
1. Generate `config/crawler.js` — a Playwright-based crawler script
2. Run it via `node config/crawler.js` and read `config/crawl-results.json`

### Step 1: Generate the Crawler Script

Write `config/crawler.js` — a self-contained Node.js script that uses Playwright. The script must:

#### Discovery Crawl

1. Launch a headless Chromium browser via `playwright.chromium.launch()`
2. Navigate to the provided `baseURL`
3. Discover routes by:
   - Collecting all `<a href>` links that are same-origin
   - Prioritizing links from `<nav>`, `<header>`, and `<footer>` elements
   - Deduplicating URLs (normalize trailing slashes, ignore query params and hash fragments)
   - Following discovered links up to `MAX_DEPTH` (default: 3) and `MAX_PAGES` (default: 50)
4. For each discovered page, record: URL path, page title, HTTP status code, redirect chain

#### Page Inspection (per discovered page)

For each page, wait for `networkidle` then inspect the DOM:

**Interactive Elements:**
- Forms: `<form>` elements with `action`, `method`, and all child inputs (`name`, `type`, `placeholder`, `required`, `aria-label`, `data-testid`)
- Buttons: `<button>` and `[role="button"]` with text content, type, and attributes
- Links: internal navigation links with text and target
- Custom interactives: elements with `onclick`, `role="tab"`, `role="dialog"` triggers

**Selector Audit:**
- Count elements with `data-testid` attributes (total and per-page)
- Count elements with ARIA roles
- Count elements with unique IDs
- Sample `data-testid` values to detect naming patterns (kebab-case, camelCase, etc.)
- Calculate `data-testid` coverage percentage

**Network Requests (via request interception):**
- Capture all XHR/Fetch requests made during page load
- For each: URL, method, response status
- Group by base path to identify API endpoint patterns

**Tech Stack Detection:**
- Check `<meta>` tags for generator/framework hints
- Check `<script src>` for framework bundles (react, vue, angular, next, nuxt, svelte)
- Check for known CSS framework patterns (Tailwind utility classes, Bootstrap, Material UI)
- Check window globals: `__NEXT_DATA__`, `__NUXT__`, `__REMIX_CONTEXT__`
- Check response headers for framework signatures (`x-powered-by`)

**Page Structure:**
- Heading hierarchy (`h1` through `h6`)
- Content landmarks (`<main>`, `<aside>`, `<section>`)
- Modal/overlay detection (`role="dialog"`, common modal class patterns)

### Step 2: Crawler Configuration

The generated script reads configuration from environment variables or a config object at the top:

```javascript
const CONFIG = {
  BASE_URL: process.env.CRAWL_BASE_URL || '<provided-url>',
  MAX_DEPTH: parseInt(process.env.CRAWL_MAX_DEPTH || '3'),
  MAX_PAGES: parseInt(process.env.CRAWL_MAX_PAGES || '50'),
  TIMEOUT: parseInt(process.env.CRAWL_TIMEOUT || '30000'),
  AUTH_COOKIE: process.env.CRAWL_AUTH_COOKIE || null,
  VIEWPORT: { width: 1280, height: 720 },
  DELAY_BETWEEN_PAGES: 500, // ms, to avoid rate limiting
};
```

### Step 3: Output Schema

The script writes `config/crawl-results.json` with this structure:

```json
{
  "metadata": {
    "baseURL": "https://example.com",
    "crawlDate": "2026-02-16T10:30:00Z",
    "pagesDiscovered": 12,
    "maxDepthReached": 2,
    "crawlDurationMs": 4500
  },
  "techStack": {
    "framework": "Next.js",
    "language": "TypeScript (inferred)",
    "cssFramework": "Tailwind CSS",
    "detectedGlobals": ["__NEXT_DATA__"],
    "detectedScripts": ["/_next/static/chunks/..."],
    "serverHeaders": {}
  },
  "routes": [
    {
      "path": "/",
      "title": "Home - MyApp",
      "status": 200,
      "redirectedFrom": null
    }
  ],
  "pages": {
    "/login": {
      "forms": [
        {
          "action": "/api/auth/login",
          "method": "POST",
          "fields": [
            { "name": "email", "type": "email", "required": true, "testid": "email-input", "ariaLabel": "Email address" }
          ]
        }
      ],
      "buttons": [
        { "text": "Sign In", "type": "submit", "testid": "submit-btn" }
      ],
      "links": [
        { "text": "Create Account", "href": "/register" }
      ],
      "headings": ["h1: Welcome Back"],
      "testidCount": 3,
      "ariaRoleCount": 5
    }
  },
  "apiEndpoints": [
    { "url": "/api/auth/login", "method": "POST", "observedStatus": 401 }
  ],
  "selectorAudit": {
    "totalElements": 342,
    "withTestId": 28,
    "withAriaRole": 65,
    "withUniqueId": 41,
    "testIdCoverage": "8.2%",
    "testIdPattern": "kebab-case",
    "sampleTestIds": ["email-input", "password-input", "submit-btn"]
  }
}
```

### Step 4: Run the Crawler

Execute the script:

```bash
npx playwright install chromium  # ensure browser is installed
node config/crawler.js
```

If the script fails, check:
- Is the URL reachable? (try `curl -I <baseURL>`)
- Is Playwright installed? (`npx playwright install chromium`)
- Is the app behind authentication? (see authenticated crawl below)

### Step 5: Handling Authentication

If the app requires login to access most routes:

**Pass 1 — Unauthenticated crawl:**
- Discovers the login page and public routes
- Captures the login form structure (fields, action endpoint)
- All routes tagged as `"access": "public"`

**Pass 2 — Authenticated crawl (after analyst provides credentials):**
- Generate an updated `config/crawler.js` that:
  1. Navigates to the login page
  2. Fills the login form with provided test credentials
  3. Submits and waits for redirect
  4. Captures the session cookie/token
  5. Crawls all routes with the authenticated session
- New routes tagged as `"access": "authenticated"`
- Merge with Pass 1 results — public routes keep their data, authenticated routes are added

### Step 6: Handling SPAs

Single-page applications render content dynamically. The crawler handles this:

- Use `page.waitForLoadState('networkidle')` on every page
- Wait an additional 2 seconds for client-side rendering to complete
- Execute `document.querySelectorAll('a[href]')` after JS has run (not just parse static HTML)
- For hash-based routing (`/#/page`), treat hash segments as distinct routes
- For client-side routers, check `window.history` state changes

### Error Handling

- **URL unreachable:** Stop immediately, report the HTTP error
- **Consistent 403/401:** Suggest authentication is needed, ask for credentials
- **Crawl exceeds `MAX_PAGES`:** Stop crawling, report what was found, note the app may be larger
- **Playwright not installed:** Report error, suggest `npx playwright install chromium`
- **Rate limiting (429):** Increase `DELAY_BETWEEN_PAGES`, retry

### Limitations

URL-mode crawling is less precise than source code analysis. Document these as "Known Unknowns" in the generated CLAUDE.md:

- **Cannot determine:** Internal state management, build tooling, exact TypeScript vs JavaScript, component boundaries, code-level validation rules, error handling patterns not triggered during crawl
- **May miss:** Elements behind user interactions (accordion expand, tab switch, modal trigger), routes requiring specific state, dynamically generated routes
- **Can determine:** Visible routes, form structure, observed API endpoints, tech stack from signatures, selector availability and patterns, page structure and headings
