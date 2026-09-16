---
title: "Playwright vs Selenium: Fixtures vs Manual Setup and Teardown"
date: 2026-03-03T07:30:00+11:00
draft: false
slug: "playwright-vs-selenium-fixtures-vs-manual-setup-teardown"
categories:
  - Playwright
  - Selenium
description: "Comparing Playwright's built in test fixtures against the manual WebDriver lifecycle management Selenium leaves entirely up to you."
---

Most comparisons between Playwright and Selenium start with API syntax. Click this way, find an element that way. That is not where the real difference shows up day to day. The real difference shows up the moment you write your tenth test and notice how much setup code you are copying between files.

This is the first of three posts comparing Playwright and Selenium on things that actually matter once a suite grows past a handful of tests. Today it is setup and teardown. The examples use Java for Selenium and TypeScript for Playwright, since that is a pairing a lot of teams genuinely work with side by side.

## Selenium Has No Built-In Lifecycle

This is worth saying plainly. Selenium WebDriver is a browser automation library. It gives you a driver object and a way to find elements and interact with them. It has no opinion at all about test structure, setup order, or cleanup. Every bit of that comes from whatever test framework you pair it with, JUnit, TestNG, NUnit, or pytest.

A typical JUnit 5 test class looks like this.

```java
public class LoginTest {
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();
        driver.manage().window().maximize();
    }

    @Test
    void userCanLogIn() {
        driver.get("https://example.com/login");
        driver.findElement(By.id("username")).sendKeys("testuser");
        driver.findElement(By.id("password")).sendKeys("Password123");
        driver.findElement(By.cssSelector("button[type='submit']")).click();
        assertTrue(driver.findElement(By.className("welcome-banner")).isDisplayed());
    }

    @AfterEach
    void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

This works fine. But notice how much of it is plumbing you wrote yourself. Create the driver. Remember to null check it in teardown. Remember to actually call quit, since a forgotten quit leaves an orphaned browser process running on whatever machine the test executed on. None of this is Selenium's fault exactly. It is just the cost of a library that does not manage test lifecycle for you.

## The Inheritance Trap

The real pain shows up once different tests need different starting states. Say half your suite needs a logged in user. The common pattern is a base class.

```java
public abstract class BaseAuthenticatedTest {
    protected WebDriver driver;

    @BeforeEach
    void setUpAuthenticated() {
        driver = new ChromeDriver();
        driver.get("https://example.com/login");
        driver.findElement(By.id("username")).sendKeys("testuser");
        driver.findElement(By.id("password")).sendKeys("Password123");
        driver.findElement(By.cssSelector("button[type='submit']")).click();
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

Every test class needing a logged in user extends this. It works, right up until you need a second independent precondition. Say some tests also need a cart pre populated with items. Java does not let you extend two classes at once, so now you are either duplicating the login logic into a second base class, or building a deeper inheritance chain, or moving everything into static helper methods called manually inside each `@BeforeEach`. None of these options are terrible on their own. All of them get messy fast once you have three or four independent preconditions that need to combine in different ways across your suite.

## Fixtures Solve This With Composition Instead of Inheritance

Playwright's `test.extend` gives you a fixture for the same login flow.

```typescript
import { test as base } from '@playwright/test';
import type { Page } from '@playwright/test';

type AuthFixtures = {
  authenticatedPage: Page;
};

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ page }, use) => {
    await page.goto('https://example.com/login');
    await page.fill('#username', 'testuser');
    await page.fill('#password', 'Password123');
    await page.click('button[type="submit"]');
    await use(page);
  },
});
```

```typescript
test('user can access their profile', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('/account/profile');
  await expect(authenticatedPage.locator('.profile-name')).toBeVisible();
});
```

Now add the second precondition, a cart with items in it, as its own separate fixture that depends on the first one.

```typescript
type Fixtures = {
  authenticatedPage: Page;
  cartWithItems: Page;
};

export const test = base.extend<Fixtures>({
  authenticatedPage: async ({ page }, use) => {
    await page.goto('https://example.com/login');
    await page.fill('#username', 'testuser');
    await page.fill('#password', 'Password123');
    await page.click('button[type="submit"]');
    await use(page);
  },
  cartWithItems: async ({ authenticatedPage }, use) => {
    await authenticatedPage.request.post('/api/cart/items', { data: { productId: 42 } });
    await use(authenticatedPage);
  },
});
```

```typescript
test('checkout works with items already in cart', async ({ cartWithItems }) => {
  await cartWithItems.goto('/checkout');
  await expect(cartWithItems.locator('.order-summary')).toBeVisible();
});
```

`cartWithItems` depends on `authenticatedPage`, and Playwright resolves that dependency automatically. There is no inheritance chain here at all. Any test can ask for any combination of fixtures it needs, and Playwright figures out the order to set them up in. This is the actual difference. It is not that Playwright has a nicer syntax for the same idea. It is that fixtures compose, and inheritance does not.

If you want a deeper look at building out a fixture library on its own, outside of this Selenium comparison, I covered that in an [earlier post](/blog/2024/03/04/supercharging-tests-playwright-native-fixtures/).

## Cleanup Happens Even When the Test Fails

The other quiet advantage is teardown. In the fixture above, everything after `await use(page)` runs after the test finishes, whether it passed or failed. You do not write a try or a finally block for this. It is built into how fixtures work.

Compare this to the Selenium example from earlier. If `setUp()` throws an exception partway through, before the driver variable is even assigned, JUnit will still call `tearDown()`, and your null check saves you there. But that null check is something you had to remember to write. Multiply that across every base class and helper method in a large suite, and it becomes one more category of thing that quietly breaks when someone refactors without thinking about it.

## Which One Should You Actually Use

If you already have a mature Selenium suite built around base classes, this is not, on its own, a reason to rewrite it. Inheritance based setup has shipped reliable test suites for well over a decade, and plenty of teams manage it fine with good discipline. But if you are starting a new project, or your existing base class hierarchy is already starting to strain under too many combinations of preconditions, the fixture model scales in a way inheritance structurally cannot. Composition does not hit a wall the way a single inheritance chain does.

## Wrapping Up

Selenium leaves lifecycle management entirely in your hands, paired with whatever hooks your test framework provides. Playwright bakes it in, and its fixture system composes cleanly instead of forcing everything through inheritance.

Next time, we look at a different kind of test, ones that need to mock or inspect network traffic, and how differently each tool handles it.
