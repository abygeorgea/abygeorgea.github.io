---
title: "Supercharging Tests with Native Playwright Fixtures"
date: 2024-03-04T07:30:00+11:00
draft: false
slug: "supercharging-tests-playwright-native-fixtures"
categories:
  - Playwright
tags:
  - Playwright
description: "Using Playwright's test.extend to build custom fixtures for page objects, authenticated sessions, and clean context isolation."
---

In the [previous post](/blog/2024/02/23/decoupling-data-playwright-test-data-management/), we sorted out static and dynamic test data. This time we tackle something that quietly bloats a lot of test suites. Setup code.

If you have written more than a handful of Playwright tests, you have probably written a `beforeEach` block that logs a user in, or sets up a page object, or seeds some starting state. Do that across twenty spec files and you end up with the same boilerplate copied everywhere, and a small change to the login flow means touching every single file.

Playwright's fixture system, through `test.extend`, solves this properly. Think of it as dependency injection for your tests. You describe what a test needs, and Playwright hands it to you already set up.

## Creating a Custom Fixture

A fixture in Playwright is just a function that sets something up, hands it to the test, and optionally cleans up afterward. Let's start by turning our `LoginPage` object into a fixture, so any test can just ask for it directly as a parameter.

```typescript
// fixtures/pages.fixture.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/login.page';

type PageFixtures = {
  loginPage: LoginPage;
};

export const test = base.extend<PageFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },
});

export { expect } from '@playwright/test';
```

Now instead of importing `test` from `@playwright/test` directly in your spec files, you import your own extended version.

```typescript
import { test, expect } from '../fixtures/pages.fixture';

test('user can log in with valid credentials', async ({ loginPage }) => {
  await loginPage.goto();
  await loginPage.login('testuser', 'Password123');
  await expect(loginPage.welcomeBanner).toBeVisible();
});
```

The test never has to construct a `LoginPage` itself. It just declares that it needs one, and Playwright builds it for you before the test body runs.

## Fixtures for Authenticated State

This pattern really shines once you apply it to authentication. Logging in through the UI for every single test that needs to be logged in is slow, and it repeats the same three or four steps constantly. A better approach is to authenticate once, save the browser storage state, and reuse it.

First, set up a small script that logs in and saves the state to a file.

```typescript
// fixtures/auth.setup.ts
import { test as setup } from '@playwright/test';
import { LoginPage } from '../pages/login.page';

const authFile = 'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('testuser', 'Password123');
  await page.context().storageState({ path: authFile });
});
```

Then reference this in `playwright.config.ts` as a dedicated setup project, and have your other projects depend on it.

```typescript
projects: [
  { name: 'setup', testMatch: /auth\.setup\.ts/ },
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome'],
      storageState: 'playwright/.auth/user.json',
    },
    dependencies: ['setup'],
  },
],
```

With this in place, every test in the `chromium` project starts already logged in, because the browser context loads the saved storage state before the test even begins. Login happens exactly once per test run, not once per test.

## Context Isolation, Automatically

One thing worth appreciating here is something Playwright gives you for free. Every test gets its own browser context by default. That means cookies, local storage, and session data from one test never leak into another, even when tests run in parallel in the same worker process. You do not need to manually clear cookies between tests the way you might have in older tools. Playwright's isolation model handles it at the context level, so each test genuinely starts from a clean slate, aside from whatever storage state you explicitly load.

## Global Setup Without the beforeEach Chains

Fixtures also give you a cleaner way to handle setup that used to live in long chains of `beforeEach` and `afterEach` blocks. Say a test needs a freshly created order before it runs, and the order needs to be cleaned up afterward regardless of whether the test passes or fails.

```typescript
// fixtures/order.fixture.ts
import { test as base } from '@playwright/test';
import { createOrderViaApi, deleteOrderViaApi } from '../utils/api-client';

type OrderFixtures = {
  existingOrder: { id: string };
};

export const test = base.extend<OrderFixtures>({
  existingOrder: async ({}, use) => {
    const order = await createOrderViaApi();
    await use(order);
    await deleteOrderViaApi(order.id);
  },
});
```

Everything before `use(order)` is setup. Everything after it is teardown, and it runs even if the test itself fails, because Playwright wraps the fixture lifecycle around the whole test. Compare that to manually remembering to clean up inside an `afterEach`, and hoping nobody forgets when they add a new test to the file.

## Wrapping Up

Fixtures turn setup and teardown into something declarative. A test asks for what it needs, whether that is a page object, an authenticated session, or a piece of freshly created data, and Playwright handles the rest behind the scenes.

Next time, we look at what might be Playwright's biggest quality of life improvement over older automation tools. Auto-waiting and web-first assertions, and why they get rid of most of the flakiness that used to plague UI test suites.
