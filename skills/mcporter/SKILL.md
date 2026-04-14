---
name: mcporter
description: Use the mcporter CLI to list, configure, auth, and call MCP servers/tools directly (HTTP or stdio).
homepage: http://mcporter.dev
metadata:
  openclaw:
    emoji: "📦"
---

# mcporter

Quick start
- `mcporter list`
- `mcporter list <server> --schema`
- `mcporter call <server.tool> key=value`

Call tools
- Selector: `mcporter call linear.list_issues team=ENG limit:5`
- Full URL: `mcporter call https://api.example.com/mcp.fetch url:https://example.com`
- JSON payload: `mcporter call <server.tool> --args '{"limit":5}'`

Auth + config
- OAuth: `mcporter auth <server | url> [--reset]`
- Config: `mcporter config list|get|add|remove|import|login|logout`

Notes
- Config default: `./config/mcporter.json` (override with `--config`).
- Prefer `--output json` for machine-readable results.