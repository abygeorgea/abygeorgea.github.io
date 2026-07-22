---
title: "Feeders 101: Driving Tests With CSV, JSON, and In-Memory Data"
date: 2026-02-19T07:30:00+11:00
draft: false
slug: "gatling-feeders-csv-json-data-driven-tests"
categories:
  - Gatling
description: "Using Gatling feeders to parameterize load tests with CSV, JSON, and custom Java data sources, without every virtual user hitting the exact same data."
---

In the [previous post](/blog/2026/02/05/gatling-scenarios-virtual-users-injection-profiles/), we looked at how many users to inject and when. Today we tackle a different problem. If every one of those users logs in with the same username, or adds the exact same product to their cart, you are not really testing realistic load. You are testing one specific code path over and over.

Feeders solve this by handing each virtual user its own piece of data before it runs through the scenario.

## CSV Feeders

The simplest and most common feeder reads from a CSV file. Put this in `src/test/resources/data/users.csv`.

```csv
username,password
user1,Password1!
user2,Password2!
user3,Password3!
```

Wiring it into a scenario looks like this.

```java
FeederBuilder<String> users = csv("data/users.csv").random();

ScenarioBuilder login = scenario("Login")
    .feed(users)
    .exec(
        http("Login")
            .post("/auth/login")
            .body(StringBody("{\"username\": \"#{username}\", \"password\": \"#{password}\"}"))
            .check(status().is(200))
    );
```

The `#{username}` and `#{password}` syntax is Gatling's expression language. It pulls the value the feeder just placed into the session for that virtual user. Calling `.random()` on the feeder means each user grabs a random row, rather than reading through the file in strict order, which matters once you have more virtual users than rows and Gatling needs to decide how to reuse them.

## Controlling How Rows Get Reused

Gatling gives you a few strategies for what happens when a feeder runs out of unique rows partway through a test.

```java
csv("data/users.csv").random();     // random row every time, can repeat
csv("data/users.csv").shuffle();    // shuffled once, then read in that order, wraps around
csv("data/users.csv").circular();   // reads top to bottom, wraps to the top when it runs out
```

If uniqueness genuinely matters, like a registration flow that needs a fresh username every single time, none of these strategies are quite right on their own, since all of them eventually repeat rows. That is exactly the situation where a small amount of dynamic generation, which we will cover next, becomes the better tool.

## JSON Feeders

For data with more structure than flat CSV columns comfortably allow, a JSON feeder works the same way. Put this in `src/test/resources/data/products.json`.

```json
[
  { "productId": 42, "name": "Wireless Mouse", "price": 29.99 },
  { "productId": 51, "name": "Mechanical Keyboard", "price": 89.99 },
  { "productId": 67, "name": "USB-C Hub", "price": 34.99 }
]
```

```java
FeederBuilder<Object> products = jsonFile("data/products.json").random();

ScenarioBuilder addToCart = scenario("Add To Cart")
    .feed(products)
    .exec(
        http("Add To Cart")
            .post("/cart/items")
            .body(StringBody("{\"productId\": #{productId}, \"quantity\": 1}"))
            .check(status().is(201))
    );
```

Each field in the JSON object becomes its own session variable, the same as a CSV column would, so `#{productId}` and `#{name}` both become available to reference anywhere later in the scenario.

## Generating Data In Memory With a Custom Feeder

Sometimes a static file is not the right fit, especially when you need genuinely unique values, like a fresh email address for every single registration attempt across a long running test. Gatling lets you build a feeder directly from a Java `Iterator`, which means you can generate values on demand rather than reading from a fixed file.

```java
import java.util.Iterator;
import java.util.Map;
import java.util.concurrent.atomic.AtomicInteger;

AtomicInteger counter = new AtomicInteger(0);

Iterator<Map<String, Object>> newUserFeeder = Stream.generate(() -> {
    int id = counter.incrementAndGet();
    return Map.<String, Object>of(
        "email", "loadtest_user" + id + "@example.com",
        "username", "loadtest_user" + id
    );
}).iterator();
```

```java
ScenarioBuilder register = scenario("Register")
    .feed(newUserFeeder)
    .exec(
        http("Register")
            .post("/auth/register")
            .body(StringBody("{\"email\": \"#{email}\", \"username\": \"#{username}\"}"))
            .check(status().is(201))
    );
```

Because the counter increments atomically, this stays safe even when many virtual users are pulling from the same feeder concurrently across multiple threads. Every user gets a genuinely unique email, no matter how long the test runs or how many users you throw at it.

## A Word on Feeder Scope

By default, a feeder is shared across every virtual user in the simulation, which is exactly what you want for something like the CSV example above, where you have a fixed pool of test accounts everyone draws from. Just be careful with feeders that represent something finite and stateful in the system under test, like a limited stock of a specific product. If your feeder hands out the same product id to two different virtual users at the same time, and your application only has one unit of that product in stock, you can end up with a test failure that has nothing to do with a real defect and everything to do with the test data itself being reused unsafely. This connects back to the data scoping concerns worth thinking through any time you parameterize a test against a shared environment.

## Wrapping Up

Feeders are what turn a single hardcoded request into something that represents a real population of users, each with their own data. CSV and JSON cover most static data needs, and a custom Java based feeder covers the cases where you need guaranteed uniqueness on demand.

Next time, we look at correlation and authentication together, since almost every real application requires handling a login flow and threading a token through the rest of the scenario.
