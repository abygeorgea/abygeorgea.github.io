---
title: "Running WireMock in CI as a Standalone Server"
date: 2025-04-15T08:25:31+10:00
draft: false
slug: "running-wiremock-in-ci"
categories:
  - WireMock
description: "Running WireMock as a standalone Docker service in a pipeline, recording real responses to bootstrap stubs, and keeping stubs in sync with the real dependency."
---

In the [previous post](/blog/2025/04/01/simulating-failures-with-wiremock/), everything ran embedded, inside a single test class's lifecycle. That is the right default for unit and component level tests. It stops working once several services in a pipeline all need to talk to the same stubbed dependency, or once integration tests running outside the JVM, a Postman collection for example, need to hit the same mock. That is what WireMock's standalone mode is for.

## Running WireMock Standalone

The same WireMock artifact that ran embedded can run as its own process, listening on a real port like any other service.

```bash
docker run -d --name wiremock-inventory \
  -p 8089:8080 \
  -v $(pwd)/stubs:/home/wiremock/mappings \
  wiremock/wiremock:3.9.1
```

Stub definitions now live as JSON files in a `stubs` directory rather than inline Java code, which is exactly what makes this shareable across languages and tools, not just a single Java test suite.

```json
{
  "request": {
    "method": "GET",
    "url": "/inventory/SKU-1042"
  },
  "response": {
    "status": 200,
    "jsonBody": { "sku": "SKU-1042", "available": 37 },
    "headers": { "Content-Type": "application/json" }
  }
}
```

## Recording Real Responses

Writing every stub by hand for a service with dozens of endpoints gets tedious fast, and it is easy to drift from what the real service actually returns. WireMock's recording mode solves that by sitting in front of the real service and capturing genuine responses as stub files.

```bash
docker run -d --name wiremock-recorder \
  -p 8089:8080 \
  wiremock/wiremock:3.9.1 \
  --proxy-all="https://inventory-staging.internal" \
  --record-mappings
```

Point your client at `localhost:8089` as usual, exercise the real flows you care about against staging, and WireMock forwards every request through to the real service while saving both the request and the actual response as a stub file. Stop the container afterward, and those recorded stubs become your starting point, edited by hand from there for the failure cases the real service will not reliably reproduce on demand.

## Wiring It Into a Pipeline

In CI, start WireMock as a service container before the tests that depend on it run.

```yaml
jobs:
  integration-tests:
    runs-on: ubuntu-latest
    services:
      wiremock:
        image: wiremock/wiremock:3.9.1
        ports:
          - 8089:8080
    steps:
      - uses: actions/checkout@v4
      - name: Load stub mappings
        run: |
          curl -X POST http://localhost:8089/__admin/mappings/import \
            -H "Content-Type: application/json" \
            -d @stubs/inventory-mappings.json
      - name: Run integration tests
        run: mvn verify -Dinventory.base-url=http://localhost:8089
```

Every job in the pipeline that needs the inventory dependency now points at the same WireMock container, configured identically, which keeps behavior consistent whether the tests are running on a developer's machine or in CI.

## Keeping Stubs From Going Stale

The one real risk with standalone stub files is drift. Nothing forces `stubs/inventory-mappings.json` to stay in sync with what the real inventory service actually returns once that service changes. A stub file checked in six months ago can happily keep passing tests long after the real API has moved on.

The most reliable mitigation is treating recorded stubs as something to refresh periodically against staging, on a schedule, rather than writing them once and assuming they stay accurate forever. This is also exactly the gap consumer-driven contract testing closes more rigorously, which is worth keeping in mind. WireMock and Pact are not competitors, they solve different problems well. WireMock gives you full control over a dependency's behavior, failure modes included, for testing your own service in isolation. Pact gives you a guarantee that your assumptions about a dependency's behavior are still true. A mature test strategy usually wants both, not one instead of the other.

That distinction is a good place to end this series, since it is the exact seam between what we just built with WireMock and what the earlier Pact series covered.
