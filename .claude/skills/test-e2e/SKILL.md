---
name: test-e2e
description: End-to-End Testing Guide. Covers e2e testing strategies, tool selection, and implementation for frontend applications. Use when setting up or writing e2e tests.
version: 1.0.0
---

# E2E Testing Skill

Skill for creating and maintaining end-to-end tests for frontend applications.

## Purpose

E2E tests verify that your application works as expected from the user's perspective:
- Test complete user flows
- Verify critical paths
- Catch regressions early
- Ensure cross-browser compatibility

## When to Use

- Setting up e2e testing for a new project
- Writing tests for user flows
- Debugging failing e2e tests
- Choosing e2e testing tools
- Improving test coverage

---

## E2E vs Other Tests

| Test Type | Scope | Speed | Purpose |
|-----------|-------|-------|---------|
| **Unit** | Single function/component | Fast | Verify logic |
| **Integration** | Multiple components | Medium | Verify interactions |
| **E2E** | Full application | Slow | Verify user flows |

### When to Use E2E

- User-critical flows (login, checkout, signup)
- Complex interactions
- Cross-browser testing
- Critical path verification

---

## Tool Selection

### Popular E2E Tools

| Tool | Best For | Language | Key Features |
|------|----------|----------|--------------|
| **Playwright** | Modern web apps | TypeScript | Auto-wait, tracing, multi-browser |
| **Cypress** | React/Vue apps | JavaScript | Time-travel debugging, dashboard |
| **Puppeteer** | Chrome automation | JavaScript | Fast, headless |
| **Selenium** | Cross-browser | Multiple | Wide browser support |
| **TestCafe** | Cross-browser | JavaScript | No setup, cloud support |

### Recommendation

- **Modern projects**: Playwright (recommended)
- **React projects**: Cypress or Playwright
- **Cross-browser**: Playwright or Selenium
- **Quick scripts**: Puppeteer

---

## Playwright Guide

### Setup

```bash
# Install Playwright
npm init playwright@latest

# Or add to existing project
npm install -D @playwright/test

# Install browsers
npx playwright install chromium
```

### Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Writing Tests

```typescript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test('should login successfully', async ({ page }) => {
    await page.goto('/login');

    // Fill form
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');

    // Submit
    await page.click('[data-testid="login-button"]');

    // Verify redirect
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="welcome"]')).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[data-testid="email"]', 'invalid@example.com');
    await page.fill('[data-testid="password"]', 'wrong');
    await page.click('[data-testid="login-button"]');

    await expect(page.locator('[data-testid="error"]')).toContainText('Invalid credentials');
  });
});
```

### Page Object Pattern

```typescript
// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email"]');
    this.passwordInput = page.locator('[data-testid="password"]');
    this.loginButton = page.locator('[data-testid="login-button"]');
    this.errorMessage = page.locator('[data-testid="error"]');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async goto() {
    await this.page.goto('/login');
  }
}

// tests/login.spec.ts
import { test } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test('login flow', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');
  // ...
});
```

---

## Cypress Guide

### Setup

```bash
# Install Cypress
npm install -D cypress

# Open Cypress
npx cypress open
```

### Configuration

```javascript
// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: false,
    screenshotOnRunFailure: true,
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
  },
});
```

### Writing Tests

```javascript
// cypress/e2e/login.cy.js
describe('Login Flow', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('should login successfully', () => {
    cy.get('[data-testid="email"]').type('user@example.com');
    cy.get('[data-testid="password"]').type('password123');
    cy.get('[data-testid="login-button"]').click();

    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="welcome"]').should('be.visible');
  });

  it('should show error for invalid credentials', () => {
    cy.get('[data-testid="email"]').type('invalid@example.com');
    cy.get('[data-testid="password"]').type('wrong');
    cy.get('[data-testid="login-button"]').click();

    cy.get('[data-testid="error"]').should('contain', 'Invalid credentials');
  });
});
```

### Page Object Pattern

```javascript
// cypress/support/pages/LoginPage.js
class LoginPage {
  visit() {
    cy.visit('/login');
  }

  fillEmail(email) {
    cy.get('[data-testid="email"]').type(email);
  }

  fillPassword(password) {
    cy.get('[data-testid="password"]').type(password);
  }

  submit() {
    cy.get('[data-testid="login-button"]').click();
  }

  login(email, password) {
    this.fillEmail(email);
    this.fillPassword(password);
    this.submit();
  }
}

export const loginPage = new LoginPage();

// Usage in tests
import { loginPage } from '../support/pages/LoginPage';

it('should login', () => {
  loginPage.visit();
  loginPage.login('user@example.com', 'password');
});
```

---

## Best Practices

### Test Organization

```bash
tests/
├── e2e/
│   ├── login/
│   │   ├── login.spec.ts
│   │   └── login-page.ts
│   ├── checkout/
│   │   ├── checkout.spec.ts
│   │   └── checkout-page.ts
│   └── common/
│       ├── navigation.ts
│       └── assertions.ts
```

### Use Data Test IDs

```tsx
// Good: Use data-testid
<button data-testid="submit-button">Submit</button>

// Bad: Use fragile selectors
<button class="btn-primary mt-4">Submit</button>
```

### Auto-Wait

```typescript
// Playwright automatically waits for elements
await page.click('button'); // Waits for button to be visible

// Cypress automatically waits for assertions
cy.get('button').click(); // Waits for button to be actionable
```

### Test Isolation

```typescript
// Each test should be independent
test('test 1', async ({ page }) => {
  // Setup: Create test data
  await createTestUser();

  // Test
  await page.goto('/profile');

  // No cleanup needed - use test accounts or teardown
});
```

### Handle Dynamic Content

```typescript
// Wait for dynamic content
await page.waitForSelector('[data-testid="loaded-content"]');

// Wait for network idle
await page.waitForLoadState('networkidle');

// Wait for API response
await page.waitForResponse(response => response.url().includes('/api/data'));
```

---

## Test Coverage Strategy

### Priority 1: Critical Paths

- Login/Authentication
- Payment/Checkout
- Registration
- Password reset

### Priority 2: Common User Flows

- Search and filter
- CRUD operations
- Form submissions
- Navigation

### Priority 3: Edge Cases

- Error states
- Empty states
- Loading states
- Permission denied

---

## CI/CD Integration

### GitHub Actions with Playwright

```yaml
name: E2E Tests

on: [push, pull_request]

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npx playwright test
```

### Cypress Dashboard

```yaml
# .github/workflows/cypress.yml
name: Cypress E2E

on: [push, pull_request]

jobs:
  cypress:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: cypress-io/github-action@v5
        with:
          build: npm run build
          start: npm start
```

---

## Debugging

### Playwright

```bash
# Open UI Mode
npx playwright test --ui

# Show traces
npx playwright show-trace trace.zip

# Debug with breakpoints
npx playwright test --debug
```

### Cypress

```bash
# Open Cypress UI
npx cypress open

# Run with video
npx cypress run --spec "path/to/spec.cy.js"
```

---

## Integration with Other Skills

When writing e2e tests, combine with:

- `frontend-framework-nextjs` - For Next.js specific patterns
- `frontend-state-management` - For testing state changes
- `frontend-styling-twind` - For testing visual elements
- `frontend-api-integration` - For mocking API calls

---

## Self-Check / Validation

### Validation Commands

```bash
# Playwright
npx playwright test                    # Run all tests
npx playwright test --ui               # UI mode
npx playwright show-report             # Show HTML report

# Cypress
npx cypress run                        # Run all tests
npx cypress open                       # Open UI
```

### Validation Checklist

- [ ] Tests cover critical user flows
- [ ] Tests use stable selectors (data-testid)
- [ ] Tests are isolated and can run in parallel
- [ ] Tests handle loading states properly
- [ ] Tests have meaningful assertions
- [ ] Page objects used for complex pages
- [ ] Tests run in CI/CD pipeline

### Test Quality Guidelines

| Check | Standard |
|-------|----------|
| Selector stability | Use data-testid |
| Assertions | Specific, meaningful |
| Test isolation | No dependencies between tests |
| Naming | Descriptive, describes scenario |
| Coverage | Critical paths covered |
