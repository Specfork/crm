# Events

**Status:** Draft · **Prefix:** EVN

## Purpose

What the CRM tells the outside world (webhooks) and open clients (live updates) when records change.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| EVN-1 | Every committed record change emits one event per record: `<object>.created`, `.updated`, `.deleted`, `.restored`, `.destroyed`, with `<object>` the object's singular name (custom objects included). Upsert emits created or updated; merge emits `updated` for the primary and `deleted` for the others. A failed or rolled-back write emits nothing. | create-many 3 people → 3 `person.created` |
| EVN-2 | Webhook subscriptions (`@spechub/webhooks:HOOK-1`) have scope = team and list event names from EVN-1; deliveries follow the webhooks base. | subscribe `company.updated` → only company updates delivered |
| EVN-3 | Webhook `data`: `{object, record (state after; for deleted and destroyed, the last state), before? (updated only), updated_fields? (updated only), actor ([FLD-11](fields.md))}`; files appear as `{file_id, name}`, never contents. | update phone → `updated_fields` = `["phones"]` |
| EVN-4 | Data-model changes emit `metadata.object.created\|updated\|deleted` and `metadata.field.created\|updated\|deleted` with the new definition. | add field → `metadata.field.created` |
| EVN-5 | Live updates: a member viewing a list, board or record sees other members' changes within 5 s without reloading, limited to records and fields they may read. | A edits deal, B's board moves card |
| EVN-6 | Workflow triggers ([WF-3](workflows.md)) receive the same events as webhooks. | `opportunity.updated` → workflow on stage change runs |

## Out of scope

- Outbound email notifications and an in-app notification inbox.

## Decisions

- Event names use dots and the object's singular name, like the source; wildcard subscriptions are proposed for the webhooks base rather than defined here.
