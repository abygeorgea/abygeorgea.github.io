---
title: "Graphify - Mapping the Codebase AI Agents Actually Need"
date: 2026-08-05T08:25:31+10:00
draft: false
categories: []
description: ""
---
# Graphify: The Missing Map for Brownfield Codebases

Back when I was digging into OpenSpec, I kept circling the same problem: on a brownfield codebase, nobodycan tell you with confidence what's actually connected to what. Specs help, but a spec still describes *behaviour*.  That's the gap that pulled me into **[Graphify](https://github.com/Graphify-Labs/graphify)**.

Graphify turns a codebase — code, docs, SQL schemas, configs, even PDFs — into a queryable knowledge graph, and ships as a skill for Claude Code, Cursor, Codex, Gemini CLI, Copilot, Aider, and a dozen-plus other assistants. Its like a dependency map for your code base. Instead of an agent guessing at architecture from whatever files happen to be open, it gets a real map to query.

---

## Why This Is Different From the Usual RAG Story

Most "understand my codebase" tools reach for embeddings and a vector store. Graphify deliberately doesn't:

* **Local AST parsing, zero LLM calls for code.** It uses tree-sitter to build the code graph, which means "code maps for free" — no API key, no token spend, no hallucinated relationships for the parts it can parse directly.
* **Every edge is labelled.** Connections are tagged `EXTRACTED` (explicit in source) or `INFERRED` (resolved by Graphify). That's a genuinely useful trust signal — you know exactly which parts of the map to double-check.
* **Real graph traversal, not similarity search.** Questions like "what connects auth to the database" get answered by walking actual edges, not by hoping the nearest embedding is the right one.
* **Broad coverage.** 36+ languages, plus docs, PDFs, and video/audio (transcribed locally via faster-whisper).
* **Local-first privacy.** Code never leaves your machine; there's no telemetry or usage tracking by default.


## Setting It Up

**Prerequisites:** Python 3.10+, and `uv` (recommended) or `pipx`.

**1. Install the package.** Note the PyPI package is `graphifyy` (double-y) but the command you actually run is `graphify`:

```bash
uv tool install graphifyy
# or
pipx install graphifyy
```

**2. Register it with your AI assistant.** For Claude Code:

```bash
graphify install
```

Or scope it to just the current project instead of globally:

```bash
graphify install --project
```

Other assistants have their own subcommand:

```bash
graphify cursor install
graphify codex install
graphify gemini install
graphify copilot install
graphify aider install
```

**3. Grab optional extras** if you need them:

```bash
uv tool install "graphifyy[pdf]"     # PDF extraction
uv tool install "graphifyy[video]"   # video/audio transcription
uv tool install "graphifyy[sql]"     # SQL schema extraction
uv tool install "graphifyy[all]"     # everything
```

**4. Fix PATH issues** if the `graphify` command isn't found after install:

```bash
uv tool update-shell     # after uv install
pipx ensurepath          # after pipx install
```

**5. Generate the graph.** Inside your AI assistant:

```
/graphify .
```

(PowerShell users: drop the leading slash — `graphify .`)

This drops three files into `graphify-out/`: `graph.html` (an interactive, clickable force-directed graph), `GRAPH_REPORT.md` (key concepts and suggested questions), and `graph.json` (the queryable graph itself).

**6. Query it directly** from the CLI once it exists:

```bash
graphify query "How does the login page connect to authentication?"
graphify path "UserService" "DatabasePool"
graphify explain "RateLimiter"
```

Useful extraction flags for larger or evolving repos:

```bash
graphify extract ./src --code-only     # local AST only, no API key needed
graphify extract ./docs --update       # re-extract only changed files
graphify extract ./docs --mode deep    # richer semantic pass
```

**7. Wire it into the team workflow.** One person runs `/graphify .` and commits `graphify-out/`; everyone else pulls it and their assistant has the map immediately. `graphify hook install` auto-rebuilds the graph on every commit, and there's a git merge driver so `graph.json` unions cleanly instead of conflicting.



What sold me isn't the visualisation.it's `graphify path` and `graphify query` as blunt instruments for regression impact analysis. "What connects checkout page to database" is, word for word, the question I ask before scoping a test suite on an unfamiliar service. Having an agent answer it by walking a real, locally-built graph instead of guessing from vibes is exactly the kind of grounding brownfield work has been missing.