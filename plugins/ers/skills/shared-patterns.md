# eRS MCP — shared patterns

Apply on every skill. Do not invent tools, IDs, hours, permissions, or business rules.

## Safety rail

- No delete / bulk-edit / bulk-move / 2+ timesheet decisions / mass create of 2+ without `confirm=true` after the user agrees.
- No Resource/Project screen search to resolve a name on booking, timesheet, requirement, report, or rate flows — pass the name on the acting tool.
- No hours × rate for cost/revenue/profit. `FINANCIAL_ACCESS_DENIED` → stop.
- No org-wide leave/sick/PTO scan. Reply `Sorry, you can't get this data about leaves.` (swap the user's word).
- Admin CREATE/EDIT/DELETE is never done through MCP (`ADMIN_ACTION_NOT_SUPPORTED`). Tags: search + create only; edit/delete in the eRS Admin UI.
- `confirmationRequired=true` + `success=false` means the write was **not** applied. Ask, then recall with the listed field.
- Archive/last-working-date blocked by bookings/timesheets: quote the error and stop. Do not delete those records unless the user explicitly asked.

## Identity

There is no who-am-I tool. Tenant comes from the OAuth token. A listed tool can still 403. HTTP 401 = re-authenticate; HTTP 403 = missing screen/module — do not try another tool as a bypass.

## Resolve names (do not invent IDs)

Pass the human-readable name on the acting tool. MCP: case-insensitive exact name → else exactly one picker row → else validation error with candidates. 0 or 2+ → ask names. Never pick at random.

Collect every detail already in the prompt before asking. Infer only when exactly one interpretation exists against fetched tenant data (including type **names** from `ers_type_get`). Inference never skips `confirm`, force flags, `replaceExistingRate`, `replaceExistingDate`, `linkedBookings`, or `connectedBookings`.

Never ask the user for numeric ids, type ids, timezone, offset, limit, or `idempotencyKey` (omit it; server generates). Never show numeric ids except requirement ids or when asked. USER SEES labels; TOOL SENDS integer codes.

## Dates and clocks

MCP does not parse `"this week"`. Convert to `yyyy-MM-dd`. Search/decision with both dates omitted = **current month**. Reports except `project_progress` require both dates.

Booking without a stated clock: date only or `T00:00:00` — never invent 09:00–17:00. Project task `startTime`/`endTime` must already be UTC `yyyy-MM-dd'T'HH:mm:ss.SSS'Z'`. Requirement: user-stated clocks kept; date-only uses the project calendar. MCP snaps to 15 minutes; tell the user `warnings[]`. Do not retry a time-grid error.

Never assume 8h = 1 FTE. FTE uses the FTE calendar capacity from `ers_calendar_get` `entity=settings`.

## Calls

Two to three calls: schema if required, then the work. Never list-then-get per row. Never fetch-all and sum. Same field on 2+ → bulk tool, not a loop. After `success=true` `complete=true`, do not re-get to verify. Union tools reject unused fields — send only what that action/report lists.

`retry=true` + `retryAfterSeconds`: wait, then the same call. `retry=false`: quote `error` (+ `note`); do not work around.

## Acting screens

| Flow | Lookup |
| --- | --- |
| Booking | Schedule List, then Chart. Names on `ers_booking_*`. |
| Role/UDF then book | Chart `resourceFilters` with option **names**. Resource VIEW not required. Split = Chart only. Bulk-move = List only. |
| Timesheet capture | Entry first, then Approval. Names on `ers_timesheet_*`. Projects on Entry need a resource first. |
| Submit/unsubmit | Timesheet Entry. Approve/reject = Approval. |
| Requirement | Requirement pickers. No resource. Never `ers_rate` for roles. |
| Reports / rates | That report or Financial Rates pickers. `resourceName`/`projectName`/`ownerId`. |
