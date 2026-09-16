---
title: "Wiring Pact Into CI/CD: The Full Contract Testing Pipeline"
date: 2025-02-25T08:25:31+10:00
draft: false
slug: "wiring-pact-into-cicd"
categories:
  - Pact
description: "Pulling consumer tests, contract publishing, provider verification, and can-i-deploy into one end to end CI/CD pipeline for both services."
---

In the [previous post](/blog/2025/02/18/versioning-and-breaking-changes-in-pact/), we walked through what happens when a contract genuinely breaks, and how tagging and versioning give both teams a precise way to reason about it. This last post in the series pulls every piece we have built, consumer tests, the broker, provider verification, and `can-i-deploy`, into one coherent pipeline view, so it is clear how this actually runs day to day rather than as a sequence of separate manual steps.

## The Consumer Pipeline

Every time checkout opens a pull request or merges to main, its pipeline runs Pact tests as part of the normal test suite, then publishes the resulting contract to the broker, tagged with the branch name.

```yaml
name: checkout-service CI

on: [push, pull_request]

jobs:
  test-and-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests including Pact
        run: mvn test
      - name: Publish contracts to broker
        run: mvn pact:publish -Dpact.tag=${{ github.ref_name }}
        env:
          PACT_BROKER_URL: ${{ secrets.PACT_BROKER_URL }}
```

Nothing here is different from a normal test and publish step, contract publishing is just one more artifact of a green build, same as a JAR file or a coverage report.

## The Provider Pipeline

Payment's pipeline runs on the same triggers, but its job is verification rather than publishing. It pulls checkout's latest contract from the broker, tagged `main`, and verifies it against payment's own real implementation.

```yaml
name: payment-service CI

on: [push, pull_request]

jobs:
  verify-contracts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Start application
        run: mvn spring-boot:run &
      - name: Run Pact provider verification
        run: mvn test -Dtest=PaymentServiceProviderPactTest
        env:
          PACT_BROKER_URL: ${{ secrets.PACT_BROKER_URL }}
```

This runs on every payment commit, not just when checkout changes something, which is exactly the point. Payment finds out immediately if their own change breaks a contract, without needing to know checkout exists as a dependency in any conscious way. The broker is doing that tracking on their behalf.

## The Deploy Gate

The piece that ties it together sits right before either service actually deploys.

```yaml
deploy:
  needs: verify-contracts
  runs-on: ubuntu-latest
  steps:
    - name: Can I deploy?
      run: |
        pact-broker can-i-deploy \
          --pacticipant payment-service \
          --version ${{ github.sha }} \
          --to-environment production \
          --broker-base-url ${{ secrets.PACT_BROKER_URL }}
    - name: Deploy
      run: ./deploy.sh
    - name: Record deployment
      if: success()
      run: |
        pact-broker record-deployment \
          --pacticipant payment-service \
          --version ${{ github.sha }} \
          --environment production
```

Deploy only runs if `can-i-deploy` passes, and a successful deploy immediately tells the broker what is now live, which keeps future `can-i-deploy` checks, on either side, accurate.

## What This Buys the Team

Put together, this is a closed loop. Checkout's client code changes trigger a new contract. Payment finds out about that change on their own pipeline, without a Slack message or a shared calendar invite. A break gets caught as a failing build on whichever side owns the mismatch. And neither service can deploy past a broken contract, because `can-i-deploy` sits directly in the path to production, not as a dashboard someone has to remember to check.

## Where This Fits Against Everything Else

None of this replaces unit tests, and it deliberately does not replace end to end tests either. It fills the specific gap between them, the place where two services agree on paper but drift apart in practice, and it does that without needing a shared staging environment or a slow, brittle integration suite. For a system built from services owned by different teams, each shipping on their own schedule, that gap is usually where the most expensive production incidents actually come from, which is exactly why it is worth the setup cost this series walked through.
