---
title: "Playwright vs Selenium: Test Isolation and Parallelization"
date: 2026-03-24T07:30:00+11:00
draft: false
slug: "playwright-vs-selenium-test-isolation-and-parallelization"
categories:
  - Playwright
  - Selenium
description: "What it actually takes to run Selenium tests safely in parallel compared to Playwright's built in browser context isolation and workers."
---

In the [previous post](/blog/2026/03/13/playwright-vs-selenium-network-interception-and-api-mocking/), we looked at controlling network traffic during a test. This is the last post in this short series comparing Playwright and Selenium, and it covers what happens once your suite grows from ten tests to a thousand and you need them to run fast without stepping on each other.

## Isolation in Selenium Is Something You Build

A single WebDriver instance is a single browser session, with its own cookies and storage. If two tests reuse that same session one after another, state can leak between them without you noticing.

```java
@Test
void firstTest() {
    driver.get("https://example.com");
    driver.manage().addCookie(new Cookie("session", "abc123"));
}

@Test
void secondTest() {
    driver.get("https://example.com");
    // if this reuses the same driver instance, that cookie from firstTest
    // might still be sitting there
}
```

The common fix is either recreating the driver fresh for every test method, which we already covered in the first post of this series, or explicitly clearing state between tests.

```java
@AfterEach
void cleanUp() {
    driver.manage().deleteAllCookies();
    ((JavascriptExecutor) driver).executeScript(
        "window.localStorage.clear(); window.sessionStorage.clear();"
    );
}
```

Either approach works. Both are things you have to remember to do, and both are easy to get quietly wrong when someone adds a new test class without following the same pattern as the rest of the suite.

## Parallel Execution Needs Its Own Driver Per Thread

Parallelization is not part of Selenium at all. It comes from whatever test runner you use, TestNG's `parallel` attribute, JUnit 5's parallel execution config, or pytest-xdist. And the moment tests run on different threads, each thread needs its own WebDriver instance, or they will collide trying to drive the same browser session at once.

The standard pattern here is a `ThreadLocal` driver factory.

```java
public class DriverFactory {
    private static final ThreadLocal<WebDriver> driverThreadLocal = new ThreadLocal<>();

    public static WebDriver getDriver() {
        if (driverThreadLocal.get() == null) {
            driverThreadLocal.set(new ChromeDriver());
        }
        return driverThreadLocal.get();
    }

    public static void quitDriver() {
        WebDriver driver = driverThreadLocal.get();
        if (driver != null) {
            driver.quit();
            driverThreadLocal.remove();
        }
    }
}
```

Paired with something like this in your TestNG suite file.

```xml
<suite name="Suite" parallel="methods" thread-count="4">
  <test name="RegressionTests">
    <classes>
      <class name="com.example.tests.LoginTest"/>
    </classes>
  </test>
</suite>
```

This works, and plenty of large Selenium suites run this way in production every day. But it is infrastructure you built and now own. Forget to call `quitDriver()` at the end of a thread's work and you leak browser processes across your whole CI fleet. And this only gets you parallelism on one machine. To actually scale across multiple machines, you need Selenium Grid, or a cloud provider that hosts one for you, which is a separate piece of infrastructure with its own hub and node architecture to configure and keep running.

## Playwright Isolates by Default

Every Playwright test gets its own browser context automatically, with its own cookies, storage, and cache, completely separate from every other test, even ones running in the same worker process at the same time.

```typescript
test('first test sets a cookie', async ({ context }) => {
  await context.addCookies([
    { name: 'session', value: 'abc123', url: 'https://example.com' },
  ]);
});

test('second test starts with a clean slate', async ({ page }) => {
  const cookies = await page.context().cookies();
  expect(cookies).toHaveLength(0);
});
```

There is no `ThreadLocal` pattern here, because there is nothing shared to protect against. Each test's context is its own sandbox, torn down automatically when the test finishes.

## Parallel Workers and Sharding Are Built In

Playwright's own test runner handles worker based parallelism through config, not through a separate test framework bolted on afterward.

```typescript
export default defineConfig({
  fullyParallel: true,
  workers: process.env.CI ? 4 : undefined,
});
```

And scaling beyond one machine is a command line flag, not a hub and node deployment.

```bash
npx playwright test --shard=1/4
```

Run that same command four times with `1/4` through `4/4` across four CI jobs, and the full suite splits across them automatically. No Grid to stand up, no infrastructure to keep patched and running between test runs. I went into tuning this properly, including merging reports back together from multiple shards, in an [earlier post](/blog/2024/04/18/maximizing-speed-playwright-workers-and-sharding/).

## The Practical Difference

None of this means Selenium suites cannot be fast and safe at scale. Plenty are, running on Grid infrastructure that teams have tuned over years. But that safety and scale is something you build and then maintain indefinitely, thread local driver factories, explicit state cleanup, a Grid deployment with its own health to monitor. Playwright gives you the same outcome, isolated tests running safely in parallel, as the default behavior, and scaling further is a config value and a shard flag rather than new infrastructure.

## Wrapping Up the Series

Across these three posts we looked at setup and teardown, network interception, and isolation and parallelization. In every case the pattern was similar. Selenium can do most of what Playwright does, but it usually takes more code, more infrastructure, or more team discipline to get there safely. Playwright bakes these concerns into the tool itself, at the cost of being a newer, more opinionated piece of software than a WebDriver implementation that has been around since long before any of this comparison mattered.

If you are starting fresh, that is worth weighing seriously. If you have years of investment in a mature Selenium suite, none of this is a mandate to rewrite it. It is a map of where the friction actually is, so you know exactly what you are trading away, and what you are gaining, if you ever do make the move.
