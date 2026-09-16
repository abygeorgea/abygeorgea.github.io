---
title: "Versioning Contracts and Catching Breaking Changes in Pact"
date: 2025-02-18T08:25:31+10:00
draft: false
slug: "versioning-and-breaking-changes-in-pact"
categories:
  - Pact
description: "How Pact tracks contract versions over time, tagging by branch and environment, and what a genuinely breaking change looks like in the broker."
---

In the [previous post](/blog/2025/02/11/gating-releases-with-pact-can-i-deploy/), `can-i-deploy` gave checkout an automated gate that blocks a deploy when payment has not verified the current contract. That check is only as good as the version history behind it. This post looks at how Pact tracks that history, and what actually happens in the broker when a contract changes in a way that breaks an existing consumer.

## Every Contract Publish Is a New Version

Each time checkout runs `pact:publish`, it does not overwrite the previous contract. It adds a new version, tied to whatever identifier you passed in, normally a git commit hash. The broker keeps every version, along with which provider versions have verified each one. This is what makes `can-i-deploy` meaningful. It is not asking "has this contract ever passed," it is asking "has this specific version passed."

## Tagging by Branch

Commit hashes are precise but not very readable, and they do not tell you anything about where a version came from. Tags solve that.

```bash
mvn pact:publish -Dpact.tag=main
mvn pact:publish -Dpact.tag=feature/split-payment-currency
```

A common pattern is publishing every branch's contracts tagged with the branch name, then having `can-i-deploy` on the provider side check specifically against contracts tagged `main`, so a provider is never blocked by a consumer's half finished feature branch.

```bash
pact-broker can-i-deploy \
  --pacticipant payment-service \
  --version $GIT_COMMIT \
  --to-environment production \
  --broker-base-url http://pact-broker:9292
```

`can-i-deploy` automatically resolves the right consumer versions to check against based on what is currently deployed in the target environment, which is why tagging by environment, alongside branch, tends to matter more as a system grows.

## What a Breaking Change Looks Like

Say the payment team decides to rename `authCode` to `authorizationCode` in their response, as part of cleaning up naming across their API. They ship the change, their own unit tests pass, and their build goes green, because nothing in payment's own test suite references the old field name anymore.

The very next time payment's provider verification runs against checkout's published contract, it fails.

```
Verifying a pact between checkout-service and payment-service
  a request to authorize a payment
    returns a response which
      has a matching body (FAILED)

$.body.authCode: Expected 'AB1234' but got no value
```

This is contract testing doing exactly its job. The change is objectively fine from payment's own perspective and objectively breaking from checkout's. Neither team is wrong in isolation, which is precisely the kind of disagreement that used to only surface once checkout was already failing in production.

## Handling It Properly

The fix is not to weaken the contract or delete the failing assertion. It is a conversation, made necessary by a failing build instead of optional. Either payment supports both field names for a transition period, or checkout gets advance notice and updates its client before payment removes the old field, with both sides re-verifying before the removal actually ships.

```java
// payment-service, during the transition
response.put("authCode", authCode);          // deprecated, remove after checkout migrates
response.put("authorizationCode", authCode);
```

Once checkout has updated its client and republished a contract that only expects `authorizationCode`, payment can safely drop the deprecated field, verify again, and this time see a clean pass.

## Why This Beats Documentation

None of this depended on anyone reading an API changelog or a deprecation notice in a wiki. The contract, generated from real consumer usage and continuously re-verified, caught the mismatch mechanically, the same day the change was made, on the team that introduced it. That is a meaningfully different guarantee than "we told people in the release notes," and it is one of the strongest arguments for contract testing on any team that owns services other teams depend on.

The final post in this series pulls everything together into one CI/CD pipeline view, from a consumer test running on a pull request through to a gated production deploy.
