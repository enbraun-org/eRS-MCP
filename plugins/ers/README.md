# eResource Scheduler — Claude plugin

Claude package for **eResource Scheduler**: manage resources, customize dashboards & track projects.

Catalogued from the repo root via `.claude-plugin/marketplace.json` (`source`: `./plugins/ers`). Bundles the remote eRS MCP connector in `.mcp.json`. Skills live under `skills/` (shared with Cursor).

## Install

Prefer the [root README](../../README.md) for full listing details. Short path:

```bash
claude plugin marketplace add enbraun-org/eRS-MCP
claude plugin install eresource-scheduler@eresource-scheduler
```

Local checkout:

```bash
claude plugin marketplace add ./path/to/eRS-MCP
claude plugin install eresource-scheduler@eresource-scheduler
```

## MCP

```json
{
  "mcpServers": {
    "eResource Scheduler": {
      "type": "http",
      "url": "https://app.eresourcescheduler.cloud/mcp"
    }
  }
}
```

Auth is OAuth 2.0 against eRS. This package points at the eRS **app** MCP endpoint.

## Skills

See [skills/README.md](skills/README.md).

## Links

- [Documentation](https://support.eresourcescheduler.cloud/hc/en-us/articles/62303968313369-How-to-Connect-eRS-to-Your-AI-Assistant)
- [Privacy policy](https://www.eresourcescheduler.com/privacy-policy)
- [Support](https://www.eresourcescheduler.com/contact)
- [Product](https://www.eresourcescheduler.com)

## License

MIT
