# Pulsetic MCP Server

Connect AI assistants to your Pulsetic monitoring platform using the [Model Context Protocol](https://modelcontextprotocol.io) (MCP).

---

## What is MCP?

The Model Context Protocol is an open standard that allows AI assistants to interact with external tools and services. With Pulsetic's MCP server, you can ask your AI assistant to check monitor status, create incidents, schedule maintenance, and more — all through natural conversation.

**Supported clients:**

- **Claude Desktop** — Anthropic's desktop app
- **Claude Code** — Anthropic's CLI for developers
- **Cursor** — AI-powered code editor
- **Windsurf** — AI-powered IDE
- Any client implementing the [MCP specification](https://modelcontextprotocol.io/specification/2025-03-26)

---

## Prerequisites

- A Pulsetic account on a plan with **API access** (Business or Enterprise).
- A Pulsetic API key — generate one from **Settings → API** in your dashboard.
- An MCP-compatible AI client.

---

## Setup

### Claude Desktop

Open **Settings → Developer → Edit Config** and add:

```json
{
  "mcpServers": {
    "pulsetic": {
      "type": "streamableHttp",
      "url": "https://api.pulsetic.com/mcp",
      "headers": {
        "Authorization": "YOUR_API_KEY"
      }
    }
  }
}
```

Replace `YOUR_API_KEY` with your actual API key. Restart Claude Desktop to apply.

### Claude Code

Add the server to your project's `.mcp.json` file:

```json
{
  "mcpServers": {
    "pulsetic": {
      "type": "streamableHttp",
      "url": "https://api.pulsetic.com/mcp",
      "headers": {
        "Authorization": "YOUR_API_KEY"
      }
    }
  }
}
```

### Cursor

Open **Settings → MCP Servers → Add Server** and configure:

- **Type:** streamableHttp
- **URL:** `https://api.pulsetic.com/mcp`
- **Headers:** `Authorization: YOUR_API_KEY`

> **Keep your API key private.** Your API key has full access to your Pulsetic account. Never commit it to version control or share it publicly.

---

## Available Tools

The MCP server exposes 17 tools organized into four categories.

### Monitors

| Tool | Description |
|------|-------------|
| `list_monitors` | List all monitors with status, uptime, and response time. |
| `get_monitor` | Get detailed info about a specific monitor. |
| `get_monitor_stats` | Uptime %, downtime, and response times across 1d/7d/30d/90d. |
| `get_monitor_events` | Online/offline events within a date range. |
| `get_monitor_downtime` | Total downtime in seconds for a given period. |
| `create_monitor` | Create a new HTTP, TCP, or ICMP monitor. |
| `update_monitor` | Update a monitor's name, URL, frequency, or settings. |
| `delete_monitor` | Permanently delete a monitor. |
| `pause_monitor` | Pause monitoring (stop checks). |
| `resume_monitor` | Resume a paused monitor. |

### Status Pages

| Tool | Description |
|------|-------------|
| `list_status_pages` | List all status pages with title, slug, and domain. |

### Incidents

| Tool | Description |
|------|-------------|
| `list_incidents` | List incidents for a status page. |
| `create_incident` | Create a new incident with an initial status update. |
| `update_incident` | Post a status update to an existing incident. |
| `resolve_incident` | Mark an incident as resolved. |

### Maintenance

| Tool | Description |
|------|-------------|
| `create_maintenance` | Schedule a maintenance window on a status page. |
| `delete_maintenance` | Delete a scheduled maintenance window. |

---

## Usage Examples

Once configured, you can interact with Pulsetic through natural language.

### Check on your monitors

```
"Show me all my monitors and their current status."

"What's the uptime for my production API monitor over the last 30 days?"

"Are any of my monitors currently down?"

"Show me the downtime events for monitor 42 in the past week."
```

### Manage monitors

```
"Create a new monitor for https://api.example.com that checks every 30 seconds."

"Pause the staging monitor while we do the migration."

"Resume all paused monitors."

"Change the check frequency of monitor 15 to 60 seconds."

"Delete the old marketing-site monitor."
```

### Handle incidents

```
"Create an incident on our main status page: 'Elevated API latency'
 with status investigating and message 'We're looking into slow response times.'"

"Update the API latency incident to identified: 'Root cause is a database
 connection pool issue. Working on a fix.'"

"Resolve incident 8 with message 'Connection pool limits increased.
 Response times back to normal.'"
```

### Schedule maintenance

```
"Schedule a maintenance window on status page 1 called 'Database upgrade'
 starting 2025-03-20 at 2am UTC and ending at 4am UTC."

"Delete the maintenance window we scheduled for next week."
```

---

## Testing with cURL

You can test the MCP endpoint directly with cURL to verify your API key works.

### Initialize

```bash
curl -X POST https://api.pulsetic.com/mcp \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2025-03-26",
      "capabilities": {},
      "clientInfo": {"name": "curl", "version": "1.0"}
    }
  }'
```

### List tools

```bash
curl -X POST https://api.pulsetic.com/mcp \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

### Call a tool

```bash
curl -X POST https://api.pulsetic.com/mcp \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "list_monitors",
      "arguments": {}
    }
  }'
```

---

## Tool Reference

### list_monitors

Returns all monitors for your account.

*No parameters.*

---

### get_monitor

Returns full details for a single monitor, including SSL certificate info.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monitor_id` | integer | Yes | The monitor ID. |

---

### get_monitor_stats

Returns uptime %, downtime, incident count, and average response time across 1-day, 7-day, 30-day, and 90-day periods.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monitor_id` | integer | Yes | The monitor ID. |

---

### get_monitor_events

Returns online/offline events in a date range (max 100, newest first).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monitor_id` | integer | Yes | The monitor ID. |
| `from` | string | No | Start date (YYYY-MM-DD). Default: 7 days ago. |
| `to` | string | No | End date (YYYY-MM-DD). Default: today. |

---

### get_monitor_downtime

Returns total downtime in seconds and incident count for a period.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monitor_id` | integer | Yes | The monitor ID. |
| `days` | integer | No | Days to look back. Default: 30. |

---

### create_monitor

Creates a new uptime monitor.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL or hostname to monitor. |
| `name` | string | No | Display name. |
| `request_type` | string | No | `http`, `tcp`, or `icmp`. Default: `http`. |
| `uptime_check_frequency` | integer | No | Check interval in seconds. Default: 60. |
| `request_method` | string | No | HTTP method. Default: GET. |
| `request_timeout` | integer | No | Timeout in seconds. Default: 30. |
| `ssl_check` | boolean | No | Enable SSL monitoring. |

---

### update_monitor

Updates an existing monitor's settings. Only include the fields you want to change.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monitor_id` | integer | Yes | The monitor ID. |
| `name` | string | No | New display name. |
| `url` | string | No | New URL or hostname. |
| `uptime_check_frequency` | integer | No | New check interval in seconds. |
| `request_method` | string | No | HTTP method. |
| `request_timeout` | integer | No | Timeout in seconds. |
| `ssl_check` | boolean | No | Enable/disable SSL monitoring. |

---

### delete_monitor

Permanently deletes a monitor and all associated data.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monitor_id` | integer | Yes | The monitor ID. |

---

### pause_monitor

Pauses a monitor. It will stop being checked until resumed.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monitor_id` | integer | Yes | The monitor ID. |

---

### resume_monitor

Resumes a paused monitor.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monitor_id` | integer | Yes | The monitor ID. |

---

### list_status_pages

Returns all status pages for your account.

*No parameters.*

---

### list_incidents

Returns incidents for a status page, including the latest update for each.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status_page_id` | integer | Yes | The status page ID. |

---

### create_incident

Creates a new incident with an initial status update that appears on the status page.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status_page_id` | integer | Yes | The status page ID. |
| `title` | string | Yes | Incident title. |
| `status` | string | Yes | `investigating`, `identified`, `monitoring`, or `resolved`. |
| `message` | string | Yes | Initial update message. |

---

### update_incident

Posts a new status update to an existing incident.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `incident_id` | integer | Yes | The incident ID. |
| `status` | string | Yes | `investigating`, `identified`, `monitoring`, or `resolved`. |
| `message` | string | Yes | Update message. |
| `title` | string | No | Update the incident title. |

---

### resolve_incident

Marks an incident as resolved with a resolution message.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `incident_id` | integer | Yes | The incident ID. |
| `message` | string | Yes | Resolution message. |

---

### create_maintenance

Schedules a maintenance window on a status page.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status_page_id` | integer | Yes | The status page ID. |
| `name` | string | Yes | Maintenance title. |
| `description` | string | No | Description of the maintenance. |
| `start` | string | Yes | Start time in ISO 8601 format. |
| `end` | string | Yes | End time in ISO 8601 format. |
| `timezone` | string | No | Timezone (e.g., `UTC`, `America/New_York`). Default: `UTC`. |

---

### delete_maintenance

Deletes a scheduled maintenance window.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `maintenance_id` | integer | Yes | The maintenance window ID. |

---

## Troubleshooting

### 401 Unauthorized

- Verify your API key is correct and hasn't expired.
- Make sure you're passing the key in the `Authorization` header directly (no `Bearer` prefix).
- Confirm your account is on a plan with API access enabled.

### "Monitor not found or access denied"

- The monitor ID doesn't exist or belongs to a different account.
- Use `list_monitors` to see your available monitor IDs.

### Tools not showing in your AI client

- Restart the AI client after editing the MCP configuration.
- Check that the URL is exactly `https://api.pulsetic.com/mcp`.
- Test the endpoint with cURL to verify connectivity.

### Getting protocol errors

- Ensure requests use `Content-Type: application/json`.
- The `jsonrpc` field must be exactly `"2.0"`.
- Include an `id` field (integer or string) for all requests except notifications.

---

## API Reference

The MCP server implements the [MCP specification (2025-03-26)](https://modelcontextprotocol.io/specification/2025-03-26) using Streamable HTTP transport over a single `POST /mcp` endpoint with JSON-RPC 2.0.
