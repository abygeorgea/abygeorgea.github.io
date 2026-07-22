---
title: "Debugging with Confidence: Traces, Screenshots, and HTML Reports"
date: 2024-03-31T07:30:00+11:00
draft: false
slug: "debugging-with-confidence-playwright-traces-and-html-reports"
categories:
  - Playwright
tags:
  - Playwright
description: "Using Playwright's HTML reporter, screenshots, video, and the trace viewer to diagnose CI failures quickly."
---

In the [previous post](/blog/2024/03/22/speeding-up-execution-playwright-api-and-ui-hybrid-testing/), we sped up tests by mixing in API calls. Today we cover something every team hits eventually. A test fails overnight in CI, nobody was watching it run, and now someone has to figure out why.

This used to mean staring at a stack trace and a screenshot, if you were lucky enough to have a screenshot at all, and guessing at what the page must have looked like. Playwright gives you a lot more to work with than that, and it is worth setting all of it up before you actually need it.

## The Built-In HTML Reporter

We already set `reporter: 'html'` back in our config in part one. After any test run, this produces a report you can open locally.

```bash
npx playwright show-report
```

This opens an interactive report in your browser, listing every test, its status, and how long it took. Click into a failed test and you get the full error message, the exact line of code that failed, and a timeline of every step Playwright took along the way. For anything beyond a trivial failure, this is always the first thing I look at.

## Capturing Screenshots and Video Automatically

Static evidence of what the browser actually looked like at the moment of failure is worth a lot. Playwright can capture this automatically, without you writing any extra code in your tests.

```typescript
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
},
```

With these two settings in your config, a screenshot and a short video recording get saved automatically whenever a test fails, and both show up as attachments in the HTML report. Passing tests do not generate this extra data, so you are not filling up disk space or CI storage for runs that already worked fine.

You can also grab a screenshot manually at any point in a test, which is handy for debugging a specific step while you are writing a new test.

```typescript
await page.screenshot({ path: 'debug-checkout-step.png', fullPage: true });
```

## The Trace Viewer

Screenshots and video tell you what the page looked like. The trace viewer tells you everything else. It is, in my experience, the single most useful debugging tool Playwright gives you.

Turn it on in your config like this.

```typescript
use: {
  trace: 'on-first-retry',
},
```

This setting records a trace only when a test fails on its first attempt and then gets retried, which keeps the overhead low while still capturing exactly the runs you need. Once you have a trace file from a failed run, either from CI artifacts or a local run, open it like this.

```bash
npx playwright show-trace trace.zip
```

This opens an interactive viewer where you can step through the test action by action. For every single step, you get a DOM snapshot exactly as it existed at that moment, a screenshot, the network requests that were in flight, and the browser console output. You can click on any action in the timeline and see precisely what the page looked like right before and right after it happened.

This turns debugging a remote CI failure from a guessing game into something closer to actually watching the test run happen. You are not reconstructing the failure from a single error message anymore. You are looking at the exact state of the page at the exact moment things went wrong.

## Generating Traces On Demand

Sometimes you want a trace for a specific local run, outside of the retry mechanism, especially while you are actively debugging something. You can do that directly from the command line.

```bash
npx playwright test checkout.spec.ts --trace on
```

This forces every test in that run to record a trace, regardless of whether it passes or fails, which is useful when you want to inspect a passing test's behavior in detail too, not just chase down a failure.

## Archiving Reports and Traces From CI

None of this helps much if the report and trace files disappear the moment the CI job finishes. Make sure your pipeline uploads them as artifacts so anyone on the team can pull them down later. We will look at the exact GitHub Actions configuration for this in the next post, but the important habit to build now is treating the HTML report and any trace files as first class outputs of every CI run, not an afterthought.

## Wrapping Up

Between the HTML reporter, automatic screenshots and video on failure, and the trace viewer, you rarely need to guess what went wrong in a failed test. The evidence is already sitting there waiting for you.

Next time, we take everything we have built so far and wire it into a GitHub Actions pipeline, so every pull request gets a full test run automatically, with reports and traces archived for anyone who needs to look at them later.
