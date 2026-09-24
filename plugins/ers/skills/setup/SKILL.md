---
name: setup
description: First-run setup for the eResource Scheduler MCP. Checks the eRS connection, explains what MCP can and cannot do, then routes to the right skill. Use when someone says "connect eRS", "set up eRS MCP", "what can this MCP do", "get started with eResource Scheduler", or "I just connected".
user-invocable: true
allowed-tools: [ers_type_get, ers_calendar_get, ers_tag_search]
---

# eRS MCP — Setup

Guide the user through first-run connection of the eRS MCP, then triage the next skill. **This skill is read-only.** It never writes.

Flow: **Trigger → Connector check → What MCP can do → Triage → Handoff**.

## Input

- Optional: user already authenticated.
- Optional: they named the job they want next (book, timesheet, report).

## Output

- Chat only. No eRS records created.
- Signed-in confirmation when a probe succeeds.
- Next-skill list matched to whether they already have people/projects or are starting empty.

## Knowledge

- Shared patterns: [../shared-patterns.md](../shared-patterns.md).
- Admin writes are blocked even for admin users (`ADMIN_ACTION_NOT_SUPPORTED`).
- There is no current-user tool. Identity is the OAuth session.

## Tools (MCP)

- `ers_type_get` — probe Resource/Project/booking profile VIEW.
- `ers_calendar_get` — calendars / settings (settings = Admin VIEW only).
- `ers_tag_search` — cheap Schedule List VIEW probe.

Do not call create/update/delete tools from this skill.

## Cross-skill handoffs

- **To workspace-builder:** no people/projects yet, or "set up my account from scratch".
- **To daily-briefing:** "what should I look at today".
- **To reports:** totals, availability, financials, gap, forecast.
- **To booking-allocation:** book / demand.
- **To timesheet-workflow:** log / submit / approve time.
- **To bulk-hygiene:** same change on many records, archive, delete.

---

## Step 0: Connector check

**Goal:** Fail fast if eRS MCP is not connected.

1. Call `ers_tag_search` with `{}` (list tags). If that 403s, try `ers_type_get` `entity=resource` (omit id).
2. **401 / tool missing / auth-style error:** print exactly:

   > I don't see the eRS MCP connector active on this session. In your AI client, open MCP settings, add the eRS server, complete OAuth, then run setup again.

   Stop. Do not guess tenant data.
3. **Success:** continue. Do not print numeric ids unless asked.

> **PAUSE:** do not write any `ers_*_create` / `update` / `delete` before a later skill takes over, and never in this skill.

---

## Step 1: What MCP can do

Print a short capability list (not a tool dump):

- People, equipment, rooms, projects, tasks, bookings, timesheets, requirements, tags, rates, reports.
- Reads of types, calendars, holidays, calendar settings (when Admin VIEW exists).

Print a short cannot-do list:

- Admin Configure writes (fields, roles, types, form layouts, calendars, users & rights, account, integrations). Tag **edit/delete** stays in the eRS Admin UI.
- Org-wide who is on leave / sick / PTO (no bulk exception API).

---

## Step 2: Triage

If the user already named a job, skip the menu and hand off.

Otherwise ask once:

> You're connected to eRS. What do you want to do?
> - Brief me on this week (free/busy, over capacity, pending timesheets)
> - Create a person, asset, or project
> - Book someone or record demand
> - Log or approve time
> - Run a report (utilization, availability, financials, gap)
> - Change many records at once / archive / delete

Zero extra questions.

---

## Step 3: Close

One line: `eRS MCP connected. Next: <skill>.`

---

## Shared patterns

See [../shared-patterns.md](../shared-patterns.md). This skill never mutates.

## Error handling reference

| Failure | Behavior |
| --- | --- |
| Connector / 401 | Step 0 stops; print connect instructions. |
| 403 on tag search | Try `ers_type_get entity=resource`; if that also 403s, say which screens are missing; still explain capabilities. |
| `ADMIN_ACTION_NOT_SUPPORTED` | Quote; direct to eRS Admin UI. |
| Leave / PTO question | `Sorry, you can't get this data about leaves.` — do not scan exceptions. |

## Completion criteria

- [ ] Step 0 ran (probe or explicit auth failure).
- [ ] No create/update/delete calls.
- [ ] User was told what MCP cannot do (admin writes, org-wide leave).
- [ ] Handoff named one of the other six skills, or the user stopped.
