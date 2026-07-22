---
title: "Checks and Validations: Making Sure Your Load Test Catches Real Failures"
date: 2026-03-19T07:30:00+11:00
draft: false
slug: "gatling-checks-and-validations"
categories:
  - Gatling
description: "Going beyond status code checks in Gatling, with response body validation, header checks, and why a fast response is not the same as a correct one."
---

In the [previous post](/blog/2026/03/05/gatling-correlation-and-authentication/), we used checks to extract tokens for correlation. This time we look at checks purely from a validation angle, because a surprising number of load tests quietly pass while the application under test is actually broken.

## The Trap of Checking Status Codes Only

Here is a scenario I have seen play out more than once. A test checks only that every response comes back with a 200 status. Midway through the run, the application starts returning a generic error page, but that error page itself happens to render with a 200 status code, because the server is misconfigured to return success even for its own error pages. The load test finishes green. The report shows a great response time. Meanwhile every single user in that window got an error page instead of what they asked for.

Status codes are necessary, but they are not sufficient on their own for anything beyond the most trivial smoke test.

## Validating Response Content

Checking that the body actually contains what you expect closes this gap.

```java
.exec(
    http("Get Product Detail")
        .get("/products/42")
        .check(status().is(200))
        .check(jsonPath("$.name").is("Wireless Mouse"))
        .check(jsonPath("$.price").exists())
        .check(jsonPath("$.inStock").is("true"))
)
```

Now the check fails if the product name does not match, if the price field is missing entirely, or if the stock flag says the item is unavailable when your test expects it to be in stock. A response that looks superficially fine but has the wrong data underneath gets caught immediately.

## Checking Response Headers

Sometimes the thing you actually care about lives in a header rather than the body. Content type mismatches, caching headers, or a custom header your application uses for tracing are all fair game.

```java
.exec(
    http("Get Product Detail")
        .get("/products/42")
        .check(status().is(200))
        .check(header("Content-Type").is("application/json"))
        .check(header("X-Cache").is("HIT"))
)
```

That last check is a genuinely useful one for performance testing specifically. If you expect a cache layer to be serving most of your traffic, asserting on the cache header lets you confirm that assumption is actually true during the run, rather than guessing at it after the fact.

## Checking Response Time Per Request

Checks are not limited to correctness. You can assert on timing at the level of an individual request too, which is useful for catching a single slow endpoint hiding inside an otherwise healthy scenario.

```java
.exec(
    http("Search Products")
        .get("/products/search?q=mouse")
        .check(status().is(200))
        .check(responseTimeInMillis().lte(800))
)
```

This fails the specific request if it takes longer than eight hundred milliseconds, independent of whatever global assertions you have configured for the whole simulation, which we will cover properly in a later post on setting performance SLAs.

## Combining Multiple Checks on One Request

A single request can carry as many checks as it needs, and Gatling evaluates all of them.

```java
.exec(
    http("Checkout")
        .post("/checkout")
        .body(StringBody("{\"cartId\": \"#{cartId}\"}"))
        .check(status().is(200))
        .check(jsonPath("$.orderId").exists())
        .check(jsonPath("$.status").is("confirmed"))
        .check(jsonPath("$.total").ofType(Double.class).gt(0.0))
        .check(responseTimeInMillis().lte(2000))
)
```

This one request now confirms the call succeeded, an order id came back, the order status is actually confirmed rather than pending or failed, the total charged is a sensible positive number, and the whole thing happened inside a reasonable time budget. That is a load test that actually tells you something meaningful about the checkout flow, not just that a server responded.

## Deciding What to Check, Practically

Checking everything on every single request is not the goal, since overly strict checks can make a test brittle in ways that have nothing to do with performance, like failing because a non-critical field's formatting changed slightly. A practical rule I follow is to check the things that would represent a genuine failure from a user's point of view. Did the order actually go through. Did the search actually return results. Is the price actually correct. Save strict field by field validation for your functional test suite, and keep performance test checks focused on the handful of signals that tell you the request genuinely succeeded and returned something usable.

## Wrapping Up

A load test that only checks status codes can hide real problems behind a green summary. Validating response content, relevant headers, and per request timing where it matters turns a load test into something that actually catches failures, not just something that measures how fast a broken response comes back.

Next time, we step back from individual requests and look at scenario design as a whole, focusing on pacing and think time, and how to make a scenario actually behave like a real user rather than a script racing through steps as fast as possible.
