# mcptools Reference

## R as MCP Server

Allow external agents to access your R session. After registering the server with your agent (below), opt the current R session in:

```r
library(mcptools)
mcp_session()  # interactive console or .Rprofile, NOT batch scripts
```

Call each time R starts. The session must be interactive.

## Agent Configuration

### Claude Code

```bash
claude mcp add r-mcptools -- Rscript -e "mcptools::mcp_server()"
```

### Claude Desktop

Add to `claude_desktop_config.json`:

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

### VS Code (Copilot Chat / Continue) / Positron Assistant

Use the same command (`Rscript -e "mcptools::mcp_server()"`) in each client's MCP server settings.

## What Agents Can Do

Once connected with at least one `mcp_session()` active, agents can:

- Execute R code in your session
- Inspect and modify objects in your environment
- Call functions from any loaded package
- Read and write files via R
- Use custom tools you provide via `mcp_server(tools = list(...))`
- Switch between sessions using `list_r_sessions` / `select_r_session` MCP tools (when `session_tools = TRUE`, the default)

## HTTP Transport (0.2.0+)

For non-stdio setups:

```r
mcp_server(type = "http", host = "127.0.0.1", port = 8080)
```

The `MCPTOOLS_PORT` environment variable provides the default port. Bind to `127.0.0.1` unless you have explicit reason to expose the server.

## Custom Tools

```r
library(ellmer)
library(mcptools)

custom_tool <- tool(
  function(x) x^2,
  name = "square",
  description = "Square a number",
  arguments = list(x = type_number())
)

mcp_server(tools = list(custom_tool))
```

## R as MCP Client

Use external MCP servers as ellmer tools:

```r
library(mcptools)
library(ellmer)

# Reads ~/.config/mcptools/config.json by default
tools <- mcp_tools()

chat <- chat_openai()
chat$set_tools(tools)
```

Override the config path via `options(.mcptools_config = "path/to/config.json")` or `mcp_tools(config = "path/to/config.json")`.

The client is stdio-only; for HTTP-served MCP servers, configure `mcp-remote` as the command in your config.

## Combining with btw

mcptools previously bundled helpers that now live in btw. Two ways to combine them:

```r
# Option 1: Run btw's standalone MCP server (different command in agent config)
btw::btw_mcp_server()

# Option 2: Load btw in your session, then mcp_session()
library(btw)
library(mcptools)
mcp_session()  # agent runs R code that uses btw helpers
```

## Session Management

| MCP tool (called by agent) | Purpose |
|----------------------------|---------|
| `list_r_sessions` | List sessions with active `mcp_session()` |
| `select_r_session` | Switch which session subsequent tool calls run in |

Toggle with `session_tools = TRUE/FALSE` in `mcp_server()`. There is no R-callable `mcp_list_sessions()` function.

## Security Notes

- Agents can execute arbitrary R code in your session
- They have access to your environment variables and any secrets the R process can read
- They can read and write files the R process can access
- Use only in trusted environments
- For HTTP transport, keep `host = "127.0.0.1"` unless you know what you are doing
