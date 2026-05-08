# RAG (Retrieval-Augmented Generation) with ragnar

## Overview

The ragnar package provides tools for building RAG workflows in R, emphasizing transparency and control at each step. It grounds LLM outputs in external, trusted content stored in a local DuckDB-backed vector store.

## The Hallucination Problem

Large language models generate fluent responses without necessarily understanding truth. LLMs operate on text sequences and do not possess a robust concept of facts. RAG addresses this by retrieving relevant excerpts from verified sources and asking the LLM to summarize or answer using only that material.

## Use Case: Documentation Chat

Rather than traditional search requiring precise wording, a RAG-powered chat tool accepts natural language questions, retrieves relevant excerpts, and returns contextualized answers with source links.

## Setting Up RAG

### Creating the Store

```r
store_location <- "quarto.ragnar.duckdb"
store <- ragnar_store_create(
  store_location,
  embed = embed_openai(model = "text-embedding-3-small")
)
```

Embedding providers (no `ragnar_` prefix): `embed_openai()`, `embed_ollama()`, `embed_lm_studio()`, `embed_azure_openai()`, `embed_bedrock()`, `embed_databricks()`, `embed_google_gemini()`, `embed_google_vertex()`, `embed_snowflake()`.

### Document Identification

```r
# Note: in 0.3.0 the default is children_only = FALSE; pass TRUE to restrict
paths <- ragnar_find_links(
  "https://quarto.org/",
  depth = 3,
  children_only = TRUE
)
```

`ragnar_find_links()` also accepts a sitemap.xml URL and supports a `validate` argument for filtering broken links.

### Converting to Markdown

`read_as_markdown()` handles PDFs, DOCX, PPTX, HTML, ZIP, EPUB, and (in 0.3.0) YouTube transcripts. The markdown form keeps token counts low and works well for both humans and LLMs.

### Chunking and Augmentation

```r
chunks <- path |>
  read_as_markdown() |>
  markdown_chunk()
```

`markdown_chunk()` adjusts chunk boundaries to semantic breaks and augments chunks with their heading hierarchy. It emits overlapping chunks that can be merged after retrieval with `chunks_deoverlap()`.

### Ingestion and Indexing

For sequential ingestion:

```r
for (path in paths) {
  chunks <- path |>
    read_as_markdown() |>
    markdown_chunk()
  ragnar_store_insert(store, chunks)
}
ragnar_store_build_index(store)
```

For concurrent ingestion (0.3.0+, mirai-backed):

```r
ragnar_store_ingest(store, paths = paths)
ragnar_store_build_index(store)
```

## Retrieval Methods

`ragnar_retrieve()` performs hybrid retrieval combining:

- **Vector Similarity Search** (`ragnar_retrieve_vss()`): semantically related content
- **BM25** (`ragnar_retrieve_bm25()`): keyword-based matching (results ordered by descending score in 0.3.0)

In 0.3.0 `ragnar_retrieve()` also accepts a pre-computed query vector. All retrieval functions accept a `filter` argument for metadata filtering.

```r
# Hybrid (default)
results <- ragnar_retrieve(store, "what is parameterized reporting?", top_k = 10)

# Filter by source
results <- ragnar_retrieve(
  store, "parameterized reports",
  filter = origin %in% c("docs/computations/parameters.qmd")
)
```

## Registering as an LLM Tool

```r
client <- ellmer::chat_openai()
ragnar_register_tool_retrieve(
  client, store,
  top_k = 10,
  description = "the quarto website"
)
```

In 0.3.0 the registered tool is named `search_{store@name}` (was `rag_retrieve_from_{store@name}`). The tool also de-duplicates previously fetched chunks within a session, so subsequent searches return new excerpts. Override naming via `name` and `title`.

## Custom Retrieval Implementation

For specialized tasks, define custom tools with a focused system prompt:

```r
client$set_system_prompt(glue::trim(
  "You are an expert in Quarto documentation. Always perform a
  search for each user request. If initial results lack relevance,
  perform up to three additional searches returning unique excerpts."
))
```

Stateful retrieval that excludes previously returned chunks (still useful when not relying on built-in de-duplication):

```r
rag_retrieve_quarto_excerpts <- local({
  retrieved_chunk_ids <- integer()
  function(text) {
    chunks <- ragnar::ragnar_retrieve(
      store, text,
      top_k = 10,
      filter = !.data$chunk_id %in% retrieved_chunk_ids
    )
    retrieved_chunk_ids <<- unique(c(retrieved_chunk_ids, chunks$chunk_id))
    chunks
  }
})
```

## Serving Stores Over MCP

In 0.3.0 a store can be exposed directly to MCP-aware clients (e.g., Claude Code) without going through ellmer:

```r
mcp_serve_store(store)
```

## Troubleshooting and Iteration

Development requires iterative refinement: source selection, markdown conversion, chunking strategy, chunk augmentation, metadata filtering, embedding model choice, LLM selection, system prompts, and tool definitions.

- `ragnar_store_inspect(store)`: launches a Shiny inspector for browsing chunks and testing queries.
- `ragnar_store_atlas(store)` (0.3.0+): visualizes the embedding space using Embedding Atlas.
- `ragnar_chunks_view(chunks)`: previews chunks before insertion.
- `options(ragnar.embed.req_retry = ...)`: configurable retry policy for embedding requests (0.3.0+).
