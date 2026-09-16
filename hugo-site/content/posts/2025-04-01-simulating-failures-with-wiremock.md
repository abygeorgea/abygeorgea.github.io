---
title: "Simulating Failures and Latency with WireMock"
date: 2025-04-01T08:25:31+10:00
draft: false
slug: "simulating-failures-with-wiremock"
categories:
  - WireMock
description: "Using WireMock to simulate timeouts, delays, connection resets, and malformed responses, so resilience code actually gets exercised in tests."
---

In the [previous post](/blog/2025/03/18/getting-started-with-wiremock/), we stubbed a happy path response from an inventory service and verified our client built the right request. Happy paths are the easy part. What actually determines whether a service survives production is how it behaves when a downstream dependency does not cooperate, and that is much harder to test against a real dependency, since you cannot exactly ask another team's service to time out on demand. WireMock's fault simulation is built for exactly this.

## Simulating a Slow Response

Timeouts and circuit breakers are only worth having if something has actually tested that they trigger correctly. WireMock's fixed delay makes that possible without waiting on a genuinely slow network.

```java
wireMock.stubFor(get(urlEqualTo("/inventory/SKU-1042"))
    .willReturn(aResponse()
        .withStatus(200)
        .withBody("{\"sku\": \"SKU-1042\", \"available\": 37}")
        .withFixedDelay(3000)));

assertThrows(TimeoutException.class, () ->
    client.getStock("SKU-1042"));
```

If your client is configured with a two second timeout, this test proves that configuration actually does something, rather than just existing as a number in a properties file nobody has verified.

## Simulating Connection Failures

Beyond slow responses, WireMock can simulate the connection itself misbehaving, which is a different failure mode your resilience code needs to handle separately.

```java
wireMock.stubFor(get(urlEqualTo("/inventory/SKU-1042"))
    .willReturn(aResponse()
        .withFault(Fault.CONNECTION_RESET_BY_PEER)));
```

Other fault types are available for different scenarios, including `EMPTY_RESPONSE` for a connection that closes with nothing sent back at all, and `MALFORMED_RESPONSE_CHUNK` for a response that starts arriving and then breaks mid-stream. Each of these exercises a distinct code path in a well written HTTP client, and each one is close to impossible to reproduce reliably against a real service on demand.

## Simulating Malformed but Valid Responses

Not every bad response is a network level failure. Sometimes a service is up, responds successfully at the HTTP level, and just returns something your code does not expect.

```java
wireMock.stubFor(get(urlEqualTo("/inventory/SKU-1042"))
    .willReturn(aResponse()
        .withStatus(200)
        .withHeader("Content-Type", "application/json")
        .withBody("{\"sku\": \"SKU-1042\"}")));

InventoryLevel level = client.getStock("SKU-1042");
assertEquals(0, level.available());
```

That response is missing the `available` field entirely. This test is really checking one specific decision in your deserialization logic, does a missing field default sensibly or does it throw an unhandled exception that takes down the calling thread. Either answer might be correct depending on the service, but it should be a decision your test suite actually verifies, not something you discover the first time the real service has a partial outage.

## Chaining Failure Into Recovery

WireMock supports stateful stubs through scenarios, which let you test a full sequence, a failure followed by a successful retry, rather than just a single failure in isolation.

```java
wireMock.stubFor(get(urlEqualTo("/inventory/SKU-1042"))
    .inScenario("retry-then-succeed")
    .whenScenarioStateIs(STARTED)
    .willReturn(aResponse().withStatus(503))
    .willSetStateTo("retried-once"));

wireMock.stubFor(get(urlEqualTo("/inventory/SKU-1042"))
    .inScenario("retry-then-succeed")
    .whenScenarioStateIs("retried-once")
    .willReturn(aResponse()
        .withStatus(200)
        .withBody("{\"sku\": \"SKU-1042\", \"available\": 37}")));
```

The first call to this endpoint returns a 503. The second call, after the scenario state changes, returns success. If your client has retry logic, this is how you prove it actually retries and actually succeeds on the second attempt, rather than just trusting the retry annotation is configured correctly.

## Why This Matters More Than the Happy Path Ever Did

A payment or inventory service that only ever gets tested against cooperative dependencies will have resilience code that has never actually run in a test. Timeouts, retries, circuit breakers, and fallback logic are the parts of a system most likely to have a bug precisely because they are hardest to exercise naturally. Simulating the failure directly, rather than hoping it eventually happens in staging, is what turns that code from theoretical to actually verified.

The next post moves this out of individual test classes and into CI, running WireMock as a standalone server so integration tests across a whole pipeline can share the same stubbed environment.
