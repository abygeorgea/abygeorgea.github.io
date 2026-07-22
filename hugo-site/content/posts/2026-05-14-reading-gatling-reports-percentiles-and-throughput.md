---
title: "Reading Gatling Reports: Percentiles, Throughput, and What Actually Matters"
date: 2026-05-14T07:30:00+10:00
draft: false
slug: "reading-gatling-reports-percentiles-and-throughput"
categories:
  - Gatling
description: "A practical guide to Gatling's HTML report, covering the global stats page, response time distribution, and how to actually find a problem in the details."
---

In the [previous post](/blog/2026/04/30/setting-performance-slas-with-gatling-assertions/), we set up assertions so a build fails automatically when performance regresses. A passing build is a good start, but the report itself still holds a lot of useful detail worth understanding properly, especially when something does go wrong and you need to figure out why.

Every Gatling run produces a self contained HTML report. Open it with the show command from part one.

```bash
mvn gatling:test
# once the run finishes, Gatling prints the path to the report,
# or open the latest one directly under target/gatling/
```

## The Global Stats Page

The landing page of the report summarizes the entire simulation. A few numbers here matter more than the rest. The percentile breakdown, shown as a chart and a table, tells you the distribution of response times across every single request in the run, not just an average. The requests per second chart over time shows whether your injection profile actually produced the load shape you intended, which is worth checking even on a passing run, since a misconfigured injection profile can silently produce far less load than you think it did.

The error percentage for the whole run is shown prominently too, and it is worth treating any non zero error rate as something to investigate, even if it stays under whatever threshold your assertions allow. A small number of consistent errors, appearing steadily throughout a run, usually points at a real and reproducible issue rather than random noise.

## Per-Request Breakdown

Below the global summary, the report breaks every named request out individually, with its own response time distribution and error rate. This is where you actually find which specific request is dragging the whole simulation down. A global ninety fifth percentile of one and a half seconds could mean every request is moderately slow, or it could mean nine out of ten requests are fast and one specific endpoint is consistently terrible. The per-request view is what tells you which of those two very different situations you are actually looking at.

Cross reference this against the request names you chose back when writing the scenario. This is exactly why naming requests clearly, like `"Get Product Catalog"` instead of a generic name, pays off later, since a report full of clearly named requests is far easier to scan for the one that actually needs attention.

## Response Time Over Time

A chart most people skim past too quickly is the response time distribution over the duration of the run, rather than aggregated across the whole test. This view is what tells you whether performance degraded progressively as load increased, stayed flat and consistent throughout, or spiked briefly at one specific point in time. If you ran a staged injection profile like the warm up, hold, and cool down pattern from earlier in this series, this chart is where you can actually see each of those phases reflected in the response time behavior, confirming the test ran the way you intended.

## Active Users Over Time

This chart shows how many virtual users were actually active at each point during the run, which is a direct visual confirmation of your injection profile. Comparing this against the response time chart side by side is one of the more useful habits to build. If response times start climbing at the exact moment active users cross a certain threshold, that threshold is a genuinely useful data point, arguably more useful than any single aggregate number in the whole report, since it tells you approximately where the system's real capacity limit sits.

## Distinguishing Errors From Slowness

The report separates failed requests from slow ones, and it is worth checking both independently rather than assuming one implies the other. A request can be fast and wrong, like the misconfigured error page returning a 200 status we talked about back in the post on checks. A request can also be slow but eventually correct, timing out right at the edge of an acceptable threshold without technically failing. Neither of these shows up clearly if you only glance at the top level pass or fail assertion result, which is exactly why it is worth opening the actual report even when the build goes green.

## Building a Habit Around the Report

The most useful habit here is simple. Do not just check whether the assertions passed and move on. Open the report, at minimum on any run against a new environment, after any significant application change, and any time a test result surprises you in either direction. The report holds detail that a single pass or fail signal from CI cannot express, and the small amount of time it takes to actually look at it regularly pays off the first time it catches something a plain assertion would have missed entirely.

## Wrapping Up

The global stats page tells you the headline numbers. The per-request breakdown tells you where a problem actually lives. The time series charts tell you how behavior evolved over the course of the run. Together they turn a report from a single pass or fail signal into a genuinely useful diagnostic tool.

Next time, we take everything we have built and wire it into a CI pipeline, including how to manage configuration across different environments cleanly.
