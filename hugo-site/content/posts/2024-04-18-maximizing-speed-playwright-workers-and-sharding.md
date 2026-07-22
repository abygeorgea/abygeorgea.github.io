---
title: "Maximizing Speed: Worker Optimization and CI Sharding"
date: 2024-04-18T07:30:00+10:00
draft: false
slug: "maximizing-speed-playwright-workers-and-sharding"
categories:
  - Playwright
tags:
  - Playwright
description: "Tuning Playwright workers and using --shard to split large test suites across parallel CI jobs for faster feedback."
---

In the [previous post](/blog/2024/04/09/automating-quality-playwright-in-github-actions/), we got our suite running automatically in GitHub Actions. That works fine when you have thirty tests. It starts to feel slow once you have three hundred. Today we look at getting that feedback loop back down to something reasonable.

There are two separate levers here, and it is worth understanding the difference. Workers control how many tests run at once on a single machine. Sharding controls splitting the whole suite across multiple separate machines entirely. You usually want both.

## Tuning Workers

By default, Playwright picks a sensible number of workers based on the CPU cores available on the machine running the tests. You can override this directly in your config, which we touched on briefly back in part one.

```typescript
export default defineConfig({
  fullyParallel: true,
  workers: process.env.CI ? 4 : undefined,
});
```

Leaving `workers` undefined locally lets Playwright use its own default based on your machine, which is usually the right call for local development. In CI, it is worth setting an explicit number, since CI runners often report more cores than they can actually give you a fair share of, and an overly high worker count there can cause tests to slow down instead of speeding up, purely from resource contention.

`fullyParallel: true` matters here too. Without it, Playwright only parallelizes across different test files, and every test within a single file still runs one after another. With it enabled, tests within the same file can run across different workers as well, which matters a lot if you have a handful of large spec files rather than lots of small ones.

Worth being honest about a tradeoff here too. More workers means more browser instances running at once, and if those tests are hitting a shared staging environment or a shared test database, too much concurrency can cause its own problems, like exhausting connection pools or hitting rate limits. Tune the worker count against what your target environment can actually handle, not just what your CI runner's core count allows.

## Splitting the Suite With Sharding

Workers help you use one machine efficiently. Sharding lets you split the entire test suite across several machines running at the same time, which is where the real wall clock time savings come from once a suite gets large.

Playwright supports this directly through a command line flag.

```bash
npx playwright test --shard=1/4
```

This tells Playwright to run only the first quarter of the test suite. Run the same command four times with `1/4`, `2/4`, `3/4`, and `4/4`, and between them, every test in the suite gets covered exactly once, split across four separate processes that can run on four separate machines simultaneously.

In GitHub Actions, the clean way to do this is with a matrix strategy.

```yaml
jobs:
  test:
    timeout-minutes: 30
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
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
        run: npx playwright test --shard=${{ matrix.shard }}/4

      - name: Upload blob report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: blob-report-${{ matrix.shard }}
          path: blob-report/
          retention-days: 14
```

This spins up four separate jobs, each running a quarter of the suite, all in parallel. A suite that took forty minutes on one runner can drop to something closer to ten minutes across four runners, assuming the suite splits reasonably evenly.

`fail-fast: false` is worth calling out specifically. Without it, GitHub Actions will cancel the other shards the moment any single shard fails, which means you lose visibility into whether the other three shards would have passed. Setting it to false lets every shard finish and report its own result independently.

## Merging Reports From Multiple Shards

One side effect of sharding is that you end up with separate report data from each shard, rather than one unified HTML report. Playwright has a blob reporter built for exactly this situation, which is why the config above uses `blob-report` as the upload path instead of the usual `playwright-report`.

Add a small follow up job that downloads every shard's blob report and merges them into a single HTML report.

```yaml
  merge-reports:
    if: always()
    needs: [test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Download blob reports
        uses: actions/download-artifact@v4
        with:
          path: all-blob-reports
          pattern: blob-report-*
          merge-multiple: true

      - name: Merge into HTML report
        run: npx playwright merge-reports --reporter html ./all-blob-reports

      - name: Upload merged report
        uses: actions/upload-artifact@v4
        with:
          name: merged-html-report
          path: playwright-report/
          retention-days: 14
```

Now anyone reviewing a pull request gets one single report to look at, covering the whole suite, regardless of how many shards it actually ran across.

## Wrapping Up

Workers make good use of a single machine's resources. Sharding spreads a large suite across several machines at once. Together they turn a slow, single threaded test run into something that finishes in a fraction of the time, without touching the tests themselves.

Next time, in our final post of this series, we look at keeping the framework itself healthy over months of active development, through linting, pre-commit hooks, and some practical rules around selector governance.
