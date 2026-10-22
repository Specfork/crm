# Activities

**Status:** Draft · **Prefix:** ACT

## Purpose

Notes, tasks and files attached to records, and each record's timeline of what happened to it.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| ACT-1 | Note: `{title, body (rich text), position}`. Task: `{title, body, due_at?, status: todo\|in_progress\|done (optional, default todo), assignee? → member}`. Both are objects ([REC](records.md)) and support custom fields. | create task → status `todo` |
| ACT-2 | A note or task links to any number of records of any object, standard or custom; a `(note\|task, record)` pair is unique. Each linked record lists it. | note linked to Acme and Ann → shown on both |
| ACT-3 | Deleting or restoring a note or task does the same to its links; destroying a linked record removes only the link. | destroy Acme → note stays, linked to Ann |
| ACT-4 | Tasks: completing sets `done`, reopening sets `todo`; a task is overdue when `due_at` < now and status ≠ done. "My tasks" = tasks assigned to me, not done, by `due_at` (nulls last), grouped by status. | due yesterday, todo → overdue |
| ACT-5 | Attachment: `{name, file (files, max 1, [FLD-10](fields.md)), parent}`; parent is one record of any object, including notes and tasks. Destroying the parent destroys its attachments. | destroy person → their files gone |
| ACT-6 | Timeline entry: `{record, happens_at, action: created\|updated\|deleted\|restored\|linked\|unlinked, actor ([FLD-11](fields.md)), diff?, linked {object, id, cached_name}?}`, newest first, grouped by month. | create company → one `created` entry |
| ACT-7 | Entries are written for record create, update, delete, restore; note, task, attachment link and unlink on each linked record; a change to a linked note's or task's title; synced emails and calendar events linked to the record ([MAIL-8](mail.md)), placed at their own time. | link note → `linked` entry on company |
| ACT-8 | Update diff `{field: {before, after}}` covers only audited fields ([MOD-3](data-model.md)); it never includes `updated_at`, `position`, search data or relation fields; an empty diff writes no entry. | change position only → no entry |
| ACT-9 | An update by the same actor on the same record within 10 minutes of that actor's last update entry merges into it: earliest `before`, latest `after` per field. | edit name twice in 2 min → one entry |
| ACT-10 | A reader sees diff fields only if they may read the field; an entry left empty is hidden. Linked entries keep `cached_name` after the linked record is destroyed. Destroying a record destroys its own entries. | destroyed note → company entry still names it |

## Out of scope

- Comments and mentions on records, reminders and notification inbox, collaborative real-time editing of note bodies.

## Decisions

- Attachments are destroyed with any parent; the source kept them (unlinked) for note and task parents only.
- Timeline entries of any destroyed record are destroyed; the source did so for people and companies only.
