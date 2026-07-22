---
title: "What Chatgpt Means for Our Test Automation Strategy"
date: 2023-03-28T11:21:31+10:00
draft: false
categories: []
description: ""
---
If your group chats look anything like mine lately, they’re probably full of screenshots of ChatGPT spitting out code. Ever since the public launch blew up, I’ve been down a rabbit hole trying to figure out what’s actually useful versus what’s just hype.

I wanted to cut through the noise and see how this stuff actually holds up in real life. Is AI going to write whole test suites for us overnight? Definitely not. But can it make the tedious, repetitive stuff a lot less painful right now? 100%.

### A Few Things I’ve Been Playing With

#### 1. Quick-Starting Page Object Models (POM)
The coolest trick I've found so far is using it to bypass the boring setup phase when building a new test suite. I’ll throw a chunk of raw HTML or a messy DOM snippet into ChatGPT (sticking with GPT-3.5) and ask it to sketch out a basic Page Object class in Python or TypeScript Playwright.

It’s definitely not perfect—it loves to hallucinate weird locators or make up fake `async` methods that don't exist—but as a starter scaffold, it easily shaves off 30–40% of the initial setup time. It’s like having a rough draft ready the second you start.