# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single Jupyter notebook (`simple_local_rag.ipynb`) that demonstrates a minimal local RAG (Retrieval-Augmented Generation) system. It is a teaching artifact, not a production app.

## Environment setup
```

Install dependencies:

```
pip install chromadb sentence-transformers groq python-dotenv
```

## Running the notebook

```
jupyter notebook simple_local_rag.ipynb
```

Or run all cells non-interactively:

```
jupyter nbconvert --to notebook --execute simple_local_rag.ipynb
```

## Architecture

The notebook is a linear, self-contained pipeline executed top-to-bottom:

1. **Load** — reads `*.txt` files from `knowledge_base/`
2. **Chunk** — splits each document into 300-char overlapping windows (50-char overlap) using `chunk_text()`
3. **Embed** — encodes chunks with `sentence-transformers/all-MiniLM-L6-v2` (384-dim vectors, ~80 MB download on first run)
4. **Store** — persists embeddings in ChromaDB at `./chroma_db/` using cosine similarity
5. **Retrieve** — `retrieve(query, n_results)` embeds the question and queries ChromaDB
6. **Generate** — `rag_answer(question)` builds a context string from retrieved chunks and calls `llama-3.1-8b-instant` via the Groq API

The `chroma_db/` directory is the persisted vector store. Delete it to force a full re-index on the next run.

## SSL note

The notebook disables SSL verification (`verify=False`) for the Groq HTTP client to work around corporate/school network SSL inspection proxies. This is intentional.

## Key tunable parameters

| Parameter | Default | Location |
|-----------|---------|----------|
| `chunk_size` | 300 chars | `chunk_text()` call |
| `overlap` | 50 chars | `chunk_text()` call |
| `n_results` | 2 | `retrieve()` / `rag_answer()` calls |
| `DOCS_FOLDER` | `./knowledge_base` | top of notebook |
| embedding model | `all-MiniLM-L6-v2` | `SentenceTransformer(...)` call |
| LLM model | `llama-3.1-8b-instant` | `groq_client.chat.completions.create(...)` |
