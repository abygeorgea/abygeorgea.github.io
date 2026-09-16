---
title: "Setting Up Pact for a Spring Boot Consumer"
date: 2025-01-14T08:25:31+10:00
draft: false
slug: "setting-up-pact-with-spring-boot"
categories:
  - Pact
description: "Adding Pact JVM to a Spring Boot project, the project layout it expects, and the first empty consumer test skeleton."
---

In the [previous post](/blog/2025/01/07/consumer-driven-contract-testing-explained/), we covered why contract testing exists and where Pact fits between unit tests and full integration tests. Before we can write an actual contract, we need Pact wired into the project. This post is the setup step, getting a Spring Boot service ready to generate its first contract.

## Adding the Dependency

Pact JVM ships a JUnit 5 module that plugs straight into the test framework you are probably already using. For a Maven project, add this to the checkout service's `pom.xml`.

```xml
<dependency>
    <groupId>au.com.dius.pact.consumer</groupId>
    <artifactId>junit5</artifactId>
    <version>4.6.5</version>
    <scope>test</scope>
</dependency>
```

If you are on Gradle, the equivalent is a single line in your test dependencies.

```groovy
testImplementation 'au.com.dius.pact.consumer:junit5:4.6.5'
```

Nothing else needs to change in the main source set. Pact only touches your test code, which is one of the reasons it is such a low friction addition to an existing service.

## Project Layout

Pact generates contract files into a `pacts` directory at the root of your build output by default, usually `target/pacts` for Maven or `build/pacts` for Gradle. You do not need to create this directory yourself, Pact creates it the first time a consumer test runs.

It is worth deciding early where these generated files eventually live long term. For now, keep them local and out of version control. Once we introduce the Pact Broker in a later post, that becomes the shared home for contracts instead of a folder in your repo.

```
.gitignore
target/pacts/
```

## The Test Skeleton

A Pact consumer test looks close to a normal JUnit 5 test, with two extra pieces. A Pact extension that manages a mock provider server for you, and an annotation that defines what the mock should return.

```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "payment-service")
class PaymentServiceConsumerPactTest {

    @Pact(consumer = "checkout-service")
    public RequestResponsePact authorizePayment(PactDslWithProvider builder) {
        return builder
            .given("a valid card is presented")
            .uponReceiving("a request to authorize a payment")
            .path("/payments/authorize")
            .method("POST")
            .body("{\"amount\": 4999, \"currency\": \"AUD\"}")
            .willRespondWith()
            .status(200)
            .body("{\"status\": \"AUTHORIZED\", \"authCode\": \"AB1234\"}")
            .toPact();
    }
}
```

Nothing here talks to a real payment service. Pact spins up a mock HTTP server that returns exactly what you defined in `willRespondWith`, and the test method you attach to this contract, which we will write next, calls your actual client code against that mock server instead of the real one.

## Why the Mock Server Matters

The mock server is the part that makes this different from a plain unit test with a stubbed HTTP client. Because Pact is the one serving the response, it also records exactly what request your client sent, and that recorded interaction is what becomes the contract file. You are not writing the contract by hand, you are writing a test the normal way and letting Pact capture the shape of the conversation for you.

That is the piece we build next, an actual test method that calls your checkout service's payment client and asserts against the mock, generating a real contract file in the process.
