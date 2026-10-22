# Views

**Status:** Draft · **Prefix:** VIEW

## Purpose

Saved ways to look at an object's records: tables, boards and calendars, shared or personal; plus favorites.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| VIEW-1 | View: `{object, name, icon?, type: table\|kanban\|calendar, visibility: workspace\|personal, position, compact (default false), filter ([QRY-1](query.md)), sorts ([QRY-5](query.md)), search_text?}`; `search_text` matches any searchable field. Views are ordered by `position` in the object's view list. | two views reordered → order kept for all |
| VIEW-2 | Each object has exactly one default view: table "All <plural label>", visibility workspace, all active fields as columns. It can't be deleted or made personal; an object's last view can't be deleted. | delete default view → refused |
| VIEW-3 | Personal views are visible and editable only by their creator; API keys can't create them. Workspace views are created and changed per [PERM-1](permissions.md). A member without that right may change a workspace view on screen; the change isn't saved, and they can save it as a new personal view. | member edits shared view filter → reload shows original |
| VIEW-4 | Columns: per field `{visible (default true), width (px, default 180), position, aggregate? ([QRY-8](query.md))}`; a field at most once; the label field is always first and visible. | move column before name → refused |
| VIEW-5 | Filter groups nest at most 2 levels; select filter values must be existing options ([MOD-6](data-model.md) keeps them in step). | 3-level filter → refused |
| VIEW-6 | Kanban: `group_by` is a select or many-to-one field. Columns `{value, visible, position}`, one per option in option order, plus a "no value" column while the field is optional. New option → column appended (visible while fewer than 40 are visible); removed option → column removed. Settings: `hide_empty_columns` (default false), column width 150–400 px, one aggregate per column, cards per column page 8\|25\|50\|100 (default 8). | add stage option → new column at end |
| VIEW-7 | Dragging a card sets the group field to the column's value and `position` to the midpoint of its neighbours (top: first − 1; bottom: last + 1; empty column: 1) in one update. Within a column, cards are ordered by `position` unless the view has sorts. | drag deal to `won` → stage `won`, between neighbours |
| VIEW-8 | Calendar: `start_field` (date or date-time) required; optional `end_field` of the same type, not the start field; layout `day`\|`week`\|`month`. | end field = start field → refused |
| VIEW-9 | Each member's last-used view per object reopens on return. | open view B, leave, return → B |
| VIEW-10 | Favorites: items `{kind: object\|view\|record\|folder\|link, target, position, folder?, owner?}`; with `owner` personal, without it shared with the workspace ([PERM-1](permissions.md)). Folders nest at most 2 levels; deleting a folder deletes its items; links are `https` URLs. Items pointing at a deleted record or view are removed. | delete favorited company → favorite gone |

## Out of scope

- Record page layout editing, dashboards and widgets, list (mobile) views.

## Decisions

- Default view of opportunities is a table like any object; the source special-cased it.
