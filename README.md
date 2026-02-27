# tailwindplus-mcp-connector

A FastMCP server for browsing and searching TailwindPlus components.

## Overview

MCP (Model Context Protocol) server that provides tools for working with TailwindPlus component data. Built with [FastMCP](https://github.com/jlowin/fastmcp) for seamless integration with AI assistants.

## Prerequisites

- Python >= 3.13
- [uv](https://docs.astral.sh/uv/) package manager

## Installation

```bash
uv sync
```

## Usage

### As an MCP Server

Configure your MCP client (e.g., Claude Desktop) with:

```json
{
  "mcpServers": {
    "tailwindplus": {
      "type": "stdio",
      "command": "uv",
      "args": ["run", "tailwindplus-mcp-connector"],
      "env": {
        "MCP_TAILWINDPLUS_DATA": "${MCP_TAILWINDPLUS_DATA}"
      }
    }
  }
}
```

## Development

```bash
# Install dev dependencies
uv sync --group dev

# Run linter
uv run ruff check .

# Run formatter
uv run ruff format .

# Run tests
uv run pytest
```

## License

MIT
