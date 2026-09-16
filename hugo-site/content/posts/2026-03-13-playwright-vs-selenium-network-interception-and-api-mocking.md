---
title: "Playwright vs Selenium: Network Interception and API Mocking"
date: 2026-03-13T07:30:00+11:00
draft: false
slug: "playwright-vs-selenium-network-interception-and-api-mocking"
categories:
  - Playwright
  - Selenium
description: "Why Selenium's approach to network interception feels bolted on compared to Playwright's page.route, and what that actually costs you in practice."
---

In the [previous post](/blog/2026/03/03/playwright-vs-selenium-fixtures-vs-manual-setup-teardown/), we compared how each tool handles setup and teardown. Today we look at something that trips up a lot of teams moving from Selenium to Playwright for the first time. Controlling what actually happens on the network during a test.

## Why Selenium Was Never Built for This

It helps to understand where Selenium comes from. The WebDriver protocol, the actual W3C specification Selenium implements, was designed to simulate a real user driving a real browser. Click here, type there, read what is on the page. Network traffic was never part of that picture.

Selenium 4 added a way in through the Chrome DevTools Protocol, which gives you low level access to what Chromium is actually doing under the hood, including its network layer. More recently, Selenium has been investing in WebDriver BiDi, a newer W3C standard aiming to bring this kind of capability to every browser through one consistent API, not just Chromium through CDP.

Here is roughly what intercepting a request looks like using the DevTools approach in Java.

```java
ChromeDriver driver = new ChromeDriver();
DevTools devTools = driver.getDevTools();
devTools.createSession();
devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));

devTools.addListener(Network.requestIntercepted(), interceptedRequest -> {
    if (interceptedRequest.getRequest().getUrl().contains("/api/products")) {
        devTools.send(Network.continueInterceptedRequest(
            interceptedRequest.getInterceptionId(),
            Optional.empty(), Optional.of("200"), Optional.empty(),
            Optional.empty(), Optional.empty(), Optional.empty(), Optional.empty()
        ));
    }
});
```

Notice how much is happening here just to react to one request. You open a DevTools session, enable the network domain, register a listener, and manually construct the continuation response with mostly empty optional parameters. Building an actual mocked response body means base64 encoding it yourself and setting headers by hand. It is powerful, but it reads like the low level protocol access it actually is, and it is tied closely to Chromium. BiDi improves the cross browser story over time, but the tooling and documentation around it are still catching up to how mature CDP support already is, and either way you are writing noticeably more code than the equivalent Playwright test.

## Playwright Treats This as a First Class Feature

Playwright's `page.route()` was built from day one with the assumption that tests need to control the network, not just observe it as an afterthought.

```typescript
test('shows fallback UI when the product API fails', async ({ page }) => {
  await page.route('**/api/products/42', (route) => {
    route.fulfill({
      status: 500,
      contentType: 'application/json',
      body: JSON.stringify({ error: 'Internal Server Error' }),
    });
  });

  await page.goto('/products/42');
  await expect(page.locator('.error-message')).toBeVisible();
});
```

That is the whole thing. A glob pattern to match the URL, and a plain object describing the response you want back. No sessions, no listeners, no base64 encoding.

`route.fulfill()` mocks a response entirely. `route.continue()` lets the real request through, optionally after you inspect or modify it. `route.abort()` simulates the request failing outright, which is exactly what you want for testing how the UI handles a dropped connection.

```typescript
test('blocks third party analytics calls during the test', async ({ page }) => {
  await page.route('**/analytics.example.com/**', (route) => route.abort());
  await page.goto('/');
});
```

## Inspecting Traffic Without Changing It

Sometimes you do not want to mock anything. You just want to confirm the frontend sent the right request. Playwright gives you this through simple event listeners on the page.

```typescript
test('checkout sends the correct payload', async ({ page }) => {
  let checkoutPayload: any;

  page.on('request', (request) => {
    if (request.url().includes('/api/checkout') && request.method() === 'POST') {
      checkoutPayload = request.postDataJSON();
    }
  });

  await page.goto('/checkout');
  await page.click('#place-order');

  expect(checkoutPayload.items.length).toBeGreaterThan(0);
});
```

This is a genuinely useful pattern. It confirms the frontend built the request correctly, independent of whatever the backend actually does with it.

## Why This Matters Beyond Convenience

The real value here is not just writing less code. It is what becomes practical to test at all. Error states like a 500 response or a malformed payload are often hard to trigger reliably against a real backend, since you would need to coordinate with whoever owns that service, or find a way to force a failure condition on demand. Mocking makes these trivial to set up on your own terms.

It also lets you test the frontend in isolation from backend availability, which matters a lot in CI where a flaky downstream dependency can fail your UI tests for reasons that have nothing to do with the UI. And you can simulate conditions that are awkward to reproduce naturally, like a slow response.

```typescript
test('shows a loading spinner while the search request is in flight', async ({ page }) => {
  await page.route('**/api/search**', async (route) => {
    await new Promise((resolve) => setTimeout(resolve, 2000));
    await route.continue();
  });

  await page.goto('/search?q=mouse');
  await expect(page.locator('.loading-spinner')).toBeVisible();
});
```

Delaying a real request by two seconds and letting it continue afterward gives you a reliable way to test a loading state without needing the backend to actually be slow.

## Wrapping Up

Selenium can get you some of this through CDP or the newer BiDi support, but it takes real effort and reads like protocol level plumbing rather than a testing feature. Playwright treats controlling the network as something you will need constantly, and the API reflects that from the first line of code.

Next time, we look at test isolation and parallelization, and what it actually takes to run each tool's tests safely at scale.
