---
title: "Running Gatling in CI/CD: Pipelines and Environment Configuration"
date: 2026-05-28T07:30:00+10:00
draft: false
slug: "running-gatling-in-cicd-pipelines-and-environments"
categories:
  - Gatling
description: "Wiring a Gatling and Maven project into a GitHub Actions pipeline, with clean environment configuration for dev, staging, and production targets."
---

In the [previous post](/blog/2026/05/14/reading-gatling-reports-percentiles-and-throughput/), we learned to actually read what Gatling produces. None of this is worth much long term if it only ever runs on someone's laptop before a release, run manually and inconsistently. Today we automate it properly, and we sort out configuration across environments while we are at it, since the two problems tend to show up together in practice.

## Externalizing the Base URL

Every simulation so far has hardcoded a base URL directly in the protocol configuration. That falls apart the moment you want to run the exact same simulation against a local environment, a staging environment, and occasionally production itself for a controlled test. The fix is to read it from a system property instead.

```java
// config/EnvironmentConfig.java
package config;

public class EnvironmentConfig {

    public static String baseUrl() {
        return System.getProperty("baseUrl", "http://localhost:8080");
    }
}
```

```java
// config/HttpProtocolConfig.java
package config;

import io.gatling.javaapi.http.HttpProtocolBuilder;

import static io.gatling.javaapi.http.HttpDsl.*;

public class HttpProtocolConfig {

    public static final HttpProtocolBuilder httpProtocol = http
        .baseUrl(EnvironmentConfig.baseUrl())
        .acceptHeader("application/json");
}
```

Now the base URL defaults to a sensible local value, but it can be overridden at run time from the command line without touching any code.

```bash
mvn gatling:test -DbaseUrl=https://staging.example.com
```

This same pattern extends naturally to anything else that varies by environment, like credentials for a test account, or the injection profile itself, since a staging environment might warrant a much smaller load than a production adjacent performance environment sized closer to real capacity.

```java
public static int targetUsersPerSec() {
    return Integer.parseInt(System.getProperty("targetUsersPerSec", "10"));
}
```

## A GitHub Actions Workflow

With configuration externalized, wiring this into a pipeline is a matter of installing Java, running Maven, and passing the right system properties for whichever environment the workflow targets.

```yaml
name: Performance Test

on:
  workflow_dispatch:
    inputs:
      baseUrl:
        description: 'Target environment base URL'
        required: true
        default: 'https://staging.example.com'
      targetUsersPerSec:
        description: 'Target users per second'
        required: true
        default: '10'

jobs:
  performance-test:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '21'

      - name: Run Gatling simulation
        run: |
          mvn gatling:test \
            -DbaseUrl=${{ inputs.baseUrl }} \
            -DtargetUsersPerSec=${{ inputs.targetUsersPerSec }}

      - name: Upload Gatling report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: gatling-report
          path: target/gatling/
          retention-days: 30
```

This one uses `workflow_dispatch`, meaning it runs on demand rather than on every push, which is the right trigger for most performance tests. Unlike a fast functional suite, a meaningful load test can take anywhere from several minutes to several hours, and running one automatically on every single commit to a shared branch is rarely the right tradeoff. A manual trigger, or a nightly scheduled run against a dedicated performance environment, both fit the actual rhythm of performance testing much better than running on every pull request the way you might for unit tests.

The `if: always()` on the upload step matters here just as much as it would in a functional test pipeline. You want the report from a failing run, when an assertion trips and the build goes red, at least as much as you want one from a passing run.

## Scheduling a Regular Baseline Run

Alongside on demand runs, it is worth scheduling a recurring run against a stable environment, purely to track how performance trends over time rather than just checking it at a single point before a release.

```yaml
on:
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:
```

This runs automatically every night at two in the morning, in addition to still being triggerable manually whenever needed. Having this history matters more than any single run, since it is what lets you notice a gradual regression creeping in over several weeks, something that a single one-off test run before a release would never catch, since it only ever compares against whatever the baseline happened to be on that one day.

## Keeping Credentials Out of the Workflow File

If your simulations need real credentials for a staging or performance environment, never put them directly in the workflow YAML. Use your CI platform's secrets management and reference them the same way you would for any other pipeline.

```yaml
      - name: Run Gatling simulation
        env:
          TEST_USER_PASSWORD: ${{ secrets.PERF_TEST_USER_PASSWORD }}
        run: |
          mvn gatling:test \
            -DbaseUrl=${{ inputs.baseUrl }} \
            -DtestUserPassword=$TEST_USER_PASSWORD
```

And read it in Java the same way as any other system property, through your `EnvironmentConfig` class, keeping the actual secret value out of both the simulation code and the workflow file itself.

## Wrapping Up

Externalizing environment specific values turns a hardcoded simulation into one that runs cleanly against dev, staging, or production adjacent environments without any code changes. Wiring that into a scheduled and on demand CI pipeline, with reports archived automatically, is what turns performance testing from an occasional manual exercise into an ongoing, trustworthy part of how the team ships software.

Next time, in our final post of this series, we look at what happens once a single Gatling instance is not enough, through distributed load generation, and the long term habits that keep a performance testing framework healthy over months of active use.
