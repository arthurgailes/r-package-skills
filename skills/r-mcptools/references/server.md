# R as an MCP Server with mcptools

## Overview

mcptools enables AI coding assistants - Claude Desktop, Claude Code, VS Code GitHub Copilot, Positron Assistant - to execute R code through the Model Context Protocol (MCP). The architecture has three roles.

## Three Core Concepts

**Clients** - Applications that speak MCP and connect to mcptools' R-based server. Examples: Claude Desktop, Claude Code, Copilot Chat in VS Code, Positron Assistant.

**Server** - A long-lived process started with `mcp_server()`, typically launched by the client itself via `Rscript -e "mcptools::mcp_server()"`. The server exposes tools (custom ones you provide and/or built-in session-routing tools) and routes tool calls into R sessions.

**Sessions** - Interactive R environments (RStudio, Positron, R console) opted in by calling `mcptools::mcp_session()`. A bare `mcp_server()` with no `tools` argument and no sessions is essentially a no-op.

## Setup

### Claude Code

```bash
claude mcp add r-mcptools -- Rscript -e "mcptools::mcp_server()"
```

### Claude Desktop

Edit `claude_desktop_config.json` (paths in API.md):

```json
{
  "mcpServers": {
    "r-mcptools": {
      "command": "Rscript",
      "args": ["-e", "mcptools::mcp_server()"]
    }
  }
}
```

### VS Code Copilot Chat / Positron Assistant

Follow each client's MCP server registration steps using the same `Rscript -e "mcptools::mcp_server()"` command.

### Opting a Session In

In the R console (or `.Rprofile`):

```r
library(mcptools)
mcp_session()
```

Must run interactively, not in batch scripts.

## Transports (0.2.0+)

### stdio (default)

The client launches `Rscript -e "mcptools::mcp_server()"` and communicates via stdin/stdout. This is the standard pattern.

### HTTP

For setups where the server runs separately (e.g. background process, container):

```r
mcp_server(type = "http", host = "127.0.0.1", port = 8080)
# Port also controllable via MCPTOOLS_PORT env var
```

Bind to `127.0.0.1` unless you fully trust the network - the server can execute arbitrary R code.

## Multiple Sessions and Clients

When several R sessions call `mcp_session()`, mcptools picks a default. The agent can call the `list_r_sessions` and `select_r_session` MCP tools to switch.

To suppress those built-in tools (e.g. when exposing a curated tool set):

```r
mcp_server(tools = list(my_tool), session_tools = FALSE)
```

Multiple clients work without extra configuration. A single client connects to one R session at a time, which can complicate simultaneous multi-chat flows.

## Custom Tools

Pass a list of `ellmer::tool()` objects (or a path to an `.R` file that returns one) to `mcp_server()`:

```r
library(ellmer)
library(mcptools)

square <- tool(
  function(x) x^2,
  name = "square",
  description = "Square a number",
  arguments = list(x = type_number())
)

mcp_server(tools = list(square))
```

Tool results are formatted consistently with ellmer (since 0.2.0).

## R as MCP Client

mcptools also acts as an MCP client - registering third-party MCP servers as ellmer tools:

```r
library(mcptools)
library(ellmer)

chat <- chat_anthropic()
chat$set_tools(mcp_tools())
```

Default config path is `~/.config/mcptools/config.json`; override via `options(.mcptools_config = "...")` or by passing `config = "..."`.

The client uses stdio only. For HTTP-served MCP servers, wrap them in the `mcp-remote` tool and configure that command in your config.

## History

The package was previously called `acquaint`. Bundled tools (R session/object/docs helpers) moved to the btw package; mcptools focuses on the MCP protocol layer. Use `btw::btw_mcp_server()` or load btw in your session if you want those helpers.
