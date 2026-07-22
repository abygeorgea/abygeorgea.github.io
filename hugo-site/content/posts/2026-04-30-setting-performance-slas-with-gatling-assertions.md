---
title: "Setting Performance SLAs With Gatling Assertions"
date: 2026-04-30T07:30:00+10:00
draft: false
slug: "setting-performance-slas-with-gatling-assertions"
categories:
  - Gatling
description: "Using Gatling's global and per-request assertions to turn subjective load test results into an objective pass or fail build."
---

In the [previous post](/blog/2026/04/16/load-stress-soak-spike-testing-with-gatling/), we covered the different types of performance tests. Today we cover something that turns any of those tests from "someone eyeballs the report and makes a judgment call" into an objective, automatable pass or fail result. Assertions.

Without assertions, a Gatling run finishes and hands you a report, and a human has to decide whether the numbers in it are acceptable. That does not scale, and it definitely does not work inside a CI pipeline where nobody is watching the run happen live. Assertions let you encode your performance requirements directly into the simulation, so the build itself fails when those requirements are not met.

## Global Assertions

Assertions attach to the `setUp` call, alongside the protocol.

```java
setUp(
    checkoutScenario.injectOpen(
        rampUsersPerSec(1).to(20).during(Duration.ofMinutes(3)),
        constantUsersPerSec(20).during(Duration.ofMinutes(15))
    )
).protocols(httpProtocol)
 .assertions(
    global().responseTime().percentile3().lt(1500),
    global().successfulRequests().percent().gt(99.0)
);
```

This says two things at once. The ninety fifth percentile response time across every single request in the whole run must be under one and a half seconds, and at least ninety nine percent of all requests must succeed. If either condition fails, Gatling exits with a non zero status code, which is exactly the signal a CI pipeline needs to fail the build automatically.

## Why Percentiles Instead of Averages

It is worth being deliberate about using a percentile rather than an average here. An average hides outliers completely. If ninety percent of your users get a response in two hundred milliseconds and ten percent wait eight full seconds, the average might still look perfectly reasonable, while a meaningful chunk of your real users are having a genuinely bad experience. A percentile like the ninety fifth or ninety ninth tells you what the slower end of your user base is actually experiencing, which is almost always the more useful number for a real SLA.

Gatling gives you access to several percentiles directly.

```java
global().responseTime().percentile1().lt(500)   // 50th percentile, the median
global().responseTime().percentile2().lt(900)   // 75th percentile
global().responseTime().percentile3().lt(1500)  // 95th percentile
global().responseTime().percentile4().lt(3000)  // 99th percentile
```

A common pattern is asserting on more than one percentile at once, a tight bound on the median for the typical experience, and a looser bound on the ninety ninth percentile to catch the genuinely bad outliers without being so strict that normal variance fails the build constantly.

## Per-Request Assertions

Global assertions cover the whole simulation, but sometimes one specific request matters more than the rest, and deserves its own explicit bound. You can scope an assertion down to a single named request.

```java
.assertions(
    global().responseTime().percentile3().lt(1500),
    global().successfulRequests().percent().gt(99.0),
    details("Checkout").responseTime().percentile3().lt(2000),
    details("Checkout").failedRequests().percent().lt(1.0)
);
```

`details("Checkout")` scopes the assertion to just the request named "Checkout" in your scenario. This matters because a single slow endpoint can easily hide inside a healthy looking global average across dozens of other fast requests. If checkout specifically is the part of the journey your business cares most about, it deserves its own explicit, and often stricter, threshold.

## Assertions on Throughput

Assertions are not limited to timing and success rate. You can also assert on the request throughput the system actually achieved during the run.

```java
.assertions(
    global().requestsPerSec().gt(15.0)
);
```

This is a useful sanity check for a scenario where you expect the system to sustain a certain throughput. If the assertion fails, it usually means the system could not keep up with the intended injection rate, which is a meaningful finding on its own, separate from whatever the individual response times looked like.

## Making SLAs a Team Conversation, Not a Guess

The numbers themselves matter less than where they come from. Picking a response time threshold because it sounds reasonable is a weak foundation for an SLA. A better approach pulls the number from somewhere real, an existing production monitoring dashboard showing what current response times actually look like, a documented business requirement, or a competitor benchmark if the application is customer facing. Whatever the source, write it down and treat the assertion as living documentation of an agreed target, not just a number embedded in test code that nobody remembers agreeing to six months later.

## Wrapping Up

Assertions turn a load test from something that produces a report someone has to interpret into something that produces a clear, automatable pass or fail result. Combine global assertions with per-request thresholds on the parts of the journey that matter most, and you get a build that fails loudly the moment performance genuinely regresses.

Next time, we go back to the HTML report itself and actually learn how to read it properly, since a passing assertion does not mean there is nothing worth investigating in the details.
