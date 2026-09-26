---
name: daily-briefing
description: Daily eRS capacity briefing. Summarizes who is over capacity, remaining availability, and timesheet entries waiting for approval for a date range. Use when someone says "morning briefing", "what should I look at today", "who's overassigned", "who's free this week", "pending timesheets", "pipeline for my team this week", or "what's hot in the schedule".
argument-hint: "[optional: period, e.g. 'this week' or '2026-09-14 to 2026-09-20']"
user-invocable: true
allowed-tools: [ers_report_get, ers_type_get, ers_calendar_get, ers_timesheet_search]
---

# Daily Briefing

Read-only digest of schedule load and timesheet backlog for the signed-in tenant. Prints in chat (eRS has no briefing-doc tool). Does not change bookings or timesheets unless the user explicitly hands off to timesheet-workflow / bulk-hygiene.

Flow: **Trigger → Period → Over capacity → Availability → Pending timesheets → Print**.

## Input

- Optional: period (`this week`, named dates). Default = **current week**.
- Optional: grain — people (default) vs team / location / department / role for over-capacity.

## Output

- **α (default):** Markdown briefing in chat. See [sample-output.md](sample-output.md).
- **β (opt-in):** Only if the user then says submit/approve — hand off to `timesheet-workflow`. Do not decide statuses inside this skill.

## Knowledge

- Shared patterns: [../shared-patterns.md](../shared-patterns.md).
- Over capacity = utilization load vs capacity, not a booking list. Availability total = remaining hours (`availability_hrs`), not booked hours.
- Grouped over-capacity: do **not** send `organizeBy` on utilization. Use `resourceFilters` with option **names** (see Step 3).

## Tools (MCP)

- `ers_report_get` `report=utilization` — over capacity.
- `ers_report_get` `report=availability` — remaining hours.
- `ers_type_get` `entity=resource` — UDF/role codes + option names for grouped grain.
- `ers_calendar_get` `entity=settings` — `starting_day_of_week` when week bounds matter and are unknown.
- `ers_timesheet_search` — pending entries (`entry_status` Submitted = 2). Never for totals.

## Cross-skill handoffs

- **To reports:** user wants full utilization/availability/financial/gap, not a digest.
- **To timesheet-workflow:** submit/approve the pending list.
- **To booking-allocation:** free someone or fill a gap.
- **To bulk-hygiene:** shift many bookings.

---

## Step 0: Connector check

401 / tools missing → connect instructions; stop. Do not invent hours.

> **PAUSE:** no mutations in this skill.

---

## Step 1: Period

Convert to `startDate`/`endDate` `yyyy-MM-dd`. MCP does not parse `"this week"`.

- Named range → use it.
- Else current week. If `starting_day_of_week` is in chat, use it. Else one `ers_calendar_get` `entity=settings` (Admin VIEW). If settings 403, ASK week start or state the Monday–Sunday range you will use and wait.

---

## Step 2: Grain for over-capacity

Scan the message **before** defaulting to people.

| Message has | Grain |
| --- | --- |
| team / teams | Team |
| location / locations | Location |
| department / departments | Department |
| role / roles | Role (primary) |
| none of the above | people |

"Which team is overassigned" is Team — do not list people. Named value is a filter ("teams in London" → grain Team, filter London).

---

## Step 3: Over capacity (`report=utilization`)

One call:

- `view=resource`
- `startDate` + `endDate`
- `limit=500`; page while `has_more`
- User said bookings/planned/allocated only → `reportType=planned`, `data=planned,capacity`
- Timesheets/actuals/logged, **or nothing** → `reportType=planned_vs_actual`, `data=planned,actual,capacity` (do not also call `planned`)
- If `planned_vs_actual` fails, one `planned` call and say actuals were unavailable

**People:** no `resourceFilters`. Compare `total_planned_hrs` / `total_actual_hrs` to `total_capacity_hrs`. Keep overage **> 0.25h**.

**Team / location / department / role:** `ers_type_get` `entity=resource`. Match grain to `display_name` → code (`udf_team`, `udf_location`, `udf_department`, `roles`). `resourceFilters` `{ "code": "<code>", "values": [<option names>] }`. Never send `{Field} Undefined` in `values`. Never `organizeBy`. Never `ers_resource_search`.

COPY hours from the payload. Do not sum bookings.

---

## Step 4: Availability (`report=availability`)

Same dates. Named person → `resourceName`. COPY `availability_hrs` **and** `availability_percentage` (remaining = capacity − booked). Never present `totalUtilization` / planned booked hours as the availability total.

Omit `reportType` unless the user named Planned or Planned vs Actual — MCP picks Planned if available. Copy `report_type`.

Skip this strand if Availability (and utilization PVA fallback) is 403; note it under hygiene.

---

## Step 5: Pending timesheets

`ers_timesheet_search` with `filters` `{"entry_status:eq":2}` + the same dates (or omit dates = current month — do not omit if Step 1 fixed a week). Cap the printed list at 15; then `+n more`. Present status **labels**, never `1/2/4/8`.

Do not call `ers_timesheet_decision` here.

---

## Step 6: Print α

Markdown in chat. No code fence around the tables. No numeric ids unless asked.

```markdown
# eRS briefing — <start> to <end>

Basis: <bookings | timesheets | bookings and timesheets>

## Over capacity
- <n> <people|teams|…>
| <Name or Team> | Capacity | Load | Over |
|---|---:|---:|---:|
| … | …h | …h | …h (…%) |

## Remaining availability
- Highlight: <name> <availability_hrs>h (<availability_percentage>)

## Pending timesheets
- <n> submitted, waiting
| Person | Project | Date | Hours |
|---|---|---|---:|
| … | … | … | … |

## Hygiene
- Strands skipped (403 / empty) listed in one line
```

Empty over-capacity: `No overassigned <grain> in <start> to <end>.`

Sort most over capacity first. Cap 25 rows; `+n more`.

---

## Step 7: Close

`Briefed <start> to <end>. <n> over capacity · <n> pending timesheets.`

If the user wants those timesheets submitted/approved: hand off to timesheet-workflow.

---

## Shared patterns

See [../shared-patterns.md](../shared-patterns.md). Include zero-hour groups when the report returned them for role %; for **this** briefing, print only over-capacity rows in the over-capacity table.

## Error handling reference

| Failure | Behavior |
| --- | --- |
| 401 | Stop; connect. |
| Utilization 403 | Skip over-capacity strand; still try availability + timesheets. |
| Availability 403 | MCP may serve remaining from utilization PVA; if that also fails, skip strand. |
| `WRONG_PARAMS` / Undefined in values | Drop Undefined; recall **once**. Do not send `organizeBy`. |
| Timesheet search 403 | Skip pending strand; note Entry/Approval missing. |
| Empty results | Say none; do not invent rows. |

## Completion criteria

- [ ] Dates were `yyyy-MM-dd`, not the phrase "this week".
- [ ] Over-capacity used `ers_report_get` utilization, not booking search.
- [ ] Grouped grain did not print a people table.
- [ ] Availability used remaining hours, not booked hours.
- [ ] No `ers_timesheet_decision` unless the user left this skill.
- [ ] Chat output matches the briefing shape (or a named skipped strand).
