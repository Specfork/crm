# Permissions

**Status:** Draft · **Prefix:** PERM

## Purpose

Team roles (`@specfork/teams:TEAM-2`) applied to CRM resources.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| PERM-1 | Resource rows (`@specfork/teams:TEAM-10`): records of any object: read, create, update, delete, restore, merge: member+; destroy: admin+. Workspace views: create, update, delete: member+. Shared favorites: admin+. Import, export: member+. Data model ([MOD](data-model.md)), duplicate rules, workspace settings ([OBJ-1](objects.md)): admin+. Workflows: read, run manual: member+; create, edit, activate, deactivate: admin+. Webhook subscriptions (scope = team): admin+. Workspace blocklist ([MAIL-9](mail.md)): admin+. | member destroys record → 403 |
| PERM-2 | The same check applies to every path that reads or writes records: UI, API, search, aggregates, export, timeline, workflow steps and merge. | member export of companies → allowed; then destroy via API → 403 |
| PERM-3 | A user-scoped API key acts as its user. A team-scoped API key (`@specfork/auth:AUTH-14`) acts with the current role of the member who created it, and stops working when that member leaves the team. | creator demoted to member → key can't destroy |
| PERM-4 | Personal data stays personal: personal views and favorites ([VIEW-3](views.md)), connected accounts and personal blocklist entries ([MAIL](mail.md)) are visible and editable only by their owner, whatever their role. | admin reads another member's personal view → 404 |

## Out of scope

- Custom roles, per-object and per-field permissions, row-level rules and record sharing (proposed for the teams base; row-level and sharing are commercial in the source).

## Decisions

- Fixed member/admin/owner roles from `@specfork/teams` instead of the source's custom roles, until the base has them.
- Team API keys borrow their creator's role, since the auth base keys carry no role.
