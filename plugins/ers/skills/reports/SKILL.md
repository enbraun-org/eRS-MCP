---
name: reports
description: Runs one eRS aggregate report — utilization, availability, financials, resource gap, capacity forecast, project progress, or timesheet report. Use when someone says "total hours", "who's free", "who's overloaded", "utilization", "availability report", "planned vs actual", "planned", "actual", "resource gap", "capacity forecast", "project progress", "timesheet report", "cost", "revenue", or "profit". Do NOT use for who is booked (booking-allocation) or who logged which entry (timesheet-workflow) or resource and project detail search.
argument-hint: "[optional: 'planned availability Chris Rose this week']"
user-invocable: true
allowed-tools: [ers_report_get, ers_calendar_get]
---

# Reports

One `ers_report_get` per question. Copy wrap totals. Never page resources/bookings and sum. Never `ers_resource_search` / `ers_project_search`.

Flow: **Trigger → Identify report + measure + scope + period → One call → Copy payload → Print**.

## Input

- Report family + optional named person/project + dates + optional Planned / Actual / Planned vs Actual.



## Output

- Chat tables/lists using **labels** and copied hours/money. Include returned zero rows (`hrs=0`, `pending_request_hrs=0`).
- Copy `report_type` and tell the user which variant ran.



## Knowledge

- Shared patterns: [../shared-patterns.md](../shared-patterns.md).
- Planned allocation ≠ logged actuals ≠ remaining availability ≠ unmet demand.
- User said **"actual"** (hours / logged/ utilization) → Timesheet Report or utilization actuals. Never `report=requirement`.
- Financials never hours × rate.



## Tools (MCP)

- `ers_report_get` — the only data tool in this skill.
- `ers_calendar_get` `entity=settings` — week start / FTE rules when unknown.



## Cross-skill handoffs

- **From daily-briefing:** user wants the full report, not a digest.
- **To booking-allocation:** who is booked on what (entries).
- **To timesheet-workflow:** individual timesheet rows / submit.
- **To setup:** `FINANCIAL_ACCESS_DENIED` or missing report screen.

---



## Step 0: Connector check

401 → connect; stop.

---



## Step 1: Map the question (do not cross)


| User meant                                                             | `report`                                     | Notes                                                   |
| ---------------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------- |
| Busy / capacity / planned hours / overallocated / billable utilization | `utilization`                                | `bookingBillingStatus=billable|non_billable` when asked |
| Remaining / free (not a booking list)                                  | `availability`                               | COPY `availability_hrs` + `availability_percentage`     |
| Cost / revenue / profit                                                | `financial`                                  | Stop on `FINANCIAL_ACCESS_DENIED`                       |
| Resource Gap                                                           | `requirement` `reportType=resource_gap`      | No Resource tab                                         |
| Capacity Forecast                                                      | `requirement` `reportType=capacity_forecast` | Five summary rows; no Requirement row                   |
| Project progress                                                       | `project_progress`                           | No dates (rejected if sent)                             |
| Timesheet **report** / logged totals                                   | `timesheet`                                  | Not `ers_timesheet_search`                              |
| Who is booked                                                          | **Leave this skill** → booking-allocation    |                                                         |
| Who logged which entry                                                 | **Leave this skill** → timesheet-workflow    |                                                         |


Worked example: *"Show me the planned Availability report's total hours for Chris Rose for this week"*


| Slot           | Value                                 |
| -------------- | ------------------------------------- |
| `report`       | `availability`                        |
| `reportType`   | `planned`                             |
| `resourceName` | `Chris Rose`                          |
| dates          | that week as `yyyy-MM-dd`             |
| Answer         | remaining hours + %, not booked hours |


---



## Step 2: Period and type

Convert relative dates (shared-patterns). Reports except `project_progress` **require** `startDate`+`endDate`.

`reportType`:

- Utilization / availability: `planned`  `planned_vs_actual`. Omit when the user did not name a method — MCP picks Planned if available. Copy `report_type`. Do not ASK.
- Financial: `planned`  `actual`  `planned_vs_actual`. Same omit rule.
- Requirement: `resource_gap`  `capacity_forecast`. If `needs_calculation_method`, ASK `available_methods`, then recall.
- Timesheet / project_progress: do not send `reportType`.

Named planned with no Planned VIEW: MCP may serve PVA planned slice. Named actual hours: Timesheet Report, else PVA actuals. No Availability VIEW: remaining from utilization PVA. Copy the **named** variant. Do not implement fallbacks yourself.

---



## Step 3: Scope

- Named person/project: `resourceName` / `projectName`. MCP resolves on that report's pickers.
- Utilization `view=resource` (how busy is Amy) vs `view=project` (hours Apollo consumes). `view` is utilization-only.
- Performing Role → `bookingFilters` `code=role_id`. Never `resourceFilters` `code=roles` (primary role).
- Timesheet report: `includeScheduledHours=true` only when asked (needs Utilization VIEW).
- `dailyBreakdown=true` only for a day grid.
- `deductCapacity=true` requires `projectName` or `projectFilters`. Never subtract hours yourself.
- Organize By: `organizeBy` is grouping, not a filter (except project_progress grouped GET). COPY `organize_by_groups` / `groups.*` **label**. Always keep `hrs=0` rows.
- Availability has no project picker unless `deductCapacity`.
- Resource Gap: `projectName`; no `resourceName`. Forecast: both.

---



## Step 4: Financial extras

Omit `costRateSource` on the first call (Admin applies). Revenue-only: never ask. Cost/profit: ASK only if `needs_cost_rate_source`.

Keys: `actual_cost`/`cost` = work; `actual_bench_cost`/`bench_cost` = unused; `actual_total_cost`/`total_cost` = work+bench. Bare "cost" → ASK Actual vs Bench.

COPY `data.admin` (`currency`, `calculation_method`, `cost_rate_source`, `profit_calculation`).

---



## Step 5: Requirement extras

**Gap:** Pending = requirement − booked, including negatives. COPY `pending_request_hrs`. Never clamp to 0.

**Forecast Overall Summary:** COPY `capacity_hrs`, `scheduled_hrs`, `actual_hrs`, `pending_request_hrs`, `balance_hrs` as five rows. No Requirement row. Never combine Pending and Balance. Omit `pendingRequestFormula` / `balanceFormula` unless the user asked to change them. Role pending: COPY `by_role[].pending_request_hrs` (performing role).

`calculateGapUsingLinkedBookings` is on the data GET only (default true).

---



## Step 6: Print

- COPY wrap totals / `by_role` / `by_project` / groups. Utilization % = `by_project[].percentage` (hours / sum(totalCapacity) × 100), never hours/sum(allHours).
- Role utilization `by_role` is **primary** unless Organize By Performing Role / gap performing roles.
- Never show bare option or project ids in charts/tables — use `label`.

---



## Shared patterns

See [../shared-patterns.md](../shared-patterns.md). Unused report fields are rejected (`WRONG_PARAMS`) — do not send timesheet-only flags on utilization, dates on project_progress, or `view` on financial.

## Error handling reference


| Failure                   | Behavior                                                                         |
| ------------------------- | -------------------------------------------------------------------------------- |
| `FINANCIAL_ACCESS_DENIED` | Stop. Do not reconstruct from rates.                                             |
| 403 named variant         | Let MCP fallback; copy returned `report_type`. Do not hop to Resource search.    |
| `needs_cost_rate_source`  | ASK only for cost/profit; revenue-only recall `costRateSource=0` without asking. |
| Empty `data`              | Say none for that range.                                                         |
| Unknown filter key        | One retry from the valid list.                                                   |




## Completion criteria

- [ ] Exactly one `ers_report_get` family (plus paging while `has_more` on utilization/progress).
- [ ] Planned vs actual vs remaining vs demand not swapped.
- [ ] Named person/project used `resourceName`/`projectName`, not Resource/Project search.
- [ ] Zeros from the payload were not dropped.
- [ ] Financial answers came from `report=financial` or an explicit denial.