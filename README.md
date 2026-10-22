---
specfork: 1
name: "@specfork/crm"
summary: Customizable CRM with companies, people, deals, custom objects, views, notes, tasks, import, workflows and mail sync.
license: CC-BY-4.0
tags: [saas, sales]
requires:
  "@specfork/auth": "^0.3.0"
  "@specfork/teams": "^0.3.0"
  "@specfork/webhooks": "^0.3.0"
derived_from:
  repo: https://github.com/twentyhq/twenty
  commit: 9044bdd7d2ac023e973b292a3b883fa535f2f34e
  url: https://twenty.com
  license: AGPL-3.0-only
---

# CRM

A team keeps its customer data in objects: companies, people and opportunities out of the box, plus
objects and fields it defines itself. Every object gets the same record operations, API, views,
notes, tasks, timeline, import/export and automation.
Behaviour derived from Twenty's open-source code (AGPL-3.0), written fresh and stack-free.
Files under Twenty's commercial licence (record sharing, row-level permissions, validation rules,
SSO, audit logs, billing) were not used.
Not affiliated with or endorsed by Twenty.com, PBC. The name is used only to credit the source.

| Area | Prefix | Covers |
|---|---|---|
| [objects.md](specs/objects.md) | OBJ | Workspace, system fields, companies, people, opportunities, members, duplicate rules |
| [fields.md](specs/fields.md) | FLD | Field types, composite values, normalisation, files, actors |
| [data-model.md](specs/data-model.md) | MOD | Custom objects and fields, options, relations, unique fields, deactivation |
| [records.md](specs/records.md) | REC | Record operations, batches, trash, position, upsert, duplicates, merge |
| [query.md](specs/query.md) | QRY | Filters, sorting, pagination, aggregates, global search |
| [views.md](specs/views.md) | VIEW | Table, kanban and calendar views, visibility, favorites |
| [activities.md](specs/activities.md) | ACT | Notes, tasks, attachments, timeline |
| [import-export.md](specs/import-export.md) | IO | Spreadsheet import, CSV export |
| [permissions.md](specs/permissions.md) | PERM | Team roles applied to records, data model, views and settings |
| [events.md](specs/events.md) | EVN | Webhook events, payloads, live updates |
| [workflows.md](specs/workflows.md) | WF | Triggers, steps, versions, runs, limits |
| [mail.md](specs/mail.md) | MAIL | Email and calendar sync, contact creation, blocklist, visibility |

A workspace is a team (`@specfork/teams`); all data belongs to exactly one workspace.
All times are stored in UTC; API names are `snake_case`.
