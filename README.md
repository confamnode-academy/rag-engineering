# RAG Engineering

Build a RAG system from scratch, and learn how to tell whether it works.

Most tutorials wire a vector database to an LLM and stop. You get something that
answers questions, no way to know if the answers are good, and no idea what to fix
when they aren't. This course is about the second half.

## Start

```bash
docker compose up
```

Open http://localhost:8888 and go to `modules/01-foundations/`.

Ctrl-C to stop.

If you'd rather not use Docker, open the notebooks in Google Colab or your own
JupyterLab. They work the same way.

## What you'll build

A RAG system over business documents — bank policies, regulatory
circulars, an annual report, board minutes, a scanned tax notice.

The documents are deliberately difficult. One policy exists in two versions with
different numbers, so a careless system quotes the outdated one. A table splits
across a page break and loses its headers. One document is a photocopy with no
extractable text. Each module fixes one of these, and you measure the improvement.

## Modules

1. **Foundations** — what RAG is, when to use something else, where it breaks
2. **Baseline pipeline** — build one in plain Python. It'll be bad. Write a small
   test that shows how bad.
3. **Document ingestion** — PDFs, Word, spreadsheets, slides, email, a scan
4. **Chunking** — how to split documents, and whether the clever methods help
5. **Embeddings and vector storage** — model choice, Qdrant, a working retriever
6. **Evaluation** — a proper test set, and why the module 02 test misled you
7. **Improving retrieval** — keyword, hybrid, reranking, each one measured

Modules 8–18 cover conversational RAG, the application layer, grounding, agents,
multimodal, security and production.

## API keys

Copy `.env.example` to `.env` and fill it in. Modules 01–04 need nothing.

**[OpenRouter](https://openrouter.ai/keys)** — needed from module 02. One key
covers generation, embeddings and reranking, so it's the only model provider
you'll sign up for.

Start on the free tier; switch to a cheap paid model like DeepSeek or Gemini
Flash if you hit limits. Compare prices at
[openrouter.ai/models](https://openrouter.ai/models).

**Embeddings run locally**, not through the API — free and unlimited, so you can
re-embed the corpus as often as you like. Module 04 depends on that. The first
run downloads the model; after that it's cached.

Model names are set in the notebooks. If you change one, note it alongside your
scores — a different model means a different baseline.

**[Qdrant Cloud](https://cloud.qdrant.io)** — needed from module 05. Free tier,
no card.

On Colab, use the secrets panel with the same variable names.

## Layout

```
modules/    the notebooks
corpus/     the documents and the test questions
results/    saved scores, so you can see progress
```

Read `corpus/README.md` early — it explains what's wrong with each document.

## The documents are synthetic

The organisations don't exist and the figures are invented; every file says so in
its footer. They behave like real Nigerian business documents so the problems are
real, but nothing in them is a fact about any actual company.
