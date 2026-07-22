---
title: "Laying the Foundation: Architecture and Setup for a Playwright Project"
date: 2024-02-05T07:30:00+11:00
draft: false
slug: "laying-the-foundation-playwright-architecture-and-setup"
categories:
  - Playwright
tags:
  - Playwright
description: "How to scaffold a clean, scalable Playwright TypeScript project, from folder structure to config."
---

Every solid test automation framework starts the same way. You pick a clean folder structure. You get your config right once, early, before you have hundreds of tests depending on it. Playwright makes this part refreshingly easy. TypeScript support is built in from day one. You do not need to bolt on extra plugins or wrestle with a transpiler.

This post walks through setting up a Playwright project the way I would actually set one up for a real team. Not a toy demo. Something that can grow to hundreds of tests without turning into a mess.

This is part one of a ten part series on building a proper Playwright framework. By the end of it, you will have a setup that handles page objects, test data, fixtures, CI, reporting, and more. Today we just lay the foundation.

## Starting With the Official CLI

Playwright ships its own scaffolding tool, and it saves a lot of manual setup. Open a terminal in an empty folder and run this.

```bash
npm init playwright@latest
```

You will get a short list of prompts. Answer them like this for a TypeScript project.

```text
✔ Do you want to use TypeScript or JavaScript? · TypeScript
✔ Where to put your end-to-end tests? · tests
✔ Add a GitHub Actions workflow? (y/N) · y
✔ Install Playwright browsers (can be done manually via 'npx playwright install')? (Y/n) · y
```

Once it finishes, you get a working project with a sample test, a config file, and browser binaries installed locally. Run the sample test right away just to confirm everything is wired up.

```bash
npx playwright test
```

If that green summary shows up in your terminal, you are ready to start shaping the project into something real.

## A Directory Layout That Scales

The default scaffold gives you a single `tests` folder and a `playwright.config.ts` file. That is enough for a handful of tests. It is not enough once you have real page objects, shared fixtures, and test data files. Here is the layout I use on most projects.

```text
.
├── tests/
│   ├── login.spec.ts
│   ├── checkout.spec.ts
│   └── search.spec.ts
├── pages/
│   ├── base.page.ts
│   ├── login.page.ts
│   └── checkout.page.ts
├── fixtures/
│   └── test-options.ts
├── data/
│   ├── users.json
│   └── products.json
├── utils/
│   ├── api-client.ts
│   └── date-helpers.ts
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

A quick word on what each folder is for, since this comes up in every code review I do.

* `tests/` holds only spec files. Each file describes a user journey or a feature. No selectors, no low level logic here.
* `pages/` holds Page Object Model classes. These wrap up the selectors and actions for a single page or component.
* `fixtures/` holds custom Playwright fixtures. This is where you wire up authenticated sessions, shared setup, and anything injected into your tests.
* `data/` holds static test data. Think reference lookups, seed users, and configuration values that do not change at runtime.
* `utils/` holds small reusable helpers. API wrappers, date formatting, string generation, that kind of thing.

Keeping this separation from day one means a new team member can open the repo and know exactly where to look for something. We will fill in `pages/` and `fixtures/` properly in the next two posts.

## Configuring playwright.config.ts

The config file is where you set the rules for your whole test run. Base URL, timeouts, which browsers to test against, and how many workers to use all live here. Below is a config close to what I use as a starting point on a real project.

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30 * 1000,
  expect: {
    timeout: 5000,
  },
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 2 : undefined,
  reporter: 'html',
  use: {
    baseURL: process.env.BASE_URL || 'https://example.com',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    headless: true,
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
});
```

A few points worth calling out here.

`baseURL` means every `page.goto('/login')` in your tests resolves against the right environment. You just swap the `BASE_URL` environment variable between local, staging, and production runs.

`fullyParallel: true` tells Playwright it can run tests within the same file in parallel, not just across files. This matters a lot once your suite grows.

`retries` is set to zero locally and two in CI. Flaky network conditions in a shared CI runner are a different problem than a genuinely broken test on your laptop, and this setting reflects that difference.

The `projects` array is what gives you cross browser coverage almost for free. Every test you write runs against Chromium, Firefox, and WebKit without any extra code.

## Wrapping Up

At this point you have a project that installs cleanly, runs a sample test, and has a folder structure ready for real page objects and fixtures. That is a solid place to stop for one post.

Next time, we will tackle the Page Object Model properly. We will build a base page class, a couple of real page objects, and talk about why keeping selectors out of your test files saves you so much pain later.
