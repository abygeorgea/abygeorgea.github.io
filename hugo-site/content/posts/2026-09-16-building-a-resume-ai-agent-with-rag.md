---
title: "Building an AI Agent That Answers Questions About My Resume"
date: 2026-09-16T08:25:31+10:00
draft: false
slug: "building-a-resume-ai-agent-with-rag"
categories:
  - Agentic development
tags:
  - Agentic development
description: "How I built a RAG agent that answers recruiter questions from my resume: the architecture, chunking, embeddings, Pinecone ingestion and retrieval."
---

A resume is a static document. A recruiter reading it has questions it cannot answer. Did you create AI agents? What did you actually do in your last role? Have you used Kafka?

So I built an agent that answers those questions for me. You can try it on my [resume page](/resume/). Ask it anything about my background and it answers in the first person, with sources.

This is the first of four posts about how I built it. Today we cover the architecture and the data pipeline. By the end you will know how a question turns into the right piece of my resume.

## What We Are Building

The pattern is called RAG, retrieval augmented generation. The idea is simple. The model does not know my career. So we find the relevant parts of my resume first, and hand only those to the model.

```
data/resume.md   (the single source of truth)
      |
      v   ingest: chunk, embed, upsert
   Pinecone      (vector index)
      |
   FastAPI backend   POST /chat
      1. input guard: refuse obvious attacks
      2. embed the question, retrieve the top matching chunks
      3. Claude writes a grounded answer with citations
      4. verify: every citation must be a chunk we really retrieved
      5. rate limits and a daily cost cap
      |
   Static chat page   (embedded on my blog)
```

Read the diagram from top to bottom. The resume file is processed once, offline, into a vector index. Then every visitor question flows through the five numbered steps in the backend, and the answer goes back to the chat page.

Steps 1, 4 and 5 are the safety net. We cover them in the next post. Today we build everything up to step 2.

## The Stack and Why

I made the stack decisions on day one so I would stop debating them.

| Piece | Choice | Why |
|---|---|---|
| Vector database | Pinecone, free tier | Managed. No servers to run. |
| Embeddings | all-MiniLM-L6-v2 | Small, free, runs locally. A resume is tiny, so quality is not a concern. |
| Generation | Claude Haiku 4.5 | Cheap and fast. Follows strict rules well. |
| Backend | Python and FastAPI | Most RAG examples are in Python. |
| Hosting | Render, free tier | A real server, no serverless surprises while learning. |
| Testing | pytest and promptfoo | Unit tests for code, evals for the AI behaviour. |

Two of these choices changed later, for good reasons. I will point them out when we get there.

## Step 1: One Source of Truth

Everything the agent knows lives in one file, `data/resume.md`. It is plain markdown. Headings for sections, bullets for achievements.

```markdown
# Aby George
## Professional Experience
### COMPANY_NAME_XYZ
**Leadership & Management**
- First achievement bullet
- Second achievement bullet
```

This layout is not just for readability. The chunker in the next step reads these headings to decide where one piece of knowledge ends and the next begins. A `###` line starts a job. A standalone bold line starts a sub topic inside that job.

One rule matters a lot here. If the agent should know something, it goes in this file. If it is not in this file, the agent has to say "I don't have that." That rule is what makes the answers trustworthy.

My phone and email live in this file too, but not as real values. They are placeholders.

```markdown
- Phone: {{CONTACT_PHONE}}
- Email: {{CONTACT_EMAIL}}
```

The ingestion step swaps them in from environment variables at the moment it builds the chunks. So the real values exist only in my environment and in the vector index. They never appear in the repository or its history. It is a small habit, and it is worth doing on day one.

## Step 2: Chunking by Heading

We cannot hand the whole resume to the model on every question. It is wasteful, and it buries the relevant part. So we split the file into chunks.

The obvious approach is to cut every N characters. It is a bad fit here. A cut can land in the middle of a sentence, or split a job from its bullets. A resume already has structure, so we use it. We split on headings.

Each chunk keeps its full heading path, like a breadcrumb.

```
Professional Experience > COMPANY_NAME_XYZ > Leadership & Management
```

Here is the core of the chunker, trimmed.

```python
HEADER_RE = re.compile(r"^(#{1,6})\s+(.*)$")
BOLD_ONLY_RE = re.compile(r"^\*\*(.+?)\*\*:?\s*$")

@dataclass
class Chunk:
    heading: str
    text: str

def chunk_markdown(text: str) -> list[Chunk]:
    heading_stack: list[tuple[int, str]] = []
    current_lines: list[str] = []
    chunks: list[Chunk] = []

    def flush() -> None:
        content = "\n".join(current_lines).strip()
        current_lines.clear()
        if content:
            path = " > ".join(title for _, title in heading_stack)
            chunks.append(Chunk(heading=path, text=content))

    for line in text.splitlines():
        header = HEADER_RE.match(line)
        bold = BOLD_ONLY_RE.match(line)
        if header:
            flush()
            level = len(header.group(1))
            heading_stack[:] = [h for h in heading_stack if h[0] < level]
            heading_stack.append((level, header.group(2).strip()))
        elif bold:
            flush()
            heading_stack[:] = [h for h in heading_stack if h[0] < 7]
            heading_stack.append((7, bold.group(1).strip()))
        else:
            current_lines.append(line)
    flush()
    return chunks
```

Let me walk through the logic.

* We read the file line by line and collect ordinary lines in `current_lines`.
* When we meet a heading, we call `flush()`. That turns the collected lines into one chunk, labelled with the current breadcrumb.
* `heading_stack` remembers where we are in the document. It holds pairs of a heading level and a title.
* When a new heading arrives, we first drop every heading at the same level or deeper. Then we push the new one. So a new `###` job replaces the previous job, but keeps the `##` section above it.
* Joining the stack with ` > ` gives the breadcrumb path.
* A standalone bold line gets an artificial level of 7. That is deeper than any real markdown heading, so it always nests under the current job and is replaced by the next bold line.
* The last `flush()` after the loop saves the final chunk.

The chunker has no dependencies at all. That means we can unit test it without loading a model or touching the network.

My resume produces about 32 chunks. That is small enough to reason about by hand.

## Step 3: Turning Text Into Numbers

Now the clever part. To find the chunks that match a question, we need a way to measure meaning. Keyword search fails here. "Agent creation" and "building AI assistants" share no words, but they are close in meaning.

An embedding model solves this. It turns a piece of text into a list of numbers, a vector. Texts with similar meaning get similar vectors. We compare vectors with cosine similarity, which is just the angle between them.

The model I picked is all-MiniLM-L6-v2. It outputs 384 numbers per text.

```python
def embed(texts: list[str]) -> list[list[float]]:
    """L2-normalised embeddings, so cosine similarity equals dot product."""
    vectors = np.asarray(list(get_model().embed(texts)), dtype=np.float32)
    vectors /= np.linalg.norm(vectors, axis=1, keepdims=True)
    return vectors.tolist()
```

What is happening here?

* `get_model().embed(texts)` runs the model over a batch of texts and returns one vector per text.
* We stack them into a NumPy array so we can do maths on all of them at once.
* The `vectors /= norm` line scales every vector to length one. That is called L2 normalisation.
* Once every vector has length one, cosine similarity becomes a plain dot product. That is faster, and it is what the vector database uses.

Notice that the same function embeds both the resume chunks at ingestion time and the visitor's question at query time. They must use the same model, or the numbers would not be comparable.

## Step 4: Storing Vectors in Pinecone

Pinecone stores the vectors and finds the nearest ones to a query. Ingestion is short. We embed every chunk, then upsert it with the heading and text as metadata.

```python
vectors = [
    {
        "id": chunk.id,
        "values": embedding,
        "metadata": {"heading": chunk.heading, "text": chunk.text},
    }
    for chunk, embedding in zip(chunks, embeddings)
]

index.delete(delete_all=True)   # never leave stale chunks behind
index.upsert(vectors=vectors)
```

The list comprehension pairs each chunk with its embedding. Each record has an id, the vector itself, and metadata. The metadata is what we get back on a search, so it carries the heading and the original text.

Notice the delete before the upsert. If I edit the resume and remove a bullet, I do not want the old version lingering in the index. Clearing first makes the script safe to run again and again. Upsert means insert or update, so re-running never creates duplicates either.

Creating the index is one call. The dimension must match the model, and the metric is cosine.

```python
pc.create_index(
    name="resume-index",
    dimension=384,
    metric="cosine",
    spec=ServerlessSpec(cloud="aws", region="<your-region>"),
)
```

The `dimension` of 384 is not a choice. It has to equal the number of values the embedding model outputs. If they disagree, Pinecone rejects every upsert. The `metric` tells Pinecone how to rank neighbours, and we match it to how we normalised the vectors. The `spec` picks the free serverless tier. The script only calls this when the index does not exist yet.

## Step 5: Retrieval, With No AI Yet

Before adding the language model, I built retrieval on its own as a command line tool. This was a good decision. If retrieval returns the wrong chunks, no prompt in the world will fix the answer. So test it in isolation first.

```python
def retrieve(query: str, top_k: int = 5):
    vector = embeddings.embed([query])[0]
    result = index.query(vector=vector, top_k=top_k, include_metadata=True)
    return [
        {"score": m["score"], "heading": m["metadata"]["heading"], "text": m["metadata"]["text"]}
        for m in result["matches"]
    ]
```

The function embeds the question with the same model, asks Pinecone for the `top_k` nearest vectors, and reshapes the matches into plain dictionaries. Each result has a similarity score, the heading, and the text.

```bash
python -m app.retrieval "Has he built AI agents?"
```

This runs the module as a script. It prints a ranked list of chunks with their scores. That is where I learned something useful. Scores swing a lot with phrasing. A concrete technical question scored 0.64. A legitimate but abstract question about visa status scored 0.21.

That killed an idea I had. I wanted to refuse any question whose best score was below some threshold. It cannot work. Any threshold that keeps the visa question would also let off topic questions through. So we need a better safety mechanism. That is the next post.

## Two Fixes That Retrieval Needed

Testing retrieval properly exposed two problems. I will tell the full story in the testing post. Here are the fixes, because they belong to the data pipeline.

**Fix one: embed the heading too.** A bullet under my last role says something like "Maintained continuous accountability for..." It never says the company name. That word only lives in the heading. So a question about that company had nothing to match against. The fix is to embed the heading together with the text.

```python
@property
def embedding_text(self) -> str:
    if not self.heading:
        return self.text
    return f"{self.heading}\n\n{self.text}"
```

This property builds the string we send to the embedding model. If there is a heading, we put it first, then a blank line, then the bullets. Now the company name is part of the vector, so a question about that company finds its own chunks.

Only the embedding sees the heading glued on. The text we later show the model stays clean, because the model already gets the heading as a label.

**Fix two: split long bullet lists.** One chunk had about fifteen bullets on many topics. Averaged into one vector, it resembled no single question. I measured a similarity of 0.054 for a leadership question against that chunk. That is barely above random. So any chunk over 500 characters made only of bullets is split into groups of three.

```python
def _split_long_bullet_chunk(chunk: Chunk) -> list[Chunk]:
    if len(chunk.text) <= 500:
        return [chunk]
    lines = [l for l in chunk.text.splitlines() if l.strip()]
    if not all(re.match(r"^- .+$", l) for l in lines):
        return [chunk]
    return [
        Chunk(heading=chunk.heading, text="\n".join(lines[i:i + 3]))
        for i in range(0, len(lines), 3)
    ]
```

The logic has two early exits and one split.

* If the chunk is short, return it unchanged. Short chunks are already focused.
* If any line is not a bullet, return it unchanged. We only want to split pure bullet lists, never prose.
* Otherwise walk the lines in steps of three and make a new chunk from each group. Every new chunk keeps the same heading, since it still belongs to the same section.

Small, focused chunks embed better. That is a rule of thumb worth remembering.

## Recap

We now have a pipeline that turns a question into the right pieces of my resume.

* One markdown file is the source of truth.
* The chunker splits on headings and keeps a breadcrumb path.
* An embedding model turns chunks and questions into vectors.
* Pinecone finds the nearest chunks.
* Embedding the heading, and splitting long bullet lists, made retrieval much better.

But retrieval only finds text. It does not stop a model from making things up, or from obeying a hostile question. In the next post we add the language model, and the guardrails that keep it honest. Read it here: [Keeping a RAG Agent Honest](/blog/2026/09/17/grounding-and-guardrails-for-a-rag-agent/).
