---
name: bulk-hygiene
description: Applies the same eRS change to many records — bulk-edit, mass create, archive/restore, bulk-move handoff, and deletes — with explicit confirm. Use when someone says "all resources", "every project", "bulk edit", "archive these", "delete all", "same note on everyone", "mass book", or "clean up old projects". Different from booking-allocation (one booking) and timesheet-workflow (status uses ers_timesheet_decision there).
argument-hint: "[optional: entity + change, e.g. 'archive inactive projects']"
user-invocable: true
allowed-tools: [ers_type_get, ers_resource_search, ers_resource_update, ers_resource_bulk_edit, ers_resource_delete, ers_manage_resource_record, ers_project_search, ers_project_update, ers_project_bulk_edit, ers_project_delete, ers_manage_project_record, ers_booking_search, ers_booking_bulk_edit, ers_booking_bulk_move, ers_booking_delete, ers_booking_create, ers_requirement_search, ers_requirement_bulk_edit, ers_requirement_create, ers_requirement_delete, ers_timesheet_search, ers_timesheet_decision, ers_timesheet_delete, ers_rate, ers_tag_search]
---

# Bulk Data Hygiene

Audit a set, let the user pick the change, then run **one** native bulk/mass call. The "do" sibling to daily-briefing's "report".

Flow: **Trigger → Resolve set → Plan → Confirm → One native write → Summary**.

## Input

- Entity (resources, projects, bookings, requirements, timesheets, notes, rates).
- The change (field, archive, delete, mass create, note text).

## Output

- **α:** One bulk/mass/delete operation after `confirm=true`.
- Chat summary of what changed from the tool result. Never claim success on `confirmationRequired`.

## Knowledge

- Shared patterns: [../shared-patterns.md](../shared-patterns.md).
- Same field on 2+ → `ers_*_bulk_edit`, not a loop of update.
- Same note on 2+/all resources: page `ers_resource_search` then **one** `ers_manage_resource_record` `entity=note` `resourceIds` + `content`. FORBIDDEN: N creates. Same for projects with `projectIds`.
- Mass book / mass requirement: one `ers_booking_create` / `ers_requirement_create` with filters or collective phrase (booking-allocation has the field rules). `confirm=true` when 2+.
- Hide project = `is_archive`. Permanent delete = `ers_project_delete`. Archive resource = `last_date=setArchive` (yesterday, never today). Restore `last_date=null`. Read resource archive with `$archive_res`.

## Tools (MCP)

Bulk-edit / delete / bulk-move / note-create-many / mass-create / `ers_timesheet_decision` (2+) / `ers_rate` delete. Profile via `ers_type_get` before bulk-edit of booking/timesheet/requirement (and resource/project when fields unknown).

## Cross-skill handoffs

- **From daily-briefing / reports:** user wants the set changed.
- **To booking-allocation:** single booking create/split (not bulk).
- **To timesheet-workflow:** single-day log; status bulk stays here only if they asked "submit all of these" — prefer timesheet-workflow; this skill may call `ers_timesheet_decision` when the set is already resolved.

---

## Step 0: Connector + safety class

401 → stop.

| User said | Class |
| --- | --- |
| How many / list | Informational — search only. No `confirm`. |
| Set field / archive / mass create | Mutation — plan + confirm. |
| Delete / bulk-move / 2+ decisions | Destructive — `confirm=true` after explicit yes. |

Never convert a list question into a delete. Never delete related bookings to unblock archive unless the user asked.

> **PAUSE:** no write before Step 4.

---

## Step 1: Resolve the set

Search with server-side filters. Page until `has_more` is false when collecting ids for notes/bulk. Never invent ids.

- Active projects: `{"is_archive":false}`. Active resources: `{"$archive_res":false}`.
- Ids N to M: one search `{"id:bt":[N,M]}` — never loop get.
- Booking/timesheet names: pass names on those search tools (acting screen).

If 0 rows → stop, say none. If the user said "all" and the set is large, print the **count** from `total_count` and confirm that count.

---

## Step 2: Schema

Bulk-edit booking/requirement/timesheet: `ers_type_get` that entity unless `fields[]` in chat. Resource/project: reuse type fields or fetch once. Option values = option **ids** (except Chart booking `resourceFilters` names on mass create).

---

## Step 3: Plan

Build one native call:

| Change | Call |
| --- | --- |
| One attribute, 2+ resources/projects/bookings/requirements | `ers_*_bulk_edit` `confirm=true` (one attribute per call; requirement `"40 hours"`/`"1 FTE"` may set unit+effort) |
| Same note, 2+ resources/projects | one note create with `resourceIds`/`projectIds` |
| Same comment, 2+ requirements | `ers_requirement_bulk_edit` `code=comments` |
| Postpone/prepone/extend | `ers_booking_bulk_move` `confirm=true` |
| Mass book / mass demand | one create + filters + `confirm` when 2+ |
| Archive resources | `code=last_date` `value=setArchive` or single `ers_resource_update` |
| Hide projects | `ers_project_update` / bulk `is_archive` |
| Delete 2+ tasks | one `ers_manage_project_record` `entity=task` `action=delete` `ids` or `deleteAllTasks=true` |
| Delete record | matching `*_delete` `confirm=true` |
| 2+ timesheet statuses | one `ers_timesheet_decision` `confirm=true` |
| Rate delete | `ers_rate` `action=delete` `confirm=true` |

Print the plan: count, field, new value, confirm question. Empty pick → summary-only, no write.

---

## Step 4: Confirm and execute

Default: one batched yes. Force flags (`forceDeleteBookings`, `forceDeleteTimesheetEntries`, `replaceExistingRate`, `replaceExistingDate`) only after a **second** yes when the backend asked.

Resource delete two-step: first `confirm=true` no force flags; if `confirmationRequired`, tell the user related records will be permanently deleted; only then force flags. Never delete those related rows with other tools.

`confirmationRequired` on mass create → ASK, recall `confirm=true`. Write was not applied until that recall succeeds.

---

## Step 5: Archive failures

If archive / last-working-date fails because bookings or timesheets exist: quote the backend message (+ `note`). Stop. `"Delete … and try again"` is for the user.

---

## Step 6: Close

`Updated <n> <entity>.` or quote failure. Partial mass: report returned counts only.

---

## Shared patterns

See [../shared-patterns.md](../shared-patterns.md). Exception list is one `resourceId`/`projectId` only — never bulk exception read; never sequential fan-out for leave.

## Error handling reference

| Failure | Behavior |
| --- | --- |
| Missing `confirm=true` | Ask the user; do not retry silently with confirm. |
| `confirmationRequired` | Not applied; ASK listed options. |
| 409 start/last date vs timesheets | ASK then `forceDeleteTimesheetEntries` only if they agree. |
| Loop of creates detected | Stop; switch to bulk/mass tool. |
| `LEAVE_NOT_AVAILABLE` / bulk exception | Quote; do not scan owners. |
| Admin write | `ADMIN_ACTION_NOT_SUPPORTED`; eRS Admin UI. |

## Completion criteria

- [ ] Informational questions did not mutate.
- [ ] Set resolved via search filters, not invented ids.
- [ ] One native bulk/mass/delete call (not a per-id loop).
- [ ] `confirm=true` only after the user agreed when required.
- [ ] Force flags only after a distinct agreement.
- [ ] Archive-blocked related records were not deleted unless asked.
- [ ] Tool `success=true` (and not `confirmationRequired`) before claiming done.
