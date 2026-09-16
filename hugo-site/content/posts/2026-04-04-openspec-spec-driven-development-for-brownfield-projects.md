---
title: "OpenSpec - Spec driven development for brownfield projects"
date: 2026-04-04T08:25:31+10:00
draft: false
categories:
  - Agentic development
description: ""
---
# Spec-Driven Development for the Rest of Us: OpenSpec

Last month I found GitHub's Spec Kit and how it flips the traditional dev workflow — spec as the durable source of truth, code as the disposable output.  Spec Kit's `/constitution`-first, plan-then-build ceremony assumes you're starting mostly from a blank slate. Almost nothing I touch day-to-day looks like that. It's always years old services, inherited conventions nobody remembers agreeing to, and test suites that are more archaeology than architecture.

That's the gap **OpenSpec** is aimed at, and it's why it's grabbed my attention this month.

---

## Specs as a Mirror, Not a Blueprint

The core difference in philosophy is subtle but important. Spec Kit's specs describe what a system *should become*. OpenSpec's specs describe what a system *currently does* — and then layers proposed changes on top as explicit, reviewable deltas.

In practice that means:

* A living `specs/` directory that mirrors actual, current system behaviour — not an aspirational design doc.
* **Change proposals** as the unit of work: before an AI agent touches code, it drafts a proposal describing the delta against the existing spec — what capability is changing, why, and what the new expected behaviour is.
* Proposals get reviewed like a pull request, *before* implementation starts, not after.
* The agent implements against both the existing spec and the approved delta, which keeps it anchored to established conventions instead of quietly inventing a fresh architecture halfway through a feature.

It's a much lighter-weight loop than Spec Kit's full constitution-to-tasks pipeline, and that's deliberate — it's built to slot into normal sprint-sized changes on a codebase that already exists, rather than kicking off a project.

## The Bit I Actually Care About

The thing that got me interested isn't the workflow ceremony, it's the byproduct: an always-current spec of what the system actually does. Every brownfield project I've worked on has the same problem — nobody can tell you with confidence what's actually supported without reading the code (or the tests, when they're trustworthy, which is inconsistent). If change proposals genuinely keep the spec directory honest, that's a standing artifact I can use directly for regression impact analysis and test-scope decisions, without reverse-engineering behaviour from a diff.

The other appeal is that it plays nicer with AI coding agents on legacy code specifically. An agent regenerating code from a green-field spec is one thing; an agent that has to respect eleven years of quiet architectural decisions is another. Anchoring it to a delta against documented current behaviour, instead of turning it loose with a fresh mental model, feels like the safer default for anything I'd actually let near production.



Spec Kit and OpenSpec aren't really competitors — they're solving for opposite ends of a project's lifecycle. Spec Kit wants to plan a system into existence; OpenSpec wants to safely evolve one that already has scars. Given how much of my work lives in the second category, I suspect I'll get more day-to-day mileage out of this one. Next step is trying a real change proposal against one of our older services and seeing how well the generated spec actually matches reality.
