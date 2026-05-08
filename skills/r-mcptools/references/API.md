# mcptools API Reference

Complete function reference for the mcptools package (CRAN 0.2.1) - Model Context Protocol for R.

mcptools provides three exported functions plus configuration files. There is no `mcp_list_sessions()` or `mcp_mcp_server()` - those names are not real. `list_r_sessions` and `select_r_session` are MCP tools the agent invokes, not R-callable functions.

## R as a Server

### `mcp_server()`

Implements a Model Context Protocol server exposing R functions as tools. Pair with `mcp_session()` calls in interactive sessions to route tool execution into those sessions.

```r
mcp_server(
  tools = NULL,
  ...,
  type = c("stdio", "http"),
  host = "127.0.0.1",
  port = as.integer(Sys.getenv("MCPTOOLS_PORT", "8080")),
  session_tools = TRUE
)
```

| Argument | Description |
|----------|-------------|
| `tools` | Optional. List of objects from `ellmer::tool()`, or a path to an `.R` file that returns such a list. Defaults to `NULL`. |
| `...` | Reserved for future use. Currently ignored. |
| `type` | Transport. `"stdio"` (default) for stdin/stdout, `"http"` for HTTP. Added in 0.2.0. |
| `host` | Host to bind for HTTP transport. Defaults to `"127.0.0.1"`. Ignored for stdio. |
| `port` | Port for HTTP transport. Defaults to `MCPTOOLS_PORT` env var, then `8080`. |
| `session_tools` | Logical. Include built-in session-routing MCP tools (`list_r_sessions`, `select_r_session`). Default `TRUE`. Added in 0.2.0. |

**Stdio (typical):** Launched by the agent itself via `Rscript -e "mcptools::mcp_server()"`.

**HTTP (new in 0.2.0):**
```r
mcp_server(type = "http", host = "127.0.0.1", port = 8080)
```

**Custom tools:**
```r
library(ellmer)
square <- tool(
  function(x) x^2,
  name = "square",
  description = "Square a number",
  arguments = list(x = type_number())
)
mcp_server(tools = list(square))
```

### `mcp_session()`

Opts the current interactive R session in to receive tool execution from `mcp_server()`. Call once per session - typically in console or `.Rprofile`, never in batch scripts.

```r
mcp_session()
```

| Returns | A nanonext socket (invisibly). Most users ignore the return value. Added invisible-return behavior in 0.2.0. |

When multiple sessions call `mcp_session()`, mcptools routes to a default; the agent can switch using the `select_r_session` MCP tool.

## R as a Client

### `mcp_tools()`

Fetches tools from MCP servers configured in a JSON config file and returns a list compatible with `ellmer::Chat$set_tools()`.

```r
mcp_tools(config = NULL)
```

| Argument | Description |
|----------|-------------|
| `config` | Path to MCP servers JSON config. If `NULL`, uses option `.mcptools_config`, falling back to `file.path("~", ".config", "mcptools", "config.json")`. Errors if the file does not exist. |

```r
library(mcptools)
library(ellmer)

chat <- chat_anthropic()
chat$set_tools(mcp_tools())
```

**Note:** mcptools client uses stdio transport only. For HTTP-served MCP servers, run them through the external `mcp-remote` tool and configure that as the command in your config.

## Configuration Files

### Server (agent -> R) config

The agent specifies how to launch R. Format and location vary per agent.

**Claude Code:**
```bash
claude mcp add r-mcptools -- Rscript -e "mcptools::mcp_server()"
```

**Claude Desktop** - `claude_desktop_config.json`:

| OS | Path |
|----|------|
| Windows | `%APPDATA%/Claude/claude_desktop_config.json` |
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

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

### Client (R -> third-party MCP) config

Default path: `~/.config/mcptools/config.json` (override via `options(.mcptools_config = "...")` or pass `config` to `mcp_tools()`).

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/some/path"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "..." }
    }
  }
}
```

## Built-in MCP Tools (NOT R functions)

When `session_tools = TRUE` (default) and at least one session is active, the server exposes:

| Tool name | Purpose |
|-----------|---------|
| `list_r_sessions` | Agent lists currently-registered R sessions |
| `select_r_session` | Agent switches which R session subsequent tool calls run in |

These are invoked by the agent, not by you in R. Disable with `session_tools = FALSE`.

## Protocol Versions Supported

mcptools 0.2.1 negotiates protocol versions from `2024-11-05` through `2025-11-25`. mcptools 0.2.0 supported up to `2025-06-18`.

## Documentation

- **Pkgdown:** https://posit-dev.github.io/mcptools/
- **GitHub:** https://github.com/posit-dev/mcptools
- **CRAN:** https://cran.r-project.org/web/packages/mcptools/
