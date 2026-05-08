# ragnar API Reference

Complete function reference for the ragnar package - Retrieval-Augmented Generation (RAG) in R. Reflects CRAN version 0.3.0.

## Document Processing

| Function | Purpose |
|----------|---------|
| `read_as_markdown()` | Convert files / URLs to Markdown (PDF, DOCX, PPTX, HTML, EPUB, ZIP, YouTube) |
| `MarkdownDocument` | S7 class returned by `read_as_markdown()` |
| `ragnar_find_links()` | Crawl a page or sitemap.xml for links |

**Notes for 0.3.0:**
- `read_as_markdown()` restored YouTube transcript fetching and gained an `origin` argument and pluggable formatter support.
- `ragnar_find_links()` default for `children_only` is now `FALSE` (was `TRUE`); pass `children_only = TRUE` to restrict to descendants. It also supports `sitemap.xml` parsing and a `validate` argument for filtering broken links (since 0.2.1).

**Example:**
```r
library(ragnar)

# Read documents (file, directory, or URL)
docs <- read_as_markdown("docs/")
docs <- read_as_markdown("https://example.com/page.html")

# Find links (returns ALL links by default in 0.3.0)
links <- ragnar_find_links("https://example.com")
links <- ragnar_find_links("https://example.com", children_only = TRUE, depth = 2)
links <- ragnar_find_links("https://example.com/sitemap.xml", validate = TRUE)
```

## Text Chunking

| Function | Purpose |
|----------|---------|
| `markdown_chunk()` | Chunk a Markdown document at semantic boundaries (with overlap and heading augmentation) |
| `MarkdownDocumentChunks` | S7 class returned by `markdown_chunk()` |
| `ragnar_chunks_view()` | View chunks with the Store Inspector |

**Example:**
```r
chunks <- markdown_chunk(docs)
chunks <- markdown_chunk(docs, max_size = 1600, overlap = 200)

# Inspect chunks interactively
ragnar_chunks_view(chunks)
```

`markdown_chunk()` was introduced in 0.2.0 as the recommended chunker; it preserves semantic boundaries and emits overlapping chunks that can be merged with `chunks_deoverlap()` after retrieval.

## Store Management

| Function | Purpose |
|----------|---------|
| `ragnar_store_create()` | Create a DuckDB-backed vector store |
| `ragnar_store_connect()` | Connect to an existing store |
| `ragnar_store_insert()` | Insert chunks into a store |
| `ragnar_store_update()` | Insert or update chunks (idempotent on origin) |
| `ragnar_store_ingest()` | Concurrently ingest documents into a store (mirai-backed, 0.3.0+) |
| `ragnar_store_build_index()` | Build retrieval index for fast queries |
| `ragnar_store_inspect()` | Launch interactive Store Inspector (Shiny) |
| `ragnar_store_atlas()` | Visualize embeddings using Embedding Atlas (0.3.0+) |

**Notes:**
- Stores are versioned. New stores default to `version = 2` (introduced 0.2.0), which supports deoverlapping and heading augmentation.
- `ragnar_store_ingest()` requires `mirai >= 2.5.1`.
- Embedding retry policy is configurable via `options(ragnar.embed.req_retry = ...)` (0.3.0+).

**Example:**
```r
# Create store with OpenAI embeddings (default)
store <- ragnar_store_create("docs.duckdb", embed = embed_openai())

# Insert chunks
ragnar_store_insert(store, chunks)

# Concurrent ingestion (read + chunk + embed in parallel)
ragnar_store_ingest(store, paths = list.files("docs/", full.names = TRUE))

# Build index after inserts
ragnar_store_build_index(store)

# Reconnect later
store <- ragnar_store_connect("docs.duckdb")

# Inspect / visualize
ragnar_store_inspect(store)
ragnar_store_atlas(store)
```

## Embeddings

All embedders share a common interface and return a function suitable for use as `embed = ` in `ragnar_store_create()`. Note: function names do **not** carry a `ragnar_` prefix.

| Function | Purpose |
|----------|---------|
| `embed_openai()` | OpenAI embeddings (default model: `text-embedding-3-small`) |
| `embed_ollama()` | Ollama embeddings (default model in 0.3.0: `embeddinggemma`) |
| `embed_lm_studio()` | LM Studio (added 0.2.1) |
| `embed_azure_openai()` | Azure AI Foundry (added 0.3.0) |
| `embed_bedrock()` | AWS Bedrock |
| `embed_databricks()` | Databricks Foundation Model APIs |
| `embed_google_gemini()` | Google Gemini API (added 0.2.1) |
| `embed_google_vertex()` | Google Vertex API |
| `embed_snowflake()` | Snowflake Cortex Embedding (added 0.3.0) |

**Example:**
```r
# OpenAI (default)
store <- ragnar_store_create("store.duckdb", embed = embed_openai())

# Local Ollama (no API key)
store <- ragnar_store_create(
  "local.duckdb",
  embed = embed_ollama()  # default: embeddinggemma in 0.3.0
)

# Azure
store <- ragnar_store_create(
  "azure.duckdb",
  embed = embed_azure_openai(
    endpoint = "https://your-resource.openai.azure.com",
    deployment_id = "your-embedding-deployment",
    api_version = "2024-02-01"
  )
)
```

## Retrieval

| Function | Purpose |
|----------|---------|
| `ragnar_retrieve()` | Hybrid retrieval (vector + BM25). Accepts text or vector queries (0.3.0+) |
| `ragnar_retrieve_vss()` | Vector similarity search (gained `query_vector` arg in 0.2.0) |
| `ragnar_retrieve_bm25()` | BM25 keyword retrieval (results ordered by descending score in 0.3.0) |
| `chunks_deoverlap()` | Merge overlapping chunks in retrieval results |
| `ragnar_register_tool_retrieve()` | Register store as an ellmer chat tool |
| `mcp_serve_store()` | Serve a Ragnar store over MCP (0.3.0+) |

**Notes:**
- Retrieval functions accept a `filter` argument (added 0.2.0) for metadata filtering.
- `ragnar_register_tool_retrieve()` registers the tool under the name `search_{store@name}` (renamed in 0.3.0 from `rag_retrieve_from_{store@name}`). Override via the `name` argument. It also de-duplicates previously fetched chunks within a session.

**Example:**
```r
# Hybrid retrieval (default)
results <- ragnar_retrieve(store, "query text", top_k = 10)

# BM25 only
results <- ragnar_retrieve_bm25(store, "query text", top_k = 5)

# Vector similarity only
results <- ragnar_retrieve_vss(store, "query text", top_k = 5)

# With metadata filter
results <- ragnar_retrieve(
  store, "query",
  filter = origin %in% c("docs/intro.md", "docs/setup.md")
)

# Merge overlapping chunks for cleaner display
results |> chunks_deoverlap()

# Register with ellmer chat (tool name: search_<store name>)
library(ellmer)
chat <- chat_openai()
ragnar_register_tool_retrieve(chat, store, top_k = 10)

# Custom tool name / title
ragnar_register_tool_retrieve(
  chat, store,
  name = "search_manual",
  title = "User manual search"
)

# Serve the store over MCP for external clients
mcp_serve_store(store)
```

## Complete RAG Workflow

```r
library(ragnar)
library(ellmer)

# 1. Identify documents (returns all links by default in 0.3.0)
paths <- ragnar_find_links("https://quarto.org/", depth = 3)

# 2. Create store
store <- ragnar_store_create("quarto.duckdb", embed = embed_openai())

# 3. Concurrent ingest (or loop with read_as_markdown + markdown_chunk + insert)
ragnar_store_ingest(store, paths = paths)
ragnar_store_build_index(store)

# 4. Connect to chat
chat <- chat_openai()
ragnar_register_tool_retrieve(chat, store, top_k = 10)

# 5. Query with retrieval
chat$chat("Explain Quarto's project structure")
```

## Configuration

```r
# Embedding retry policy (0.3.0+)
options(ragnar.embed.req_retry = list(max_tries = 5, backoff = ~ 2 ^ .x))
```

## Documentation

**Package site:** https://ragnar.tidyverse.org/
**GitHub:** https://github.com/tidyverse/ragnar
**Blog post:** https://tidyverse.org/blog/2025/08/ragnar-0-2/
