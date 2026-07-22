---
title: "Token Optimization and prompt caching"
date: 2024-12-17T08:25:31+10:00
draft: false
categories: []
description: ""
---
As I’ve been leaning heavier into AI-assisted coding and scripting for my own side projects over the past year, my API token usage shot up fast—it went from a cheap experiment to an invoice that made me do a double-take. Lately, my focus has been all about **Token Optimization, Context Caching, and keeping my setup lean**.

### Things I’ve Been Tweaking

#### 1. Prompt Caching & Smart Context Truncation
When OpenAI and Anthropic rolled out prompt caching, I jumped on it immediately for my personal scripts. By caching heavy system prompts, common schemas, and reusable code patterns, I managed to slash my monthly LLM API spend by over 45% while actually speeding up execution times. 

#### 2. Building Safety & Reliability Checks for My Own Tools
As I've built out more LLM-powered side projects, I've had to get serious about testing the models themselves. I've been tinkering with personal evaluation harnesses to check my scripts for hallucinations, prompt injection vulnerabilities, and weird edge-case failures before relying on them.

### What I’ve Learned from the Whole Journey

Looking back, this was the year using AI moved from throwing random prompts at a chat window to building actual structured, reliable tools. But if there's one takeaway from playing around with all this stuff, it's that human intuition and solid code structure are still non-negotiable. 

Left unchecked, letting AI generate everything leads to messy repos, duplicate code, and an unnecessary cloud bill. Moving forward, keeping things modular, well-architected, and cost-effective is way more satisfying than just writing clever prompts.