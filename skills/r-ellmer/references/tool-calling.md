# Tool/Function Calling in ellmer

## Overview

Chat models can request external tool execution. **The chat model does not directly execute any external tools!** Instead, models request that callers execute functions, then return results for the model to incorporate into responses.

## Key Conceptual Distinction

**Incorrect**: User -> Assistant executes code -> Result back to user
**Correct**: User -> Assistant requests tool call -> ellmer executes the R function -> Assistant uses results

The model's value lies in deciding when tools are useful, selecting appropriate arguments, and integrating tool results into a coherent answer.

## Practical Implementation

### Defining Tools

Wrap an R function with `tool()`:

```r
tool(
  fun,
  description,
  ...,
  arguments  = list(),
  name       = NULL,
  convert    = TRUE,
  annotations = list()
)
```

Required components:

- A function reference (`fun`)
- A `description` string (used by the model to decide when to call)
- Argument specs in `arguments`, built with `type_*()` helpers

`create_tool_def()` can auto-generate a tool spec from an existing function using an LLM. Always review the result.

### Registration and Usage

Use `chat$register_tool()` for a single tool, or `chat$register_tools()` for a list. (`chat$set_tools()` also exists but is documented as expert-use only - it replaces the entire tool set.) Once registered, the model autonomously decides when invocation is appropriate.

```r
get_time <- tool(
  function() format(Sys.time(), "%Y-%m-%d %H:%M:%S"),
  description = "Get the current local date and time"
)

chat <- chat_anthropic()
chat$register_tool(get_time)
chat$chat("What time is it right now?")
```

### Input/Output Specifications

**Inputs** are described with `type_boolean()`, `type_integer()`, `type_number()`, `type_string()`, `type_enum()`, `type_array()`, or `type_object()`.

Note (0.4.0+): `type_array()` and `type_enum()` take items/values as the first argument and `description` as the second.

**Outputs** should remain simple - text or atomic vectors. Complex data automatically serializes to JSON. For direct JSON control, wrap results in `I()`. Use `tool_reject()` from inside a tool function to refuse a call (e.g., when arguments fail validation).

## Advanced Capabilities

### Data Structures

Tools can return data frames; ellmer converts them to a JSON row-major form that LLMs read well.

### Multimodal Output

Tools may return images or PDFs via `content_image_file()` (and `content_image_pdf()` for PDFs), enabling vision when the underlying API supports it.

### Tool Annotations

`tool_annotations()` attaches metadata (e.g., destructive, idempotent, read-only) consumed by some clients to drive UI confirmations.

### Built-in Provider Tools

Several providers expose first-party tools that you can register directly:

- `claude_tool_web_search()`, `claude_tool_web_fetch()`
- `google_tool_web_search()`, `google_tool_web_fetch()`
- `openai_tool_web_search()`

Example:

```r
chat <- chat_anthropic()
chat$register_tool(claude_tool_web_search())
chat$chat("Summarise the latest tidyverse blog posts.")
```

### Streaming and Cancellation

When streaming, you can cancel a long-running tool-using turn programmatically with `stream_controller()` (added in 0.4.1).

## Example Applications

- Current time retrieval for temporal reasoning
- Weather API simulation with batch city queries
- Website screenshots for visual analysis
- Database lookups returning data frames

Each example shows autonomous tool invocation: the model decides when a tool solves the problem.
