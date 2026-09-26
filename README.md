# eResource Scheduler

**Manage resources, customize dashboards & track projects**

Connects [eResource Scheduler](https://www.eresourcescheduler.com) to AI clients via a remote MCP server. After you install and sign in, the client can work with your schedules, resources, projects, bookings, and timesheets using your account permissions.

| | |
| --- | --- |
| **Category** | Productivity |
| **Publisher** | Enbraun Technologies Private Limited |
| **Plugin ID** | `eresource-scheduler` |
| **Display name** | eResource Scheduler |
| **Claude marketplace** | `eresource-scheduler` |
| **MCP Registry name** | `cloud.eresourcescheduler/eresource-scheduler` |

## What you can do

- View and manage resource schedules and availability
- Track projects, bookings, and allocations
- Work with timesheets and related status
- Run guided skills for briefing, reports, bookings, and bulk updates

Actions run as the eRS user who authorizes the connection. Clients only use data that user can access.

## Requirements

- An active eResource Scheduler account
- Ability to complete OAuth sign-in when prompted
- One of: Cursor, Claude (with plugins), or an MCP client that can load Official MCP Registry remotes (e.g. GitHub Copilot)


###  Install — Cursor

1. Open **Cursor Settings → Plugins**.
2. Search for **eResource Scheduler**.
3. Click **Install**, then complete the eRS sign-in prompt.

Or run `/add-plugin eresource-scheduler` in chat.

## Install — Claude

```bash
claude plugin marketplace add enbraun-org/eRS-MCP
claude plugin install eresource-scheduler@eresource-scheduler
```

From a local checkout:

```bash
claude plugin marketplace add ./path/to/eRS-MCP
claude plugin install eresource-scheduler@eresource-scheduler
```

Then enable the plugin and complete the eRS sign-in prompt when Claude connects to the MCP server.

## Install — GitHub Copilot / MCP Registry

Listing metadata lives in `server.json` (Official MCP Registry). Clients that support remote Streamable HTTP MCP (including GitHub Copilot) discover the connector from the registry, then connect to:

`https://app.eresourcescheduler.cloud/mcp`

### Publish (outside this repo)

1. Host `/.well-known/mcp-registry-auth` on `eresourcescheduler.cloud`.
2. `mcp-publisher login http --domain eresourcescheduler.cloud …`
3. `mcp-publisher publish` (from a directory containing this `server.json`).
4. Email `partnerships@github.com` for GitHub MCP Registry curation.

Do not commit `key.pem`.

## MCP

Root Cursor connector (`mcp.json`), Claude bundled connector (`plugins/ers/.mcp.json`), and Registry remote (`server.json`):

```json
{
  "mcpServers": {
    "eresource-scheduler": {
      "type": "http",
      "url": "https://app.eresourcescheduler.cloud/mcp"
    }
  }
}
```

- **Transport:** Streamable HTTP (remote)
- **Auth:** OAuth 2.0 against eResource Scheduler
- **Endpoint:** the eRS **app** MCP URL

## Skills

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

## Example prompts

1. Show the overall utilization for this quarter.
2. Show resources who are overutilized.
3. List all working and non-working exceptions for David this month.
4. List all bookings for Ava starting this week or next week.
5. Henry, Ava, and Lucas have worked 4 hours on Cobalt Mobile App every day this week. Enter and submit their timesheets.

## Privacy, docs, and support

- [Documentation — Connect eRS to your AI assistant](https://support.eresourcescheduler.cloud/hc/en-us/articles/62303968313369-How-to-Connect-eRS-to-Your-AI-Assistant)
- [Privacy policy](https://www.eresourcescheduler.com/privacy-policy)
- [Contact support](https://www.eresourcescheduler.com/contact)
- Product: https://www.eresourcescheduler.com
- Support email: support@enbraun.com

## Repository layout

```text
.
├── .cursor-plugin/plugin.json               # Cursor marketplace manifest
├── mcp.json                                 # Cursor MCP connector
├── .claude-plugin/marketplace.json          # Claude marketplace catalog
├── server.json                              # Official MCP Registry / Copilot
├── assets/logo.png                          # Plugin logo
└── plugins/ers/
    ├── .claude-plugin/plugin.json           # Claude plugin manifest
    ├── .mcp.json                            # Claude MCP connector
    └── skills/                              # Shared agent skills
```

## License

MIT — see [LICENSE](LICENSE).
