# btw API Reference

Complete function reference for the btw package - a toolkit for connecting R and LLMs.

Reflects upstream CRAN version 1.2.1 (2026-03-23). 1.2.0 introduced several tool renames; old names still work with deprecation warnings.

## Core Functions

| Function | Purpose |
|----------|---------|
| `btw(..., clipboard = TRUE)` | Concatenate plain-text descriptions of R objects; copies to clipboard when interactive |
| `btw_this(x, ...)` | S3 generic producing LLM-targeted descriptions |
| `btw_client(..., client, tools, path_btw, path_llms_txt)` | btw-enhanced ellmer chat client (console) |
| `btw_app(..., client, tools, path_btw, messages)` | btw-enhanced ellmer Shiny chat app |
| `btw_tools(...)` | Register tools / tool groups against the current ellmer chat |
| `btw_mcp_server()` | Launch an MCP server exposing btw tools |
| `btw_mcp_session()` | Initialize an MCP session with btw tools |
| `install_btw_cli()` | Install the `btw` command-line tool |

**Example:**
```r
library(btw)
library(ellmer)

# Clipboard
btw(mtcars)

# Register all btw tools with an ellmer chat (in place)
chat <- ellmer::chat_anthropic()
chat$register_tools(btw_tools())
```

`btw_tools()` accepts one or more **tool group** names or individual tool names (with or without the `btw_tool_` prefix). Available groups:

`agent`, `cran`, `docs`, `env`, `files`, `git`, `github`, `ide`, `pkg`, `run`, `sessioninfo`, `skills`, `web`

```r
chat$register_tools(btw_tools("docs"))                 # only docs tools
chat$register_tools(btw_tools("docs", "env", "files")) # several groups
```

## Project Context Management

| Function | Purpose |
|----------|---------|
| `use_btw_md()` | Create a `btw.md` context file for the project |
| `edit_btw_md()` | Open / modify an existing `btw.md` |
| `btw_task_create_btw_md()` | Pre-built task that drafts a project context file |

`btw_client()` / `btw_app()` auto-discover `btw.md`, `AGENTS.md`, and `CLAUDE.md` in the current directory and its parents. They also discover agents from `.btw/agent-*.md` and `.claude/agents/`.

## R Object Description (`btw_this`)

| Method | Purpose |
|--------|---------|
| `btw_this(<character>)` | Describe a referenced object/package by name |
| `btw_this(<data.frame>)` / `btw_this(<tbl>)` | Compact JSON-style description of a data frame |
| `btw_this(<environment>)` | Document environment contents |

## Tasks & Agents (1.2.0+)

| Function | Purpose |
|----------|---------|
| `btw_task(path, ..., client = NULL, mode = c("app","console","client","tool"))` | Run a markdown task file (YAML frontmatter + body) with `{{ var }}` interpolation. Named `...` are template vars; unnamed `...` are extra context (passed through `btw()`). |
| `btw_agent_tool(path, client = NULL)` | Build an `ellmer::tool()` from an agent markdown spec; register with `chat$register_tools(list(tool))`. |
| `btw_task_create_readme()` | Task: generate a polished README |
| `btw_task_create_skill()` | Task: scaffold a new skill |
| `btw_task_create_btw_md()` | Task: initialize project context file |

**Example:**
```r
# Task: pass package_name as a template variable
btw_task("tasks/release-notes.md", package_name = "dplyr", mode = "console")

# Custom subagent
tool <- btw_agent_tool(".btw/agent-reviewer.md")
chat$register_tools(list(tool))
```

## Skills Management

| Function | Purpose |
|----------|---------|
| `btw_tool_skill(name)` | Tool that loads a skill (`name = ""` lists all available skills) |
| `btw_skill_install_github(repo)` | Install a skill from a GitHub repo |
| `btw_skill_install_package(pkg)` | Install a skill bundled in an R package (`inst/skills/`) |

Skills are discovered, in priority order, from: btw's bundled skills, attached R packages with `inst/skills/`, user-level skill directories, and project-level skill folders.

## Tools (for LLM Integration)

These functions are registered against an ellmer chat via `btw_tools()`. Names below are the **current (1.2.0+)** names.

### Documentation Tools (`docs`)

| Function | Purpose |
|----------|---------|
| `btw_tool_docs_package_news()` | Read package release notes |
| `btw_tool_docs_package_help_topics()` | List help topics for a package |
| `btw_tool_docs_help_page()` | Display a help page |
| `btw_tool_docs_available_vignettes()` | List package vignettes |
| `btw_tool_docs_vignette()` | Display a vignette |

### Environment Tools (`env`)

| Function | Purpose |
|----------|---------|
| `btw_tool_env_describe_data_frame()` | Describe a data frame's structure |
| `btw_tool_env_describe_environment()` | Document environment contents |

### File Tools (`files`)

| Function | Purpose | Renamed from (1.2.0) |
|----------|---------|----------------------|
| `btw_tool_files_list()` | List project files | `btw_tool_files_list_files()` |
| `btw_tool_files_read()` | Read a text file | `btw_tool_files_read_text_file()` |
| `btw_tool_files_write()` | Write a text file | `btw_tool_files_write_text_file()` |
| `btw_tool_files_search()` | Code search in project | `btw_tool_files_code_search()` |
| `btw_tool_files_edit()` | Targeted line-based edit with hash anchoring (new in 1.2.0) | - |
| `btw_tool_files_replace()` | Find/replace exact strings (new in 1.2.0) | - |

### Git Tools (`git`)

| Function | Purpose |
|----------|---------|
| `btw_tool_git_status()` | Working tree status |
| `btw_tool_git_diff()` | Display diffs |
| `btw_tool_git_log()` | View history |
| `btw_tool_git_commit()` | Create commits |
| `btw_tool_git_branch_list()` | List branches |
| `btw_tool_git_branch_create()` | Create branches |
| `btw_tool_git_branch_checkout()` | Switch branches |

### Package Development Tools (`pkg`, 1.1.0+)

| Function | Purpose |
|----------|---------|
| `btw_tool_pkg_document()` | Generate package documentation |
| `btw_tool_pkg_test()` | Run package tests |
| `btw_tool_pkg_check()` | `R CMD check` |
| `btw_tool_pkg_coverage()` | Test coverage |
| `btw_tool_pkg_load_all()` | Verify package code loads (new in 1.2.0) |

### CRAN Tools (`cran`, renamed from `search` in 1.2.0)

| Function | Purpose | Renamed from |
|----------|---------|--------------|
| `btw_tool_cran_search()` | Search CRAN packages | `btw_tool_search_packages()` |
| `btw_tool_cran_package()` | Describe a CRAN package | `btw_tool_search_package_info()` |

### Session Info Tools (`sessioninfo`, renamed from `session` in 1.2.0)

| Function | Purpose | Renamed from |
|----------|---------|--------------|
| `btw_tool_sessioninfo_platform()` | Describe system platform | `btw_tool_session_platform_info()` |
| `btw_tool_sessioninfo_package()` | Gather package information | `btw_tool_session_package_info()` |
| `btw_tool_sessioninfo_is_package_installed()` | Check if a package is installed | `btw_tool_session_check_package_installed()` |

### Web Tools (`web`)

| Function | Purpose |
|----------|---------|
| `btw_tool_web_read_url()` | Convert a web page to markdown |

### Agent / IDE / GitHub / Run / Skills (other groups)

| Function | Group | Purpose |
|----------|-------|---------|
| `btw_tool_agent_subagent()` | `agent` | Hierarchical subagent dispatch (new in 1.2.0) |
| `btw_tool_github()` | `github` | GitHub API operations |
| `btw_tool_ide_read_current_editor()` | `ide` | Read the IDE's currently-active file |
| `btw_tool_run_r()` | `run` | Execute R code (experimental; opt-in, disabled by default) |
| `btw_tool_skill()` | `skills` | Load a skill |

**Example:**
```r
chat <- ellmer::chat_anthropic()

# Register all btw tools (in place)
chat$register_tools(btw_tools())

# Or register only specific groups
chat$register_tools(btw_tools("docs", "cran"))

chat$chat("What does dplyr::mutate do, and how has it changed recently?")
# LLM may call btw_tool_docs_help_page() and btw_tool_docs_package_news()
```

## Configuration Options

- `btw.files_search.extensions`, `btw.files_search.exclusions` (renamed from `btw.files_code_search.*` in 1.2.0)
- Multiple client configurations supported in `btw.md` frontmatter (1.2.0+)

## Documentation

**Package site:** https://posit-dev.github.io/btw/
**GitHub:** https://github.com/posit-dev/btw
**CRAN:** https://cran.r-project.org/web/packages/btw/
