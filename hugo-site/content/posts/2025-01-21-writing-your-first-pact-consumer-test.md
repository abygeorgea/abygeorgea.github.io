---
title: "Writing Your First Pact Consumer Test"
date: 2025-01-21T08:25:31+10:00
draft: false
slug: "writing-your-first-pact-consumer-test"
categories:
  - Pact
description: "Turning a Pact contract definition into a real test method, calling actual client code against the mock server, and generating a contract file."
---

In the [previous post](/blog/2025/01/14/setting-up-pact-with-spring-boot/), we added the Pact JVM dependency and sketched out a contract definition for authorizing a payment. That definition on its own does not generate anything yet. We still need a test method that actually exercises it, by calling real client code against Pact's mock server. That is what turns a description of an interaction into a generated contract file on disk.

## Connecting the Test to the Mock

Pact's JUnit 5 extension injects a `MockServer` into your test method, giving you the actual host and port the mock is running on. Your job is to point your real HTTP client at that address instead of the real payment service.

```java
@Test
@PactTestFor(pactMethod = "authorizePayment")
void authorizesAPaymentSuccessfully(MockServer mockServer) {
    PaymentClient client = new PaymentClient(mockServer.getUrl());

    PaymentResponse response = client.authorize(
        new PaymentRequest(4999, "AUD")
    );

    assertEquals("AUTHORIZED", response.status());
    assertEquals("AB1234", response.authCode());
}
```

`PaymentClient` here is the same class checkout uses in production to call the payment service, the only difference is the base URL, which now points at Pact's mock instead of a real environment. If that client class does not exist yet, this is a good forcing function to write it, since the whole point of contract testing is exercising real client code, not a stand in.

## Running the Test

Run this the same way you run any other JUnit test.

```bash
mvn test -Dtest=PaymentServiceConsumerPactTest
```

Assuming the assertions pass, look in `target/pacts`. You should see a file named something like `checkout-service-payment-service.json`, containing exactly the interaction you defined, request and response both, in Pact's standard contract format.

```json
{
  "consumer": { "name": "checkout-service" },
  "provider": { "name": "payment-service" },
  "interactions": [
    {
      "description": "a request to authorize a payment",
      "providerState": "a valid card is presented",
      "request": {
        "method": "POST",
        "path": "/payments/authorize",
        "body": { "amount": 4999, "currency": "AUD" }
      },
      "response": {
        "status": 200,
        "body": { "status": "AUTHORIZED", "authCode": "AB1234" }
      }
    }
  ],
  "metadata": { "pactSpecification": { "version": "2.0.0" } }
}
```

## Keeping Contracts Focused

It is tempting to write one enormous test that covers every field and every status code payment might return. Resist that. A contract should describe interactions checkout genuinely relies on, not a full specification of payment's API. If checkout never reads a `processedAt` timestamp from the response, do not assert on it in the contract, since that just gives payment one more thing they cannot change without breaking a consumer that never actually cared.

A second, separate test method for a declined card is worth adding now, since it is a real path checkout has to handle differently.

```java
@Pact(consumer = "checkout-service")
public RequestResponsePact declinedPayment(PactDslWithProvider builder) {
    return builder
        .given("a card with insufficient funds is presented")
        .uponReceiving("a request to authorize a declined payment")
        .path("/payments/authorize")
        .method("POST")
        .body("{\"amount\": 4999, \"currency\": \"AUD\"}")
        .willRespondWith()
        .status(402)
        .body("{\"status\": \"DECLINED\", \"reason\": \"INSUFFICIENT_FUNDS\"}")
        .toPact();
}
```

Two interactions in one contract file now, a happy path and a realistic failure path, both driven from how checkout actually behaves.

## What This Contract Does Not Prove Yet

Right now, this contract only proves that checkout's client code correctly builds the request and correctly parses the response, against a mock that returns whatever we told it to. It says nothing about whether the real payment service actually behaves this way. That is the gap the next post closes, by taking this exact file and replaying it against payment's real implementation.
