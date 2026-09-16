---
title: "Gating Releases with Pact's can-i-deploy"
date: 2025-02-11T08:25:31+10:00
draft: false
slug: "gating-releases-with-pact-can-i-deploy"
categories:
  - Pact
description: "Using Pact's can-i-deploy CLI check as an automated release gate, so a service cannot deploy against a contract nobody has verified."
---

In the [previous post](/blog/2025/02/04/publishing-contracts-to-a-pact-broker/), both checkout and payment started publishing contracts and verification results to a shared Pact Broker. The broker knows exactly which version of payment has verified exactly which version of checkout's contract. What it does not do on its own is stop anyone from deploying a version that has not been verified. That is what `can-i-deploy` is for.

## The Question It Answers

Before checkout deploys a new version to production, there is one question worth asking automatically rather than trusting someone to remember it. Has every provider checkout depends on already verified this exact version of the contract. If the answer is no, the deploy should not happen, full stop.

`can-i-deploy` is a CLI command, part of the Pact Broker client tooling, that asks the broker exactly that question and returns a pass or fail result you can wire straight into a pipeline.

```bash
pact-broker can-i-deploy \
  --pacticipant checkout-service \
  --version $GIT_COMMIT \
  --to-environment production \
  --broker-base-url http://pact-broker:9292
```

## Reading the Result

When every provider has verified the contract for this version, you get a clean pass.

```
Computer says yes \o/

CONSUMER          | C.VERSION | PROVIDER        | P.VERSION | SUCCESSFUL?
checkout-service  | a1b2c3d   | payment-service | f9e8d7c   | true

All required verification results are published and successful
```

When they have not, the same command fails, with a message that tells you exactly which provider is missing a passing verification.

```
Computer says no

CONSUMER          | C.VERSION | PROVIDER        | P.VERSION | SUCCESSFUL?
checkout-service  | a1b2c3d   | payment-service | (none)    | (no verification found)

There is no verified pact between version a1b2c3d of checkout-service
and a suitable version of payment-service
```

That second output is the whole point. It catches exactly the situation where checkout's contract changed, payment has not run verification against the new version yet, and someone is about to deploy anyway.

## Wiring It Into the Pipeline

Add this as a step right before the actual deploy step in checkout's pipeline, and treat a non-zero exit code the same way you would treat a failing test.

```yaml
- name: Verify contracts before deploy
  run: |
    pact-broker can-i-deploy \
      --pacticipant checkout-service \
      --version ${{ github.sha }} \
      --to-environment production \
      --broker-base-url ${{ secrets.PACT_BROKER_URL }}

- name: Deploy to production
  if: success()
  run: ./deploy.sh
```

The deploy step now literally cannot run unless the previous step exited zero, which only happens when every dependency payment relates to has a passing verification recorded against this exact commit.

## Environments Matter Here

Notice `--to-environment production` in the command. The broker tracks which version of each service is currently deployed where, which lets `can-i-deploy` ask a more precise question than just "has this ever been verified." It asks whether this version is safe to deploy against whatever is actually running in production right now, which is a meaningfully different and more useful question than checking against the latest contract in isolation.

This does mean the broker needs to be told what is deployed where, usually through a companion `record-deployment` call right after a successful deploy.

```bash
pact-broker record-deployment \
  --pacticipant checkout-service \
  --version $GIT_COMMIT \
  --environment production
```

## What This Buys You

With this in place, a breaking change to payment's API can no longer reach production through checkout's pipeline without someone on the payment side explicitly verifying it first. The failure moves from a production incident, discovered by a customer, to a red step in CI, discovered by whoever pushed the change. That shift, catching the break before deploy instead of after, is the entire value proposition of consumer-driven contract testing in one pipeline step.

The next post looks at what happens once contracts start changing over time, and how Pact handles versioning so a breaking change gets flagged clearly instead of just quietly failing verification.
