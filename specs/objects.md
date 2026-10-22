# Objects

**Status:** Draft · **Prefix:** OBJ

## Purpose

What every record has, and the standard objects a new workspace starts with.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| OBJ-1 | A workspace is a team (`@specfork/teams:TEAM-1`). Workspace settings: `trash_retention_days` (integer ≥ 0, default 14, [REC-5](records.md)), `default_currency` (ISO 4217, default `USD`). A new workspace gets the standard objects OBJ-4..7, notes and tasks ([ACT](activities.md)), each with its default view ([VIEW-2](views.md)). | create team → companies, people, opportunities exist, empty |
| OBJ-2 | Every record of every object has system fields: `id` (UUID v4; a client may supply one on create), `created_at`, `updated_at`, `deleted_at?` ([REC-3](records.md)), `created_by`, `updated_by` (actor, [FLD-11](fields.md)), `position` (finite number, [REC-6](records.md)). Clients can't write them except `id` on create, `position`, and per FLD-11. | update `created_at` → `read_only` |
| OBJ-3 | Each object names one `label_field` (text, full name or UUID; default its `name` field) used as the record's title in lists, links, search and timeline, and an optional `image_field` (files or links, same object, active). | person titled by full name |
| OBJ-4 | Company: `name` (text), `domain` (links, domain form, max 1 value, unique, [FLD-5](fields.md)), `address`, `linkedin` (links), `annual_revenue` (currency), `account_owner` → member (many-to-one, set null). Label `name`; image `domain` (logo looked up from the domain). | two companies `acme.com` → second `conflict` |
| OBJ-5 | Person: `name` (full name), `emails` (unique on primary), `phones`, `linkedin` (links), `job_title` (text), `avatar` (files, max 1), `company` → company (many-to-one, set null). Label `name`; image `avatar`. | destroy company → its people keep existing, `company` empty |
| OBJ-6 | Opportunity: `name` (text), `amount` (currency), `close_date` (date), `stage` (select, required, default `new`; options `new`, `qualified`, `meeting`, `proposal`, `won`, `lost`, editable like any select [MOD-5](data-model.md)), `company` → company, `point_of_contact` → person, `owner` → member (all many-to-one, set null). | create without stage → `new` |
| OBJ-7 | Member: one read-only record per accepted membership (`@specfork/teams:TEAM-2`) with `name`, `email`, `avatar`, and own-editable preferences `time_zone` (IANA or `system`), `date_format` (`system`\|`month_first`\|`day_first`\|`year_first`), `time_format` (`system`\|`24h`\|`12h`), `number_format` (`system`\|`1,234.5`\|`1 234,5`\|`1.234,5`\|`1'234.5`). Leaving or removal (`@specfork/teams:TEAM-8`) empties every relation pointing at the member. | removed member's companies → `account_owner` empty |
| OBJ-8 | Duplicate rules ([REC-9](records.md)) per object, as a list of field groups: company: `name`, or `domain`; person: `name` (first + last), or primary email, or primary `linkedin`. Custom objects start with none; admins can set groups of up to 3 fields. | person "Ann Lee" twice → flagged |
| OBJ-9 | Standard objects and fields can be renamed in labels, hidden (deactivated, [MOD-11](data-model.md)) and extended with custom fields, never deleted or retyped. | delete `person.emails` → refused |

## Out of scope

- Dashboards and charts, AI agents and chat, apps/extensions platform, email campaigns, call recordings.
- Company enrichment from external data providers.

## Decisions

- Opportunity stages add `won` and `lost`; the source's default pipeline had no closing stages.
- Member is a projection of team membership, so roles and invites stay in `@specfork/teams`.
