---
title: "Speeding Up Execution: Combining API Setup with UI Validation"
date: 2024-03-22T07:30:00+11:00
draft: false
slug: "speeding-up-execution-playwright-api-and-ui-hybrid-testing"
categories:
  - Playwright
tags:
  - Playwright
description: "Using Playwright's built-in APIRequestContext to seed state through the API and speed up hybrid UI and API test flows."
---

In the [previous post](/blog/2024/03/13/eliminating-flakiness-playwright-auto-waiting-and-web-first-assertions/), we looked at why Playwright tests do not need manual sleeps. Today we look at a different kind of speed problem. Tests that spend most of their time on setup steps that have nothing to do with what they are actually testing.

Say you are testing the checkout flow. To get there, a real user has to log in, search for a product, add it to the cart, and then go to checkout. If every single checkout test drives all of that through the UI first, you are spending most of your test run clicking through steps you already tested thoroughly somewhere else. That adds up fast across a whole suite.

Playwright gives you a clean way around this. It ships with a built in API testing client, so you can set up state directly through your backend, and only use the browser for the part you actually care about.

## The Built-In Request Context

Every test automatically gets access to an `APIRequestContext` through the `request` fixture. You do not need to install a separate HTTP client.

```typescript
test('check product details via API', async ({ request }) => {
  const response = await request.get('/api/products/42');
  expect(response.ok()).toBeTruthy();

  const product = await response.json();
  expect(product.name).toBe('Wireless Mouse');
});
```

This works for POST, PUT, and DELETE too, and it handles headers, query params, and JSON bodies naturally.

```typescript
const response = await request.post('/api/cart/items', {
  data: {
    productId: 42,
    quantity: 2,
  },
  headers: {
    Authorization: `Bearer ${authToken}`,
  },
});
```

Because this is a real HTTP client under the hood, it is fast. There is no browser rendering involved, no waiting for a page to load, just a direct call to your backend and a response.

## Seeding State Before the UI Takes Over

This is where things get genuinely useful. Instead of driving the browser through login, search, and add to cart, we can do all of that through direct API calls, and only open the browser once we are ready for checkout.

```typescript
// utils/api-client.ts
import { APIRequestContext } from '@playwright/test';

export async function loginViaApi(request: APIRequestContext, username: string, password: string) {
  const response = await request.post('/api/auth/login', {
    data: { username, password },
  });
  const { token } = await response.json();
  return token;
}

export async function addProductToCartViaApi(request: APIRequestContext, token: string, productId: number) {
  await request.post('/api/cart/items', {
    data: { productId, quantity: 1 },
    headers: { Authorization: `Bearer ${token}` },
  });
}
```

And the test itself becomes short and focused entirely on checkout, which is the part we are actually validating.

```typescript
test('user can complete checkout with an item already in cart', async ({ page, request }) => {
  const token = await loginViaApi(request, 'testuser', 'Password123');
  await addProductToCartViaApi(request, token, 42);

  // Hand the authenticated session to the browser context
  await page.context().addCookies([
    { name: 'auth_token', value: token, url: 'https://example.com' },
  ]);

  const checkoutPage = new CheckoutPage(page);
  await checkoutPage.goto();
  await checkoutPage.completeOrder();

  await expect(checkoutPage.confirmationBanner).toBeVisible();
});
```

The login and add to cart steps that used to take several seconds of clicking through the UI now happen in a couple of fast API calls. The browser only opens for the part of the test that actually matters, checkout itself.

## Verifying Backend State After a UI Action

The same idea works in reverse too. Sometimes you want to confirm that an action a user took in the browser actually changed something correctly on the backend, beyond what is visible on the screen.

```typescript
test('placing an order updates inventory count', async ({ page, request }) => {
  const checkoutPage = new CheckoutPage(page);
  await checkoutPage.goto();
  await checkoutPage.completeOrder();
  await expect(checkoutPage.confirmationBanner).toBeVisible();

  const response = await request.get('/api/products/42');
  const product = await response.json();
  expect(product.stockCount).toBe(17);
});
```

This gives you a level of confidence that a pure UI check cannot. The confirmation banner showing up tells you the user experience worked. Checking the API afterward tells you the backend state is actually correct too.

## When to Use the UI and When Not To

The rule I follow is simple. If a step is not the thing the test is actually verifying, it is a candidate for the API. Logging in, creating prerequisite records, cleaning up test data, all of that belongs in API calls wherever your backend supports it. The UI is reserved for the actual behavior under test.

This does not replace end to end coverage entirely. You still want some tests that walk through a full real user journey from start to finish, UI only, because that is the only way to catch integration issues between steps. But for the bulk of your suite, hybrid tests like the ones above will run noticeably faster and fail less often, because you have fewer UI interactions that could hit a rendering quirk or a slow network call.

## Wrapping Up

Playwright's request context turns API setup into a first class part of your test suite, without needing a separate HTTP library. Use it to skip repetitive UI steps and to verify backend state directly, and save full UI journeys for the parts of your application where the user interface itself is what you are testing.

Next time, we look at what happens when a test does fail. Screenshots, videos, and the Playwright trace viewer, and how they turn a frustrating overnight CI failure into something you can diagnose in a couple of minutes.
