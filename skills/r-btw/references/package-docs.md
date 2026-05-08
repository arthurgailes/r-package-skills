# btw Reference

## Quick Context (clipboard)

```r
btw("dplyr")           # package documentation
btw(my_data)           # data frame structure
btw(my_function)       # function source/docs
btw(a, b, c)           # multiple objects in one prompt
btw(clipboard = FALSE) # don't write to clipboard
```

`btw()` returns an `ellmer::ContentText` object; printing it interactively copies the prompt to the clipboard.

## Tool Groups

When registered via `btw_tools()`, tools become available to the LLM. Groups (1.2.0+):

| Group | Description |
|-------|-------------|
| `docs` | Package help pages, vignettes, news |
| `env` | Data frame structure, environment contents |
| `files` | Read, write, list, search, edit, replace text files |
| `git` | Status, diff, log, commit, branch operations |
| `github` | GitHub API operations |
| `pkg` | `R CMD check`, tests, coverage, document, load_all |
| `cran` | CRAN search and package info (was `search` pre-1.2.0) |
| `sessioninfo` | Platform info, package info, install checks (was `session`) |
| `web` | Fetch URLs as markdown |
| `ide` | Read current editor file |
| `agent` | Subagent dispatch |
| `skills` | Load Agent Skills |
| `run` | Execute R code (opt-in) |

## Integration with ellmer

```r
library(btw)
library(ellmer)

chat <- ellmer::chat_anthropic()

# All tools (in place: btw_tools updates chat$register_tools(...))
chat$register_tools(btw_tools())

# Or register selectively
chat$register_tools(btw_tools("docs", "env"))

chat$chat("Show me the structure of mtcars")
chat$chat("What does dplyr::mutate() do?")
chat$chat("What files are in the R/ directory?")
```

> Migration note: pre-1.2.0 examples used `chat$set_tools(btw_tools())`. Use `chat$register_tools(...)` going forward.

## Interactive Apps

```r
# Console chat client (auto-loads btw.md / AGENTS.md / CLAUDE.md from project)
client <- btw_client()
client$chat("Help me analyze this data")

# Shiny chat app
btw_app()

# Both accept a `tools` argument:
btw_client(tools = c("docs", "env"))   # only those groups
btw_app(tools = FALSE)                 # no tools
```

`btw_client()` and `btw_app()` accept:

- `client` - an `ellmer::Chat`, a `"provider/model"` string, or an alias from your project config
- `tools` - tool/group selector (defaults to all)
- `path_btw` - explicit path to `btw.md` / `AGENTS.md` / `CLAUDE.md` (otherwise auto-discovered)
- `path_llms_txt` - path to `llms.txt`
- `messages` (`btw_app` only) - initial chat messages

## Tasks (1.2.0+)

Run a markdown task file with YAML frontmatter that configures client/tools, plus body that supports `{{ variable }}` interpolation.

```r
# tasks/release-notes.md frontmatter declares the client/tools.
# Named ... become template vars; unnamed ... become context via btw().
btw_task(
  "tasks/release-notes.md",
  package_name = "dplyr",
  mode = "console"   # or "app" (default), "client", "tool"
)
```

Pre-built tasks: `btw_task_create_readme()`, `btw_task_create_skill()`, `btw_task_create_btw_md()`.

## Custom Subagents (1.2.0+)

```r
tool <- btw_agent_tool(".btw/agent-reviewer.md")
chat$register_tools(list(tool))
chat$chat("Review the latest commit for style issues.")
```

Agent markdown files use the same YAML/body pattern (name, description, system prompt) and are auto-discovered from `.btw/agent-*.md` and `.claude/agents/` by `btw_client()` / `btw_app()`.

## MCP Server

Expose btw tools to external agents (Claude Desktop, VS Code, etc.):

```r
btw_mcp_server()  # serve btw tools over MCP
```

## Project Configuration

Create `btw.md` in the project root for persistent context:

```markdown
# btw.md
This project uses tidyverse conventions.
Data lives in data/ directory.
Tests use testthat.
```

Multiple client configurations are supported in `btw.md` frontmatter (1.2.0+) with interactive selection.

## CLI

`install_btw_cli()` installs the standalone `btw` command-line tool, which exposes the same tool groups (`docs`, `pkg`, `info`/`sessioninfo`, `cran`, ...) for use outside R.
