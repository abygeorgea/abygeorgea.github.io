---
title: "Decoupling Data: Managing Test Inputs and Dynamic States"
date: 2024-02-23T07:30:00+11:00
draft: false
slug: "decoupling-data-playwright-test-data-management"
categories:
  - Playwright
tags:
  - Playwright
description: "Managing static and dynamic test data in Playwright with JSON files and @faker-js/faker, and avoiding data collisions in parallel runs."
---

In the [previous post](/blog/2024/02/14/scaling-maintenance-playwright-page-object-model/), we cleaned up how our tests interact with the UI using the Page Object Model. This time we look at a different kind of mess. Test data.

Here is a scenario that plays out on almost every team at some point. Two tests both try to register a new account using the email `test@example.com`. Run them one at a time and everything is fine. Run them at the same time in a parallel CI job, and one of them fails because the account already exists. Nobody touched the test code. The problem is the data.

A good Playwright framework treats test data as its own concern, separate from test logic. That means static reference data lives in files, and anything that needs to be unique gets generated fresh for every run.

## Externalizing Static Data

Some data genuinely does not change between runs. Country codes, product categories, a set of known test accounts that live permanently in a test environment. This kind of data belongs in a plain JSON file, not scattered across test files as string literals.

```json
// data/users.json
{
  "standardUser": {
    "username": "standard_user",
    "password": "SecretPass1!"
  },
  "adminUser": {
    "username": "admin_user",
    "password": "AdminPass1!"
  }
}
```

Loading it in a test is direct. TypeScript will even give you type checking on the shape of the file if you import it properly.

```typescript
import users from '../data/users.json';

test('admin can access the settings page', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login(users.adminUser.username, users.adminUser.password);
  // continue the test
});
```

This works well for anything that is genuinely fixed. It falls apart the moment two tests need their own unique version of that data at the same time.

## Generating Dynamic Data With Faker

For anything that needs to be unique per test run, like a new user registration or a new order, you want data generated on the fly. This is exactly what `@faker-js/faker` is built for. Install it alongside your other dev dependencies.

```bash
npm install --save-dev @faker-js/faker
```

Then use it right inside your test, or better, inside a small helper function.

```typescript
// utils/data-factory.ts
import { faker } from '@faker-js/faker';

export interface NewUser {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
}

export function createNewUser(): NewUser {
  return {
    firstName: faker.person.firstName(),
    lastName: faker.person.lastName(),
    email: faker.internet.email(),
    password: faker.internet.password({ length: 12 }),
  };
}
```

Now a registration test looks like this.

```typescript
import { createNewUser } from '../utils/data-factory';

test('a new user can register', async ({ page }) => {
  const newUser = createNewUser();
  const registerPage = new RegisterPage(page);

  await registerPage.goto();
  await registerPage.register(newUser);

  await expect(registerPage.successMessage).toBeVisible();
});
```

Run this test a hundred times in a hundred parallel workers, and every single one gets its own unique email address. No collisions, no cleanup needed between runs, and no more flaky failures caused by a duplicate account.

You can lean on Faker for a lot more than names and emails too. Addresses, phone numbers, company names, even realistic looking transaction amounts, all with sensible formats out of the box.

```typescript
const orderAmount = faker.finance.amount({ min: 10, max: 500, dec: 2 });
const shippingAddress = {
  street: faker.location.streetAddress(),
  city: faker.location.city(),
  postcode: faker.location.zipCode(),
};
```

## Scoping Data So Parallel Tests Do Not Collide

Faker solves the uniqueness problem for brand new data. But some tests need to work with existing records in a shared environment, and that brings its own risk. If two parallel tests both grab "the first product in the catalog" to add to a cart, you can end up with race conditions around stock counts or shared state.

A few practical rules I follow here.

Never hardcode an id or a record that another test might also be using. If a test needs an existing product, either seed one specifically for that test through the API, or query for one dynamically and use whatever comes back, rather than assuming record id 1 will always be free.

Prefer creating fresh data per test over reusing shared fixtures whenever the workflow allows it. A test that registers its own new user and then acts on that user's own data cannot collide with anything else running in parallel.

Where you truly must share a fixed data set, like a list of countries or currencies, treat it as read only. Nothing should ever be a test that mutates shared reference data, because the next parallel test relies on that data staying exactly as it was.

## Wrapping Up

Static data belongs in JSON files. Anything unique belongs in Faker. And anything shared across parallel tests should be treated as read only unless you have a very good reason not to.

Next time, we move on to fixtures. Playwright's `test.extend` gives you a clean way to inject page objects, authenticated sessions, and test data directly into your tests, and it removes a lot of repetitive setup code in the process.
