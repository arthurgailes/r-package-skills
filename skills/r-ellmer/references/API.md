# ellmer API Reference

Complete function reference for the ellmer package (current upstream: 0.4.1).

## Chatbots

| Function | Purpose |
|----------|---------|
| `chat()` | Unified provider entry point - accepts strings like `"anthropic"` or `"openai/gpt-4.1-nano"` |
| `chat_anthropic()` / `chat_claude()` | Connect to Anthropic Claude models (default: claude-sonnet-4-5); `chat_claude()` is an alias |
| `models_claude()` / `models_anthropic()` | List available Claude models |
| `chat_aws_bedrock()` | Access models through AWS Bedrock (gains `cache` argument in 0.4.1) |
| `models_aws_bedrock()` | Enumerate AWS Bedrock available models |
| `chat_azure_openai()` | Connect to Azure-hosted OpenAI models |
| `chat_cloudflare()` | Access CloudFlare-hosted models |
| `chat_databricks()` | Connect to Databricks-hosted models (OpenAI-compatible) |
| `chat_deepseek()` | Access DeepSeek-hosted models |
| `chat_github()` | Use GitHub model marketplace (uses `chat_openai_compatible()` under the hood as of 0.4.1) |
| `models_github()` | List GitHub marketplace models |
| `chat_google_gemini()` / `chat_google_vertex()` | Connect to Google Gemini or Vertex AI (default: Gemini 2.5 flash) |
| `models_google_gemini()` / `models_google_vertex()` | Enumerate Google model options |
| `chat_groq()` | Access Groq-hosted models (supports structured output as of 0.4.1) |
| `chat_huggingface()` | Connect to Hugging Face Serverless Inference API |
| `chat_lmstudio()` | Run local LM Studio models (added in 0.4.1) |
| `models_lmstudio()` | List local LM Studio models (added in 0.4.1) |
| `chat_mistral()` | Use Mistral's La Platforme |
| `models_mistral()` | List Mistral models |
| `chat_ollama()` | Run local Ollama models (supports `params(top_k = )` as of 0.4.1) |
| `models_ollama()` | List local Ollama models |
| `chat_openai()` | Connect to OpenAI models (default: GPT-4.1; uses Responses endpoint as of 0.4.0) |
| `models_openai()` | Enumerate OpenAI models |
| `chat_openai_compatible()` | Access non-official OpenAI-compatible endpoints; new `preserve_thinking` arg (0.4.1) |
| `chat_openrouter()` | Use OpenRouter model marketplace |
| `chat_perplexity()` | Connect to perplexity.ai models |
| `chat_portkey()` | Access PortkeyAI-hosted models |
| `models_portkey()` | List PortkeyAI models |
| `chat_snowflake()` | Connect to Snowflake-hosted models (supports tool calling) |
| `chat_vllm()` | Use vLLM-hosted models |
| `models_vllm()` | List vLLM available models |
| `Chat` | Core R6-style chat object (see Chat methods below) |
| `token_usage()` | Report total token consumption across the current R session |

**Common arguments (most `chat_*()` constructors):** `system_prompt`, `params`, `model`, `api_args`, `base_url`, `api_key`, `credentials`, `api_headers`, `echo`. Use `credentials` (a function returning a token) instead of `api_key` for dynamic auth (introduced in 0.4.0).

**Removed in 0.4.0:** `chat_azure()` (use `chat_azure_openai()`), `chat_bedrock()` (use `chat_aws_bedrock()`), `chat_cortex()` (use `chat_snowflake()`), `chat_gemini()` (use `chat_google_gemini()`), `chat_openai(seed = )`, `chat_anthropic(max_tokens = )`.

**Example:**
```r
library(ellmer)

# Explicit constructor
chat <- chat_anthropic(model = "claude-sonnet-4-5")

# String interface (0.3.0+)
chat <- chat("openai/gpt-4.1-nano")

chat$chat("Explain R's pipe operator")
token_usage()
```

## Chat Object Methods

The `Chat` object exposes ~25 methods. Most-used:

| Method | Purpose |
|--------|---------|
| `$chat(...)` | Submit input, return string response (Markdown) |
| `$chat_async(...)` | Async variant |
| `$chat_structured(..., type)` | Extract structured data conforming to a `type_*()` schema (replaces removed `extract_data()`) |
| `$chat_structured_async(..., type)` | Async structured extraction |
| `$stream(...)` | Streaming response |
| `$register_tool(tool)` | Register a single `tool()` (preferred) |
| `$register_tools(list_of_tools)` | Register multiple tools |
| `$set_tools(tools)` | Replace the full tool set (expert use only) |
| `$get_tools()` | List registered tools |
| `$on_tool_request(callback)` / `$on_tool_result(callback)` | Hook into tool lifecycle |
| `$set_system_prompt(text)` / `$get_system_prompt()` | Manage system prompt |
| `$get_turns()` / `$set_turns(turns)` | Inspect / mutate conversation history |
| `$last_turn()` | Most recent assistant turn |
| `$get_tokens()` | Data frame of tokens + cost per turn (0.4.0+) |
| `$get_cost()` | Cumulative cost estimate |
| `$get_model()` | Model id |
| `$clone()` | Copy the chat object |

**Removed in 0.4.0:** `$extract_data()`, `$extract_data_async()` - use `$chat_structured()` / `$chat_structured_async()`.

## Provider-Specific Helpers

| Function | Purpose |
|----------|---------|
| `google_upload()` | Upload files to Gemini |
| `claude_file_upload()` | Upload files for Claude use |
| `claude_file_list()` | View Claude-uploaded files |
| `claude_file_get()` | Retrieve Claude file metadata |
| `claude_file_download()` | Download Claude files |
| `claude_file_delete()` | Remove Claude files |
| `claude_tool_web_search()` | Built-in Claude web search tool (now exposes description + annotations in 0.4.1) |
| `claude_tool_web_fetch()` | Built-in Claude URL fetching tool |
| `google_tool_web_search()` | Built-in Google web search tool |
| `google_tool_web_fetch()` | Built-in Google URL fetching tool |
| `openai_tool_web_search()` | Built-in OpenAI web search tool |

**Example:**
```r
# Upload file to Claude
claude_file_upload("data.csv")

# Register a built-in provider tool
chat$register_tool(claude_tool_web_search())
```

## Chat Helpers

| Function | Purpose |
|----------|---------|
| `create_tool_def()` | LLM-assisted generation of tool metadata (the `model` argument was removed in 0.4.0) |
| `content_image_url()` / `content_image_file()` / `content_image_plot()` | Prepare images for chat input |
| `content_pdf_file()` / `content_pdf_url()` | Prepare PDF content for chat input |
| `live_console()` / `live_browser()` | Open interactive chat interfaces |
| `interpolate()` / `interpolate_file()` / `interpolate_package()` | Insert data into prompts (works with devtools-loaded packages) |
| `stream_controller()` | Create a stream controller for programmatic cancellation of streaming responses (added in 0.4.1) |

**Example:**
```r
# Multimodal input
chat$chat(
  "What's in this image?",
  content_image_file("plot.png")
)

# Interactive chat
live_console(chat)

# Interpolate data into a prompt
interpolate("Summarize: {{data}}", data = df)
```

## Parallel and Batch Chat

Stable since 0.4.0 (no longer experimental).

| Function | Purpose |
|----------|---------|
| `batch_chat()` | Submit many prompts via the provider's batch API; logs token usage on retrieval |
| `batch_chat_text()` | Batch submit, return text responses |
| `batch_chat_structured()` | Batch submit, return structured outputs |
| `batch_chat_completed()` | Check batch completion status |
| `parallel_chat()` | Concurrent (non-batch) chat requests |
| `parallel_chat_text()` | Parallel text responses |
| `parallel_chat_structured()` | Parallel structured outputs (returns a tibble; warns instead of erroring on parse failure) |

Both families accept `on_error` to control failure handling.

**Example:**
```r
# Process multiple prompts in parallel
results <- parallel_chat_text(
  chat,
  prompts = c("Explain R", "Explain Python", "Explain Julia")
)

# Structured extraction over many inputs
schema <- type_object(sentiment = type_enum(c("pos","neg","neu"), "Sentiment"))
parallel_chat_structured(chat, prompts = reviews, type = schema)
```

## Tools and Structured Data

| Function | Purpose |
|----------|---------|
| `tool()` | Define a callable tool. Signature: `tool(fun, description, ..., arguments = list(), name = NULL, convert = TRUE, annotations = list())` |
| `tool_annotations()` | Add metadata (e.g., destructive/idempotent flags) to tool definitions |
| `tool_reject()` | Decline a tool invocation from inside a tool function |
| `type_boolean()` / `type_integer()` / `type_number()` / `type_string()` | Scalar parameter types |
| `type_enum(values, description)` | Enumerated string parameter (description is the SECOND argument since 0.4.0) |
| `type_array(items, description)` | Array parameter (description is the SECOND argument since 0.4.0) |
| `type_object(...)` | Object/struct parameter built from named type fields |
| `type_from_schema()` | Generate a type from a JSON schema |
| `type_ignore()` | Mark tool arguments not provided by the LLM (pulled from R-side defaults) |

**Example:**
```r
# Define a tool (modern signature)
calc_tool <- tool(
  function(a, b, op) switch(op, "+" = a + b, "-" = a - b, "*" = a * b, "/" = a / b),
  description = "Arithmetic on two numbers",
  arguments = list(
    a  = type_number("First number"),
    b  = type_number("Second number"),
    op = type_enum(c("+", "-", "*", "/"), "Operation")
  )
)

chat$register_tool(calc_tool)

# Structured extraction
schema <- type_object(
  city  = type_string("City name"),
  pop   = type_integer("Population")
)
chat$chat_structured("Tokyo has about 14 million people.", type = schema)
```

## Objects

| Class | Purpose |
|-------|---------|
| `Provider` | Abstract provider implementation |
| `Turn` / `UserTurn` / `SystemTurn` / `AssistantTurn` | Message turn representations (`AssistantTurn` has `@duration` slot since 0.4.0) |
| `AssistantPartialTurn` | Incomplete streaming turn (added in 0.4.1) |
| `Content` / `ContentText` / `ContentImage` / `ContentImageRemote` / `ContentImageInline` / `ContentPDF` / `ContentToolRequest` / `ContentToolResult` / `ContentThinking` | Content payload classes |
| `TypeBasic` / `TypeEnum` / `TypeArray` / `TypeObject` / `TypeJsonSchema` / `TypeIgnore` | Schema classes for tool/structured-output parameters |

## Utilities

| Function | Purpose |
|----------|---------|
| `contents_text()` / `contents_html()` / `contents_markdown()` | Format outputs as text, HTML, or Markdown |
| `contents_record()` / `contents_replay()` | Persist a conversation and restore it into a new chat (handles custom Turn/Content classes since 0.3.0) |
| `df_schema()` | Generate data-frame schema descriptions for an LLM |
| `params()` | Standard model parameters (now includes `reasoning_effort` and `reasoning_tokens` since 0.4.0; `top_k` for Ollama since 0.4.1) |

**Example:**
```r
# Record conversation
contents_record(chat, "conversation.rds")

# Replay later
chat_new <- chat_openai()
contents_replay(chat_new, "conversation.rds")

# Describe a data frame for the LLM
df_schema(mtcars)

# Reasoning model parameters
chat <- chat_openai(
  model = "o3-mini",
  params = params(reasoning_effort = "high")
)
```

## Options and Observability

- `options(ellmer_echo = ...)` - default `echo` behavior
- `options(ellmer_timeout_s = ...)` - request + connection timeout (combined since 0.4.0)
- Default retries: 3 attempts (since 0.4.0)
- OpenTelemetry instrumentation activates automatically when the `otel` package is installed (0.4.1)

## Documentation

**Package site:** https://ellmer.tidyverse.org/
**GitHub:** https://github.com/tidyverse/ellmer
**CRAN:** https://cran.r-project.org/package=ellmer
