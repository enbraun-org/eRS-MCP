---
name: workspace-builder
description: Create eRS people, equipment, rooms, projects, and project tasks from a plain-language description. Use ONLY when someone says "create a resource", "add a person", "new project", "create a task", "add equipment", "set up my first project", or "I have no resources yet". Do NOT use for booking (booking-allocation), logging time (timesheet-workflow), or reports.
argument-hint: "[optional: 'person Jane Doe' or 'project Apollo']"
user-invocable: true
allowed-tools: [ers_type_get, ers_calendar_get, ers_resource_search, ers_resource_create, ers_resource_update, ers_project_search, ers_project_create, ers_project_update, ers_manage_project_record, ers_manage_resource_record, ers_tag_search, ers_tag_create]
---

# Workspace Builder

Turns "add a laptop" / "create project Apollo" into typed eRS records. Replaces guessing Person vs Equipment vs Building.

Flow: **Trigger → Schema → Infer type → Ask remaining required → Confirm → Create (α) → Optional sub-records (β)**.

## Input

- Optional: name + kind in the argument ("Dell Laptop", "project Apollo").
- Optional: fields the user already stated (role, dates, title).

## Output

- **α (default):** One resource or one project created. Chat names the record (no numeric id unless asked).
- **β (opt-in):** Project task / phase / milestone, or resource timing, only if the user asked in the same request.

## Knowledge

- Shared patterns: [../shared-patterns.md](../shared-patterns.md).
- Required fields differ by **tenant type**. Never hardcode a Person/Equipment/Building table.
- "Create a task" = `ers_manage_project_record` `entity=task` (Project CREATE). Never `ers_requirement_create`.

## Tools (MCP)

- `ers_type_get` — list types (omit id) then `fields[]` for one type id.
- `ers_resource_create` / `ers_project_create` — writes after schema.
- `ers_resource_search` / `ers_project_search` — collision check by name/title.
- `ers_manage_project_record` — tasks (clocks already UTC with `Z`).
- `ers_tag_search` / `ers_tag_create` — reuse a tag label before creating.
- `ers_calendar_get` — `calendarId` before resource timings.

## Cross-skill handoffs

- **From setup:** first records in an empty account.
- **To booking-allocation:** after a person and project exist.
- **To timesheet-workflow:** logging time on an existing person.
- **To bulk-hygiene:** same field on 2+ resources/projects.

---

## Step 0: Connector + intent

If MCP is missing / 401: same connect message as setup; stop.

Classify: **resource** (person, equipment, room, asset) vs **project** vs **project task**. Ambiguous → ask those three labels only.

> **PAUSE:** no create until Step 5 confirm.

---

## Step 1: Collision check

- Resource: `ers_resource_search` `filters` `{"name:has":"<name>"}`.
- Project: `ers_project_search` `{"title:has":"<title>"}`.

One exact match → stop and offer update (`ers_*_update`) instead of a duplicate. Multiple → ask which name.

---

## Step 2: Load types

Reuse type list/`fields[]` already in this chat. Else:

1. `ers_type_get` `entity=resource` or `entity=project` (omit id).
2. Match user words to that tenant's type **names**. Exactly one match (e.g. laptop ↔ Equipment) → use it. A person name ("Jane") is not a type → ask type **names** only. 0 or 2+ → ask names. Never ask for type id.

---

## Step 3: Load fields

If that type's `fields[]` are not in chat: `ers_type_get` with that type `id`. Collect every value already in the prompt. ASK only `is_required=true` by `display_name`. Project `title` is always required.

Option fields: write `options[].id`, never the label. Roles on a resource: `fields.roles` as an integer id **list** (`[6]`, never `role` / `role_id`). Photo: `fields.image` http(s) or data URI — never invent URLs.

---

## Step 4: Confirm

Print one plan:

> I'll create `<name>` **`<type>`** with: `<fields>`. Proceed? (yes / change type / no)

No writes until yes.

---

## Step 5: Create α

- Resource: `ers_resource_create` `resourceTypeId` + `fields` by code.
- Project: `ers_project_create` `projectTypeId` + `fields` by code.

Do not re-fetch types on later creates in this chat unless the user asks for types again. Report labels from the response. `success=false` → quote error; do not claim created.

---

## Step 6: Optional β sub-records

Only if the user asked:

- Task: `ers_manage_project_record` `entity=task` `action=create`. Convert local clocks to UTC `.000Z` first. IST 09:00 → `03:30:00.000Z`. Never local wall-clock without `Z`.
- Timing: `ers_manage_resource_record` `entity=timing` after `ers_calendar_get` for `calendarId`.

Never create a requirement here.

---

## Step 7: Close

`Created <type> (<name>).`

---

## Shared patterns

See [../shared-patterns.md](../shared-patterns.md). Confirm in one batch, not per field after the schema ask.

## Error handling reference

| Failure | Behavior |
| --- | --- |
| 401 | Stop; connect instructions. |
| 403 Resource/Project | Quote; do not invent a Chart workaround for **creating** a resource/project. |
| Duplicate name | Step 1; offer update. |
| Missing required | ASK `display_name`; do not call create to discover gaps. |
| Validation `fieldErrors` | Fix named fields once; do not retry the same payload. |

## Completion criteria

- [ ] `ers_type_get` (or reused chat schema) ran before create.
- [ ] Type chosen from tenant type names (one match or user pick).
- [ ] Only remaining `is_required` fields were asked.
- [ ] User confirmed the create plan.
- [ ] Response `success=true` before telling the user it exists.
- [ ] No requirement created for a project task.
- [ ] Task clocks, if any, were UTC with `Z`.
