# Daily Briefing — Sample Output

Example chat output for `daily-briefing` on week 2026-09-14 to 2026-09-20 (Monday start). Illustrative numbers only — never copy these hours into a live answer.

---

# eRS briefing — 2026-09-14 to 2026-09-20

Basis: bookings and timesheets
Grouped by: people

## Over capacity

- Overassigned: 2

| Name | Capacity | Booked | Over |
|---|---:|---:|---:|
| Chris Rose | 40h | 52h | 12h (130%) |
| MCP QA Person | 40h | 44.5h | 4.5h (111%) |

## Remaining availability

| Name | Remaining | Remaining % |
|---|---:|---:|
| Alex Kumar | 16h | 40% |
| Chris Rose | 0h | 0% |

## Pending timesheets

- 3 submitted, waiting

| Person | Project | Date | Hours |
|---|---|---|---:|
| Chris Rose | Apollo | 2026-09-15 | 8h |
| Chris Rose | Apollo | 2026-09-16 | 8h |
| MCP QA Person | Mobile App | 2026-09-17 | 6h |

## Hygiene

- None skipped

---

## Run metadata (for authors)

| Field | Value |
| --- | --- |
| Utilization | `report=utilization` `reportType=planned_vs_actual` `view=resource` |
| Availability | `report=availability` COPY `availability_hrs` + `availability_percentage` |
| Pending | `ers_timesheet_search` `entry_status` Submitted |
| Grain | people (no team/location/department/role in the prompt) |
