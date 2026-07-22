---
title: "Structured Output, Function Calling, and Autonomous Test Authoring"
date: 2024-03-26T08:25:31+10:00
draft: false
categories: []
description: ""
---
Entering 2024, it feels like the whole AI tooling ecosystem has leveled up dramatically over the last year. We’re finally moving past hacky string parsing and regex workarounds—OpenAI’s native **Function Calling** has completely changed how I build scripts that interact with LLMs.

### What I’ve Been Experimenting With Lately

#### 1. Reliable, Deterministic Scripting via Function Calling
Instead of asking the model for raw code or text and hoping for the best, I’ve been defining strict JSON schemas for core test actions—things like `click_element(selector)`, `fill_input(selector, text)`, and `assert_text(expected)`. 

Now, I can pass a page’s state representation to the LLM and get back structured, executable function calls every time. It completely eliminates the random pipeline crashes caused by rogue Markdown formatting or unexpected conversational text.

#### 2. Trying Out First-Gen Test Copilots
I’ve also been taking a look at dedicated QA copilots from platforms like Tricentis and Testim. Unlike general-purpose coding assistants that just guess, these specialized tools actually understand wait strategies, test assertions, and environment configs right out of the box. 

### The Tricky Part: Hallucinations Inside Tool Calls

Even with structured function calling, the models still pull some weird moves. Every so often, an LLM will invent non-existent parameters or try to execute actions out of order—like attempting to hit a submit button before filling out the required form fields.

