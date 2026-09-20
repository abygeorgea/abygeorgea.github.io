---
title: "Shipping the Agent: Eval Costs, Free Tier Deployment, and the Chat UI"
date: 2026-09-19T08:25:31+10:00
draft: false
slug: "shipping-a-resume-ai-agent-on-a-free-tier"
categories:
  - Agentic development
tags:
  - Agentic development
description: "Measuring and cutting AI eval costs, deploying a RAG agent to a free tier without running out of memory, and embedding the chat UI in a Hugo blog."
---

In the [previous post](/blog/2026/09/18/testing-an-ai-agent-with-pytest-and-promptfoo/), we built a test strategy and found real bugs. The agent works and it is tested. Now we ship it.

This post has three parts. First, what the evals cost and how I cut it. Second, three problems I found in a pre deployment check that would each have broken the live site. Third, the chat page itself, and how it ended up on my resume page.

As in the last post, the repository is private and this blog is public. So the configuration below is simplified, and I have left out anything an attacker could use, such as exact limits and trust settings.

## Part One: What Do Evals Actually Cost?

Every eval case that reaches the model is a paid API call. My full suite was running at what I guessed was 25 cents per run. And CI was running it on every push.

I did not want to guess. So I measured.

### Measure First

I added one log line per model call, with the real token counts.

```python
usage = getattr(response, "usage", None)
if usage is not None:
    print(
        f"[usage] input_tokens={getattr(usage, 'input_tokens', '?')} "
        f"output_tokens={getattr(usage, 'output_tokens', '?')}",
        file=sys.stderr,
    )
```

The logic is short.

* The API response carries a `usage` object with the input and output token counts.
* `getattr` with a default means the code keeps working when a test fake has no `usage` field. A logging line must never break the request.
* We print to standard error, one greppable line per call. Then I can add the numbers up with a script instead of estimating.

Then I ran the suites and added up the log lines. Here is what came back.

| Run | Cases | Paid calls | Tokens in, avg | Tokens out, avg | Cost | Time |
|---|---|---|---|---|---|---|
| Smoke | 12 | 10 | 2,632 | 182 | about 3.5 cents | 71 s |
| Full | 34 | 28 | 2,691 | 165 | about 9.9 cents | 3 min 26 s |

The cost uses the published Haiku 4.5 list price, one dollar per million input tokens and five per million output. That is my assumption, so check your own console. The result is about a third of a cent per paid case.

My guess had been wrong in both directions. Answers are short, about 170 tokens, because the prompt asks for concise ones. Measuring beat estimating. Notice that paid calls are fewer than cases. The injection cases are refused before any model call, so they are free.

### Cut the Duplicates

I found four redundant cases. One question was asked twice with slightly different assertions. Three topics were asked once as "he" and once as "you".

I merged each pair into one case, in the second person, with the union of both assertions. Then I proved nothing got weaker by comparing every case against the previous version with a script. Thirty two assertions were byte for byte identical. Exactly two were changed, each now stricter. Four cases were removed. None were added.

I kept some third person questions on purpose. Real visitors type both styles, so both must stay covered.

### Add a Smoke Tier

Now the useful trick. I tagged twelve cases as a cheap everyday tier. The tag is just a description prefix.

```yaml
- description: "smoke: ASX recency regression"
  vars:
    question: "Where did he work before COMPANY_NAME_XYZ?"
```

promptfoo can filter test cases by matching a regular expression against the description.

```bash
npx --yes promptfoo@0.118.0 eval -c promptfooconfig.yaml --no-cache --filter-pattern "^smoke"
```

Here is what each part does. `npx --yes` runs promptfoo without a global install. Pinning the version keeps runs repeatable. `-c` names the config file. `--no-cache` forces real calls, and I explain why below. `--filter-pattern "^smoke"` runs only cases whose description starts with "smoke".

Now I have three tiers.

| Tier | When | Cost |
|---|---|---|
| Free | Always. pytest and Node. | Nothing |
| Smoke | After each small change | About 3.5 cents |
| Full | Before a deploy, or after changing the prompt, resume, chunking or retrieval | About 10 cents |

The twelve smoke cases cover the highest risk behaviours. Recency, first person voice, a chip question, refusal, a missing tool, injection, and a control case.

### The Cache Trap

This one surprised me, and I think it is the most important lesson in this section.

promptfoo caches responses. I ran the smoke tier a third time with the backend completely stopped. It passed 12 of 12 in zero seconds.

That is what a free rerun looks like. It is also the danger. The cache is keyed on the request, which is the question. It is not keyed on my code. So after I change the prompt, the resume or retrieval, a cached run replays old answers and goes green against a system that no longer exists.

The rule is simple. After changing the agent, always use `--no-cache`.

There is a second surprise. `--no-cache` neither reads nor writes the cache, so a following run without the flag is not free either. And `--repeat 5` with the cache on only replays one saved answer five times. Repeat runs must use `--no-cache`, and they multiply the cost, so use them on one or two risky questions only.

If you take one thing from this section, take this. A green test that cannot fail is worse than no test.

### Stop Paying on Every Push

The last change was in CI. I split the pipeline into two workflows.

`ci.yml` runs the free unit tests on every push and pull request.

```yaml
on:
  push:
    branches: ["**"]
  pull_request:
    branches: ["**"]

jobs:
  unit-tests:
    steps:
      - run: python -m pytest tests/unit -v
      - run: node --test
```

The trigger block says run on a push to any branch, and on every pull request. The single job runs the Python tests and the browser side Node tests. Neither needs secrets or the network, so this workflow costs nothing.

`evals.yml` holds the paid job. It runs only when something the answers depend on changes, or when I start it by hand.

```yaml
on:
  push:
    branches: [master]
    paths:
      - "backend/**"
      - "data/**"
      - "tests/evals/**"
      - ".github/workflows/evals.yml"
  workflow_dispatch:

concurrency:
  group: evals-${{ github.ref }}
  cancel-in-progress: true
```

Read it like this.

* It triggers only on pushes to the main branch that touch the backend, the resume data, the eval cases, or the workflow itself. A docs or frontend change spends nothing.
* `workflow_dispatch` adds a manual run button in the Actions tab.
* The `concurrency` block groups runs by branch. With `cancel-in-progress`, a newer push cancels a run that is still going, so a quick second push does not pay twice.

There is a trade off, and I wrote it in the file. The two workflows now run in parallel. A push that breaks a unit test can still spend the eval money. I accepted that.

A third workflow, `ingest.yml`, re-indexes the resume in Pinecone whenever `data/resume.md` changes. So editing my resume updates the live agent automatically.

## Part Two: Three Problems Before Launch

When I said I was ready to deploy, I did not push. I ran a pre flight first. It found three problems, and any one of them would have hurt.

The check was clean on secrets. No API keys, phone or email anywhere in tracked files or in the whole git history. The repo is private. Then the real findings started.

### Problem One: Not Enough Memory

Render's free tier gives 512 MB of memory. Would my backend fit? I did not want to find out after deploying, so I measured it locally first.

Here is how I checked. The idea is simple. Run the backend the way production runs it, use it a bit, then read the memory of that one process.

**Step 1.** Start the backend without the reload option, so there is exactly one process to look at.

```bash
uvicorn app.main:app --port 8000
```

**Step 2.** Warm it up. Memory grows as the model loads and as requests arrive, so a reading at startup is too optimistic. I waited for the model to load, then sent a few real questions through the chat endpoint.

**Step 3.** Read the memory of the process that owns the port. In PowerShell:

```powershell
$conn = Get-NetTCPConnection -LocalPort 8000 -State Listen
Get-Process -Id $conn.OwningProcess |
  Select-Object Id, ProcessName,
    @{n='WorkingSetMB'; e={[math]::Round($_.WorkingSet64 / 1MB)}}
```

Here is the logic.

* `Get-NetTCPConnection` finds the listening socket on port 8000. That socket belongs to exactly one process, and `OwningProcess` is its ID. This avoids guessing which of many Python processes is the server.
* `Get-Process -Id` looks that process up.
* `WorkingSet64` is the physical memory the process currently holds, in bytes. Dividing by 1MB and rounding gives megabytes.

If you prefer plain Command Prompt, `tasklist /FI "IMAGENAME eq python.exe"` lists the same figure in its Mem Usage column. Repeat step 3 after each round of requests and write the numbers down.

One caution. Windows working set and Linux resident memory are close, but they are not identical. So treat the local number as a strong warning, and confirm it later on Render's own metrics page after the first deploy.

The readings were **511 MB after warm up**, and **541 MB after a few requests**. That is over the 512 MB limit. It would have crashed with an out of memory error.

Almost all of it was PyTorch. I was pulling in a huge library to run a model with only 22 million parameters. The fix was to run the same model on ONNX Runtime through the `fastembed` package.

I measured again with the same three steps. The result was 211 to 240 MB, and startup dropped from 64 seconds to 4. That leaves plenty of headroom. Same method before and after is what makes the comparison trustworthy.

### Problem Two: A Silent Quality Drop

Here is the subtle part. After switching, I compared vectors against the PyTorch ones.

* Query vectors matched exactly. Cosine 1.000000.
* Document vectors did not. The worst match was 0.91.

That is a silent retrieval quality drop. Nothing would have crashed. Answers would just have quietly got worse.

My first guess was wrong. I thought it was truncation at 256 tokens, but no chunk is that long. The real cause was that fastembed's tokenizer truncates at **128** tokens, while the model uses 256. Eleven of my 32 chunks are longer than 128 tokens, so their tails were dropped from the embedding.

Passing a max length option is silently ignored. So the fix overrides the tokenizer directly, and then verifies that it took.

```python
tokenizer = model.model.tokenizer
padding = tokenizer.padding
tokenizer.enable_truncation(max_length=256)
tokenizer.enable_padding(pad_id=padding["pad_id"], pad_token=padding["pad_token"])

if tokenizer.truncation["max_length"] != 256:
    raise RuntimeError("embedding tokenizer truncation is not 256")
```

Walk through it.

* We reach into the loaded model and grab its tokenizer object.
* We save the current padding settings first, because changing truncation resets them.
* `enable_truncation(max_length=256)` raises the cut off from 128 to what the model was trained for.
* `enable_padding(...)` puts padding back, using the saved pad id and token. Without this, batches of different length texts would not line up.
* The last check reads the setting back and raises an error if it did not stick. That turns a silent failure into a loud one.

After that, every chunk matched the PyTorch vector at cosine 1.000000. Notice the last three lines. When a setting can be silently ignored, assert that it applied.

### Problem Three: One Rate Limit for Everyone

Behind Render's proxy, the address my app sees is the proxy's, not the visitor's. So my per address rate limit would be shared by all visitors together. A handful of real people could lock everyone out.

I reproduced this faithfully, and there is a trap in doing so. A naive local test hides the bug. Uvicorn trusts localhost by default, so calling through localhost makes the server honour the forwarded address header. I had to call through the machine's network address, so the "proxy" was untrusted, like on Render.

The fix is in the start command.

```yaml
startCommand: uvicorn app.main:app --host 0.0.0.0 --port $PORT --proxy-headers
```

The `--proxy-headers` flag tells uvicorn to read the visitor's real address from the header the proxy adds, instead of using the proxy's own address. There is one more setting that says which proxies to trust. I am deliberately not publishing it. Trust settings like that are easy to get wrong, and the right value depends on your host. If you do this yourself, scope it as tightly as your platform allows and test it the way I described above.

### The Render Blueprint

Both services are defined in one file, `render.yaml`. A blueprint means no manual dashboard clicking. Here is a simplified version.

```yaml
services:
  - type: web
    name: resume-agent
    runtime: python
    rootDir: backend
    plan: free
    healthCheckPath: /health
    buildCommand: pip install -r requirements.txt && python -c "from app import embeddings; embeddings.get_model()"
    startCommand: uvicorn app.main:app --host 0.0.0.0 --port $PORT --proxy-headers
    buildFilter:
      paths:
        - backend/**
    envVars:
      - key: PINECONE_API_KEY
        sync: false
      - key: ANTHROPIC_API_KEY
        sync: false
      - key: ALLOWED_ORIGINS
        value: https://your-chat-page.example.com

  - type: web
    name: resume-web
    runtime: static
    staticPublishPath: ./frontend
    buildFilter:
      paths:
        - frontend/**
```

The file describes two services. Here is how to read the API one.

* `rootDir` points Render at the backend folder, and `plan: free` picks the free tier.
* `healthCheckPath` lets Render probe the app and know when it is ready.
* `buildCommand` installs dependencies and then runs a one line Python command that loads the embedding model once. That downloads the model files at build time, so they ship with the app and a cold start does not re-download them.
* `startCommand` is what runs in production.
* `buildFilter` rebuilds the service only when files under `backend` change.
* `sync: false` marks a secret. Render asks for its value in the dashboard, so the secret never lives in git.
* `ALLOWED_ORIGINS` is the CORS allow list. It must be the chat page's exact origin, with no path and no trailing slash. A mismatch shows up as a CORS error in the browser.

The second service is just the static chat page. It has no build step, it publishes the frontend folder as it is, and it rebuilds only when that folder changes.

One hosting decision changed. I first planned to use GitHub Pages for the chat page. Pages is not available for private repos on a free account, and I wanted to keep the repo private. So the chat page is a Render static site too.

After the deploy I did not trust the word "Deployed". That only means the build and start succeeded. I checked behaviour from outside. The static site served my page, and the API health endpoint answered with a 200 in about a quarter of a second.

## Part Three: The Chat Page

The frontend is deliberately plain. HTML, one stylesheet and a small script. No framework, no build step.

### Cold Starts

The free API server sleeps after about fifteen minutes idle. The first request afterwards can take up to a minute. That is a bad first impression, so the page does two things.

It wakes the server as soon as the page opens, before the visitor has typed anything.

```javascript
fetch(RESUME_AGENT_API_URL.replace(/\/chat$/, "/health")).catch(() => {});
```

The chat address ends in `/chat`. The regular expression swaps that ending for `/health`, so we call the cheap health check. We do not use the response at all. The `.catch(() => {})` swallows any failure, because a failed wake up ping must never show an error to a visitor.

And if an answer is still pending after six seconds, it explains why, instead of leaving a silent spinner.

```javascript
const coldStartTimer = setTimeout(() => {
  loadingEl.textContent =
    "This runs on a free server that sleeps when idle, so the first answer can take up to a minute. Thanks for your patience!";
}, 6000);
```

When the visitor sends a question, we start a six second timer. If the answer arrives first, the code clears the timer. If not, the timer fires and swaps the loading message for the explanation. A fast answer never shows the note.

The API also warms its own dependencies at startup, so the first question does not pay for loading the model.

### Suggestion Chips and Sources

Four chips sit above the input. Test automation, test management, agentic development and leadership. Each chip sends a fuller, first person question, so retrieval has enough to work with.

Every answer has a collapsible "How I answered" section listing the sources it relied on. That is the citation list from the verifier, shown to the visitor. Transparency is a feature. It lets a recruiter see the answer is grounded in my actual resume.

### The Architecture Panel

There is also a collapsed "Technical architecture" card describing the design. It is written impersonally, in engineering language, and it lists the guards, the retrieval settings and the tests.

Accuracy was the main constraint. A public page must describe what really exists. So every number on it was read from the code, and, as I mentioned in the testing post, a unit test pins each claim to the source. It is collapsed by default, so the chat stays the star of the page. Open, it scrolls inside itself, so it cannot bury the input box on a phone.

### Embedding It in This Blog

The last step is the one you may have already seen. My blog is a Hugo site with the PaperMod theme, and the agent now sits at the top of my [resume page](/resume/).

Embedding is an iframe. But first I checked that the agent's host allows framing. The response has no `X-Frame-Options` header and no `frame-ancestors` policy, so a frame works. Always check this first, because a blocked frame fails silently.

The URL lives in the site config, so I can change or remove it without touching a template.

```toml
[params.resume]
  agentUrl = "https://your-chat-page.example.com/"
```

That is one line in `hugo.toml`. Any template can read it as `site.Params.resume.agentUrl`.

And the layout renders it only if the parameter exists.

```html
{{- with site.Params.resume.agentUrl }}
<section class="resume-section resume-agent">
  <h2>Ask about my resume</h2>
  <iframe class="resume-agent-frame" src="{{ . }}" title="Resume AI agent"
          loading="lazy" allow="clipboard-write"></iframe>
  <p><a href="{{ . }}" target="_blank" rel="noopener">Open in a new tab</a> if it does not load.</p>
</section>
{{- end }}
```

The `with` block in Hugo templates runs only when the value is set, and inside it `.` means that value. So if I delete the config line, the whole section disappears cleanly, with no empty box left behind. The same value feeds both the iframe and the fallback link.

Two small choices are worth explaining. `loading="lazy"` means the frame loads only when a visitor scrolls near it. And there is a plain link underneath, as a fallback if the frame fails. A free server that is asleep is a real failure mode, so the page should still work. The `rel="noopener"` on that link stops the new tab from getting a handle back to this page.

## Known Limitations

I keep a short honest list in the README. You should too.

* **Cold starts.** Free servers sleep. The first answer can be slow.
* **External dependencies.** Pinecone and the model provider are outside my control. If either is down, the agent is down.
* **One small instance.** The limits are designed for a low traffic portfolio, not for scale.

## What I Would Do Next

* Give chunks short numeric IDs, and have the model cite IDs instead of long heading strings. That is more robust than any string matching. I noted it after the citation bug, and I would do it if mismatches come back.
* Show the parent heading in the "How I answered" list. Right now an answer citing four employers shows "Leadership & Management" four times.
* Move the rate limiter state to a shared store if traffic ever grows.

## Series Recap

Across four posts we built and shipped an agent.

1. [Building the pipeline](/blog/2026/09/16/building-a-resume-ai-agent-with-rag/). Chunking, embeddings, Pinecone and retrieval.
2. [Keeping it honest](/blog/2026/09/17/grounding-and-guardrails-for-a-rag-agent/). Structured output, verification, injection defence and API limits.
3. [Testing it](/blog/2026/09/18/testing-an-ai-agent-with-pytest-and-promptfoo/). Unit tests, evals and the bugs they found.
4. Shipping it. Costs, deployment and the chat page.

If I had to compress it to one sentence, it would be this. Do not trust the model, check its output, and test the whole system with real questions before anyone else does. Go and try it on my [resume page](/resume/).
