---
name: timesheet-workflow
description: Log actual time, copy scheduled hours into timesheets, comment on entries, and submit/unsubmit/approve/reject. Use when someone says "log time", "timesheet", "copy scheduled hours", "submit my timesheet", "approve timesheets", "pending approvals", or "who logged time". Do NOT use for planned bookings (booking-allocation) or timesheet/utilization **reports** (reports).
argument-hint: "[optional: person + dates, e.g. 'Chris Rose this week']"
user-invocable: true
allowed-tools: [ers_type_get, ers_timesheet_search, ers_timesheet_get, ers_timesheet_create, ers_timesheet_update, ers_timesheet_delete, ers_manage_timesheet_record, ers_timesheet_decision]
---

# Timesheet Workflow

Timesheet entries are **actual** logged time. Capture (Timesheet Entry) first; Approval only when Entry VIEW is missing or the decision is approve/reject.

Flow: **Trigger → Profile → Capture or search → Decision (opt-in)**.

## Input

- Person / project names, hours or clocks, date or range, optional comment, optional submit/approve.

## Output

- Created/updated entries with `hours_display` / status **labels** (Draft, Submitted, Approved, Rejected — never `1/2/4/8`).
- Search lists for "who logged" / pending.

## Knowledge

- Shared patterns: [../shared-patterns.md](../shared-patterns.md).
- Copy scheduled hours = `ers_timesheet_create` `fromSchedule=true` (Timesheet Entry utilization). Never `ers_booking_search`.
- Comment on **create** = `comment` / `fields.comments` on the same POST. Comment on an **existing** entry = `ers_manage_timesheet_record`.
- Submit/unsubmit = Entry (`1,8→2` and `2,8→1`). Approve/reject = Approval (`2,8→4` and `2,4→8`).

## Tools (MCP)

- `ers_type_get` `entity=timesheet` — **mandatory** before create/update unless `fields[]` in this chat.
- `ers_timesheet_search` / `get` / `create` / `update` / `delete`.
- `ers_manage_timesheet_record` `entity=note`.
- `ers_timesheet_decision` — one call for many ids; never a loop.

Never `ers_resource_search` / `ers_project_search`. List people to log time: omit `resource_id` (Entry `/resources-lookup`). Project names on Entry need a resource first. No all-projects lookup.

## Cross-skill handoffs

- **From daily-briefing:** pending list → decision here.
- **From booking-allocation:** copy schedule → `fromSchedule=true` here.
- **To reports:** timesheet **report** totals.
- **To bulk-hygiene:** not for status — status is this skill's `ers_timesheet_decision`.

---

## Step 0: Connector check

401 → stop. 403 with no Entry and no Approval → quote that timesheet access is required.

> **PAUSE:** no `ers_timesheet_decision` until the user asked to submit/unsubmit/approve/reject.

---

## Step 1: Profile (writes)

`ers_type_get` `entity=timesheet` (omit id) unless fields already in this chat. Collect from the prompt. ASK remaining `is_required` by `display_name`. WAIT.

---

## Step 2: Create

| Shape | Call |
| --- | --- |
| One day | `fields.date` + `hours` **or** `time_start`+`time_end` (never both) |
| Uniform range | `fields` + `startDate`≠`endDate` + exactly one of `hoursPerDay`, `totalHours`, `timeStart`+`timeEnd` |
| Copy scheduled | `fromSchedule=true` + resource name/id + `startDate`/`endDate`. COPY `dailyUtilization` cells; omit `bookingId` on create. `capacity` in that payload is calendar capacity — use when asked; do **not** copy capacity hours as timesheet entries |
| + comment | same create |

Never loop single-day creates for a uniform range. MCP snaps to 15 min; tell the user `warnings[]`.

---

## Step 3: Edit / comment / delete

- Hours/fields: `ers_timesheet_update` (profile first). Resolve id via search.
- Comment on existing: `ers_manage_timesheet_record` list then create/update. Delete note: `confirm=true`.
- Delete entry: `ers_timesheet_delete` `confirm=true` only if the user asked to delete.

---

## Step 4: Search (informational)

`ers_timesheet_search` + dates (both or neither; omit = current month). Pending: `{"entry_status:eq":2}`. Names on resource/project filters.

Do not use search for the timesheet **report** or billable utilization (reports skill).

---

## Step 5: Decision

One `ers_timesheet_decision`:

- `decision=submit\|unsubmit\|approve\|reject`
- `ids` from search **or** `filters` + dates (MCP collects; omit dates = current month)
- 2+ entries: `confirm=true`
- Optional `comment`

Eligible from: submit Draft/Rejected; unsubmit Submitted/Rejected; approve Submitted/Rejected; reject Submitted/Approved. Never a per-id loop. Max 100 ids.

---

## Step 6: Close

Report labels and hours from the response. `success=false` → quote; do not claim submitted.

---

## Shared patterns

See [../shared-patterns.md](../shared-patterns.md). Confirm 2+ decisions and deletes; do not confirm a search.

## Error handling reference

| Failure | Behavior |
| --- | --- |
| Capture CREATE 403 but VIEW exists | Do not fall through to Approval to create. Quote Entry CREATE required. |
| Approve without Approval screen | Quote Approval required. |
| Project name without resource on Entry | Resolve resource first. |
| Range missing effort unit | ASK exactly one of hoursPerDay / totalHours / clocks. |
| Multiple name matches | Ask; do not invent ids. |

## Completion criteria

- [ ] Profile fetched (or reused) before create/update.
- [ ] Copy-from-schedule did not use `ers_booking_search`.
- [ ] Range used one create, not a day loop.
- [ ] Status changes used one `ers_timesheet_decision`.
- [ ] 2+ decisions / deletes had user `confirm=true`.
- [ ] Status presented as labels.
