---
name: r-ragnar
description: Use when code loads or uses ragnar (library(ragnar), ragnar::), implementing RAG in R, creating vector stores, embedding documents for semantic search, serving stores over MCP, or building retrieval pipelines
---

# ragnar: Retrieval Augmented Generation in R

## Overview

**ragnar implements RAG in R.** Create DuckDB-backed vector stores, embed documents, register retrieval tools with ellmer chat sessions, or serve stores over MCP. LLMs search your documents before answering.

**Install:** `install.packages("ragnar")` (CRAN 0.3.0)

## References

Read `references/API.md` before writing code.

- `references/API.md` - Complete function reference
- `references/package-docs.md` - Vector store setup and usage
- `references/rag.md` - RAG patterns and ellmer integration

## When to Use

- LLM needs to search your documentation
- Implement semantic or hybrid (vector + BM25) search over documents
- Create persistent vector database from documents
- Expose a vector store to MCP-aware clients (e.g., Claude Code)
- RAG workflows in R

## When NOT to Use

- Just need keyword search (use grep/Grep tool)
- Documents fit in single prompt (<100k tokens)
- Building chatbot without document search (use r-ellmer only)

## Quick Reference

```r
library(ragnar)

# 1. Read and chunk documents
docs <- read_as_markdown("docs/")
chunks <- markdown_chunk(docs)

# 2. Create store (uses embed_openai() by default; OPENAI_API_KEY env var)
store <- ragnar_store_create("docs.duckdb", embed = embed_openai())
ragnar_store_insert(store, chunks)
ragnar_store_build_index(store)

# 3. Register with ellmer chat
library(ellmer)
chat <- chat_openai()
ragnar_register_tool_retrieve(chat, store)

# 4. Now chat searches docs before answering
chat$chat("What does the documentation say about RAG?")

# Local embeddings (Ollama; defaults to embeddinggemma in 0.3.0)
store <- ragnar_store_create(
  "local.duckdb",
  embed = embed_ollama()
)

# Concurrent ingestion (added 0.3.0; uses mirai)
ragnar_store_ingest(store, paths = list.files("docs/", full.names = TRUE))

# Serve store over MCP (added 0.3.0)
mcp_serve_store(store)
```

## Common Mistakes

| Issue | Solution |
|-------|----------|
| Wrong function name | Functions are `embed_openai()`, `embed_ollama()`, etc. (no `ragnar_` prefix on embedders) |
| Embedding mismatch | Store and retrieval must use same embedding provider/model |
| Store not registered | Call `ragnar_register_tool_retrieve(chat, store)` after creating chat |
| Ollama embeddings not working | Start `ollama serve` first; pull model (`ollama pull embeddinggemma`) |
| `ragnar_find_links()` returns more than expected | Default changed in 0.3.0 to `children_only = FALSE`; pass `children_only = TRUE` to restrict |
| Tool name unexpected | 0.3.0 renamed retrieval tool prefix to `search_{store@name}` (was `rag_retrieve_from_*`) |
| Index not built | Call `ragnar_store_build_index(store)` after inserts for fast retrieval |
| Poor retrieval quality | Inspect with `ragnar_store_inspect(store)` or `ragnar_store_atlas(store)`; revisit chunking |

## Core Functions

**Document Processing:**
- `read_as_markdown()`: Convert files/URLs to markdown (PDF, DOCX, PPTX, HTML, EPUB, ZIP, YouTube)
- `ragnar_find_links()`: Crawl page or sitemap.xml for links
- `markdown_chunk()`: Semantic chunker with overlap and heading augmentation

**Store Management:**
- `ragnar_store_create()` / `ragnar_store_connect()`: Create or open a DuckDB store
- `ragnar_store_insert()` / `ragnar_store_update()`: Add or update chunks
- `ragnar_store_ingest()`: Concurrent ingestion (mirai-backed, 0.3.0+)
- `ragnar_store_build_index()`: Build retrieval index
- `ragnar_store_inspect()`: Launch interactive Store Inspector
- `ragnar_store_atlas()`: Visualize embeddings via Embedding Atlas (0.3.0+)

**Embeddings (no `ragnar_` prefix):**
- `embed_openai()`, `embed_ollama()`, `embed_lm_studio()`
- `embed_azure_openai()` (0.3.0+), `embed_bedrock()`, `embed_databricks()`
- `embed_google_gemini()`, `embed_google_vertex()`, `embed_snowflake()` (0.3.0+)

**Retrieval & Integration:**
- `ragnar_retrieve()`: Hybrid retrieval (also accepts vector queries in 0.3.0)
- `ragnar_retrieve_vss()`: Vector similarity search
- `ragnar_retrieve_bm25()`: Keyword/BM25 (results ordered by descending score in 0.3.0)
- `chunks_deoverlap()`: Merge overlapping chunks in results
- `ragnar_register_tool_retrieve()`: Expose store as ellmer tool
- `mcp_serve_store()`: Serve store via MCP (0.3.0+)

## Advanced

See `references/` for:
- **API.md**: Complete function reference
- **rag.md**: RAG workflow details
- **package-docs.md**: Full package documentation

## Integration

**With ellmer:** `ragnar_register_tool_retrieve(chat, store)`
**With MCP clients:** `mcp_serve_store(store)` (Claude Code, etc.)
**With vitals (evaluation):** See r-vitals skill for RAG testing
**Cross-package patterns:** See r-ai meta-skill
