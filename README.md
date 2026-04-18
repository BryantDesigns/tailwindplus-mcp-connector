# TailwindPlus MCP Connector

An [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) server that exposes downloaded [TailwindPlus](https://tailwindcss.com/plus) component data to AI assistants like Claude, Cursor, and other MCP-compatible clients.

Search, browse, and retrieve TailwindPlus UI components — including **Application UI**, **Marketing**, and **eCommerce** categories — in HTML, React, or Vue, for Tailwind CSS v3 or v4, directly from your AI assistant.

---

## Prerequisites

1. **A TailwindPlus components JSON file** — generate one with the [TailwindPlus Downloader](https://github.com/BryantDesigns/ts-tailwindplus-downloader) (`>= 1.0.0`).
2. **[`uv`](https://docs.astral.sh/uv/getting-started/installation/)** — the Python project/package manager used to run the server.

---

## Quick Start

### 1. Obtain the components data file

```bash
npx github:BryantDesigns/ts-tailwindplus-downloader --output=components.json
```

See the [TailwindPlus Downloader README](https://github.com/BryantDesigns/ts-tailwindplus-downloader#readme) for detailed usage, authentication, and options.

### 2. Install `uv`

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or via Homebrew
brew install uv
```

### 3. Add to Claude Code

```bash
claude mcp add tailwindplus -- \
  uv run --directory /path/to/tailwindplus-mcp-connector \
  tailwindplus-mcp-connector \
  --tailwindplus-data /path/to/components.json
```

### 4. Store your TailwindPlus preferences in memory

Tell Claude your project defaults so it can use the correct tool parameters automatically:

```
# Use TailwindPlus React components for Tailwind CSS v4 in light mode.
  ⎿  Got it.
```

---

## Shared Config (team)

Add with project scope and set the env var so each developer can point to their own data file:

```bash
claude mcp add -s project tailwindplus -- \
  uv run --directory /path/to/tailwindplus-mcp-connector \
  tailwindplus-mcp-connector
```

Then set `MCP_TAILWINDPLUS_DATA=/path/to/components.json` in each developer's environment. This keeps `.mcp.json` committable to the repo while allowing different data files per developer.

---

## Other Clients

### JSON config (VS Code, Cursor, Windsurf, etc.)

```json
{
  "mcpServers": {
    "tailwindplus": {
      "type": "stdio",
      "command": "uv",
      "args": [
        "run",
        "--directory", "/path/to/tailwindplus-mcp-connector",
        "tailwindplus-mcp-connector",
        "--tailwindplus-data", "/path/to/components.json"
      ]
    }
  }
}
```

Or use the environment variable instead of the CLI flag:

```json
{
  "mcpServers": {
    "tailwindplus": {
      "type": "stdio",
      "command": "uv",
      "args": [
        "run",
        "--directory", "/path/to/tailwindplus-mcp-connector",
        "tailwindplus-mcp-connector"
      ],
      "env": {
        "MCP_TAILWINDPLUS_DATA": "/path/to/components.json"
      }
    }
  }
}
```

### Run directly

```bash
uv run --directory /path/to/tailwindplus-mcp-connector \
  tailwindplus-mcp-connector \
  --tailwindplus-data /path/to/components.json
```

### Environment Variables

| Variable | Description |
|----------|-------------|
| `MCP_TAILWINDPLUS_DATA` | Path to the TailwindPlus components JSON file. Overridden by `--tailwindplus-data`. |

---

## Claude Desktop

Claude Desktop requires HTTPS for HTTP-based MCP servers and does not permit localhost URLs. Use [ngrok](https://ngrok.com/) to expose the server:

```bash
# Terminal 1: start the MCP server over HTTP
uv run tailwindplus-mcp-connector --transport http --tailwindplus-data /path/to/components.json

# Terminal 2: tunnel via ngrok (terminates TLS at its edge)
ngrok http 8000
```

Then add the ngrok URL (`https://<id>.ngrok-free.app/mcp`) as a custom connector in Claude Desktop (Settings > Connectors > Add custom connector).

ngrok handles TLS termination, so the MCP server itself runs plain HTTP.

Claude Desktop also supports **MCP Apps**, which renders component previews visually in the conversation rather than returning raw HTML. This activates automatically when using the `get_component_preview_by_full_name` tool.

---

## Tips

The TailwindPlus components JSON file has a **version** (timestamp of the download).

To make future markup maintenance easy, tell Claude to add the component **name** and **version** as a comment in your source code:

```jsx
{/* Start: Application UI > Forms > Input Groups > Label with leading icon | v.2026-02-27-120000 */}

// ... component code ...

{/* End: Application UI > Forms > Input Groups > Label with leading icon | v.2026-02-27-120000 */}
```

If you add a version comment to your code before requesting a component, the AI assistant can infer the correct framework and version automatically:

```html
<!-- Using Tailwind CSS v4 with React -->
```

---

## Example Usage

Ask for UI:

```
> I need a simple one-line search input to put in my app's header.

⏺ tailwindplus - Search Component Names (MCP)(search_term: "input")
  ⎿  [
       "Application UI.Forms.Input Groups.Input with leading icon",
       "Application UI.Forms.Input Groups.Input with keyboard shortcut",
       … +20 lines

⏺ tailwindplus - Get Component by Full Name (MCP)(
    full_name: "Application UI.Forms.Input Groups.Input with leading icon",
    framework: "react",
    tailwind_version: "4",
    mode: "light"
  )
  ⎿  { "version": "2026-02-27-120000", "full_name": "...", "code": "..." }

 Here's a search input component based on the "Input with leading icon" from
 TailwindPlus. Swap the EnvelopeIcon for MagnifyingGlassIcon and you're set.
```

Claude appreciates other types of guidance too:

```
# Referring to "component" means a TailwindPlus component.
  ⎿  Got it.
```

Ask for code for a specific use-case:

```
> Show me the code for a URL input with the schema prefix, e.g. `https:// [INPUT]`

⏺ tailwindplus - Search Component Names (MCP)(search_term: "input")
  ⎿  [ ... list of matching components ... ]

⏺ tailwindplus - Get Component by Full Name (MCP)(
    full_name: "Application UI.Forms.Input Groups.Input with inline add-on",
    framework: "react",
    tailwind_version: "4",
    mode: "light"
  )
  ⎿  { "version": "2026-02-27-120000", ... }

⏺ Here's a TailwindPlus React component for URL input with "https://" prefix:
   <shows React code>
```

---

## Available Tools

All tools are **read-only** and **idempotent**.

### `list_component_names`

Returns a sorted list of all available component names (dot-separated paths).

### `search_component_names`

Search for components by keyword (case-insensitive partial match).

| Parameter | Type | Description |
|-----------|------|-------------|
| `search_term` | `string` | Keyword to match against component names |

### `get_component_by_full_name`

Retrieve component code for a specific component.

| Parameter | Type | Description |
|-----------|------|-------------|
| `full_name` | `string` | Dot-separated path, e.g. `Application UI.Forms.Input Groups.Label with leading icon` |
| `framework` | `html` \| `react` \| `vue` | Target framework |
| `tailwind_version` | `3` \| `4` | Tailwind CSS version (`4` for new projects, `3` for legacy) |
| `mode` | `light` \| `dark` \| `system` \| `none` | Theme mode (see [Mode Rules](#mode-rules)) |

### `get_component_preview_by_full_name`

Same parameters as `get_component_by_full_name`; returns the **preview HTML** instead of the component code. In Claude Desktop, this renders the component visually via MCP Apps rather than returning raw HTML.

### `list_tailwindplus_information`

Returns metadata about the loaded data file: version, download date, component count, download duration, and downloader version.

---

## Available Resources

MCP resources provide direct URI access to components:

| URI Template | Returns |
|---|---|
| `twplus://{full_name}/{framework}/{version}/{mode}` | Component code (JSON) |
| `twplus://{full_name}/{framework}/{version}/{mode}/preview` | Preview HTML |

Example: `twplus://Application UI.Forms.Input Groups.Label with leading icon/react/4/light`

> **Note:** Resources require MCP clients that support "resource templates". Not all clients support this feature yet.

### Parameters

| Parameter | Values | Description |
|-----------|--------|-------------|
| `full_name` | dot-separated path | e.g. `Application UI.Forms.Input Groups.Label with leading icon` |
| `framework` | `html`, `react`, `vue` | Target framework |
| `version` | `3`, `4` | Tailwind CSS version |
| `mode` | `light`, `dark`, `system`, `none` | Theme mode (see [Mode Rules](#mode-rules)) |

---

## Mode Rules

Mode requirements are **enforced by the server** — an invalid combination returns a descriptive error.

| Component Category | Allowed Modes |
|---|---|
| **Application UI** | `light`, `dark`, `system` |
| **Marketing** | `light`, `dark`, `system` |
| **eCommerce** | `none` only |

---

## Component Organization

Components are organized hierarchically using dot-separated paths following the TailwindPlus category structure:

```
Application UI.Forms.Input Groups.Label with leading icon
Marketing.Sections.Heroes.With angled image on right
Ecommerce.Components.Shopping Carts.Simple
```

- **Categories:** Application UI, Marketing, Ecommerce
- **Full Names:** Dot-separated paths like `Marketing.Hero Sections.Simple centered`
- **Search:** Supports partial keyword matching — use `search_component_names` to find components

Use `list_component_names` to browse all available paths.

---

## Cache Management

Component data is parsed into a **SQLite cache** on first load and reused on subsequent starts. The cache is keyed by the source file's size and modification time — it rebuilds automatically when the data file changes.

```bash
uv run tailwindplus-mcp-connector --clear-cache   # remove all cached databases
```

---

## Data Source

This server requires a TailwindPlus components JSON file generated by the [TailwindPlus Downloader](https://github.com/BryantDesigns/ts-tailwindplus-downloader) (`>= 1.0.0`).

**Generate the data file:**

```bash
npx github:BryantDesigns/ts-tailwindplus-downloader --output=components.json
```

**Configure the data file path** via either method:

- **Command line:** `--tailwindplus-data /path/to/components.json`
- **Environment:** `MCP_TAILWINDPLUS_DATA=/path/to/components.json`

The CLI flag takes priority over the environment variable.

---

## Development

```bash
# Install dependencies
uv sync

# Run server locally
uv run tailwindplus-mcp-connector --tailwindplus-data /path/to/components.json

# Run tests
uv run pytest

# Format + lint (run before every commit)
uv run ruff format . && uv run ruff check . --fix
```

### Running a development version in another project

To test a dev version of this MCP in another project, add to that project's `.mcp.json`:

```bash
claude mcp add -s project tailwindplus -- \
  uv run --directory /absolute/path/to/tailwindplus-mcp-connector \
  tailwindplus-mcp-connector
```

Resulting `.mcp.json`:

```json
{
  "mcpServers": {
    "tailwindplus": {
      "type": "stdio",
      "command": "uv",
      "args": [
        "run",
        "--directory", "/absolute/path/to/tailwindplus-mcp-connector",
        "tailwindplus-mcp-connector"
      ],
      "env": {
        "MCP_TAILWINDPLUS_DATA": "/path/to/components.json"
      }
    }
  }
}
```

### Inspecting the server directly

Use the [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector) to execute MCP commands interactively, review tool/resource schemas, etc.:

```bash
MCP_TAILWINDPLUS_DATA=/path/to/components.json \
  npx @modelcontextprotocol/inspector \
  uv run tailwindplus-mcp-connector
```

---

## Acknowledgments

Inspired by [mcp-tailwindplus](https://github.com/richardkmichael/mcp-tailwindplus) by Richard Michael. This is an independent Python/FastMCP rewrite with added mode support and SQLite caching.

---

## License

MIT
