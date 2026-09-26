# Changelog

All notable changes to this repository will be documented here.

## 1.0.0 — Cursor plugin

- Added the `eResource Scheduler` MCP server pointing at `https://test.eresourcescheduler.cloud/mcp`.
- Logo: `assets/logo.png`.
- Skills under `plugins/ers/skills/`: `setup`, `workspace-builder`, `daily-briefing`, `reports`, `booking-allocation`, `timesheet-workflow`, `bulk-hygiene`.
- Cursor manifest loads skills via `"skills": "./plugins/ers/skills"`.

## 1.0.0 — Claude plugin packaging

- Claude marketplace: `.claude-plugin/marketplace.json` → `./plugins/ers`.
- Claude plugin: `plugins/ers/.claude-plugin/plugin.json`.
- Bundled MCP: `plugins/ers/.mcp.json` → `https://test.eresourcescheduler.cloud/mcp` (`eResource Scheduler`).

## 1.0.0 — Official MCP Registry metadata

- Added `server.json` for Official MCP Registry / GitHub Copilot connector listing.
- Name: `cloud.eresourcescheduler/eresource-scheduler`.
- Remote: `https://test.eresourcescheduler.cloud/mcp` (`streamable-http`).
