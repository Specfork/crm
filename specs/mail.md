# Mail and calendar sync

**Status:** Draft · **Prefix:** MAIL

## Purpose

Members connect mailboxes and calendars; messages and events attach to the people and companies involved.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| MAIL-1 | Connected account `{owner (member), provider: google\|microsoft\|imap_smtp_caldav, handle (email), aliases [], scopes, status}`; credentials encrypted at rest; one message channel and one calendar channel each. Only the owner sees or manages it ([PERM-4](permissions.md)); the owner leaving the team disconnects it (MAIL-12). | other member lists accounts → doesn't see it |
| MAIL-2 | Message channel: `visibility: metadata\|subject\|everything` (default metadata), `create_contacts: none\|sent\|sent_and_received` (default sent), `folders: all\|selected` (default all), `skip_personal_domains` (default true), `skip_group_addresses` (default true), `sync_enabled` (default true). Calendar channel: `visibility: metadata\|everything` (default metadata), `create_contacts` (bool, default true), `skip_private_events` (default false), `sync_enabled`. | new channel → metadata visibility |
| MAIL-3 | Sync: an initial import (mail from the last 365 days; calendar from 365 days back to 365 days ahead; instance settings), then incremental changes at least every 5 min, or by provider push with a poll when no sync happened for 1 h. Trash, spam and junk are never imported; provider bulk categories (promotions, social, updates, forums) are skipped under `all`. Status: `not_synced`\|`ongoing`\|`active`\|`failed_permissions`\|`failed`. | message in spam → not imported |
| MAIL-4 | Failures: temporary errors back off 1, 2, 4, 8, 16 min, then `failed`; `failed` channels are retried every 30 min; revoked or insufficient credentials → `failed_permissions`, owner asked to reconnect, no retry; an invalid sync cursor → full re-list. A stage stuck > 30 min is restarted. | 5 timeouts → `failed`, retried 30 min later |
| MAIL-5 | Import filters, in order: drop if any participant is blocklisted (MAIL-9); drop if every participant is on the owner's own work domain (workspace setting can allow internal mail); with `skip_group_addresses`, drop mail not sent by the owner that has bulk-mail headers or comes from role addresses (noreply, info, contact, hello, support, team, notifications, news, …). Calendar: cancelled events removed; private events skipped if set; events with a blocklisted attendee skipped. | newsletter with List-Unsubscribe → not imported |
| MAIL-6 | A message is stored once per workspace by Message-ID and linked to every channel holding it. Thread: the provider's thread id, else first References id, else In-Reply-To, else its own Message-ID; a thread's subject is its latest message's. | same email in 2 members' boxes → one message, 2 channels |
| MAIL-7 | Participants match people by any of their emails (case-insensitive; oldest live person wins) and members by email; matches are recomputed when a person is created, destroyed or their emails change. | add email to Ann → past threads link to Ann |
| MAIL-8 | Each message thread and calendar event is linked to its matched people and their companies, listed on those records and on their timeline ([ACT-7](activities.md)) at the message or event time. | email with Ann → on Ann and Acme timelines |
| MAIL-9 | Blocklist entries: an email or `@domain` (covers subdomains), case-insensitive, scope member (default) or workspace ([PERM-1](permissions.md)); own handles can't be blocked. Adding one deletes matching imported messages in its scope; removing one re-syncs the affected channels. | block `@spam.io` → existing messages from `a.spam.io` removed |
| MAIL-10 | Contact creation (per `create_contacts`; `sent` = only mail the owner sent): for participants that aren't the owner, aliases, members, the owner's work domain, personal-provider domains (if skipped) or group addresses (if skipped): reuse a person with that email (restoring a trashed one), else create one named from the display name; for work domains reuse or create a company with that domain, named from it. Actor `email` or `calendar` ([FLD-11](fields.md)). | owner mails `bob@globex.com` → person Bob + company Globex |
| MAIL-11 | Visibility: anyone sees everything if any linked channel shares `everything`, and the owner of a linked channel always does; `subject` hides the body; `metadata` hides subject and body (calendar: title and description), keeping participants and times. Applies to UI, API, search, export and timeline. | metadata channel → other member sees sender, not subject |
| MAIL-12 | Disconnecting stops sync and push, revokes the provider grant where possible, deletes the channels and every message, thread and event no longer linked to a channel. People and companies created stay. | disconnect → thread gone; person Bob remains |

## Out of scope

- Composing and replying from the CRM (except workflow send-email), SMS, email campaigns and tracking, shared team inboxes.

## Decisions

- Initial import limited to 365 days (configurable); the source imported whole mailboxes.
- Calendar contact creation is a boolean; the source's organizer/participant policy values were unused.
