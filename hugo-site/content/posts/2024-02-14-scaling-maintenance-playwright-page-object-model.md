---
title: "Scaling Maintenance: Implementing the Page Object Model (POM)"
date: 2024-02-14T07:30:00+11:00
draft: false
slug: "scaling-maintenance-playwright-page-object-model"
categories:
  - Playwright
tags:
  - Playwright
description: "How to build a maintainable Page Object Model in Playwright with TypeScript, including base pages and reusable components."
---

In the [previous post](/blog/2024/02/05/laying-the-foundation-playwright-architecture-and-setup/), we scaffolded a Playwright project and set up a folder structure with an empty `pages/` directory sitting there waiting to be used. Today we fill it in properly.

Here is a problem I see all the time. A test file has ten tests in it. Every single one of them repeats the same selector for the login button. Then the front end team renames a CSS class, and suddenly all ten tests break at once. You end up doing a find and replace across a dozen files just to fix one small UI change.

The Page Object Model fixes this. It is not a Playwright specific idea. It has been around test automation for years. But Playwright and TypeScript make it especially clean to implement.

## Keeping Tests Focused on Behavior

The core idea is simple. Your test files should read like a description of user behavior, not a list of CSS selectors. Compare these two approaches.

Without a page object, a login test tends to look like this.

```typescript
test('user can log in with valid credentials', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'Password123');
  await page.click('button[type="submit"]');
  await expect(page.locator('.welcome-banner')).toBeVisible();
});
```

It works, but the test is mixing two concerns. It describes what the user does, and it describes exactly how the page is built. With a page object, the same test looks like this instead.

```typescript
test('user can log in with valid credentials', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('testuser', 'Password123');
  await expect(loginPage.welcomeBanner).toBeVisible();
});
```

Now the test reads like a sentence. Go to the login page, log in, check the welcome banner. All the messy selector detail moved somewhere else, and that somewhere else is a page object class.

## Building a Base Page Class

Most pages in an application share some common behavior. Waiting for the page to be ready, grabbing the page title, that sort of thing. It makes sense to put that shared logic in one base class that every other page object extends.

```typescript
// pages/base.page.ts
import { Page } from '@playwright/test';

export class BasePage {
  constructor(protected readonly page: Page) {}

  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  async getTitle(): Promise<string> {
    return this.page.title();
  }
}
```

Now the login page can extend this and add its own locators and actions.

```typescript
// pages/login.page.ts
import { Page, Locator } from '@playwright/test';
import { BasePage } from './base.page';

export class LoginPage extends BasePage {
  readonly usernameInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly welcomeBanner: Locator;

  constructor(page: Page) {
    super(page);
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.submitButton = page.locator('button[type="submit"]');
    this.welcomeBanner = page.locator('.welcome-banner');
  }

  async goto(): Promise<void> {
    await this.page.goto('/login');
    await this.waitForPageLoad();
  }

  async login(username: string, password: string): Promise<void> {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}
```

Notice that locators are declared once, as readonly properties, and set up in the constructor. Every method after that just uses `this.usernameInput` instead of repeating the selector string. If the front end team changes the id from `#username` to `#user-email`, you update one line in one file. Every test that uses `LoginPage` keeps working without a single change.

## Reusing Shared Components

A lot of applications have UI pieces that show up on many pages. A header with a search bar. A footer with links. A data table used across three different admin screens. Copying the same locators into every page object that touches these pieces gets messy fast.

The fix is to treat these shared pieces as components of their own, separate from any single page.

```typescript
// pages/components/header.component.ts
import { Page, Locator } from '@playwright/test';

export class HeaderComponent {
  readonly searchInput: Locator;
  readonly cartIcon: Locator;
  readonly accountMenu: Locator;

  constructor(private readonly page: Page) {
    this.searchInput = page.locator('[data-testid="header-search"]');
    this.cartIcon = page.locator('[data-testid="header-cart"]');
    this.accountMenu = page.locator('[data-testid="header-account"]');
  }

  async search(term: string): Promise<void> {
    await this.searchInput.fill(term);
    await this.searchInput.press('Enter');
  }
}
```

Any page object that needs the header just creates an instance of it in its constructor.

```typescript
// pages/product-listing.page.ts
import { Page } from '@playwright/test';
import { BasePage } from './base.page';
import { HeaderComponent } from './components/header.component';

export class ProductListingPage extends BasePage {
  readonly header: HeaderComponent;

  constructor(page: Page) {
    super(page);
    this.header = new HeaderComponent(page);
  }
}
```

Now a test can do `productListingPage.header.search('running shoes')` and it just works, without the product listing page object needing to know anything about how the header is built.

## Wrapping Up

We now have a base page, a real page object, and a reusable component pattern for shared UI. This alone removes most of the maintenance pain that comes from front end changes.

Next time, we will look at test data. Hardcoded usernames and product ids look fine at first, but they cause real headaches once you start running tests in parallel against a shared environment. We will cover static data files and dynamic data generation with Faker, and how to keep parallel tests from stepping on each other.
