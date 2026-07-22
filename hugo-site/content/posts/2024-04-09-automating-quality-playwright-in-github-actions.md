---
title: "Automating Quality: Running Playwright in GitHub Actions"
date: 2024-04-09T07:30:00+10:00
draft: false
slug: "automating-quality-playwright-in-github-actions"
categories:
  - Playwright
tags:
  - Playwright
description: "Setting up a GitHub Actions workflow for Playwright using the official Docker image and archiving reports as artifacts."
---

In the [previous post](/blog/2024/03/31/debugging-with-confidence-playwright-traces-and-html-reports/), we set up reporting and traces so failures are easy to diagnose. None of that matters much if tests only run on your own laptop. Today we wire everything into a CI pipeline, so every pull request gets tested automatically before it can be merged.

We will use GitHub Actions here, since it is what most teams already have available, and Playwright has solid first party support for it.

## A Basic Workflow

If you answered yes to the GitHub Actions prompt back when you ran `npm init playwright@latest` in part one, you already have a starting workflow file at `.github/workflows/playwright.yml`. Here is a version close to what I actually use on real projects, with a bit more structure around artifact uploads.

```yaml
name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    timeout-minutes: 30
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test

      - name: Upload HTML report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 14
```

A couple of details worth calling out. The `if: always()` on the upload step means the report gets uploaded whether the tests passed or failed. You want the report from a failing run more than anything, so this cannot be conditional on success.

`npx playwright install --with-deps` installs both the browser binaries and any system level dependencies those browsers need on a fresh Ubuntu runner. Skipping the `--with-deps` flag is a common cause of tests failing in CI with confusing browser launch errors that never happen locally.

## Using the Official Docker Image Instead

Installing browsers fresh on every CI run works, but it adds time to every single job, and there is always a small risk of a subtle difference between your local OS and the CI runner's rendering behavior. Playwright publishes an official Docker image with browsers and all their dependencies already baked in, which avoids both problems.

```yaml
jobs:
  test:
    timeout-minutes: 30
    runs-on: ubuntu-latest
    container:
      image: mcr.microsoft.com/playwright:v1.42.0-jammy
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run Playwright tests
        run: npx playwright test

      - name: Upload HTML report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 14
```

Notice there is no separate browser install step here at all. The container already has everything Playwright needs, matched to the exact Playwright version in the image tag. This is worth pinning to a specific version rather than using `latest`, so an unrelated image update never quietly changes browser behavior out from under your pipeline.

## Archiving Traces Alongside Reports

Reports alone are useful, but if a test fails, you want the trace file too, since that gives you the full step by step DOM and network detail we covered in the last post. Add a second upload step scoped to just the trace output.

```yaml
      - name: Upload traces
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-traces
          path: test-results/
          retention-days: 14
```

This one uses `if: failure()` instead of `always()`, since there is no point archiving trace files for a run that had nothing to investigate. Playwright writes trace files into the `test-results` directory automatically whenever the `trace: 'on-first-retry'` setting from our config triggers a capture.

With both of these artifacts in place, anyone on the team can go to a failed workflow run, download the report and the trace, and start debugging immediately without needing to reproduce the failure locally first.

## Wrapping Up

At this point every push and pull request against `main` runs the full suite automatically, using the same browser environment every time, with a report and traces waiting for anyone who needs them.

Next time, we look at what happens once this suite grows large enough that a single CI run starts taking too long, and how worker tuning and test sharding keep feedback loops fast even as the number of tests keeps climbing.
