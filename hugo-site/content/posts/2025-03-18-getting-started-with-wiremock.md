---
title: "Getting Started with WireMock: Your First Stub"
date: 2025-03-18T08:25:31+10:00
draft: false
slug: "getting-started-with-wiremock"
categories:
  - WireMock
description: "Adding WireMock to a Spring Boot test suite, writing a first stub mapping, and matching on path, headers, and request body."
---

In the [previous post](/blog/2025/03/04/wiremock-vs-mountebank/), we looked at why WireMock's embedded mode is a strong fit for a Java test suite specifically. This post gets into the actual setup, adding WireMock to a Spring Boot project and writing a real stub against a downstream service.

## Adding the Dependency

```xml
<dependency>
    <groupId>org.wiremock</groupId>
    <artifactId>wiremock-standalone</artifactId>
    <version>3.9.1</version>
    <scope>test</scope>
</dependency>
```

The standalone artifact bundles everything needed to run WireMock either embedded in a test or as its own process later, which keeps things simple while we are just getting started.

## Starting WireMock in a Test

WireMock's JUnit 5 extension manages the server's lifecycle for you, starting it before each test class and stopping it afterward.

```java
@ExtendWith(WireMockExtension.class)
class InventoryClientTest {

    @RegisterExtension
    static WireMockExtension wireMock = WireMockExtension.newInstance()
        .options(wireMockConfig().port(8089))
        .build();

    private final InventoryClient client = new InventoryClient("http://localhost:8089");
}
```

Point your real client's base URL at `localhost:8089`, the same way you would point it at any other environment. Nothing about the client code itself needs to know it is talking to a mock.

## Writing a Stub

A stub tells WireMock what request to expect and what response to return when it sees one that matches.

```java
@Test
void returnsAvailableStock() {
    wireMock.stubFor(get(urlEqualTo("/inventory/SKU-1042"))
        .willReturn(aResponse()
            .withStatus(200)
            .withHeader("Content-Type", "application/json")
            .withBody("{\"sku\": \"SKU-1042\", \"available\": 37}")));

    InventoryLevel level = client.getStock("SKU-1042");

    assertEquals(37, level.available());
}
```

Run that test and the client makes a real HTTP call, just against a mock server instead of the actual inventory service. WireMock matches the incoming request against every stub it knows about and returns the first one that fits.

## Matching on More Than the URL

Real requests carry headers, query parameters, and bodies, and WireMock can match on all of them, which matters once you need different stubs for slightly different inputs.

```java
wireMock.stubFor(post(urlEqualTo("/inventory/reserve"))
    .withHeader("Authorization", matching("Bearer .*"))
    .withRequestBody(matchingJsonPath("$.sku", equalTo("SKU-1042")))
    .withRequestBody(matchingJsonPath("$.quantity", equalTo("5")))
    .willReturn(aResponse()
        .withStatus(200)
        .withBody("{\"reservationId\": \"RES-9981\"}")));
```

`matchingJsonPath` is worth calling out specifically. Rather than matching the entire request body as one exact string, which breaks the moment field order or whitespace changes, it lets you assert on individual fields within a JSON body, which is a much more resilient way to define a stub.

## Verifying What Your Client Actually Sent

Stubbing the response is half the value. WireMock also lets you verify, after the fact, exactly what your client sent, which turns a stub into a genuine test of your client's request building logic, not just its response parsing.

```java
wireMock.verify(postRequestedFor(urlEqualTo("/inventory/reserve"))
    .withRequestBody(matchingJsonPath("$.quantity", equalTo("5"))));
```

If your client has a bug that sends the wrong field name or the wrong content type, this verification catches it, even though the stubbed response would have returned successfully regardless.

## Where This Leaves Us

At this point WireMock is standing in cleanly for a happy path dependency, matching requests precisely and letting us verify outbound calls. Real dependencies do not only return happy paths though. They time out, they return malformed responses, and they occasionally just fall over. The next post covers simulating exactly those failure conditions, since that is where service virtualization earns its keep the most.
