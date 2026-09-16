---
title: "Publishing and Sharing Contracts with a Pact Broker"
date: 2025-02-04T08:25:31+10:00
draft: false
slug: "publishing-contracts-to-a-pact-broker"
categories:
  - Pact
description: "Standing up a Pact Broker, publishing consumer contracts to it automatically, and pulling them into provider verification tests."
---

In the [previous post](/blog/2025/01/28/verifying-pact-contracts-on-the-provider-side/), payment verified checkout's contract by reading it from a local folder path. That works for a demo, but it does not scale. The provider team should not need a copy of the consumer's build output sitting on disk to run their tests. This is what the Pact Broker exists to fix.

## What the Broker Actually Does

A Pact Broker is a small standalone service that stores contracts, tracks which versions of which services have verified which contracts, and exposes that history through a web UI and an API. Instead of checkout emailing a JSON file to payment, checkout publishes its contract to the broker after every build, and payment pulls the latest version from the broker when it runs verification.

The broker becomes the single source of truth for who depends on whom, and whether that dependency is currently healthy.

## Running a Broker

For local development or a small team, the official Docker image is the fastest way to get one running.

```bash
docker run -d --name pact-broker \
  -e PACT_BROKER_DATABASE_ADAPTER=sqlite \
  -e PACT_BROKER_DATABASE_NAME=pact_broker.sqlite3 \
  -p 9292:9292 \
  pactfoundation/pact-broker
```

For anything beyond local experimentation, point it at a real Postgres instance instead of sqlite, and run it somewhere both the consumer and provider pipelines can reach, since this needs to be a shared, always available service rather than something on someone's laptop.

## Publishing From the Consumer Side

Checkout's build needs one extra step after its Pact tests run, publishing the generated contract to the broker.

```xml
<plugin>
    <groupId>au.com.dius.pact.provider</groupId>
    <artifactId>maven</artifactId>
    <version>4.6.5</version>
    <configuration>
        <pactBrokerUrl>http://pact-broker:9292</pactBrokerUrl>
        <projectVersion>${git.commit.id}</projectVersion>
    </configuration>
</plugin>
```

```bash
mvn pact:publish
```

Using the actual git commit hash as the version, rather than a static string, matters a lot here. It is what lets the broker later answer a very specific question. Was the contract published by this exact build ever verified by payment.

## Pulling Contracts on the Provider Side

Payment's verification test now points at the broker instead of a local folder.

```java
@Provider("payment-service")
@PactBroker(url = "http://pact-broker:9292")
class PaymentServiceProviderPactTest {
    // verification logic unchanged from the previous post
}
```

Run `mvn test` as before. Pact fetches the latest contracts for `payment-service` from the broker, verifies each one, and publishes the verification result back to the broker automatically. That last part matters just as much as the fetch. The broker now has a record showing exactly which version of payment successfully verified exactly which version of checkout's contract.

## Reading the Broker's Network Diagram

Open the broker's web UI and you get a visual dependency graph, built entirely from published contracts and verification results, no manual documentation required. For a system with a handful of services this is convenience. For a system with thirty services calling each other in ways nobody has fully diagrammed in over a year, this becomes the most accurate architecture diagram the team has, because it is generated from what services actually do, not what someone remembers deciding.

## The Question We Still Cannot Answer

Right now, both teams can see contracts and verification results, but there is no automated gate stopping checkout from deploying a version that payment has never actually verified. Someone still has to manually check the broker before hitting deploy. That manual step is exactly what the next post removes, with Pact's `can-i-deploy` check wired directly into the release pipeline.
