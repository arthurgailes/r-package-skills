---
name: r-mcptools
description: Use when code loads or uses mcptools (library(mcptools), mcptools::), connecting AI agents to R sessions via MCP, exposing R as an MCP server (stdio or HTTP), or using R as an MCP client to register third-party MCP servers as ellmer tools
---

# mcptools: Model Context Protocol for R

## Overview

**mcptools is bidirectional MCP for R.**

- **R as a server** (`mcp_server()` + `mcp_session()`): Agents (Claude Code, Claude Desktop, VS Code Copilot, Positron Assistant) execute R code in your live session.
- **R as a client** (`mcp_tools()`): Register third-party MCP servers (GitHub, filesystem, etc.) as ellmer tools.

**btw vs mcptools:** btw gives tools TO ellmer (R -> LLM, single chat). mcptools serves R to external agents (LLM -> R) AND lets R register external MCP servers (R chats -> third-party MCP).

**Note on history:** mcptools was previously called `acquaint`. Per-package tool helpers now live in btw; mcptools focuses on the MCP protocol layer.

**Install:** `install.packages("mcptools")` (CRAN 0.2.1, 2026-03-17).

## References

Read `references/API.md` before writing code.

- `references/API.md` - Function signatures, arguments, defaults, config file formats
- `references/server.md` - R as MCP server (clients, sessions, custom tools, HTTP transport)
- `references/package-docs.md` - Setup, agent configuration, security notes

## When to Use

- Let Claude Code, Claude Desktop, VS Code Copilot, or Positron Assistant run R code in your active session
- Expose custom R functions as MCP tools via `ellmer::tool()` -> `mcp_server(tools = ...)`
- Run an HTTP-transport MCP server for R (new in 0.2.0)
- Register third-party MCP servers (filesystem, GitHub, etc.) as ellmer tools from R via `mcp_tools()`
- Switch agent attention between multiple R sessions (`list_r_sessions`, `select_r_session` MCP tools)

## When NOT to Use

- Want to call LLMs FROM R (use r-ellmer)
- Just need R object descriptions (use r-btw)
- Building a chatbot in R that needs btw helpers (use r-ellmer + r-btw)

## Quick Reference

### R as a server (most common)

Configure your agent once, then opt R sessions in by calling `mcp_session()`.

```r
# In R console (or .Rprofile) for each session you want agents to access
library(mcptools)
mcp_session()
```

Agent (Claude Code) configuration:

```bash
claude mcp add r-mcptools -- Rscript -e "mcptools::mcp_server()"
```

The agent can now run R code, inspect objects, and call functions in any R session that called `mcp_session()`.

### R as a server with custom tools

```r
library(ellmer)
library(mcptools)

square <- tool(
  function(x) x^2,
  name = "square",
  description = "Square a number",
  arguments = list(x = type_number())
)

# Path to file or list of ellmer::tool() outputs
mcp_server(tools = list(square))
```

### R as a server over HTTP (new in 0.2.0)

```r
mcp_server(type = "http", host = "127.0.0.1", port = 8080)
# Or set MCPTOOLS_PORT env var to control port
```

### R as a client

Config file at `~/.config/mcptools/config.json` (override via `options(.mcptools_config = "path")`):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
    }
  }
}
```

```r
library(mcptools)
library(ellmer)

chat <- chat_anthropic()
chat$set_tools(mcp_tools())  # registers filesystem tools from config
```

## Function Signatures

```r
mcp_server(
  tools = NULL,                                # list of ellmer::tool() or .R file path
  ...,
  type = c("stdio", "http"),                   # 0.2.0+
  host = "127.0.0.1",                          # HTTP only
  port = as.integer(Sys.getenv("MCPTOOLS_PORT", "8080")),
  session_tools = TRUE                         # 0.2.0+: include list/select session tools
)

mcp_session()  # invisibly returns nanonext socket (0.2.0+)

mcp_tools(config = NULL)  # path to JSON config; defaults to .mcptools_config option
```

## Common Mistakes

| Issue | Solution |
|-------|----------|
| btw vs mcptools confusion | btw = tools TO an R chat; mcptools = agent INTO R, or R registering external MCP servers |
| `mcp_session()` in scripts | Call interactively or in `.Rprofile`; not in batch scripts |
| `list_r_sessions` / `select_r_session` as R functions | These are MCP tools the agent calls, not R-callable functions; toggle with `session_tools = FALSE` |
| Bare `mcp_server()` does nothing useful | Without `tools = ...` and without sessions, no tools are exposed; pair with `mcp_session()` or pass `tools` |
| HTTP transport over public network | `host = "127.0.0.1"` is the default for a reason; agents executing arbitrary R code is unsafe over public hosts |
| Expecting R-side HTTP MCP client | mcptools client is stdio only; for HTTP-served MCP, use the external `mcp-remote` tool |
| Needing per-package helpers (e.g. session info, docs) | Those moved to btw; use `btw::btw_mcp_server()` or btw tools alongside mcptools |

## Multiple Sessions

When several R sessions call `mcp_session()`, mcptools picks one by default. The agent can switch via the built-in `list_r_sessions` and `select_r_session` MCP tools (set `session_tools = FALSE` in `mcp_server()` to disable).

## Security

`mcp_server()` lets the connected agent execute arbitrary R code: full filesystem and environment-variable access for the R process, ability to call any installed package. Use only in trusted environments. Bind HTTP transport to `127.0.0.1` unless you know what you are doing.

## Integration

- **With Claude Code:** `claude mcp add r-mcptools -- Rscript -e "mcptools::mcp_server()"`, then `mcp_session()` in console
- **With ellmer chats:** `chat$set_tools(mcp_tools())` to import third-party MCP servers as tools
- **With btw:** Combine `btw::btw_mcp_server()` for package introspection or load btw in a session and use mcptools to expose it
- **Cross-package patterns:** See r-ai meta-skill
