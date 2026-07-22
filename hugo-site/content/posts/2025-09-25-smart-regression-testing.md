---
title: "Smart Regression Testing"
date: 2025-09-25T08:25:31+10:00
draft: false
categories: []
description: ""
---
Running huge regression suites every time I push a small change to a repo is super inefficient and slow, especially as projects start growing and getting complex. Lately, I've been diving deep into **AI-Driven Risk-Based Selection** to make my testing workflow fast, targeted, and lean.

### Smarter Builds with Risk-Based Selection

Instead of blindly firing off every single test on every commit, I’ve been tinkering with a script that inspects the exact files changed in a Git diff. The script parses the modified code, maps out the underlying dependencies, and picks only the top 15% or so of tests that are actually affected by those changes.

By focusing execution strictly on what could actually break, I've managed to cut my local test run and CI build times in half. It turns a ten-minute wait into a quick coffee sip, all without letting obvious bugs slip through into the main branch.

###  How claude code build it ( not me :-) ) 

To get this working, I hooked a lightweight Python script into my local Git workflow and CI pipeline. Here is the step-by-step mechanism:

1. **Extracting the Git Diff:** First, the script runs `git diff --name-only HEAD~1` to snag a precise list of modified files in the latest commit.
2. **AST Parsing for Dependency Graphs:** For Python or JavaScript files, I feed the changed code into an Abstract Syntax Tree (AST) parser (like Python's `ast` module or `@babel/parser`). This builds a call-graph map showing every class, function, and module importing or touching those changed lines.
3. **LLM Context Vector Matching:** For harder-to-trace relationships—like API routes or decoupled event listeners—I pass the file diffs alongside a lightweight JSON map of my test suite to a small LLM endpoint. The prompt asks the model to rank test files by relevance (0.0 to 1.0 confidence score) based on semantic overlap and risk impact.
4. **Dynamic Test Filtering:** The script filters out any test scoring under a 0.7 confidence threshold and outputs a custom execution flag straight into the test runner (e.g., passing specific file paths to `pytest` or `playwright test`).

### The Bumpy Part: Getting the Dependency Mapping Right

The trickiest part of getting this working was avoiding false negatives—situations where the script gets too aggressive with trimming tests and skips a suite that *should* have run. 
