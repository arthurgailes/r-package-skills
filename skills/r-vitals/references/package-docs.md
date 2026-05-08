# vitals Reference: Workflow Patterns

Patterns drawn from the vitals vignettes (Getting started, Custom solvers,
Writing evals, Analyzing results).

## Task Structure

```r
library(vitals)

task <- Task$new(
  dataset = my_dataset,    # tibble with input/target columns
  solver  = my_solver,     # generate(chat) or custom function
  scorer  = my_scorer,     # built-in or custom
  metrics = NULL,          # custom metrics list (NULL uses default accuracy)
  epochs  = NULL,          # repeat each input N times for variance
  name    = "my_task",
  dir     = vitals_log_dir()
)

task$eval()                # solve + score + measure + log + view
```

`$eval()` is the canonical entry point. Use `$solve()`, `$score()`, `$log()`
separately only when you need fine-grained control.

## Datasets

Required columns: `input`, `target`. Extra columns flow through to solver and
scorer if their signatures accept them.

```r
ds <- tibble::tibble(
  input  = c("Question 1", "Question 2"),
  target = c("Expected answer 1", "Expected answer 2")
)

# Optional context column
ds <- tibble::tibble(
  input   = c("What is X?", "Explain Y"),
  target  = c("X is ...",   "Y means ..."),
  context = c("Background 1", "Background 2")
)
```

Per the "Writing evals for your LLM product" vignette: prefer real production
inputs over synthetic ones; do not sanitize user typos; structure inputs
around features, scenarios, and personas.

## Built-in Solvers

```r
library(ellmer)

# Direct chat
solver <- generate(chat_anthropic(model = "claude-sonnet-4-5"))

# System prompt: configure on the chat object
solver <- generate(
  chat_anthropic(system_prompt = "You are a terse R expert.")
)

# Zero-arg factory: fresh chat per call
solver <- generate(\() chat_anthropic())

# Defer chat selection until $eval()
task <- Task$new(ds, solver = generate(), scorer = model_graded_qa())
task$eval(solver_chat = chat_anthropic())
```

## Custom Solvers (Tool Calling, RAG)

Custom solvers must take `inputs` (vector of input strings) plus a named
`solver_chat` arg, and return a list with `result` (character vector of final
text) and `solver_chat` (chat objects for logging). Always `$clone()` the
chat so the original is not mutated.

```r
# Tool-calling solver (e.g. with btw)
btw_solver <- function(inputs, ..., solver_chat) {
  ch <- solver_chat$clone()
  ch$set_tools(btw::btw_tools(tools = "docs"))
  res <- ellmer::parallel_chat(ch, as.list(inputs))
  list(
    result      = purrr::map_chr(res, \(c) c$last_turn()@text),
    solver_chat = res
  )
}

# RAG solver (with ragnar)
rag_solver <- function(inputs, ..., solver_chat) {
  ch <- solver_chat$clone()
  ragnar::ragnar_register_tool_retrieve(ch, store = my_store)
  res <- ellmer::parallel_chat(ch, as.list(inputs))
  list(
    result      = purrr::map_chr(res, \(c) c$last_turn()@text),
    solver_chat = res
  )
}

# Multi-step solver
two_pass_solver <- function(inputs, ..., solver_chat) {
  ch  <- solver_chat$clone()
  step1 <- ellmer::parallel_chat(ch, as.list(paste("Analyze:", inputs)))
  txt   <- purrr::map_chr(step1, \(c) c$last_turn()@text)
  step2 <- ellmer::parallel_chat(ch$clone(), as.list(paste("Summarize:", txt)))
  list(
    result      = purrr::map_chr(step2, \(c) c$last_turn()@text),
    solver_chat = step2
  )
}

task <- Task$new(are(), solver = btw_solver, scorer = model_graded_qa())
task$eval(solver_chat = chat_anthropic(model = "claude-sonnet-4-5"))
```

## Built-in Scorers

```r
# LLM-as-judge for open-ended answers
scorer <- model_graded_qa()

# LLM-as-judge for factual inclusion
scorer <- model_graded_fact()

# Substring match (returns explanation slot)
scorer <- detect_includes(case_sensitive = FALSE)

# Where to look for the target string
scorer <- detect_match(location = "end", case_sensitive = FALSE)

# Regex match
scorer <- detect_pattern("\\d+", case_sensitive = FALSE, all = FALSE)

# Exact equality
scorer <- detect_exact(case_sensitive = FALSE)

# Extract whatever follows "ANSWER:" in the model output
scorer <- detect_answer(format = "letter")
```

Built-in scorers return an ordered factor `I < P < C` (Incorrect, Partially
Correct, Correct).

## Custom Scorers

A scorer is a function that takes solver outputs and dataset columns and
returns either a vector of scores or a list with `value`, `answer`, and
optional `explanation` slots.

```r
# Simple character return
my_scorer <- function(outputs, target, ...) {
  ifelse(grepl(target, outputs, ignore.case = TRUE), "C", "I")
}

# Numeric return with explanation
keyword_scorer <- function(outputs, target, ...) {
  scores <- vapply(seq_along(outputs), function(i) {
    kws  <- strsplit(target[i], ",")[[1]] |> trimws()
    hits <- sum(vapply(kws, grepl, logical(1), x = outputs[i],
                       ignore.case = TRUE))
    hits / length(kws)
  }, numeric(1))
  list(
    value       = scores,
    explanation = sprintf("Matched %.0f%% of keywords", 100 * scores)
  )
}
```

## Running Evaluations

```r
# Standard run
task$eval()

# Multiple epochs for variance
task$eval(epochs = 3)

# Pass extra args; vitals routes by signature
task$eval(
  solver_chat = chat_anthropic(),  # routed to generate()
  scorer_chat = chat_openai()      # routed to model_graded_qa()
)
```

`$eval()` errors on unnamed `...` arguments.

## Inspecting Results

```r
# Sample-level data: input, target, result, score, metadata
samples <- task$get_samples()

# Token usage and cost by model and role
costs <- task$get_cost()

# Re-open the Inspect Log Viewer
task$view()

# Or open all logs in vitals_log_dir()
vitals_view()
```

There is no `task$results()` or `task$summary()` method; use `$get_samples()`
and aggregate yourself, or rely on the Inspect Log Viewer.

## Multi-Task Comparison

```r
task_claude <- Task$new(ds, generate(chat_anthropic()), model_graded_qa())
task_gpt    <- Task$new(ds, generate(chat_openai()),    model_graded_qa())

task_claude$eval()
task_gpt$eval()

# vitals_bind() concatenates samples with a labeling column
combined <- vitals_bind(claude = task_claude, gpt = task_gpt)

# Open viewer to compare side by side
vitals_view()
```

## Logging and Deployment

```r
vitals_log_dir_set("~/eval-logs")

# Bundle logs for static hosting (writes listing.json manifest)
vitals_bundle("eval_results.zip")
```

Logs are written to the Task's `dir` argument (set at `$new()`); both
`$eval()` and `$log()` write to the same directory. Tasks within a directory
are grouped by name and identified by `task_name/model/hash` so different
models or args appear as separate entries rather than collapsed retries.

## Analyzing Results Statistically

The "Analyzing evaluation results" vignette recommends ordinal regression
because scores are ordered factors `I < P < C`:

```r
library(ordinal)

results <- vitals_bind(claude = task_claude, gpt = task_gpt)

# Single epoch: cumulative link model
fit <- clm(score ~ source, data = results)

# Multi-epoch: random effect for sample id
fit_mm <- clmm(score ~ source + (1 | id), data = results)
```

For Bayesian inference use `brms::brm()` with `family = cumulative()`.
