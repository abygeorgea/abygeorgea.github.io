---
title: "Spec Kit- Spec Driven development"
date: 2026-03-02T08:25:31+10:00
draft: false
categories:
  - Agentic development
description: ""
---
# Flipping the Script: A Look at GitHub's Spec Kit

I've spent the last few posts obsessing over agents that write, run, and heal tests. This time I want to zoom out one level, because I think I've been skipping over the artifact that actually matters most to an AI coding agent: the spec itself.

That's what pulled me into **Spec Kit**, GitHub's open-source toolkit for what they're calling "Spec-Driven Development" (SDD). The pitch is deceptively simple but genuinely inverts how most of us have worked for the last twenty years. In traditional development, the spec (if one even exists past the kickoff meeting) is a scaffold — you lean on it briefly, then throw it away the moment code starts shipping. Code becomes the source of truth, and the spec rots in Confluence somewhere, quietly lying to whoever reads it next.

Spec Kit flips that. The spec doesn't get discarded — it becomes the durable, executable artifact, and the code becomes the disposable, regenerable output of it.

---

## How It Actually Works

Spec Kit structures the workflow into a handful of slash commands that map cleanly onto how a good tech lead would run a project:

* **`/constitution`** — establish the non-negotiable principles and constraints for the project up front, before any feature work starts.
* **`/specify`** — define the *what* and the *why* of a feature, deliberately keeping implementation details out.
* **`/clarify`** — force the ambiguity-resolution step that normally happens three days into a sprint via a confused Slack thread.
* **`/plan`** — translate the spec into a concrete technical approach, tech stack included.
* **`/tasks`** — break the plan into small, reviewable, executable chunks.
* **`/implement`** — let the agent actually build it, task by task, against the spec.

## Why This Matters to a QA Brain

Reading through this workflow, I kept mentally overlaying it onto the multi-agent pipeline I sketched out a couple of months back. `/specify` and `/clarify` are essentially doing the job I assigned to my hypothetical "Analyst Agent" — mapping out the exact test surface before anyone writes a line of test automation. The difference is Spec Kit does it as a first-class, structured step rather than something bolted on afterward.

The part I find most promising from a quality standpoint is `/clarify`. So much test flakiness and "well, that's not what I meant" bug triage traces back to ambiguous acceptance criteria that nobody forced anyone to resolve before implementation started. Baking that resolution into the workflow, before the agent starts generating code, should in theory produce specs that are already halfway to being good test cases.

## The Catch

Spec Kit is clearly built with greenfield work in mind — a `/constitution` step assumes you're setting principles for a project that doesn't have fifteen years of legacy decisions already baked in. Most of what crosses my desk is nowhere near that clean. Which, conveniently, is exactly the gap I want to dig into next.
