# MCP plugin monorepo (template)

Scaffold for **Cursor** + **Claude** plugins and Official **MCP Registry** metadata. No product content on `main` — replace placeholders on feature branches, then PR.

## Layout

```text
.
├── server.json                      # Official MCP Registry (connector listing)
├── mcp.json                         # Cursor MCP wire
├── .cursor-plugin/plugin.json       # Cursor marketplace
├── .claude-plugin/marketplace.json  # Claude catalog → ./plugins/ers
├── plugins/ers/                     # Claude plugin + shared skills
│   ├── .claude-plugin/plugin.json
│   ├── .mcp.json
│   └── skills/
│       ├── example-skill/           # copy-me stub only
│       └── README.md
├── assets/logo.png                  # replace with your logo
├── README.md
├── CHANGELOG.md
└── LICENSE
```

| Platform | This template provides |
| --- | --- |
| Cursor | Manifest + MCP + skills path |
| Claude | Marketplace → nested plugin |
| GitHub Copilot (MCP Registry) | `server.json` only — no Copilot CLI skills yet |

## Branch strategy

1. Keep `main` as template / placeholders.
2. Create a feature branch for real names, URLs, and skills.
3. PR into `main`.

## Before you submit anywhere

Replace every `TODO`, `YOUR_*`, and `example.com` value. Point all MCP URLs at the same production endpoint. Do not commit secrets (`key.pem`).

## License

MIT
