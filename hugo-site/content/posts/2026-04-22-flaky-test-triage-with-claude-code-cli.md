---
title: "Using Claude Code CLI for Flaky Test Triage"
date: 2026-04-22T08:25:31+10:00
draft: false
slug: "flaky-test-triage-with-claude-code-cli"
categories:
  - Agentic development
description: "Using Claude Code CLI to triage flaky tests, teaching an agent to tell a real regression apart from environmental noise, and where a human still needs to make the final call."
---

Every test suite past a certain size accumulates flaky tests, ones that fail occasionally for reasons that have nothing to do with the code under test. A slow CI runner, a race condition in test setup, a shared resource another job happened to be using at the same time. The problem is never that flaky tests exist. The problem is triage time. Someone has to look at a failure, decide whether it is real or noise, and that decision eats far more time across a team than it should, especially on a suite with hundreds of tests running on every merge.

That is the specific, narrow problem I wanted an AI coding agent to help with, using Claude Code CLI against our own CI failure history.

## Why This Is a Good Fit for an Agent

Triage is fundamentally a pattern matching task before it is a fixing task. Has this exact test failed before. Does the failure message match a known flaky signature. Did other, unrelated tests fail in the same run, which usually points at an environment problem rather than the test itself. A human doing this well is mostly cross referencing history, not reasoning from first principles, and that is exactly the kind of task an agent with access to CI logs and test history can do quickly and consistently.

It is worth being explicit about what this does not mean. The agent is not deciding what is broken and fixing it unattended. It is doing the first, most repetitive step, sorting failures into likely-flaky and likely-real, so a human spends their attention on the smaller, harder pile.

## Setting Up the Input

The agent needs failure history to pattern match against, not just the single failure in front of it. A simple approach is exporting recent CI run results into a structured log the agent can read.

```bash
gh run list --workflow=ci.yml --limit 50 --json databaseId,conclusion \
  | jq -c '.[] | select(.conclusion == "failure")' > recent-failures.json

for id in $(jq -r '.databaseId' recent-failures.json); do
  gh run view "$id" --log-failed >> failure-logs.txt
done
```

That gives Claude Code CLI a real dataset, actual failure messages and stack traces across recent runs, rather than asking it to guess from a single failing test in isolation.

## Asking for Triage, Not a Fix

The prompt matters here. Asking an agent to fix a failing test invites it to change assertions or add retries just to make the red go green, which is the opposite of what you want. Asking it to triage keeps the scope narrow and the output reviewable.

```
Here is our test failure history from the last 50 CI runs (failure-logs.txt)
and today's failing test output (today-failure.txt).

For the failure in today-failure.txt, tell me:
1. Has this exact test failed before in the history, and how often
2. Does the failure message match a pattern seen in other, unrelated
   tests in the same time window (suggesting an environment issue)
3. Your confidence this is flaky vs a real regression, with reasoning
4. What evidence would change your assessment

Do not suggest a fix. Only triage.
```

A response worth trusting looks something like this, specific and checkable rather than a vague guess.

```
PaymentAuthorizationTest.testConcurrentAuthorization has failed 6 times
in the last 50 runs, always with a connection pool timeout, never with
an assertion failure on the actual authorization logic. Two other,
unrelated tests failed in the same run window on 3 of those 6 occasions,
suggesting a shared database connection pool under load during CI,
not a bug in the test or the code it exercises.

Confidence: likely flaky (not a regression)

This assessment would change if the failure ever included an assertion
mismatch on the authorization result itself, rather than a timeout
during setup.
```

That last section, what evidence would change the assessment, is worth insisting on in every triage prompt. It forces the agent to state its reasoning in a falsifiable way, rather than handing back a confident sounding conclusion with nothing underneath it.

## Where Trust Has to Stop

This workflow earns trust for exactly one narrow claim, sorting failures by likely cause based on pattern matching against history the agent can actually see. It does not earn trust for silently marking a test as flaky and skipping it going forward, and it should never be allowed to do that without a human confirming first. A genuinely new bug can absolutely look like noise on its very first occurrence, before any history exists to distinguish it, and an agent triaging purely on pattern frequency has no way to catch that on the first pass.

The practical guardrail that works well is treating the agent's output as a suggestion attached to the CI failure notification, not an automatic action. A test flagged as likely flaky still shows up for a human to glance at and confirm, it just gets deprioritized in the queue rather than dropped from consideration entirely. The time saved is real, most of a team's triage time goes into the easy, obviously flaky cases, and this clears those quickly so people can spend their attention on the smaller number of failures that actually need real investigation.

## What This Looks Like at Scale

Once this triage step is reliable enough to trust for prioritization, wiring it into the CI failure notification itself is a natural next step, so a failing build in a pull request comes with a triage note attached automatically rather than requiring someone to run the prompt by hand each time. That is the direction worth taking this next, treating the agent's triage output as one more piece of context in the failure report, right alongside the stack trace, rather than a separate manual step someone has to remember to run.
