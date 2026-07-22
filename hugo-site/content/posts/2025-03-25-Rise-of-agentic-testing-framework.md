---
title: "Rise Of Agentic Testing Framework"
date: 2025-03-25T08:25:31+10:00
draft: false
categories: []
description: ""
---
Lately, it feels like we’ve crossed into a totally new era with AI tools. Over the past few months, my focus has shifted from simple code auto-completions to playing around with **Autonomous Agentic Workflows**.

Instead of just having an assistant that finishes a line of code, an agent acts more like a goal-oriented script. You give it a high-level target, and it can plan out multi-step actions, check the results, and adjust its approach on the fly.

### What I’ve Been Tinkering With

#### 1. Autonomous End-to-End Test Generation
Instead of writing Playwright scripts step-by-step, I’ve been experimenting with feeding high-level user journey specs into autonomous agents. 

The agent inspects the application, navigates a live local build, generates the Playwright code, verifies that the test actually passes, and even prepares a clean Git commit—all with barely any manual intervention. Seeing a script map out its own execution path in real time is pretty incredible.

#### 2. Automated Accessibility Checks & Fixes
Accessibility compliance is always one of those things that can feel tedious to audit by hand. Lately, I’ve been pairing traditional accessibility scanning tools like `axe-core` with LLM agents. When a component fails WCAG guidelines, the agent doesn't just flag the error—it actually suggests specific code fixes to make the component accessible.

### The Big Headache: Code Churn & Bloat

The biggest problem I’ve run into with autonomous agents is that they write code at blistering speed, which quickly leads to **codebase bloat**.

Because agents don't have a global memory of every utility script in a project, they tend to recreate redundant helper functions and duplicate existing tests. To keep my personal repos from turning into a chaotic mess, I’ve had to enforce strict linting rules, set code complexity thresholds, and make sure I carefully review every single agent-generated change before merging it.