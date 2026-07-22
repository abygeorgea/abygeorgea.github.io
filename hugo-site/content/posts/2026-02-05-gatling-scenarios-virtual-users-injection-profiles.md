---
title: "Scenarios and Virtual Users: Understanding Injection Profiles"
date: 2026-02-05T07:30:00+11:00
draft: false
slug: "gatling-scenarios-virtual-users-injection-profiles"
categories:
  - Gatling
description: "A guide to Gatling's injection profiles, from atOnceUsers and rampUsers to constantUsersPerSec and staged load patterns."
---

In the [previous post](/blog/2026/01/22/your-first-gatling-simulation-java-dsl/), we built a proper chained scenario and separated it cleanly from load setup. Today we focus entirely on that load setup, because how you inject virtual users into a scenario has a huge effect on what your test actually measures.

A lot of people new to Gatling reach for `atOnceUsers` and stop there. It has its place, but it does not represent how real traffic behaves, and using it for everything will give you misleading results.

## atOnceUsers: The Blunt Instrument

`atOnceUsers` fires every single virtual user at the exact same instant.

```java
scn.injectOpen(atOnceUsers(50))
```

This is useful for exactly one thing, deliberately testing how your system handles a sudden burst, like a flash sale starting at midnight or a cache expiring across your whole user base at once. Outside of that specific scenario, it is a poor default, because it puts an unrealistic instantaneous spike on your system that most real world traffic never actually produces.

## rampUsers: A Gradual Increase

`rampUsers` spreads a fixed number of users evenly across a time window, arriving progressively rather than all at once.

```java
scn.injectOpen(rampUsers(200).during(Duration.ofMinutes(5)))
```

This ramps two hundred users in over five minutes. It is a solid default for a basic load test, since it gives your system time to warm up caches and scale connection pools naturally, the same way real traffic tends to build up over the course of a morning rather than appearing instantly.

## constantUsersPerSec: A Steady Arrival Rate

Sometimes you care less about a fixed total number of users and more about a consistent rate of new arrivals, which maps closely to how you would describe real traffic in terms of requests per second.

```java
scn.injectOpen(
    constantUsersPerSec(10).during(Duration.ofMinutes(10))
)
```

This injects ten new users every second, for ten minutes straight. It is a good fit when your capacity planning is expressed in terms of throughput, like "we need to handle two hundred checkouts per minute during a sale."

## rampUsersPerSec: Building Up a Rate Gradually

Combine the idea of a ramp with the idea of a rate, and you get `rampUsersPerSec`, which increases the arrival rate smoothly over time rather than jumping straight to a fixed rate.

```java
scn.injectOpen(
    rampUsersPerSec(1).to(20).during(Duration.ofMinutes(5))
)
```

This starts at one new user per second and climbs steadily to twenty per second over five minutes. This is one of the most realistic shapes for a genuine load test, since it mimics traffic gradually building through a busy period instead of appearing as a step function.

## Staging Multiple Phases Together

Real world traffic rarely follows one single pattern for an entire test. A more realistic run stages several profiles back to back, and Gatling lets you chain injection steps directly.

```java
scn.injectOpen(
    rampUsersPerSec(1).to(10).during(Duration.ofMinutes(2)),
    constantUsersPerSec(10).during(Duration.ofMinutes(10)),
    rampUsersPerSec(10).to(1).during(Duration.ofMinutes(2))
)
```

This warms up over two minutes, holds a steady sustained load for ten minutes, and then ramps back down over two minutes. This shape, warm up, hold, cool down, is close to what I use as a starting template for most sustained load tests, and it produces far more useful data than a single flat profile, because you can clearly see in the report how the system behaves during ramp up versus during a sustained steady state.

## Open Versus Closed Models

Everything above uses `injectOpen`, which describes an open workload model. New users arrive according to a schedule you define, completely independent of how quickly existing users finish their scenarios. This matches most public facing web traffic well, since real visitors do not wait for someone else to finish before showing up.

Gatling also supports a closed model through `injectClosed`, where you specify a fixed number of concurrent users active at any time, and a new virtual user only starts once an existing one finishes.

```java
scn.injectClosed(
    constantConcurrentUsers(50).during(Duration.ofMinutes(10))
)
```

This fits systems where concurrency itself is the constraint you care about, like an internal tool used by a fixed size support team, or a system in front of a resource pool with a hard concurrent connection limit. Choosing between open and closed models is really a question about what your real traffic actually looks like, not a question of which one is technically more advanced.

## Picking a Profile That Matches Your Goal

Before writing any injection profile, it is worth being explicit about what question the test is trying to answer. If the question is "can we survive a sudden spike," reach for `atOnceUsers` on top of some existing baseline load. If the question is "how does the system behave under steady sustained traffic," a staged ramp up, hold, and ramp down like the example above is the right shape. If the question is about a hard concurrency limit somewhere in the system, a closed model fits better than an open one.

## Wrapping Up

Injection profiles are not just a technical detail, they define what your test is actually measuring. Match the shape of the profile to the real traffic pattern or the specific question you are trying to answer, and the results will actually mean something.

Next time, we look at feeders, and how to drive a scenario with real data instead of hardcoded values, so every virtual user is not making the exact same request.
