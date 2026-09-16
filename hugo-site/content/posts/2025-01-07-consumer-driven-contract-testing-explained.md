---
title: "Consumer-Driven Contract Testing Explained"
date: 2025-01-07T08:25:31+10:00
draft: false
slug: "consumer-driven-contract-testing-explained"
categories:
  - Pact
description: "What consumer-driven contract testing actually solves, and why it fits microservices better than end-to-end tests or shared integration environments."
---

Every team running more than a handful of microservices eventually hits the same wall. Integration tests that spin up three or four real services are slow and flaky. End-to-end tests that go through the whole system catch real problems, but only after everything is already deployed, and a single unrelated service being down fails the whole suite. Somewhere in between those two extremes sits a much cheaper question. Does my service still honor what the other services expect from it.

That question is what contract testing answers, and Pact is the tool I want to spend the next few posts on.

## The Problem With Testing Microservices in Isolation

Say you own a checkout service that calls a payment service to authorize a transaction. You write unit tests for checkout with the payment call mocked out. The payment team writes their own unit tests for their service. Both suites go green. Both services deploy. And then checkout starts failing in production because the payment team renamed a field in their response, or changed a status code they return on a declined card.

Nobody lied. Nobody skipped testing. The two services just drifted apart quietly, because nothing was checking that the mock checkout used still matched what payment actually returns.

## What a Contract Actually Is

A contract, in this context, is a recorded set of expectations. The consumer, checkout in this example, states what requests it will send and what responses it expects back. That expectation gets captured as a contract file. The provider, payment, then replays those exact requests against its own real implementation and checks the responses still match.

This is the part that makes it consumer-driven. The contract is not written by the provider team guessing what consumers might need. It comes directly from how consumers actually use the service, which means it only ever covers real usage, not a full specification of every endpoint.

## Where Pact Fits

Pact is the most common tool for this pattern, with solid support across Java, JavaScript, .NET, and several other languages. It gives you two halves of the workflow.

On the consumer side, you write a test that defines the interaction you expect, and Pact generates a contract file from it, usually as JSON.

On the provider side, Pact takes that same contract file and replays it against your real service, failing the build if the actual response does not match what the consumer expects.

```json
{
  "consumer": { "name": "checkout-service" },
  "provider": { "name": "payment-service" },
  "interactions": [
    {
      "description": "a request to authorize a payment",
      "request": {
        "method": "POST",
        "path": "/payments/authorize",
        "body": { "amount": 4999, "currency": "AUD" }
      },
      "response": {
        "status": 200,
        "body": { "status": "AUTHORIZED", "authCode": "string" }
      }
    }
  ]
}
```

That file is the whole point. It is small, readable, and it lives independently of either service's source code, which means it can be shared, versioned, and checked in CI on both sides.

## Why Not Just Use Integration Tests

Integration tests that stand up real dependencies are not wrong, they just answer a different question and cost more to run. A contract test does not need payment's database, its downstream fraud checks, or its message queue. It only needs to know, does payment still return what checkout expects for this one interaction. That narrower scope is exactly what makes contract tests fast enough to run on every single commit, on both sides, without anyone waiting on a shared staging environment to be free.

## What Is Coming Next

Over the next several posts we are going to build this out properly against a Java Spring Boot service. We will write a consumer test, generate a real contract, verify it on the provider side, publish contracts to a Pact Broker so both teams can see them, and finally wire the whole thing into a CI pipeline with Pact's can-i-deploy check gating releases. By the end, the goal is a setup where a breaking change to an API gets caught in CI, on the provider's own pipeline, before it ever reaches the consumer team as a production incident.
