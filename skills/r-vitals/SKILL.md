---
name: r-vitals
description: Use when code loads or uses vitals (library(vitals), vitals::), evaluating LLM output quality, scoring AI responses, testing RAG retrieval accuracy, or benchmarking prompt changes in R
---

# vitals: LLM Evaluation and Testing

## Overview

**vitals tests LLM output quality.** Build evaluation tasks from a dataset
(`input`/`target`), a solver (your LLM pipeline), and a scorer (judges output).
Run with `task$eval()` to solve, score, log, and view in one call. Inspired by
Inspect AI; logs render in the Inspect Log Viewer for side-by-side comparison
across models and prompt revisions.

**Install:** `install.packages("vitals")` (CRAN 0.2.0)

## References

Read `references/API.md` before writing code.

- `references/API.md` - Complete function reference with current signatures
- `references/package-docs.md` - Workflow patterns (custom solvers, tool calling, analysis)

## When to Use

- Test LLM output quality across prompts or models
- Evaluate RAG retrieval accuracy
- Benchmark different prompts/models with statistical comparison
- Score AI-generated responses with model-graded or rule-based scorers
- Build LLM regression test suites for production agents

## When NOT to Use

- One-off manual prompt testing (just call `chat$chat()` directly)
- Traditional unit testing (use testthat)
- Non-LLM code testing
- Streaming chat UIs (vitals is offline batch evaluation)

## Quick Reference

```r
library(vitals)
library(ellmer)

# 1. Dataset: tibble with input + target columns
test_cases <- tibble::tibble(
  input  = c("What is 2+2?", "Capital of France?"),
  target = c("4", "Paris")
)

# 2. Solver: wrap an ellmer chat with generate()
#    generate() accepts a chat object OR a zero-argument factory.
solver <- generate(chat_anthropic(model = "claude-sonnet-4-5"))

# 3. Task: dataset + solver + scorer
task <- Task$new(
  dataset = test_cases,
  solver  = solver,
  scorer  = model_graded_qa()
)

# 4. Evaluate (solve + score + measure + log + view in one call)
task$eval()

# 5. Inspect samples and cost programmatically
samples <- task$get_samples()
costs   <- task$get_cost()

# Compare another model: build a second task and eval it
task2 <- Task$new(
  dataset = test_cases,
  solver  = generate(chat_openai(model = "gpt-4o")),
  scorer  = model_graded_qa()
)
task2$eval()

# Combine multiple task results for analysis
all <- vitals_bind(claude = task, gpt = task2)
```

**Built-in scorers (return ordered factor `I < P < C`):**

| Scorer | Use for |
|--------|---------|
| `model_graded_qa()` | LLM-as-judge for open-ended Q&A |
| `model_graded_fact()` | LLM-as-judge for factual inclusion |
| `detect_includes(case_sensitive = FALSE)` | Substring match |
| `detect_match(location, case_sensitive)` | Match at begin/end/any/exact |
| `detect_pattern(pattern, ...)` | Regex match |
| `detect_exact(case_sensitive)` | Exact equality |
| `detect_answer(format)` | Extract after `ANSWER:` marker |

**Solvers:**

| Solver | Use for |
|--------|---------|
| `generate(solver_chat)` | Standard chat-in/text-out |
| `generate_structured(solver_chat, type)` | Structured extraction via `parallel_chat_structured()` (dev version) |
| custom function | Tool calling, RAG, multi-step pipelines |

## Common Mistakes

| Issue | Solution |
|-------|----------|
| Calling `task$run()` | Method does not exist. Use `task$eval()` (or `$solve()` + `$score()` + `$log()`) |
| Solver = bare chat object | Wrap with `generate(chat)`, or write a custom function with signature `function(inputs, ..., solver_chat)` |
| Custom solver with closure over `chat` | Vignette pattern: take `solver_chat` as named arg, `$clone()` it, call `ellmer::parallel_chat()`, return `list(result, solver_chat)` |
| Passing unnamed args to `$eval()` | `$eval()` errors on unnamed args; it routes named args to solver/scorer by signature |
| Using `model_graded_qa()` without API key for grader | Set `scorer_chat = chat_anthropic(...)` if grading model differs from solver |
| Treating `score` as numeric | Built-in scorers return ordered factor `I < P < C`; convert with `as.numeric()` only after understanding levels |
| Looking for `task$results()` / `vitals_logs()` | These do not exist. Use `task$get_samples()`; logs live in `vitals_log_dir()` and open via `vitals_view()` |
| Forgetting log directory | Set once with `vitals_log_dir_set("path")`, or pass `dir = ` to `Task$new()` |

## Custom Solvers (Tool Use, RAG)

Custom solvers take inputs and a `solver_chat` and return a list:

```r
btw_solver <- function(inputs, ..., solver_chat) {
  ch <- solver_chat$clone()
  ch$set_tools(btw::btw_tools(tools = "docs"))
  res <- ellmer::parallel_chat(ch, as.list(inputs))
  list(
    result      = purrr::map_chr(res, \(c) c$last_turn()@text),
    solver_chat = res
  )
}

task <- Task$new(dataset = are(), solver = btw_solver, scorer = model_graded_qa())
task$eval(solver_chat = chat_anthropic(model = "claude-sonnet-4-5"))
```

For RAG, register a retrieval tool on the cloned chat (e.g. ragnar's
`ragnar_register_tool_retrieve(ch, store)`) before calling `parallel_chat()`.

## Logs and Viewer

```r
vitals_log_dir_set("~/eval-logs")  # set storage location
vitals_view()                      # open Inspect Log Viewer (random port)
vitals_bundle("eval_results.zip")  # bundle for sharing / Posit Connect
vitals_bind(model_a = task1, model_b = task2)  # join for analysis
```

`task$eval()` and `task$log()` write to the Task's `dir` (set at `$new()`).
Tasks within the same `dir` are grouped by name; different model/args produce
distinct entries via Inspect's `task_identifier` (`task_name/model/hash`).

## Built-in Eval

```r
are()  # An R Eval: 29 challenging R coding problems with reference solutions
```

Use `are()` as a sanity-check dataset before writing your own.

## Integration

- **ellmer**: solver chats and scorer chats are ellmer chat objects
- **ragnar**: register retrieval tools on the cloned chat inside a custom solver
- **btw**: register doc-lookup tools the same way
- **mcptools**: register MCP tools via ellmer before evaluation
- **Cross-package patterns:** see `r-ai` meta-skill
