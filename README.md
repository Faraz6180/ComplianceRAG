---
title: ComplianceRAG
emoji: 🔎
colorFrom: blue
colorTo: indigo
sdk: gradio
app_file: app.py
pinned: false
license: mit
---

# ComplianceRAG — Multi-Agent RAG with Live Groundedness Scoring......

**Live demo:** [[Open the Space](https://huggingface.co/spaces/Faraz6180/ComplianceRAG)](https://huggingface.co/spaces/Faraz618/ComplianceRAG) <!-- update with your actual URL after deploying -->
<img width="1340" height="608" alt="image" src="https://github.com/user-attachments/assets/1518dafc-95c0-4452-bee1-b5242c3a1913" />


A small, self-contained Retrieval-Augmented Generation system that answers questions from a set of documents — but instead of just returning an answer, it shows you **how grounded that answer actually is** in the source material, which sentence it came from, and flags low-confidence answers for human review.

This was built to demonstrate the part of RAG systems that's usually invisible: the evaluation and trust layer, not just the retrieval-and-generate pipeline.

## Why this exists

Most RAG demos show retrieval working. They rarely show what happens when retrieval *fails* — when the answer drifts from the source, or the model fills a gap with something that sounds right but isn't backed by the document. This project makes that failure mode visible and measurable instead of hiding it.

## How it works

1. **Ingestion** — Upload a `.txt` or `.pdf`, or use the included sample compliance document. The document is split into overlapping chunks.
2. **Retrieval Agent** — Embeds the chunks with `sentence-transformers` and indexes them with FAISS. On a query, retrieves the top-k most relevant chunks.
3. **Generation Agent** — Sends the retrieved chunks + question to an LLM (via the Hugging Face Inference API) to produce an answer with inline citations like `[1]`, `[2]`.
4. **Critic / Evaluation Agent** — Independently re-checks the generated answer against the retrieved chunks using sentence-level similarity scoring, and produces:
   - A **groundedness score** (0–1): how well-supported the answer is by the retrieved text
   - A **citation accuracy check**: whether each `[n]` citation actually points to a chunk that supports the claim near it
   - A **confidence flag**: 🟢 Grounded / 🟡 Partially grounded / 🔴 Low confidence — needs human review
5. **Human-in-the-loop view** — Low-confidence answers are visually flagged and the underlying retrieved chunks are shown side-by-side with the answer, so a reviewer can verify in seconds without re-reading the whole document.

## Architecture

```
 ┌──────────────┐      ┌───────────────────┐      ┌────────────────────┐
 │   Document    │ ──▶  │   Retrieval Agent   │ ──▶ │   Generation Agent  │
 │  (chunked +   │      │  (FAISS + sentence- │     │  (LLM via HF API,   │
 │   embedded)   │      │   transformers)     │     │   cited answer)     │
 └──────────────┘      └───────────────────┘      └────────┬───────────┘
                                                              │
                                                              ▼
                                                  ┌────────────────────────┐
                                                  │     Critic Agent        │
                                                  │  (groundedness scoring, │
                                                  │   citation validation)  │
                                                  └────────┬───────────────┘
                                                              │
                                                              ▼
                                                  🟢 / 🟡 / 🔴 + flagged for review
```

This separation — retrieval, generation, and an independent critic that checks the generation against the retrieval — is a small-scale version of the orchestrator/specialist/critic pattern used in production multi-agent systems.

## Tech stack

- **Gradio** — UI and app framework
- **sentence-transformers** (`all-MiniLM-L6-v2`) — embeddings for retrieval and for groundedness scoring
- **FAISS** — vector search
- **Hugging Face Inference API** — LLM generation (no local GPU required)
- **PyPDF** — PDF text extraction

## Try it yourself

1. Click "Load Sample Document" or upload your own `.txt`/`.pdf`
2. Ask a question about the document
3. Watch the three-agent pipeline run: Retrieval → Generation → Critic
4. Check the groundedness score and citation breakdown
5. Try asking something the document *doesn't* cover — watch the confidence flag drop to 🔴 and see why

## What I'd build next

- Swap single-document FAISS for a proper vector DB (ChromaDB/Pinecone) to support multi-document corpora
- Add hybrid retrieval (BM25 + dense) for queries with exact keyword requirements (clause numbers, names)
- Persist a ground-truth Q&A set and track groundedness drift over time (the evaluation-as-CI/CD pattern)
- Add a second specialist agent for numeric/date claims specifically, since those are where hallucination is costliest in compliance contexts

## Author

Built by [Faraz Mubeen Haider](https://github.com/Faraz6180) — AI Engineer focused on agentic RAG systems, LLM evaluation, and production-grade AI applications.

[LinkedIn](https://linkedin.com/in/farazmubeen-ai) · [GitHub](https://github.com/Faraz6180)
