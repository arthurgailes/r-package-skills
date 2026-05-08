# ragnar Reference

## Document Processing

```r
# Read documents (returns markdown)
docs <- read_as_markdown("path/to/file.pdf")
docs <- read_as_markdown("path/to/docs/")      # Directory
docs <- read_as_markdown("https://example.com/page.html")

# Supported formats: PDF, markdown, HTML, text, DOCX, PPTX, EPUB, ZIP, YouTube
# (YouTube transcript fetching restored in 0.3.0)

# Extract links from a webpage or sitemap.xml
# Note: in 0.3.0 the default is children_only = FALSE (was TRUE)
links <- ragnar_find_links("https://docs.example.com")
links <- ragnar_find_links(
  "https://docs.example.com",
  children_only = TRUE,
  depth = 2
)
links <- ragnar_find_links(
  "https://docs.example.com/sitemap.xml",
  validate = TRUE
)
```

## Chunking

```r
# Semantic chunking (preserves headings, supports overlap)
chunks <- markdown_chunk(docs)

# Custom chunk size and overlap
chunks <- markdown_chunk(docs, max_size = 1600, overlap = 200)

# Inspect chunks interactively
ragnar_chunks_view(chunks)
```

Chunks include metadata: source file (`origin`), heading hierarchy, position. Overlapping chunks can be merged after retrieval with `chunks_deoverlap()`.

## Embedding Providers

Embedder functions return a callable suitable for `embed = ` in `ragnar_store_create()`. Names do **not** carry a `ragnar_` prefix.

```r
# OpenAI (default; uses OPENAI_API_KEY)
store <- ragnar_store_create("store.duckdb")
store <- ragnar_store_create(
  "store.duckdb",
  embed = embed_openai(model = "text-embedding-3-small")
)

# Ollama (local; default model in 0.3.0 is embeddinggemma)
store <- ragnar_store_create(
  "store.duckdb",
  embed = embed_ollama()
)

# LM Studio (local)
store <- ragnar_store_create(
  "store.duckdb",
  embed = embed_lm_studio(model = "...")
)

# Cloud providers
embed_azure_openai(
  endpoint = "https://your-resource.openai.azure.com",
  deployment_id = "your-embedding-deployment",
  api_version = "2024-02-01"
)
embed_bedrock(model = "...")
embed_databricks(model = "...")
embed_google_gemini(model = "...")
embed_google_vertex(model = "...")
embed_snowflake(model = "...")
```

## Store Operations

```r
# Create a new store
store <- ragnar_store_create("docs.duckdb", embed = embed_openai())

# Open an existing store
store <- ragnar_store_connect("docs.duckdb")

# Insert chunks
ragnar_store_insert(store, chunks)

# Update chunks (idempotent on origin)
ragnar_store_update(store, chunks)

# Concurrent ingestion (0.3.0+; requires mirai >= 2.5.1)
ragnar_store_ingest(store, paths = list.files("docs/", full.names = TRUE))

# Build retrieval index after inserts
ragnar_store_build_index(store)

# Inspect or visualize the store
ragnar_store_inspect(store)
ragnar_store_atlas(store)   # 0.3.0+
```

## Retrieval

```r
# Hybrid retrieval (vector + BM25)
results <- ragnar_retrieve(store, "search query")
results <- ragnar_retrieve(store, "query", top_k = 10)

# Vector-only or BM25-only
results <- ragnar_retrieve_vss(store, "query", top_k = 5)
results <- ragnar_retrieve_bm25(store, "query", top_k = 5)

# Pre-computed query vector (0.3.0+)
qvec <- embed_openai()("query text")
results <- ragnar_retrieve(store, qvec, top_k = 10)

# Metadata filter (0.2.0+)
results <- ragnar_retrieve(
  store, "query",
  filter = origin %in% c("docs/intro.md", "docs/setup.md")
)

# Consolidate overlapping chunks
results <- chunks_deoverlap(results)
```

## Chat Integration

```r
library(ellmer)

# Register retrieval as a chat tool
# 0.3.0: tool is named search_<store@name> by default
chat <- chat_openai()
ragnar_register_tool_retrieve(chat, store, top_k = 10)

# LLM automatically retrieves when needed
chat$chat("What does the documentation say about X?")

# Custom tool name/title
ragnar_register_tool_retrieve(
  chat, store,
  name = "search_manual",
  title = "Search the user manual"
)
```

## MCP Server (0.3.0+)

```r
# Expose a store to MCP-aware clients (Claude Code, etc.)
mcp_serve_store(store)
```

## Multiple Stores

```r
# Different stores for different doc types
api_store    <- ragnar_store_connect("api_docs.duckdb")
manual_store <- ragnar_store_connect("user_manual.duckdb")

ragnar_register_tool_retrieve(chat, api_store,    name = "search_api")
ragnar_register_tool_retrieve(chat, manual_store, name = "search_manual")
```

## Incremental Updates

```r
# Add new documents to an existing store
store <- ragnar_store_connect("docs.duckdb")
new_docs   <- read_as_markdown("new_file.md")
new_chunks <- markdown_chunk(new_docs)
ragnar_store_insert(store, new_chunks)
ragnar_store_build_index(store)
```

## Configuration

```r
# Configurable embedding retry policy (0.3.0+)
options(ragnar.embed.req_retry = list(max_tries = 5, backoff = ~ 2 ^ .x))
```
