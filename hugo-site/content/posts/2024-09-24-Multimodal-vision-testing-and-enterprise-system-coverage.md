---
title: "Multimodal Vision Testing"
date: 2024-09-24T08:25:31+10:00
draft: false
categories: []
description: ""
---
With multimodal models like GPT-4 Vision and Claude 3 Opus getting so much sharper, test automation is finally moving beyond fighting with the HTML DOM and into actual **Visual and Spatial Verification**. 

### Things I’ve Been Experimenting With

#### 1. Screenshot-Based Visual Assertions
In the past, visual testing meant wrestling with rigid pixel-by-pixel diff tools that would blow up your build over a tiny font rendering shift. Lately, I’ve been feeding raw UI screenshots directly to vision-capable LLMs along with plain English questions: *"Is the checkout button properly aligned below the order summary? Is any text overlapping?"*

It actually evaluates layout aesthetics and obvious layout bugs almost like a human reviewer would, which is pretty mind-blowing to watch in real time.

#### 2. Tackling Horrible Dynamic Monoliths (Salesforce, ServiceNow)
Platforms like Salesforce have always been my least favorite thing to test—deep shadow DOMs, nested iFrames, and randomized element IDs make writing stable locators nearly impossible. 

Using vision models alongside intent-based prompts lets me automate flows across these enterprise tools visually, completely bypassing the need to maintain fragile, deeply nested CSS selectors.

### The Catch: Cost and False Positives

As cool as vision testing is, it comes with two big drawbacks: **cost and speed**. 

Multimodal API calls are way more expensive and noticeably slower than text-only endpoints. Plus, visual models can sometimes be a bit *too* sensitive, flagging tiny, non-functional padding tweaks as layout issues. Because of that, I’ve learned to be selective—reserving visual LLM checks for critical, high-value user journeys rather than slapping them onto every single PR.