# Knowi MCP Server

Connect Claude Code, Claude Desktop, Cursor, or any MCP-compatible AI client to [Knowi](https://www.knowi.com). Query your data, create dashboards, manage reports, and more — in natural language, directly from your AI assistant.

The MCP server is built into the Knowi application. No separate installation required.

## Quick Start

### 1. Generate an MCP Token

1. Log into Knowi
2. Go to **Settings → AI Settings**
3. Under **MCP Token**, choose an expiry and click **Generate Token**
4. Copy the token — it is shown only once

### 2. Connect to Claude Code

```bash
claude mcp add knowi https://your-instance.knowi.com/api/2.0/mcp \
  --transport http \
  --header "Authorization: Bearer YOUR_MCP_TOKEN"
```

For **knowi.com** accounts:

```bash
claude mcp add knowi https://www.knowi.com/api/2.0/mcp \
  --transport http \
  --header "Authorization: Bearer YOUR_MCP_TOKEN"
```

### 3. Connect to Claude Desktop

Edit your config file:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux:** `~/.config/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "knowi": {
      "type": "streamable-http",
      "url": "https://your-instance.knowi.com/api/2.0/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

Restart Claude Desktop after saving.

### 4. Verify

```
"What Knowi tools are available?"
```

You should see tools including `knowi_do`, `knowi_ask`, `knowi_search`, `knowi_list_dashboards`, and others.

---

## Available Tools

### AI-Powered Tools

Natural language tools that use Knowi's AI orchestrator to interpret instructions and chain operations.

| Tool | Description |
|------|-------------|
| `knowi_do` | Execute any Knowi operation via natural language |
| `knowi_ask` | Ask a data question — finds the dataset, generates a query, returns results |
| `knowi_search` | Search across all dashboards, widgets, datasets, and reports |

### Deterministic Tools

Direct, fast operations with predictable behavior.

| Tool | Description |
|------|-------------|
| `knowi_list_dashboards` | List all accessible dashboards |
| `knowi_get_data` | Retrieve data rows from a widget or dataset |
| `knowi_push_data` | Push data rows to a dataset (creates it if it doesn't exist) |
| `knowi_get_details` | Get full metadata for any asset |
| `knowi_find_widget` | Find widgets by name |
| `knowi_create_datasource` | Create a new datasource connection |
| `knowi_create_query` | Create and run a query against a datasource |
| `knowi_create_widget` | Create a new chart or visualization |
| `knowi_update_widget` | Update an existing widget |
| `knowi_create_report` | Create a scheduled report |
| `knowi_create_alert` | Create a data alert |
| `knowi_list_alerts` | List configured alerts |
| `knowi_test_alert` | Test an alert |
| `knowi_update_alert` | Update an alert |
| `knowi_run` | Trigger execution of a query or report |
| `knowi_export_pdf` | Export a dashboard or widget as PDF |
| `knowi_export_csv` | Export data as CSV |
| `knowi_get_embed_url` | Generate an embeddable share URL |
| `knowi_get_insights` | Get AI-generated insights for a widget |
| `knowi_layout_dashboard` | Arrange widgets on a dashboard |
| `knowi_import_file` | Import a file (CSV, Excel) as a dataset |
| `knowi_ingest_document` | Ingest a document for AI Q&A |
| `knowi_ask_document` | Ask questions against ingested documents |
| `knowi_extract_document_data` | Extract structured data from a document |
| `knowi_list_documents` | List ingested documents |
| `knowi_delete_document` | Delete an ingested document |
| `knowi_delete` | Delete an asset (requires explicit confirmation) |

---

## Usage Examples

```
"Show me my Knowi dashboards"

"Ask Knowi: what were our top customers by revenue last month?"

"Create a sales dashboard with revenue trends and top products"

"Search Knowi for dashboards related to customer analytics"

"Get the data from Knowi widget 12345"

"Export Knowi dashboard 67890 to PDF"

"Push these sales records to a Knowi dataset called 'Daily Sales'"

"Set up a daily email report for the executive dashboard"
```

---

## Token Management

**Rotate a token:**
1. Go to **Settings → AI Settings → MCP Token**
2. Click **Revoke Token**
3. Generate a new one and update your config:

```bash
claude mcp remove knowi
claude mcp add knowi https://your-instance.knowi.com/api/2.0/mcp \
  --transport http \
  --header "Authorization: Bearer YOUR_NEW_TOKEN"
```

**One token per user.** Generating a new token automatically revokes the previous one.

---

## API Reference

The MCP server exposes both Streamable HTTP (for MCP clients) and REST endpoints (for direct programmatic use).

### Health Check

```bash
curl https://your-instance.knowi.com/api/2.0/mcp/health
```

```json
{
  "status": "healthy",
  "toolCount": 28,
  "tools": ["knowi_do", "knowi_ask", "knowi_search", "..."],
  "activeSessions": 0
}
```

### List Tools

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://your-instance.knowi.com/api/2.0/mcp/tools
```

### Call a Tool (REST)

```bash
curl -X POST https://your-instance.knowi.com/api/2.0/mcp/tools/call \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "knowi_list_dashboards",
    "arguments": {}
  }'
```

### Streamable HTTP (JSON-RPC)

All MCP client operations use `POST /api/2.0/mcp` with standard JSON-RPC:

```bash
# Initialize
curl -X POST https://your-instance.knowi.com/api/2.0/mcp \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'
```

The response includes an `Mcp-Session-Id` header to use on subsequent requests.

---

## Security

- **Scoped credentials**: MCP tokens only access MCP endpoints — they cannot be used against the broader Management API.
- **Authorization on every request**: Every tool verifies the authenticated user has access to the requested resource before executing.
- **Prompt injection protection**: AI-powered tools scan instructions for known injection patterns and reject them before reaching the AI model.
- **Destructive operation safeguard**: `knowi_do` blocks delete/drop/purge instructions. Deletions require the explicit `knowi_delete` tool with a required `confirm: true` parameter.
- **Data truncation**: `knowi_get_data` and `knowi_ask` return at most 100 rows by default.
- **Session security**: Streamable HTTP sessions are bound to the authenticated user. Session IDs alone do not grant access.

### Data Flow

When Claude calls a Knowi MCP tool, the tool response (data rows, metadata, insights) is returned to Claude and processed on Anthropic's infrastructure. This is inherent to MCP — the client must receive results to act on them.

For maximum data isolation, use Knowi's in-product AI agents instead of MCP.

---

## Troubleshooting

**Tools not appearing in Claude:**
- Restart Claude Code / Claude Desktop completely
- Verify the server is reachable: `curl https://your-instance.knowi.com/api/2.0/mcp/health`
- Verify the token is valid: `curl -H "Authorization: Bearer TOKEN" https://your-instance.knowi.com/api/2.0/mcp/tools`

**"Failed to connect" in `claude mcp list`:**
- Confirm you used `--transport http` (required for Streamable HTTP)
- Remove and re-add the server if the transport was set incorrectly

**401 errors:**
- Token may be expired or revoked. Generate a new token from Settings → AI Settings and update your config.

**Tool calls return access denied:**
- The user whose token you configured does not have permission for the requested resource.
- For production use, create a dedicated Knowi user with the minimum required permissions.

---

## Requirements

- Knowi account with **AI Agents** enabled (Settings → AI Settings)
- Claude Code CLI, Claude Desktop, or any MCP-compatible client

---

## Documentation

- [MCP Server Reference](https://www.knowi.com/docs/mcp-server.html) — full API reference, all tool schemas, security model
- [Claude Code Setup Guide](https://www.knowi.com/docs/claude-code-setup.html) — step-by-step setup instructions
- [Knowi Documentation](https://www.knowi.com/docs/) — full product docs

---

## Support

- **Docs:** [knowi.com/docs](https://www.knowi.com/docs/)
- **Support:** [support.knowi.com](https://support.knowi.com)
- **Issues with this repo:** Open a GitHub issue
