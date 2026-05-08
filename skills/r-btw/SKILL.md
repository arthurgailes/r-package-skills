---
name: r-btw
description: Use when code loads or uses btw (library(btw), btw::), providing R session context to LLMs, registering tools for ellmer chat, or copying R object descriptions to clipboard
---

# btw: Context Tools for R and LLMs

## Overview

**btw connects R's context to LLMs.** Provides tools that describe R objects, fetch documentation, and expose R session/project information to ellmer chat sessions.

**btw vs mcptools:** btw gives tools TO ellmer (R -> LLM). mcptools lets agents INTO R (LLM -> R).

**Install:** `install.packages("btw")` (CRAN 1.2.1, 2026-03-23)

## References

Read `references/API.md` before writing code.

- `references/API.md` - Complete function reference (current names + 1.2.0 renames)
- `references/package-docs.md` - Tool registration, tool groups, MCP integration

## When to Use

- Copy R object descriptions to clipboard for pasting into LLM chats
- Register R tools (docs, env, files, git, pkg, ...) with an ellmer chat
- Launch a btw-enhanced chat client or Shiny app (`btw_client()`, `btw_app()`)
- Run pre-formatted task files with template variables (`btw_task()`)
- Build subagent tools from markdown specs (`btw_agent_tool()`)
- Create / edit project context files (`use_btw_md()`, `edit_btw_md()`)
- Expose tools to external agents over MCP (`btw_mcp_server()`)

## When NOT to Use

- Want agents to run R code in your IDE session (use r-mcptools)
- Just need to print an object (use `print()` / `str()`)
- One-off manual description, no LLM involved (just copy/paste)

## Quick Reference

```r
library(btw)

# 1. Clipboard descriptions for pasting into a chat
btw(mtcars)            # describe data frame, copies to clipboard
btw("ggplot2")         # package documentation
btw(my_fn, my_data)    # multiple objects at once
btw(clipboard = FALSE) # disable clipboard (e.g. in scripts)

# 2. Register tools with an ellmer chat
library(ellmer)
chat <- ellmer::chat_anthropic()
chat$register_tools(btw_tools())          # all tools
chat$register_tools(btw_tools("docs"))    # only docs group
chat$register_tools(btw_tools("docs", "env", "files"))  # multiple groups

# 3. Pre-configured chat client / app
client <- btw_client()                    # console chat with btw tools
btw_app()                                 # Shiny chat UI

# 4. Project context file
use_btw_md()                              # create btw.md
edit_btw_md()                             # open existing btw.md

# 5. Task files (markdown w/ YAML frontmatter, {{ var }} interpolation)
btw_task("tasks/release-notes.md", pkg = "dplyr")

# 6. Custom subagent from a markdown spec
tool <- btw_agent_tool(".btw/agent-reviewer.md")
chat$register_tools(list(tool))

# 7. Expose tools over MCP to external agents
btw_mcp_server()
```

## Tool Groups

`btw_tools()` accepts group names as shorthand. Available groups:

`agent`, `cran`, `docs`, `env`, `files`, `git`, `github`, `ide`, `pkg`, `run`, `sessioninfo`, `skills`, `web`

```r
chat$register_tools(btw_tools("docs", "pkg"))   # docs + package dev tools
chat$register_tools(btw_tools(tools = FALSE))   # explicit empty (apps only)
```

## Common Mistakes

| Issue | Solution |
|-------|----------|
| Using `chat$set_tools(btw_tools())` | Use `chat$register_tools(btw_tools())`. `btw_tools()` registers in place against the current chat client. |
| Calling renamed/old tool names | 1.2.0 renamed several tools (see below). Old names work but emit deprecation warnings. |
| btw vs mcptools confusion | btw = tools TO chat; mcptools = agent INTO R |
| Expecting code execution by default | `btw_tool_run_r()` is opt-in (experimental, off by default). Add explicitly via `btw_tools("run")`. |
| `btw()` not copying in scripts | Clipboard is interactive-only by default; pass `clipboard = TRUE` or run in console. |

## 1.2.0 Renames (still work with deprecation warnings)

| Old | New |
|-----|-----|
| `btw_tool_session_platform_info()` | `btw_tool_sessioninfo_platform()` |
| `btw_tool_session_package_info()` | `btw_tool_sessioninfo_package()` |
| `btw_tool_session_check_package_installed()` | `btw_tool_sessioninfo_is_package_installed()` |
| `btw_tool_search_packages()` | `btw_tool_cran_search()` |
| `btw_tool_search_package_info()` | `btw_tool_cran_package()` |
| `btw_tool_files_list_files()` | `btw_tool_files_list()` |
| `btw_tool_files_read_text_file()` | `btw_tool_files_read()` |
| `btw_tool_files_write_text_file()` | `btw_tool_files_write()` |
| `btw_tool_files_code_search()` | `btw_tool_files_search()` |

Tool group aliases also renamed: `session` -> `sessioninfo`, `search` -> `cran`. Option keys `btw.files_code_search.*` -> `btw.files_search.*`.

## Core Functions

**Description (clipboard-friendly):**
- `btw(..., clipboard = TRUE)`: concatenate plain-text descriptions; copies to clipboard interactively
- `btw_this()`: S3 generic (methods for `character`, `data.frame`, `tbl`, `environment`) producing LLM-targeted descriptions

**ellmer integration:**
- `btw_tools(...)`: register tools / tool groups against the current chat
- `btw_client(..., client, tools, path_btw, path_llms_txt)`: btw-enhanced ellmer chat (console)
- `btw_app(..., client, tools, path_btw, messages)`: btw-enhanced Shiny chat app

**Project context:**
- `use_btw_md()`, `edit_btw_md()`: create/edit `btw.md`
- `btw_task_create_btw_md()`: task to initialize project context file
- Auto-discovery of `btw.md`, `AGENTS.md`, `CLAUDE.md`, and agents in `.btw/agent-*.md`, `.claude/agents/`

**Tasks & agents (1.2.0+):**
- `btw_task(path, ..., client, mode)`: run a markdown task with `{{ var }}` template vars; modes `app`/`console`/`client`/`tool`
- `btw_agent_tool(path, client)`: build an `ellmer::tool()` from an agent markdown spec
- `btw_task_create_readme()`, `btw_task_create_skill()`: pre-built tasks

**Skills (Agent Skills framework):**
- `btw_tool_skill(name)`: load a skill (used by chat as a tool)
- `btw_skill_install_github()`, `btw_skill_install_package()`: install skills

**MCP & CLI:**
- `btw_mcp_server()`, `btw_mcp_session()`: expose btw tools over MCP
- `install_btw_cli()`: install the `btw` command-line tool (1.2.0+)

## Integration

**With ellmer:** `chat$register_tools(btw_tools())`
**Cross-package patterns:** see r-ai meta-skill
