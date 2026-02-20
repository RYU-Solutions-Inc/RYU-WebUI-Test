---
name: playwright-scripting
description: Playwright test script generation patterns including Page Object Model, selector strategy, fixtures, network interception, tagging, and data seeding. Use when generating or updating Playwright test scripts.
user-invocable: false
---

## Instructions

### Page Object Model (POM)

Every page in the app gets a Page Object in `pages/{page-name}.page.ts`:

```typescript
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  // Declare all locators as readonly properties
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    // Initialize locators using the app's selector strategy
    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.submitButton = page.locator('[data-testid="submit-button"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
  }

  // Navigation methods
  async navigate() {
    await this.page.goto('/login');
    await this.page.waitForLoadState('networkidle');
  }

  // Action methods — map to user actions, not DOM operations
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  // Helper methods — encapsulate common checks
  async getErrorText(): Promise<string | null> {
    return this.errorMessage.textContent();
  }
}
```

### Selector Strategy (Priority Order)

1. `data-testid` attributes — most stable, explicitly for testing
2. ARIA roles and labels — `page.getByRole('button', { name: 'Submit' })`
3. Text content — `page.getByText('Welcome Back')`
4. CSS selectors — last resort, use the most specific stable class

Read the app's CLAUDE.md for its specific selector conventions.

### Test File Structure

```typescript
// Reference TC-IDs for traceability
// TC-AUTH-001: Login with valid credentials
// Requirement: REQ-AUTH-001
// Category: Happy Path
// Review Status: reviewed

import { test, expect } from '@playwright/test';
import { LoginPage } from '../../pages/login.page';
import { validUser } from '../../fixtures/users';
import { resetTestData } from '../../helpers/data-seed';

test.describe('Auth: Login @smoke @p0', () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.navigate();
  });

  test.afterEach(async ({ request }) => {
    await resetTestData(request);
  });

  test('TC-AUTH-001: Login with valid credentials @new', async ({ page }) => {
    const user = validUser();
    await loginPage.login(user.email, user.password);

    // VP: Redirect to dashboard
    await expect(page).toHaveURL(/\/dashboard/);

    // VP: Welcome message displayed
    await expect(page.locator('[data-testid="welcome-msg"]'))
      .toContainText(user.name);

    await page.screenshot({ path: 'screenshots/TC-AUTH-001-success.png' });
  });
});
```

### Fixtures

Fixture factories must return new objects each call. Use dynamic data for uniqueness:

```typescript
// fixtures/users.ts
export const validUser = () => ({
  email: `test-${Date.now()}@example.com`,
  password: 'ValidP@ss123',
  name: 'Test User',
});
```

### Network Interception

For validation points that check API responses:

```typescript
// Wait for a specific API call and validate it
const responsePromise = page.waitForResponse(
  resp => resp.url().includes('/api/coupons') && resp.status() === 200
);
await cartPage.applyCoupon('SAVE10');
const response = await responsePromise;
const body = await response.json();
expect(body.discount).toBe(10);
```

### Error Scenario Testing

For testing error conditions (API failures, timeouts):

```typescript
// Mock an API failure
await page.route('**/api/coupons', route =>
  route.fulfill({ status: 500, body: 'Internal Server Error' })
);
await cartPage.applyCoupon('SAVE10');
await expect(cartPage.errorMessage).toContainText('Something went wrong');
```

### Tagging Convention

| Tag | Meaning |
|---|---|
| `@smoke` | Critical happy path — one per top-level component |
| `@p0` | Highest priority — core business flows |
| `@p1` | High priority — important but has workarounds |
| `@new` | Added in the current version — for filtered runs |

Tags are applied in `test.describe()` titles or individual `test()` titles.

### Configuration

Generate `config/playwright.config.ts` with **Chromium desktop only** as the default:

```typescript
export default defineConfig({
  projects: [
    { name: 'desktop-chrome', use: { ...devices['Desktop Chrome'] } },
  ],
});
```

Additional browsers (Safari/WebKit, Firefox) and device emulation (mobile, tablet) can be added later. The analyst can add more browser projects to the config at any time.

### Data Seeding Integration

Always include setup and teardown hooks:

```typescript
test.beforeEach(async ({ request }) => {
  await seedTestData(request, 'scenario-name');
});

test.afterEach(async ({ request }) => {
  await resetTestData(request);
});
```

Read the app's CLAUDE.md for its specific seeding mechanism (API endpoints, database approach, etc.).

### Annotation-to-Code Patterns

When a scenario has annotations, translate them to Playwright constructs:

**@skip:**
```typescript
test('TC-CART-008: Add item to cart @p0', async ({ page }) => {
  test.skip(true, 'Temporarily disabled — selector refactor in progress');
  // ... test implementation (kept for reference, not executed)
});
```

**@known-flaky:**
```typescript
test.describe('Cart: Add Item', () => {
  test.describe.configure({ retries: 2 });

  test('TC-CART-008: Add item to cart @p0', async ({ page }) => {
    // ... test implementation (retries up to 2 times on failure)
  });
});
```

**@blocked:**
```typescript
test('TC-CART-008: Add item to cart @p0', async ({ page }) => {
  test.skip(true, 'Blocked by JIRA-456 — payment API returns 500');
  // ... test implementation (kept for reference, not executed)
});
```

When annotations are removed, regenerate the script without these constructs.

### Retry Configuration

Configure retries in `config/playwright.config.ts`:

```typescript
export default defineConfig({
  // Global retries — default 0, set higher for flaky environments
  retries: 0,

  projects: [
    { name: 'desktop-chrome', use: { ...devices['Desktop Chrome'] } },
  ],
});
```

Per-test retries for `@known-flaky` tests use `test.describe.configure({ retries: 2 })`. This keeps global retries at 0.

A test that "passed on retry" is a flaky signal — the results-analysis skill tracks this.
