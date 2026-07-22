---
title: "Playwright's Dominance And the low code AI Influx"
date: 2023-09-18T08:25:31+10:00
draft: false
categories: []
description: ""
---
Lately, it feels like we’ve hit a real tipping point with web automation. Selenium had a great decade-plus run, but for me, Microsoft Playwright has officially taken over as the go-to tool for modern web testing. It’s just so much faster and smoother to work with.

At the same time, I’ve been keeping an eye on the flood of new commercial "low-code" AI test tools popping up everywhere, trying to see if they actually deliver on bridging the gap between manual testing and automated pipelines.

### Things I’ve Been Experimenting With

#### 1. Combining Playwright Codegen + Copilot
Playwright’s built-in auto-waiting and network interception alone have made a massive dent in test flakiness, but the real fun has been tweaking the workflow. 

Lately, I’ve been using Playwright’s native `codegen` tool to quickly record the core steps of a user journey, and then handing that raw output over to Copilot to refactor into a clean Page Object Model. Pairing the two turns what used to be a tedious recording into production-ready code in minutes.

#### 2. Taking a Look at AI "Low-Code" Builders
It feels like every week a new vendor is pitching an "AI-driven visual test builder." The pitch is always the same: let non-technical team members or product folks record flows, while AI automatically handles selector fixes and self-healing in the background. It’s an interesting space, but the reality is a bit more nuanced.

### My Take: Code-First Wins Over Vendor Lock-In

While these low-code platforms promise crazy fast onboarding, I’m still pretty skeptical. The biggest dealbreaker for me is how many of them store tests in proprietary formats, which totally breaks standard Git workflows and PR reviews. 

I’m much more interested in a **code-first** approach: keeping everything in solid open-source frameworks (like Playwright ) and using AI to speed up writing the code, rather than getting boxed into a black-box commercial platform.