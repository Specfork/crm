# Data model

**Status:** Draft · **Prefix:** MOD

## Purpose

Admins add objects and fields at run time; the API, views, search, import and automation pick them up without a deploy.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| MOD-1 | Custom object: `{name_singular, name_plural, label_singular, label_plural, description?, icon?, active (default true), searchable (default true)}`. Names match `^[a-z][a-z0-9_]{0,62}$`, singular ≠ plural, and are unique across every object's singular and plural names; names of system objects and API words (e.g. `object`, `field`, `view`, `search`, `and`, `or`, `not`) are reserved → `reserved_name`. Labels 1–63 chars, trimmed. Names default to `snake_case` of the labels (digit-first → `n` prefix; reserved → `_custom` suffix). | object named `views` → `reserved_name` |
| MOD-2 | Creating an object adds the system fields ([OBJ-2](objects.md)), a `name` text field as label field, search on it ([QRY-9](query.md)), a default view ([VIEW-2](views.md)), and makes it linkable from notes, tasks, attachments and timeline ([ACT](activities.md)). | new object `project` → note can link a project |
| MOD-3 | Custom field: `{name, label (1–63), type (FLD-1), description?, icon?, required (default false), unique (default false), default?, options?, settings?, active (default true), searchable (default false), audited (default true)}`. Name rules as MOD-1, unique within the object including relation reference columns and composite parts. Type can't change after creation. | change field type → refused |
| MOD-4 | A required field needs a non-null default; turning `required` on fills existing nulls with it. Relation, files and actor fields take no default. | make `industry` required with default `other` → empty records get `other` |
| MOD-5 | Select/multi-select options: non-empty list of `{id, value, label, color, position}`; `value` matches `^[a-z][a-z0-9_]{0,62}$`; label 1–63 chars, no comma; ids, values and positions unique; `color` from a fixed palette of 25 named colours (default `gray`). A select default must be one option; a multi-select default a non-empty set of options. | two options `won` → refused |
| MOD-6 | Editing options matches them by `id`: a changed value is rewritten in every record, view filter and kanban column; a removed option is cleared from records (select → null, multi-select → dropped from the set) and its kanban column removed. | remove `meeting` stage → those deals' stage null |
| MOD-7 | Relation field: `{kind: many_to_one\|one_to_many, target_object, inverse_label}` creates the inverse field on the target. The many-to-one side holds the reference and `on_destroy`: `set_null` (default) \| `cascade` \| `restrict`; it applies on destroy ([REC-3](records.md)), never on delete. Deleting either side deletes both. | `restrict` + destroy target → refused |
| MOD-8 | Many-to-many uses a junction object with two many-to-one fields; a one-to-many field may name the junction's other field to read the far records directly. | project ↔ person via `assignment` → project lists people |
| MOD-9 | `unique` is allowed on text, number, UUID, emails (primary), links (primary URL), phones (primary) and many-to-one relations, not with a non-empty default. Turning it on while duplicates exist → `duplicates_exist` with up to 10 offending values. Deleted (trashed) records count ([REC-8](records.md)). | enable unique with two equal values → `duplicates_exist` |
| MOD-10 | Editable later: labels, names (custom only), description, icon, active, searchable, unique, required, default, options, settings. Standard objects/fields: labels, description, icon, active, searchable only. System fields: nothing. Renames keep data, views and filters. | rename custom field → views still show it |
| MOD-11 | Deactivating a field or object hides it from views, forms, filters, search, import, export, workflows and the API (reads return no value; writes → `unknown_field`); data is kept and comes back on reactivation. Views grouped by the field, and its view columns, filters and sorts, are removed. The label field can't be deactivated or made unsearchable. | deactivate `stage` → kanban view on stage removed |
| MOD-12 | Only custom objects and fields can be deleted, and only while inactive. Deleting removes their data permanently, the inverse relation field, indexes and views. | delete active custom field → `deactivate_first` |
| MOD-13 | Every data-model change takes effect for all members and API clients without a restart; the API schema lists active objects and fields. | add field → next API read includes it |

## Out of scope

- Remote (external database) objects, formula fields, per-object validation rules, data-model versioning as code.

## Decisions

- `snake_case` names (the source used camelCase) to match other Specfork specs.
- Delete only after deactivation, a safety step the source didn't have.
- Enabling `unique` reports the duplicates instead of failing the migration.
- No limit on object or field count: none was found in the source.
