---
title: "Keeping Code Clean: Linting, Hooks, and Long-Term Governance"
date: 2024-04-27T07:30:00+10:00
draft: false
slug: "keeping-code-clean-playwright-linting-hooks-and-governance"
categories:
  - Playwright
tags:
  - Playwright
description: "Using ESLint, Prettier, Husky, and lint-staged to keep a Playwright TypeScript framework healthy over months of active development, plus selector governance rules."
---

In the [previous post](/blog/2024/04/18/maximizing-speed-playwright-workers-and-sharding/), we got a large suite running fast in CI. This is the last post in the series, and it covers something that matters more the longer a framework lives. Keeping it healthy after the initial build is done.

A framework that looks clean on day one can turn into a mess after six months of multiple people adding tests under deadline pressure. Inconsistent formatting, selectors that break constantly, tests nobody trusts anymore. None of this is really a Playwright problem. It is a team habits problem, and it is solvable with the right tooling in place from early on.

## ESLint and Prettier

Consistent formatting removes an entire category of pointless pull request comments. Nobody should be leaving a review comment about spacing when a tool can just fix it automatically. Install both alongside the TypeScript ESLint tooling.

```bash
npm install --save-dev eslint prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-config-prettier
```

A reasonable starting ESLint config for a Playwright TypeScript project looks like this.

```javascript
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier',
  ],
  rules: {
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'off',
    'no-console': 'warn',
  },
  env: {
    node: true,
    es2022: true,
  },
};
```

And a Prettier config, kept deliberately small.

```json
// .prettierrc
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100
}
```

Add a couple of scripts to `package.json` so these are easy to run manually and in CI.

```json
{
  "scripts": {
    "lint": "eslint . --ext .ts",
    "format": "prettier --write ."
  }
}
```

## Enforcing It With Husky and lint-staged

Having linting available is one thing. Having it actually run before broken code gets committed is another. Husky and lint-staged together give you exactly that, without slowing down every commit by re-checking the entire codebase.

```bash
npm install --save-dev husky lint-staged
npx husky init
```

Configure lint-staged in `package.json` to only touch files that are actually staged for commit.

```json
{
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"]
  }
}
```

Then wire it into the pre-commit hook Husky just created.

```bash
# .husky/pre-commit
npx lint-staged
```

Now every commit automatically lints and formats only the files being committed, and fixes what it can fix automatically. Anyone on the team gets this for free the moment they clone the repo and run `npm install`, since Husky's setup hooks into `npm install` itself.

It is worth going a step further and also running the test suite itself, or at least a fast subset of it, in a pre-push hook rather than pre-commit, since a full Playwright run is too slow to sit in front of every single commit.

```bash
# .husky/pre-push
npm run test:smoke
```

Where `test:smoke` is a small, fast tagged subset of your suite, not the full three hundred test run. This catches an obviously broken change before it even reaches a pull request, without making every commit feel painfully slow.

## Selector Governance

Tooling handles formatting and syntax, but it cannot stop someone from writing a brittle selector. This has to be a team agreement, backed by a bit of review discipline.

The rule I push hardest on any team is preferring dedicated test attributes over anything tied to styling or DOM structure.

```typescript
// Fragile. Breaks the moment a class name changes for styling reasons.
page.locator('.btn.btn-primary.submit-btn');

// Fragile. Breaks if the DOM structure shifts even slightly.
page.locator('div > form > div:nth-child(3) > button');

// Stable. Survives styling and structural changes.
page.locator('[data-testid="submit-order-button"]');
```

A `data-testid` attribute exists for exactly one purpose, and nobody refactoring styles or restructuring markup has a reason to touch it. This one habit alone prevents most of the selector breakage that causes maintenance headaches down the line. It does mean getting the front end team on board with adding these attributes, which is worth raising early rather than working around with fragile selectors indefinitely.

## Reviewing Flaky Tests on Purpose

The last piece of governance worth setting up deliberately is a recurring, scheduled look at flaky tests. It is easy for a team to develop a habit of just hitting rerun when a test fails intermittently, without ever circling back to actually fix it. Over months, this quietly erodes trust in the whole suite, to the point where a real failure gets dismissed as "probably just flaky" without anyone checking.

A simple habit that works well is a short recurring review, maybe every couple of weeks, where someone looks specifically at which tests needed a retry to pass over that period. Most of the time this points at one of a small number of root causes. A missing wait for a genuinely async operation that auto-waiting cannot see, like a background job that finishes seconds after the UI stops loading. A shared piece of test data that occasionally collides, which we covered back in part three. Or a selector that matches more than one element under specific conditions. Treating this as a regular, expected part of maintaining the framework, rather than something that only gets attention when it becomes a crisis, is what keeps a suite trustworthy for the long haul.

## Wrapping Up the Series

Over these ten posts we went from an empty folder to a framework with a clean architecture, a proper Page Object Model, solid test data handling, fixtures for setup and teardown, reliable waiting behavior, a hybrid API and UI testing approach, rich failure diagnostics, a working CI pipeline, fast parallel execution across shards, and now the governance habits to keep all of it healthy over time.

None of this needs to happen all at once on a real project. Pick it up in roughly this order, and each piece builds cleanly on the one before it, the same way we walked through it here.
