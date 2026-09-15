---
name: verdimillious
description: "Verdimillious: casts a green light as the all-clear signal. Toolkit for interacting with and testing local web applications using Playwright — verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and reading browser console logs. Use when the user wants to test a running web app in a real browser, verify a UI change actually renders and works, or debug frontend behaviour that's hard to reason about from code alone."
license: Apache-2.0
---

# Verdimillious

Browser-level verification: does the app actually work when a real browser drives it, not just does the code compile. This sits above `arresto-momentum` (unit-level TDD) — use this for what a unit test can't see: rendering, navigation, real network responses, console errors.

Adapted from Anthropic's `webapp-testing` (anthropics/skills, Apache-2.0). Changes from the original: locators now default to accessible queries instead of raw CSS/id selectors, the base URL is read from config instead of hardcoded, console errors are treated as failures by default, and a workflow section covers what the original left as bare syntax.

## Before testing

1. Confirm the dev server is actually running and reachable — don't assume; check or start it.
2. Read the base URL from `playwright.config.ts` (`use: { baseURL: ... }`), never hardcode a port in the test itself:

```typescript
// playwright.config.ts
export default defineConfig({
  use: { baseURL: process.env.BASE_URL ?? 'http://localhost:3000' },
});

// test file
await page.goto('/'); // resolves against baseURL
```

3. If setup hasn't run yet:
```bash
npm init playwright@latest
```

## Locators — prefer accessible queries over CSS/id selectors

A selector coupled to markup breaks on a restyle even when the behaviour is unchanged — the same failure mode TDD's tests avoid at the unit level. Default to how a user or screen reader finds the element:

```typescript
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByLabel('Email').fill('test@example.com');
await page.getByText('Welcome').isVisible();
```

Fall back to `data-testid` only when no accessible role or text distinguishes the element. Avoid raw class/id selectors (`.submit-btn`, `#modal`) — they're the first thing to break on a restyle, including a `weaving`-driven tlglobal restyle.

## Basic test structure

```typescript
import { test, expect } from '@playwright/test';

test('homepage has title', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveTitle(/My App/);
});

test('can navigate to about page', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('link', { name: 'About' }).click();
  await expect(page).toHaveURL(/.*about/);
});
```

## Common actions

### Navigation
```typescript
await page.goto('/dashboard');
await page.goBack();
await page.reload();
```

### Form input
```typescript
await page.getByLabel('Email').fill('test@example.com');
await page.getByLabel('Password').fill('secret123');
await page.getByRole('combobox', { name: 'Country' }).selectOption('USA');
await page.getByRole('checkbox').check();
```

### Waiting
```typescript
await page.waitForURL('**/dashboard');
await page.waitForResponse('**/api/data');
// Avoid page.waitForTimeout() — it's a fixed delay, not a real signal something is ready.
```

## Assertions

```typescript
await expect(page.getByRole('heading')).toHaveText('Welcome');
await expect(page.getByRole('listitem')).toHaveCount(5);
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
await expect(page.getByRole('dialog')).toBeVisible();
```

## Console errors are failures, not noise

Attach this in every test file (or a shared fixture) and fail loudly on unexpected errors — a silent console error is exactly the kind of bug a passing assertion can miss:

```typescript
test.beforeEach(({ page }) => {
  page.on('pageerror', err => { throw err; });
  page.on('console', msg => {
    if (msg.type() === 'error') throw new Error(`Console error: ${msg.text()}`);
  });
});
```

If a specific error is expected and safe (a known third-party warning), filter it explicitly rather than removing this check.

## Screenshots — use for visual state, not as a substitute for assertions

A screenshot shows you a moment; an assertion states what must be true every time. Use assertions first, and add a screenshot only when the thing under test is inherently visual (layout, a chart render, a generated slide):

```typescript
await page.screenshot({ path: 'screenshot.png', fullPage: true });
await page.locator('.chart').screenshot({ path: 'chart.png' });
```

## Network interception

Mock only what's external or slow, the same boundary rule `arresto-momentum` uses for mocking in unit tests:

```typescript
await page.route('**/api/data', route => {
  route.fulfill({ status: 200, body: JSON.stringify({ items: [] }) });
});
```

## Testing sandboxed or user-generated content

If the page under test renders untrusted content — e.g. a TuringLab prototype slide executing generated HTML/CSS/JS — treat it as testing a boundary, not your own UI:
- Assert the sandbox's own outer chrome (load state, error boundary, container), not the arbitrary content inside it.
- Don't write assertions that assume specific content the generator produced — that content is expected to vary.
- Route/mock any network calls the sandboxed content might attempt; don't let a test make real external requests on your behalf.

## Running tests

```bash
npx playwright test
npx playwright test tests/login.spec.ts
npx playwright test --headed
npx playwright test --ui
```

## Stopping condition

Done when every seam agreed with the user is covered by a passing assertion (not just a screenshot), console errors are checked and none are unexplained, and the test passes against the real dev server, not a mock of your own app's routes.
