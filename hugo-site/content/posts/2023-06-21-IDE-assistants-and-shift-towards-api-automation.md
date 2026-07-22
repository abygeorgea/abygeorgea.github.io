---
title: "IDE Assistants and shift towards API Automation"
date: 2023-06-21T09:21:31+10:00
draft: false
categories: []
description: ""
---
The initial novelty of pasting prompts into web browser windows has pretty much worn off. Lately, I've been way more interested in how AI fits directly into the everyday coding workflow—mostly tinkering with GitHub Copilot and inline code completions right inside the editor.

At the same time, my own approach to test automation has been shifting. I've been leaning much harder into API and microservice testing lately, mostly because keeping heavy UI tests reliable as apps change fast is a constant uphill battle.

### Things I’ve Been Experimenting With Lately

#### 1. In-IDE Autocomplete for Quick Test Scaffolding
GitHub Copilot has become a daily staple in my setup. I’ve been trying out a habit of writing a descriptive comment or function name first—something like `// Test standard checkout flow with invalid credit card`—and letting Copilot fill in the function body.

It’s surprisingly good at standard unit test structures in frameworks like Jest or PyTest. Where it stumbles, though, is when you try to get it to work with custom project wrappers or highly specific helper functions—it tends to guess wrong or fall back to generic pattern matching.

#### 2. Doubling Down on API Automation Over Brittle UI Tests
UI tests are notorious for flaking out at the worst times. To keep things cleaner, I’ve been shifting more focus toward API testing using Postman, RestAssured, and a few custom Python scripts. 

One really neat trick I stumbled across: AI tools are remarkably good at parsing OpenAPI / Swagger specs. Feeding a spec into an LLM and asking it to build out edge-case coverage (like 4xx validation errors and 5xx handling) generates surprisingly solid starting points.

### The Real Headache: Hitting Token Limits

The biggest bump in the road I've run into lately is dealing with context window limitations. 

Whenever I try to feed full HTML pages or complex DOM trees into a model for analysis, I run straight into GPT-3.5’s 4,000-token ceiling. To get around this, I ended up writing a small helper script using `BeautifulSoup` to strip away `<style>`, `<script>`, and inline SVG tags before passing the HTML along. It’s a hacky workaround, but it’s been essential for fitting everything into the token budget!