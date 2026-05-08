# vitals API Reference

Complete function reference for the vitals package (CRAN 0.2.0). LLM evaluation
toolkit for R, modeled on Inspect AI; logs render in the Inspect Log Viewer.

## Task (R6 class)

```r
Task$new(
  dataset,
  solver,
  scorer,
  metrics = NULL,
  epochs  = NULL,
  name    = deparse(substitute(dataset)),
  dir     = vitals_log_dir()
)
```

`dataset` must contain `input` and `target` columns. Additional columns are
passed as named args to the solver/scorer if their signatures accept them.

### Public methods

| Method | Signature | Purpose |
|--------|-----------|---------|
| `$eval()` | `(..., epochs = NULL, view = interactive())` | Solve + score + measure + log + view in one call. All `...` args must be named; routed to solver and scorer by function signature. |
| `$solve()` | `(..., epochs = NULL)` | Run solver only |
| `$score()` | `(...)` | Run scorer on solved samples |
| `$measure()` | `()` | Apply metrics to scored samples |
| `$log()` | `(dir = self$dir)` | Persist samples to disk in Inspect log format |
| `$view()` | `()` | Open the Inspect Log Viewer |
| `$get_samples()` | `()` | Tibble of input, target, result, score, metadata |
| `$get_cost()` | `()` | Tibble of token usage and cost by solver/scorer and model |
| `$set_solver(solver)` | | Replace solver, reset downstream results |
| `$set_scorer(scorer)` | | Replace scorer, reset score/metrics |
| `$set_metrics(metrics)` | | Replace metrics list |
| `$clone(deep = FALSE)` | | R6 clone |

**Score levels:** built-in scorers return ordered factor `I < P < C`
(Incorrect, Partially Correct, Correct). The default `accuracy()` metric
normalizes by the number of factor levels, not the observed max.

**Example:**
```r
task <- Task$new(
  dataset = test_cases,
  solver  = generate(chat_anthropic(model = "claude-sonnet-4-5")),
  scorer  = model_graded_qa()
)
task$eval()
samples <- task$get_samples()
```

## Solvers

### `generate()`

```r
generate(solver_chat = NULL)
```

Wraps an ellmer chat into a vitals solver. Calls `ellmer::parallel_chat()`
internally.

- `solver_chat`: an ellmer chat object (e.g. `ellmer::chat_anthropic()`) **or**
  a zero-argument function returning one. A factory yields a fresh chat per
  invocation rather than cloning.

```r
# Direct chat
solver <- generate(chat_anthropic(model = "claude-sonnet-4-5"))

# Factory: fresh chat per call
solver <- generate(function() chat_openai(system_prompt = "You are concise."))

# Defer until $eval()
task <- Task$new(dataset, solver = generate(), scorer = model_graded_qa())
task$eval(solver_chat = chat_anthropic())
```

### `generate_structured()` (development version)

```r
generate_structured(solver_chat = NULL, type = NULL)
```

Like `generate()` but uses `ellmer::parallel_chat_structured()` to extract
typed data. `type` is built with ellmer's `type_*()` helpers. Raw structured
output is returned in `solver_metadata`.

```r
solver <- generate_structured(
  chat_anthropic(),
  type = ellmer::type_object(
    sentiment = ellmer::type_string(),
    score     = ellmer::type_number()
  )
)
```

### Custom solvers

Solver signature: `function(inputs, ..., solver_chat)`. Return
`list(result, solver_chat)` so chat objects are logged.

```r
custom_solver <- function(inputs, ..., solver_chat) {
  ch  <- solver_chat$clone()
  ch$set_tools(my_tools)
  res <- ellmer::parallel_chat(ch, as.list(inputs))
  list(
    result      = purrr::map_chr(res, \(c) c$last_turn()@text),
    solver_chat = res
  )
}
```

## Scorers

### Model-graded scorers

```r
model_graded_qa(
  template       = NULL,
  instructions   = NULL,
  grade_pattern  = "(?i)GRADE\\s*:\\s*([CPI])(.*)$",
  partial_credit = FALSE,
  scorer_chat    = NULL
)

model_graded_fact(
  template       = NULL,
  instructions   = NULL,
  grade_pattern  = "(?i)GRADE\\s*:\\s*([CPI])(.*)$",
  partial_credit = FALSE,
  scorer_chat    = NULL
)
```

- `template`: glue string with `{input}`, `{answer}`, `{criterion}`,
  `{instructions}` substitutions
- `instructions`: replaces default grading instructions (must keep the
  `GRADE: C/P/I` output contract or update `grade_pattern`)
- `grade_pattern`: regex extracting `[CPI]` from grader output
- `partial_credit`: allow `P` grade (otherwise only `C`/`I`)
- `scorer_chat`: ellmer chat used as judge; defaults to a sensible flagship

`model_graded_qa()` judges open-ended answers; `model_graded_fact()` judges
whether the response includes a target fact.

```r
scorer <- model_graded_qa(
  scorer_chat    = chat_anthropic(model = "claude-sonnet-4-5"),
  partial_credit = TRUE
)
```

### Detect-based scorers

| Function | Signature |
|----------|-----------|
| `detect_includes()` | `(case_sensitive = FALSE)` |
| `detect_match()` | `(location = c("end", "begin", "any", "exact"), case_sensitive = FALSE)` |
| `detect_pattern()` | `(pattern, case_sensitive = FALSE, all = FALSE)` |
| `detect_exact()` | `(case_sensitive = FALSE)` |
| `detect_answer()` | `(format = c("line", "word", "letter"))` |

All detect scorers return an `explanation` slot describing why the score was
assigned. `detect_answer()` extracts whatever follows `ANSWER:` in the model's
response (handy for chain-of-thought prompts).

```r
# substring
scorer <- detect_includes()

# regex with multi-capture
scorer <- detect_pattern("\\b[A-Z]{3}\\b", all = TRUE)

# extract after ANSWER: marker
scorer <- detect_answer(format = "letter")

# multiple scorers
task <- Task$new(
  dataset = data,
  solver  = generate(chat_anthropic()),
  scorer  = list(
    detect_includes(),
    model_graded_qa()
  )
)
```

## Log Management and Deployment

| Function | Purpose |
|----------|---------|
| `vitals_log_dir()` | Get current log directory |
| `vitals_log_dir_set(path)` | Set log directory (session-wide default) |
| `vitals_view()` | Open Inspect Log Viewer on a random available port |
| `vitals_bundle(out, dir = vitals_log_dir())` | Bundle logs (writes `listing.json` manifest) for static hosting / Posit Connect |
| `vitals_bind(...)` | Concatenate samples from multiple Tasks into one tibble for analysis |

```r
vitals_log_dir_set("~/eval-logs")
vitals_view()
vitals_bundle("eval_results.zip")

combined <- vitals_bind(
  claude = task_claude,
  gpt    = task_gpt
)
```

`vitals_bind()` adds a column for each name passed in, suitable for ordinal
regression (`ordinal::clm()` / `clmm()`) when comparing models.

## Example dataset

```r
are()
```

"An R Eval": 29 challenging R coding problems with reference solutions and
metadata. Useful as a sanity-check dataset.

```r
task <- Task$new(
  dataset = are(),
  solver  = generate(chat_anthropic()),
  scorer  = model_graded_qa()
)
task$eval()
```

## Environment

During `$eval()`, vitals sets `IN_VITALS_EVAL="true"` so solver code can
detect it is running inside an evaluation.

## Complete workflow

```r
library(vitals)
library(ellmer)

vitals_log_dir_set("~/eval-logs")

ds <- tibble::tibble(
  input  = c("Explain |>", "What is a tibble?", "How do I filter rows?"),
  target = c(
    "The native pipe |> passes the LHS as the first argument of the RHS.",
    "A tibble is a modern data.frame from the tibble package.",
    "Use dplyr::filter() with logical predicates."
  )
)

task_claude <- Task$new(
  dataset = ds,
  solver  = generate(chat_anthropic(model = "claude-sonnet-4-5")),
  scorer  = model_graded_qa()
)
task_claude$eval()

task_gpt <- Task$new(
  dataset = ds,
  solver  = generate(chat_openai(model = "gpt-4o")),
  scorer  = model_graded_qa()
)
task_gpt$eval()

results <- vitals_bind(claude = task_claude, gpt = task_gpt)
costs   <- dplyr::bind_rows(
  claude = task_claude$get_cost(),
  gpt    = task_gpt$get_cost(),
  .id    = "model"
)
```

## Documentation

- Package site: https://vitals.tidyverse.org/
- GitHub: https://github.com/tidyverse/vitals
- Vignettes: getting started, custom solvers and tool calling, writing evals
  for your LLM product, analyzing evaluation results
