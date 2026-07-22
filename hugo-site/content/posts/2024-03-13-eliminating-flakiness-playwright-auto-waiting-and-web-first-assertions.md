---
title: "Eliminating Flakiness: Auto-Waiting and Web-First Assertions"
date: 2024-03-13T07:30:00+11:00
draft: false
slug: "eliminating-flakiness-playwright-auto-waiting-and-web-first-assertions"
categories:
  - Playwright
tags:
  - Playwright
description: "How Playwright's auto-waiting and web-first assertions remove the need for arbitrary sleeps and reduce flaky UI tests."
---

In the [previous post](/blog/2024/03/04/supercharging-tests-playwright-native-fixtures/), we used fixtures to clean up test setup. Today we talk about the thing that probably causes more wasted engineering hours than anything else in test automation. Flakiness.

If you have worked with older browser automation tools, you know the pattern. A test fails intermittently. Someone adds `sleep(2000)` right before the failing step. The test passes for a while. Then it starts failing again on a slower CI runner, so someone bumps it to `sleep(5000)`. Now your test suite takes twenty minutes longer to run and it is still not fully reliable.

Playwright takes a genuinely different approach, and it is worth understanding why it works so well.

## How Auto-Waiting Actually Works

Every action Playwright performs on a locator, like `click`, `fill`, or `check`, automatically waits for a set of conditions before it does anything. The element has to be attached to the DOM. It has to be visible. It has to be stable, meaning it is not in the middle of a CSS transition or animation. It has to receive events, meaning nothing else is covering it. And for things like inputs, it has to actually be enabled.

Playwright checks all of this before every action, and it keeps retrying those checks until they pass or the timeout is reached. You do not write any of this logic yourself. You just call the action.

```typescript
await page.locator('#submit-button').click();
```

Behind that one line, Playwright is silently waiting for the button to exist, become visible, stop moving, and become clickable, before the click actually happens. If a loading spinner is covering the button for half a second, Playwright waits it out instead of throwing an error immediately.

This is the single biggest reason Playwright tests tend to be more stable than tests written with tools that click immediately and let you deal with the fallout.

## Web-First Assertions

The same philosophy carries over into assertions. A traditional assertion checks a condition once, right now, and either passes or fails immediately. A web-first assertion in Playwright keeps checking, on a short interval, until the condition is true or the timeout runs out.

```typescript
await expect(page.locator('.welcome-banner')).toBeVisible();
```

This single line will keep polling for up to the configured timeout, checking whether that banner has become visible. If the banner takes three hundred milliseconds to render after some API call finishes, this assertion just waits for it naturally. No manual wait needed anywhere.

There are a lot of these built in matchers, and it is worth knowing the common ones.

```typescript
await expect(page.locator('.error-message')).not.toBeVisible();
await expect(page.locator('#order-status')).toHaveText('Confirmed');
await expect(page.locator('.cart-count')).toHaveText('3');
await expect(page.locator('input#email')).toHaveValue('test@example.com');
await expect(page).toHaveURL(/.*\/checkout\/success/);
await expect(page).toHaveTitle(/Order Confirmation/);
```

Every one of these retries automatically. You are not writing polling loops. You are describing the end state you expect, and letting Playwright handle the timing.

## What This Replaces

It is worth being explicit about what you should no longer be reaching for. `page.waitForTimeout()` exists in the API, and it is tempting to use it the way you might have used a hard sleep in an older tool. Resist that urge. It almost always means there is a specific condition you should be waiting for instead, and Playwright almost certainly has a way to wait for that exact condition already.

```typescript
// Avoid this
await page.waitForTimeout(3000);
await page.locator('.results-list').click();

// Prefer this
await expect(page.locator('.results-list')).toBeVisible();
await page.locator('.results-list').click();
```

The second version waits exactly as long as it needs to and no longer. On a fast day it might only wait fifty milliseconds. On a slow day it might wait two seconds. Either way, the test only proceeds once the real condition is met.

## Sensible Timeouts and Retries

Auto-waiting handles most timing issues on its own, but it is still worth configuring sane defaults for the situations that genuinely are slower, like a flaky third party network call in a staging environment. Two settings matter here, both of which we touched on back in our config file in part one.

```typescript
export default defineConfig({
  timeout: 30 * 1000,
  expect: {
    timeout: 5000,
  },
  retries: process.env.CI ? 2 : 0,
});
```

The `expect.timeout` setting controls how long any single web-first assertion will keep retrying before it gives up. The top level `timeout` controls how long an entire test is allowed to run. And `retries` gives you a safety net in CI specifically, where shared infrastructure and network variance are more likely to cause a one-off failure that would not reproduce on a second attempt.

Retries are a safety net, though, not a fix for a genuinely broken test. If a test only passes on the second or third attempt consistently, that is a sign something in the test or the application needs attention, not a sign to increase the retry count further.

## Wrapping Up

Auto-waiting and web-first assertions remove almost every reason to reach for a manual sleep. Your tests wait exactly as long as they need to, and no longer, which makes both fast and slow days behave consistently.

Next time, we look at combining Playwright's API testing capabilities with UI testing, so we can skip repetitive UI setup steps entirely and let the API do the boring parts.
