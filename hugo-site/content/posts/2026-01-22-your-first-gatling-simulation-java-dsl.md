---
title: "Your First Real Simulation: The Gatling Java DSL Explained"
date: 2026-01-22T07:30:00+11:00
draft: false
slug: "your-first-gatling-simulation-java-dsl"
categories:
  - Gatling
description: "A practical walkthrough of Gatling's Java DSL, covering exec, chaining, HTTP protocol configuration, and separating scenarios from simulations."
---

In the [previous post](/blog/2026/01/08/getting-started-gatling-java-project-setup/), we got a bare bones Gatling project running with a single request. That is enough to prove your setup works, but it is nowhere near what a real load test looks like. Today we build something closer to reality, a scenario with several requests chained together, and we talk properly about how the Java DSL fits together.

## How the DSL Reads

Gatling's Java DSL is built to be read almost like a script, top to bottom. Once you get used to the shape of it, most of what you write ends up looking like a sentence describing user behavior. The three static imports you will use constantly are these.

```java
import static io.gatling.javaapi.core.CoreDsl.*;
import static io.gatling.javaapi.http.HttpDsl.*;
```

`CoreDsl` gives you the general building blocks, things like `scenario`, `exec`, and `feed`, which we will get to in a later post. `HttpDsl` gives you everything specific to HTTP, like `http`, `status`, and the various header helpers. Almost every Gatling file you write starts with both of these.

## Configuring the HTTP Protocol Properly

The protocol builder is where shared HTTP behavior lives, so you are not repeating the same headers and settings on every single request. Here is a more complete version than the minimal one from the last post.

```java
HttpProtocolBuilder httpProtocol = http
    .baseUrl("https://api.example.com")
    .acceptHeader("application/json")
    .contentTypeHeader("application/json")
    .userAgentHeader("gatling-load-test")
    .maxConnectionsPerHost(20)
    .shareConnections();
```

`maxConnectionsPerHost` and `shareConnections` matter more than they look like they do. They control how Gatling manages its connection pool per virtual user, and getting this wrong is a common reason people see artificially bad response times that have nothing to do with the application under test and everything to do with the load generator itself starving for connections.

## Chaining Requests Into a Real Scenario

A scenario is a sequence of steps a virtual user performs. Chaining them together with `exec` reads naturally.

```java
ScenarioBuilder browseAndCheckout = scenario("Browse and Checkout")
    .exec(
        http("Get Product Catalog")
            .get("/products")
            .check(status().is(200))
    )
    .pause(2)
    .exec(
        http("View Product Detail")
            .get("/products/42")
            .check(status().is(200))
            .check(jsonPath("$.name").exists())
    )
    .pause(1)
    .exec(
        http("Add To Cart")
            .post("/cart/items")
            .body(StringBody("{\"productId\": 42, \"quantity\": 1}"))
            .check(status().is(201))
    )
    .pause(3)
    .exec(
        http("Checkout")
            .post("/checkout")
            .check(status().is(200))
            .check(jsonPath("$.orderId").exists())
    );
```

Every `exec` block is one HTTP call, and `pause` between them simulates a real user actually reading the page and deciding what to do next, rather than hammering the server with zero delay between requests. We will spend a whole post later in this series on getting pacing right, since it makes a big difference to how realistic your load actually is.

## Separating Scenarios From Simulations

Back in the folder structure from part one, we set aside a `scenarios` package separate from `simulations`. The reasoning is simple. A scenario describes a user journey. A simulation describes how many of those users you want, and when. Keeping them apart means you can reuse the same scenario across different load profiles without copying the request logic.

```java
// scenarios/CheckoutScenario.java
package scenarios;

import io.gatling.javaapi.core.ScenarioBuilder;

import static io.gatling.javaapi.core.CoreDsl.*;
import static io.gatling.javaapi.http.HttpDsl.*;

public class CheckoutScenario {

    public static ScenarioBuilder browseAndCheckout() {
        return scenario("Browse and Checkout")
            .exec(
                http("Get Product Catalog")
                    .get("/products")
                    .check(status().is(200))
            )
            .pause(2)
            .exec(
                http("View Product Detail")
                    .get("/products/42")
                    .check(status().is(200))
            )
            .pause(1)
            .exec(
                http("Add To Cart")
                    .post("/cart/items")
                    .body(StringBody("{\"productId\": 42, \"quantity\": 1}"))
                    .check(status().is(201))
            )
            .pause(3)
            .exec(
                http("Checkout")
                    .post("/checkout")
                    .check(status().is(200))
            );
    }
}
```

And the simulation class becomes short, focused only on load configuration, not request detail.

```java
// simulations/CheckoutLoadSimulation.java
package simulations;

import config.HttpProtocolConfig;
import io.gatling.javaapi.core.Simulation;
import scenarios.CheckoutScenario;

import static io.gatling.javaapi.core.CoreDsl.*;

public class CheckoutLoadSimulation extends Simulation {

    {
        setUp(
            CheckoutScenario.browseAndCheckout()
                .injectOpen(atOnceUsers(10))
        ).protocols(HttpProtocolConfig.httpProtocol);
    }
}
```

This mirrors a pattern you have probably seen in other kinds of test automation. Keep the thing that describes behavior separate from the thing that describes how much load to throw at it. It means a new load profile is a one line change in the simulation class, with zero risk of accidentally breaking the scenario logic itself.

## A Shared Protocol Config

While we are at it, let's move the protocol builder into its own class too, so every simulation in the project shares one consistent configuration.

```java
// config/HttpProtocolConfig.java
package config;

import io.gatling.javaapi.http.HttpProtocolBuilder;

import static io.gatling.javaapi.http.HttpDsl.*;

public class HttpProtocolConfig {

    public static final HttpProtocolBuilder httpProtocol = http
        .baseUrl("https://api.example.com")
        .acceptHeader("application/json")
        .contentTypeHeader("application/json")
        .userAgentHeader("gatling-load-test");
}
```

Now if the base URL ever needs to change, or a shared header needs adding, there is exactly one place to make that change across the whole framework.

## Wrapping Up

We now have a chained, multi step scenario, a clean separation between scenario logic and load setup, and a shared protocol configuration. This is starting to look like an actual framework rather than a single script.

Next time, we look at virtual users and injection profiles properly. `atOnceUsers` is the simplest possible profile, and there is a lot more control available once you need something closer to a realistic ramp up.
