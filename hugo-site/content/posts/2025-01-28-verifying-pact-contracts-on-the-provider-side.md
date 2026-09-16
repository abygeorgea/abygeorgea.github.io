---
title: "Verifying Pact Contracts on the Provider Side"
date: 2025-01-28T08:25:31+10:00
draft: false
slug: "verifying-pact-contracts-on-the-provider-side"
categories:
  - Pact
description: "Replaying a Pact contract against the real provider service, setting up provider states, and reading a verification failure."
---

In the [previous post](/blog/2025/01/21/writing-your-first-pact-consumer-test/), checkout's test suite generated a real contract file describing two interactions with the payment service, an authorized payment and a declined one. That file sitting in `target/pacts` proves nothing about payment's actual behavior yet. This post covers the other half of Pact, taking that same file and replaying it against payment's real implementation.

## Adding the Provider Dependency

On the payment service side, add Pact's provider JVM module.

```xml
<dependency>
    <groupId>au.com.dius.pact.provider</groupId>
    <artifactId>junit5</artifactId>
    <version>4.6.5</version>
    <scope>test</scope>
</dependency>
```

## Pointing Pact at Your Real Service

A provider verification test starts your actual Spring Boot application, or at least the controller layer, and lets Pact fire the recorded requests at it directly.

```java
@Provider("payment-service")
@PactFolder("../checkout-service/target/pacts")
class PaymentServiceProviderPactTest {

    @BeforeEach
    void setUp(PactVerificationContext context) {
        context.setTarget(new HttpTestTarget("localhost", 8080));
    }

    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void pactVerificationTestTemplate(PactVerificationContext context) {
        context.verifyInteraction();
    }
}
```

`@PactFolder` points at wherever the contract file lives. In a real setup this usually points at a Pact Broker instead of a local folder path, which we get to in the next post, but a local folder is the simplest way to see verification working end to end first.

## Provider States

Notice the contract includes a `providerState` of "a valid card is presented" for the authorized interaction, and "a card with insufficient funds is presented" for the declined one. Pact calls these provider states, and they exist because payment's real behavior depends on data that does not exist by default. There is no card with insufficient funds sitting in a database unless you put one there.

A `@State` method handles that setup before the matching interaction runs.

```java
@State("a valid card is presented")
void validCardPresented() {
    testCardRepository.save(new TestCard("4111111111111111", Status.VALID));
}

@State("a card with insufficient funds is presented")
void insufficientFundsCard() {
    testCardRepository.save(new TestCard("4111111111111111", Status.INSUFFICIENT_FUNDS));
}
```

This is the piece that trips people up first. Provider states are not decorative text, they are an instruction to the provider test suite, telling it exactly what data or system state needs to exist for the following interaction to make sense.

## Running Verification

```bash
mvn test -Dtest=PaymentServiceProviderPactTest
```

When this passes, Pact prints a summary confirming both interactions matched, request and response shape included. When it fails, the output tells you precisely which field did not match and how.

```
Verifying a pact between checkout-service and payment-service
  a request to authorize a payment
    returns a response which
      has status code 200 (OK)
      has a matching body (FAILED)

Failures:
1) Verifying a pact between checkout-service and payment-service - a request to authorize a payment
    Actual: {"result": "AUTHORIZED", "authCode": "AB1234"}
    Expected: {"status": "AUTHORIZED", "authCode": "AB1234"}
    $.body.status: Expected 'AUTHORIZED' but got no value
```

That specific failure, `result` instead of `status`, is exactly the kind of drift that would otherwise sit undetected until checkout deploys and its response parsing quietly breaks. Here it fails payment's own build, on payment's own pipeline, before it ever ships.

## Where This Still Falls Short

Right now both teams have to manually copy contract files around, or point at a shared local folder, which does not scale past a single pair of services talking directly to each other. The next post fixes that by introducing a Pact Broker, a shared service both consumer and provider publish to and verify against, so this whole exchange happens automatically as part of each team's own CI pipeline.
