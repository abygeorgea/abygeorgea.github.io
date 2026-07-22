---
title: "Load, Stress, Soak, and Spike Testing: Choosing the Right Injection Strategy"
date: 2026-04-16T07:30:00+10:00
draft: false
slug: "load-stress-soak-spike-testing-with-gatling"
categories:
  - Gatling
description: "The difference between load, stress, soak, and spike testing, and how to configure each type of performance test in Gatling."
---

In the [previous post](/blog/2026/04/02/modeling-realistic-user-journeys-in-gatling/), we made scenarios behave more like real users. With everything we have built so far, injection profiles, feeders, correlation, checks, and pacing, we finally have enough pieces to talk about the different types of performance tests properly, since "run a load test" actually covers several genuinely different testing goals.

## Load Testing: Expected Traffic

A load test answers a simple question. Does the system perform acceptably under the traffic level you actually expect. This is the baseline test you should have running regularly, ideally on every significant release.

```java
setUp(
    checkoutScenario.injectOpen(
        rampUsersPerSec(1).to(20).during(Duration.ofMinutes(3)),
        constantUsersPerSec(20).during(Duration.ofMinutes(15))
    )
).protocols(httpProtocol);
```

Ramp up to your expected peak rate, hold it there long enough to see stable behavior, and check response times and error rates stay within your targets throughout. Twenty users per second here is a stand in for whatever your actual expected peak traffic looks like, based on real analytics rather than a guess.

## Stress Testing: Finding the Breaking Point

A stress test deliberately pushes past expected traffic to find out where the system actually starts to fail, and how it fails when it does. This is not about proving the system is fine. It is about understanding its limits before a real traffic spike finds them for you.

```java
setUp(
    checkoutScenario.injectOpen(
        rampUsersPerSec(1).to(100).during(Duration.ofMinutes(10))
    )
).protocols(httpProtocol);
```

Ramp well beyond your expected peak, in this case to a rate five times higher than the load test above, and watch closely for where things start to degrade. Response times climbing steadily is one signal. A sudden spike in error rates is another. The most useful outcome of a stress test is not a pass or fail, it is knowing exactly which component buckles first, whether that is a database connection pool, a downstream API, or the application server itself running out of threads.

## Soak Testing: Long Duration Stability

A soak test, sometimes called an endurance test, runs a moderate and sustainable load for a long duration, often several hours, looking for problems that only show up over time. Memory leaks, slow resource exhaustion, log files filling up disk space, database connections that never quite get released properly.

```java
setUp(
    checkoutScenario.injectOpen(
        rampUsersPerSec(1).to(10).during(Duration.ofMinutes(5)),
        constantUsersPerSec(10).during(Duration.ofHours(4))
    )
).protocols(httpProtocol);
```

The load level here is deliberately moderate, well within normal capacity. The point is not to stress the system, it is to give slow, gradual problems enough time to actually surface. A memory leak that adds a few megabytes per hour is invisible in a fifteen minute load test and very visible after four hours.

## Spike Testing: Sudden Bursts

A spike test checks how a system handles a sudden, sharp increase in traffic, then how it recovers once that spike passes. This is the closest fit for `atOnceUsers`, the profile we were cautious about back in part three.

```java
setUp(
    checkoutScenario.injectOpen(
        constantUsersPerSec(5).during(Duration.ofMinutes(5)),
        atOnceUsers(300),
        constantUsersPerSec(5).during(Duration.ofMinutes(5))
    )
).protocols(httpProtocol);
```

This holds a light baseline load, throws three hundred users in all at once to simulate something like a flash sale opening or a marketing email going out to a large list, and then drops back to baseline. What you want to see here is that the system survives the spike without falling over entirely, and that it recovers cleanly back to normal response times once the spike passes, rather than staying degraded long after the burst of traffic is gone.

## Choosing the Right Test for the Right Question

None of these test types replace the others, and running only one of them gives you an incomplete picture. A load test tells you whether normal operation is healthy. A stress test tells you where the ceiling is. A soak test tells you whether the system stays healthy over time. A spike test tells you how gracefully the system handles sudden change. A mature performance testing practice runs all four at different points in a release cycle, load tests frequently since they are quick, and stress, soak, and spike tests at a slower cadence since they take longer and are usually run against a more production like environment.

## Wrapping Up

The injection profile you choose is really a direct expression of the question you are trying to answer. Being explicit about which of these four test types you are running, before you write a single line of Gatling code, keeps the results focused and the report meaningful to whoever reads it afterward.

Next time, we look at turning "the results looked fine to me" into something objective, by setting explicit performance SLAs with Gatling's assertion API.
