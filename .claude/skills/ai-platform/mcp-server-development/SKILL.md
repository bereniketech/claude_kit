---
name: mcp-server-development
description: Build MCP (Model Context Protocol) servers. Covers tool/resource/prompt registration, stdio/SSE/HTTP transports, authentication, testing, and publishing to the MCP ecosystem.
---

# MCP Server Development

Build Model Context Protocol servers that expose tools, resources, and prompts to Claude.

---

## 1. Core Concepts

```
MCP primitives:
- Tools:     Functions Claude can call (with parameters, return values)
- Resources: Files/data Claude can read (URI-addressed)
- Prompts:   Reusable prompt templates Claude can use

Transports:
- stdio:     CLI tools, local integrations (default)
- SSE:       Server-Sent Events — web-based, one-directional push
- HTTP:      REST-style, stateless
```

---

## 2. Quickstart (Node.js)

```bash
npm install @modelcontextprotocol/sdk zod
```

```typescript
// server.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "my-mcp-server",
  version: "1.0.0",
});

// Register a tool
server.tool(
  "get_weather",
  "Get current weather for a city",
  {
    city: z.string().describe("City name"),
    units: z.enum(["celsius", "fahrenheit"]).default("celsius"),
  },
  async ({ city, units }) => {
    const data = await fetchWeather(city, units);
    return {
      content: [{ type: "text", text: JSON.stringify(data, null, 2) }],
    };
  }
);

// Register a resource
server.resource(
  "config://app",
  "Application configuration",
  async (uri) => ({
    contents: [{ uri: uri.href, mimeType: "application/json", text: JSON.stringify(config) }],
  })
);

// Register a prompt
server.prompt(
  "analyze_code",
  "Analyze code for issues",
  { code: z.string(), language: z.string() },
  ({ code, language }) => ({
    messages: [{
      role: "user",
      content: { type: "text", text: `Analyze this ${language} code:\n\n${code}` },
    }],
  })
);

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## 3. Python Implementation

```bash
pip install mcp
```

```python
# server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
def search_database(query: str, limit: int = 10) -> list[dict]:
    """Search the database for records matching query."""
    return db.search(query, limit=limit)

@mcp.resource("data://{table}")
def get_table_data(table: str) -> str:
    """Read all records from a table."""
    return json.dumps(db.get_table(table))

if __name__ == "__main__":
    mcp.run()
```

---

## 4. Tool Design Guidelines

```typescript
// GOOD: specific, typed, documented
server.tool(
  "create_issue",           // lowercase_snake_case name
  "Create a GitHub issue in the specified repository",  // clear description
  {
    owner: z.string().describe("Repository owner (username or org)"),
    repo: z.string().describe("Repository name"),
    title: z.string().max(256).describe("Issue title"),
    body: z.string().optional().describe("Issue body in Markdown"),
    labels: z.array(z.string()).default([]).describe("Label names to apply"),
  },
  async ({ owner, repo, title, body, labels }) => { ... }
);

// BAD: vague, untyped
server.tool("do_thing", "Does something", { input: z.any() }, async () => {});
```

**Tool design rules:**
- Name: `verb_noun` format (`get_file`, `create_user`, `search_docs`)
- Description: tell Claude when to use this tool (not just what it does)
- Parameters: always use `z.string().describe(...)` — Claude needs param descriptions
- Return: always include `type: "text"` content; add `isError: true` for failures
- Idempotency: GET-like tools should be safe to call multiple times

---

## 5. Error Handling

```typescript
server.tool("risky_operation", "...", schema, async (params) => {
  try {
    const result = await doOperation(params);
    return { content: [{ type: "text", text: JSON.stringify(result) }] };
  } catch (error) {
    // Return error as tool result (not thrown exception)
    return {
      content: [{ type: "text", text: `Error: ${error.message}` }],
      isError: true,
    };
  }
});
```

---

## 6. Testing

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { InMemoryTransport } from "@modelcontextprotocol/sdk/inMemory.js";

const [clientTransport, serverTransport] = InMemoryTransport.createLinkedPair();
await server.connect(serverTransport);

const client = new Client({ name: "test-client", version: "1.0" }, {});
await client.connect(clientTransport);

// Call tool
const result = await client.callTool({ name: "get_weather", arguments: { city: "London" } });
console.assert(result.content[0].type === "text");
```

---

## 7. Claude Desktop Configuration

```json
// ~/.claude/claude_desktop_config.json (macOS)
// %APPDATA%\Claude\claude_desktop_config.json (Windows)
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/absolute/path/to/server.js"],
      "env": {
        "API_KEY": "your-key-here"
      }
    }
  }
}
```

---

## 8. Publishing

```json
// package.json
{
  "name": "@yourorg/mcp-server-name",
  "version": "1.0.0",
  "bin": { "mcp-server-name": "dist/server.js" },
  "files": ["dist/"],
  "engines": { "node": ">=18" }
}
```

```bash
# Build and publish
npm run build
npm publish --access public
```

List on: `modelcontextprotocol.io/servers` and Claude's MCP marketplace.
