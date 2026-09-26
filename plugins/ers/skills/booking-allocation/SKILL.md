---
name: booking-allocation
description: Book people or assets onto projects, record unmet demand (requirements), split or shift bookings. Use when someone says "book", "allocate", "schedule X on Y", "create a requirement", "staff a role", "split this booking", "postpone/prepone/extend bookings", or "who is booked". Do NOT use for logging time (timesheet-workflow) or utilization totals (reports).
argument-hint: "[optional: person, project, dates]"
user-invocable: true
allowed-tools: [ers_type_get, ers_booking_search, ers_booking_get, ers_booking_create, ers_booking_update, ers_booking_bulk_edit, ers_booking_bulk_move, ers_booking_delete, ers_requirement_search, ers_requirement_get, ers_requirement_create, ers_requirement_update, ers_requirement_bulk_edit, ers_requirement_delete, ers_manage_requirement_record]
---

# Booking and Allocation

Bookings assign a resource to a project. Requirements are unmet demand (role + effort, no person). Who-is-booked is `ers_booking_search`, not a utilization report.

Flow: **Trigger → Profile → Resolve on Schedule/Requirement → Confirm writes → Native operation**.

## Input

- Person and/or project names, dates, unit/effort, or a demand (role + hours/FTE).
- Optional: split date, postpone/prepone/extend, requirement conditions.

## Output

- Created/updated booking or requirement, or a booking search list.
- Chat uses `effort_display` / labels, not raw unit codes.

## Knowledge

- Shared patterns: [../shared-patterns.md](../shared-patterns.md).
- Schedule List first, then Chart. Split = Chart only. Bulk-move = List only.
- Book against a requirement: `fields.requirement_id` on Schedule pickers — never `ers_requirement_get`.
- "Create a task" is workspace-builder (`entity=task`), not a requirement.

## Tools (MCP)

- `ers_type_get` `entity=booking|requirement` — **mandatory** before create/update/bulk_edit unless that profile's `fields[]` already returned in this chat.
- `ers_booking_*` — search/get/create/update/bulk_edit/bulk_move/delete.
- `ers_requirement_*` — demand CRUD. No resource on a requirement.
- `ers_manage_requirement_record` — comments. Same comment on 2+ → `ers_requirement_bulk_edit` `code=comments`.

Never `ers_resource_search` / `ers_project_search` for a known person/project title. Never `ers_rate` as a role catalog.

## Cross-skill handoffs

- **To reports:** totals / free / gap.
- **To timesheet-workflow:** copy scheduled hours into timesheets (`fromSchedule=true` there, not booking search).
- **To bulk-hygiene:** same attribute on 2+ bookings already in that skill; postpone lives **here** (`ers_booking_bulk_move`).
- **To workspace-builder:** person or project does not exist yet.

---

## Step 0: Connector + classify

401 → stop.

| Intent | Path |
| --- | --- |
| Who is booked | Search only (Step 6). No confirm. |
| Book named person on a project | Create booking |
| Book all/active/first-N / role / UDF (no person name) | Mass create |
| Unfilled role + hours/FTE | Requirement create |
| Split | `ers_booking_update` `splitOn` |
| Postpone / prepone / extend dates | `ers_booking_bulk_move` |
| Same field on 2+ bookings | `ers_booking_bulk_edit` |
| Delete a booking | `confirm=true` (Skill bulk-hygiene also covers delete; stay here if they named one booking) |

---

## Step 1: Profile

Booking or requirement **write**: `ers_type_get` `entity=booking` or `entity=requirement` (omit id) unless `fields[]` in this chat. Collect prompt values. ASK remaining `is_required` by `display_name` and WAIT. Do not call create to discover required fields.

---

## Step 2: Booking create

Always send `fields.unit` (`1` Capacity %, `2` Total Booking Hours, `4` FTE) and `fields.effort`. ASK both if omitted; never invent; same on List and Chart.

- Named person/project on `resource_id` / `project_id` (names OK).
- Against requirement: `fields.requirement_id` (id or Schedule substring e.g. `ID 14`). Omit clocks to use the requirement range. Never Requirement screen tools.
- No clock: date only or `T00:00:00`. Stated clocks `yyyy-MM-dd'T'HH:mm:ss`; end after start. Tell user `warnings[]` snaps.
- Recurring: `recurPattern` (`repeat` 1–5). Never a create loop.
- Mass / role / UDF: `resourceFilters` e.g. `{"$archive_res":false}`, `{"roles:any":["BA"]}`, `{"udf_location:any":["London"]}` (option **names**) or `resourceIds` or collective `resource_id` `"all active resources"`. `confirm=true` when 2+. If `confirmationRequired`, ASK then recall. Chart VIEW required for UDF/role expand. FORBIDDEN: claim Resource/UDF inaccessible; ask for a person name instead of `resourceFilters`.

---

## Step 3: Booking change

- One booking fields: `ers_booking_update` (profile first). Connected series: `connectedBookings=this|future|all` if the user chose; else ASK on `confirmationRequired`.
- Split: `id` + `splitOn` `yyyy-MM-dd`, **no** `fields`. Inclusive last day of first segment. Never update-then-create.
- Dates on 1+: `ers_booking_bulk_move` `bookingIds` from search + `confirm=true`. Infer postpone→`MOVEFORWARD`, prepone→`MOVEBACKWARD`, extend-by-N-days→`EXTENDDAYS` when unambiguous. ASK if shift vs extend is unclear.
- Same non-date field on 2+: `ers_booking_bulk_edit` `confirm=true`.

---

## Step 4: Requirement create

Project name on `project_id`. Dates `yyyy-MM-dd'T'HH:mm:ss`. User-stated clocks including `00:00` kept; date-only omit `T` (project calendar). Unit `2` Hours or `4` FTE only (never Capacity %). Omit unit → MCP sends Hours. "N hours"/"N FTE" → set **unit and effort**. `role_id` name, omit, or `"any"` (unset). Stated Conditions → `conditions[]` (names). Never `ers_type_get entity=resource` for Conditions.

Mass: `projectFilters` / `projectIds` / `"first 100 projects"` / `"all active projects"` + `confirm=true` when 2+. Project VIEW not required (Chart fallback). FORBIDDEN: invent Project/UDF inaccessibility; loop one create per project.

---

## Step 5: Assigned requirement

Update/delete with linked bookings: `linkedBookings=unlink|delete|update` when the user said so. Never `ers_booking_update` to clear `requirement_id`. If `confirmationRequired`, ASK then recall. Delete needs `confirm=true`.

---

## Step 6: Search (informational)

`ers_booking_search` `filters` + `startDate`/`endDate` (both or neither; omit = current month). Names on resource/project filters. Present labels.

---

## Step 7: Close

Quote `success`. If `confirmationRequired`, do not claim booked.

---

## Shared patterns

See [../shared-patterns.md](../shared-patterns.md). One native operation — no create loops.

## Error handling reference

| Failure | Behavior |
| --- | --- |
| Multiple name matches | Ask candidates; do not invent an id. |
| No Schedule List or Chart | Quote `Schedule List or Schedule Chart access is required`. |
| Split without Chart | Quote split-requires-Chart. |
| Bulk-move without List | Quote List-only. |
| Missing unit/effort | ASK; never default. |
| `confirmationRequired` | Write not applied; ASK; recall. |

## Completion criteria

- [ ] Profile fetched (or reused) before booking/requirement writes.
- [ ] Booking create included integer `unit` + `effort` (or user was asked).
- [ ] Names resolved on Schedule/Requirement tools, not Resource/Project search.
- [ ] Mass/UDF used one create + `confirm` when 2+, not a loop.
- [ ] Destructive/date-shift used `confirm=true` after user agreement.
- [ ] Search-only questions did not create bookings.
