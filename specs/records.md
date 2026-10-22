# Records

**Status:** Draft · **Prefix:** REC

## Purpose

The operations every object gets, the same in UI and API.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| REC-1 | Every active object, standard or custom, has: get, list ([QRY](query.md)), create (one or many), update (one, or many by filter), delete (to trash), restore, destroy (permanent), upsert, find duplicates, merge. They are served by a JSON HTTP API authenticated by session (`@specfork/auth:AUTH-13`) or API key (`@specfork/auth:AUTH-14`), checked by [PERM](permissions.md), and the UI uses the same operations. | custom object → all operations work via API key |
| REC-2 | Batches: create-many ≤ 200 records (else `too_many`); an update-many touching > 200 records → `too_many`; update/delete/restore/destroy-many need a non-empty filter (else `filter_required`). A batch is atomic. | create 201 → `too_many`, none created |
| REC-3 | Delete sets `deleted_at`; deleted records are hidden from get, list, search, views and relations unless the query filters on `deleted_at`. Restore clears it. Destroy removes the record (deleted or not), applies each relation's `on_destroy` ([MOD-7](data-model.md)) and removes its files. Missing id → `not_found`. | delete then list → absent; filter deleted → present |
| REC-4 | Deleting a note or task deletes its links; restoring restores them ([ACT-3](activities.md)). Other relations are untouched by delete. | delete note → hidden on linked company; restore → back |
| REC-5 | A daily job destroys records whose `deleted_at` is more than `trash_retention_days` ([OBJ-1](objects.md)) days old; 0 → destroyed on the next run. | retention 14, deleted 15 days ago → gone |
| REC-6 | `position` on create or update: a number, `first` or `last`; missing → `first`. `first` = current minimum − 1, `last` = maximum + 1 (1 when empty), continuing −1/+1 within a batch. NaN or infinite → `invalid_position`. | create 2 with `last` on max 5 → 6, 7 |
| REC-7 | Upsert matches by `id`, then by each unique field whose value is given; matching two different records → `ambiguous_match`; a matched deleted record is restored and updated; no match → created. | upsert email of trashed person → restored, updated |
| REC-8 | Unique values include deleted records: a create clashing with a live record → `conflict`, with a deleted one → `conflict_in_trash` carrying its `id` so the client can offer restore. | create company with trashed domain → `conflict_in_trash` |
| REC-9 | Find duplicates (objects with duplicate rules, [OBJ-8](objects.md)): input is record ids or field values, not both; a rule group counts only if each of its values has ≥ 3 chars; values compared trimmed and case-insensitively; the record itself excluded; ≤ 200 results per input. | "ann lee" vs "Ann Lee" → duplicate |
| REC-10 | Merge 2–9 records of one object into a chosen primary, in one transaction: each field takes the primary's value, else the first non-empty in the given order; emails, phones, links keep the primary's primary value and combine all others into `additional` without duplicates; arrays and multi-selects are unioned. Records pointing at the merged ones (relations, notes, tasks, attachments, timeline) are re-pointed to the primary. The others go to the trash. `dry_run` returns the result without saving. | merge 2 people with 3 emails → 1 primary + 2 additional |
| REC-11 | A re-point during merge that would break a unique field moves that related record to the trash instead, and the merge result lists it. | merge where both have junction row to same project → one row trashed, listed |
| REC-12 | Concurrent updates to different fields of one record both persist; the same field: last write wins. Every write sets `updated_at`. | two members edit name and phone → both kept |

## Out of scope

- Per-record sharing and row-level access (commercial in the source), record versioning/undo beyond the trash.

## Decisions

- Merged-away records go to the trash; the source destroyed them.
- Duplicate matching is case-insensitive; the source compared exactly.
- REST/GraphQL specifics are not specified; any JSON API exposing these operations conforms.
