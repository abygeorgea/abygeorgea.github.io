---
title: "Building Run Self healing"
date: 2024-06-30T08:25:31+10:00
draft: false
categories: []
description: ""
---
Flaky tests caused by shifting DOM element IDs have been the bane of my existence for as long as I’ve been writing test automation. I got tired of constant pipeline failures over minor UI tweaks, so lately I’ve been tinkering with **Runtime Self-Healing Mechanisms** directly inside my Playwright test runners.

### What I’ve Been Building Recently

#### 1. Dynamic Selector Healing
I set up dynamic error interceptors inside my test suites. When a locator lookup fails (say, a `TimeoutError: element '#submit-order-v2' not found`), the test runner catches the exception, snags a snapshot of the surrounding DOM tree, and hands it off to an LLM to request an alternative CSS or XPath selector. 

If the new selector works, the test keeps chugging along without blowing up the build, and it drops a warning log so I can clean up the selector later.

[Test Execution] -> Locator Failed (#submit-btn)
│
▼
[Catch Interceptor] -> Capture DOM Snapshot
│
▼
[LLM Evaluator] -> Propose Healed Locator (#submit-checkout)
│
▼
[Resume Run] -> Test Passes (Log PR for Selector Repair)



### The Trade-Off: Dealing with Latency

Self-healing feels like a total superpower when you first see it work, but it comes with a catch: **latency**. 

Pausing mid-test to make an LLM call adds 3 to 5 seconds every single time a locator fails. If a suite has a dozen broken locators, the total execution time explodes real fast. It’s been a great reminder that self-healing is just a temporary safety net to keep CI green—not an excuse to ignore clean, robust locator strategies in the first place!