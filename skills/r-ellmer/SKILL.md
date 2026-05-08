---
name: r-ellmer
description: Use when code loads or uses ellmer (library(ellmer), chat_openai, chat_anthropic, chat_claude, chat_ollama, chat()), chatting with LLMs from R, building chatbots, or extracting structured data from text with LLMs
---

# ellmer: R Interface to Large Language Models

## Overview

**ellmer is the hub of the R AI stack.** Provides unified interface to 20+ LLM providers (OpenAI, Anthropic, Google, Ollama, AWS Bedrock, Mistral, Groq, LM Studio, etc.) with stateful chat sessions, tool calling, structured data extraction, parallel/batch processing, and token/cost tracking.

**Install:** `install.packages("ellmer")`

**API keys:** Set `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, etc. in `.Renviron`

## References

Read `references/API.md` before writing code.

- `references/API.md` - Complete function reference
- `references/getting-started.md` - Core workflow and chat patterns
- `references/tool-calling.md` - Tool registration and function calling

## When to Use

- Chat with any LLM from R
- Create custom chatbots with domain knowledge
- Extract structured data from unstructured text (`$chat_structured()`)
- Need multi-provider support (switch between OpenAI/Claude/Ollama/etc.)
- Run many prompts concurrently (`parallel_chat_*()`) or async batches (`batch_chat_*()`)
- Build LLM-powered R applications

## When NOT to Use

- Single one-off HTTP request to a non-chat endpoint (use httr2)
- Non-chat ML workflows (use tidymodels)
- Pure embedding/RAG retrieval (use ragnar; ellmer is the chat front-end)

## Quick Reference

```r
library(ellmer)

# Provider-specific constructors (most explicit)
chat <- chat_openai(model = "gpt-4.1")
chat <- chat_anthropic(model = "claude-sonnet-4-5")  # default Sonnet 4.5
chat <- chat_claude()                                # alias of chat_anthropic
chat <- chat_ollama(model = "llama3.2")              # local
chat <- chat_lmstudio()                              # local LM Studio (0.4.1+)

# Unified string interface
chat <- chat("anthropic")
chat <- chat("openai/gpt-4.1-nano")

# Stateful chat (history preserved across turns)
chat$chat("Explain this error")
chat$chat("How do I fix it?")  # remembers context

# System prompt
chat <- chat_openai(system_prompt = "You are a terse R expert.")

# Structured data extraction (replaces removed extract_data())
schema <- type_object(
  name = type_string("Person name"),
  age  = type_integer("Age in years")
)
chat$chat_structured("Maria is 32 years old.", type = schema)

# Tool calling - prefer register_tool() over set_tools()
add <- tool(
  function(x, y) x + y,
  description = "Add two numbers",
  arguments = list(x = type_number(), y = type_number())
)
chat$register_tool(add)
chat$chat("What is 2 + 3?")

# Parallel and batch (no longer experimental as of 0.4.0)
prompts <- c("Explain R", "Explain Python", "Explain Julia")
results <- parallel_chat_text(chat, prompts)            # one row each
tbl <- parallel_chat_structured(chat, prompts, type = schema)  # tibble

# Track tokens and cost
token_usage()              # session-wide
chat$get_tokens()          # data frame with cost per turn
print(chat)                # shows costs

# Record/replay conversations
contents_record(chat, "conv.rds")
chat2 <- chat_openai()
contents_replay(chat2, "conv.rds")
```

## Common Mistakes

| Issue | Solution |
|-------|----------|
| Forgot API key | Set in `.Renviron`, restart R |
| Ollama connection error | Run `ollama serve` first |
| Calling `chat$extract_data()` | Removed in 0.4.0 - use `chat$chat_structured(..., type = ...)` |
| Calling `chat_azure()` / `chat_bedrock()` / `chat_gemini()` / `chat_cortex()` | Removed in 0.4.0 - use `chat_azure_openai()` / `chat_aws_bedrock()` / `chat_google_gemini()` / `chat_snowflake()` |
| Using `chat$set_tools()` for typical flows | Use `chat$register_tool()` (set_tools is for expert use only) |
| Old `tool()` signature with `...` for types | Pass types via `arguments = list(...)`; description is now a named/positional arg |
| Old `type_array("desc", items)` / `type_enum("desc", values)` order | Since 0.4.0 the data argument comes first: `type_array(items, description)`, `type_enum(values, description)` |
| Conversation too long/expensive | Start fresh chat for new topics; check `chat$get_tokens()` |
| Expecting accuracy | LLMs for prototyping, not critical work |

## Use Cases

**Custom chatbots:** Preload with documentation, package info, educational materials

**Structured data extraction:** Sentiment analysis, geocoding, recipe parsing, document indexing via `chat_structured()`

**Programming assistance:** Code modernization, documentation lookup, explanation, security analysis

**Parallel workloads:** Score thousands of texts with `parallel_chat_structured()` (returns tibble)

**Other:** Alt text generation, statistical reasoning, brand style guide enforcement

## Advanced

See `references/` for:
- **API.md**: Complete function reference (all providers, models, methods, content/type classes)
- **getting-started.md**: Core vocabulary, tokens, system prompts
- **tool-calling.md**: Tool/function calling patterns

Notable advanced features:
- `stream_controller()` (0.4.1+) for programmatic cancellation of streaming responses
- `params(reasoning_effort = , reasoning_tokens = )` for reasoning models
- OpenTelemetry instrumentation when the `otel` package is installed
- Built-in provider tools: `claude_tool_web_search()`, `openai_tool_web_search()`, `google_tool_web_search()`, etc.
- Multimodal input: `content_image_file()`, `content_image_plot()`, `content_pdf_file()`

## Integration

**With btw (context tools):** See r-btw skill
**With ragnar (RAG):** See r-ragnar skill
**With vitals (evaluation):** See r-vitals skill
**Cross-package patterns:** See r-ai meta-skill
