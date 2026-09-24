# eResource Scheduler (eRS MCP)

Monorepo for the eResource Scheduler AI plugin.

**This branch:** Cursor plugin — connector, logo, and skills. Claude marketplace stubs and `server.json` stay as template until their branches.

## Cursor (this branch)

| File | Role |
| --- | --- |
| `.cursor-plugin/plugin.json` | Cursor marketplace manifest |
| `mcp.json` | eRS MCP (`https://test.eresourcescheduler.cloud/mcp`) |
| `plugins/ers/skills/` | Skills (Cursor loads via `"skills": "./plugins/ers/skills"`) |
| `assets/logo.png` | Plugin logo |

### Install

1. Open **Cursor Settings → Plugins**.
2. Search for **eResource Scheduler**.
3. Click **Install**, then complete the ERS sign-in prompt.

Or run `/add-plugin eresource-scheduler` in chat.

### MCP

```json
{
  "mcpServers": {
    "eResource Scheduler": {
      "type": "http",
      "url": "https://test.eresourcescheduler.cloud/mcp"
    }
  }
}
```

Auth is OAuth 2.0 against ERS.

### Skills

| Skill | What it does |
| --- | --- |
| `setup` | Connect, first run, what MCP can and cannot do |
| `workspace-builder` | Create people, equipment, projects, and tasks from scratch |
| `daily-briefing` | Today’s free/busy, over capacity, and pending timesheets |
| `reports` | Utilization, availability, financials, gap, forecast, progress, timesheet report |
| `booking-allocation` | Book, unfilled demand, split, shift dates |
| `timesheet-workflow` | Log time, copy from schedule, submit/approve |
| `bulk-hygiene` | Same change on many records, archive, mass create, delete |

Shared rules: [plugins/ers/skills/shared-patterns.md](plugins/ers/skills/shared-patterns.md).

### Notes

- Tool calls run as the ERS user who authorizes the connection.
- This branch points at the ERS **test** MCP endpoint.

## Layout

```text
.
├── .cursor-plugin/plugin.json       # Cursor (this branch)
├── mcp.json
├── plugins/ers/skills/              # Skills (this branch)
├── assets/logo.png
├── .claude-plugin/marketplace.json  # Claude — template (other branch)
├── server.json                      # MCP Registry — template (other branch)
└── plugins/ers/.claude-plugin/      # Claude — template (other branch)
```

## Docs

- Product: https://www.eresourcescheduler.com
- Server URL: https://test.eresourcescheduler.cloud/mcp

## License

MIT
