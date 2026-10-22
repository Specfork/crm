# Workflows

**Status:** Draft · **Prefix:** WF

## Purpose

No-code automations: a trigger starts a run that executes steps on records, email and HTTP.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| WF-1 | Workflow `{name, versions, last_published}`; version `{trigger, steps, status: draft\|active\|deactivated\|archived}`, new ones `draft`. At most one active version per workflow; at most one draft. Editing an active workflow edits its draft; the active version keeps running. Any version can be copied into the draft. | edit active → draft created, runs use active |
| WF-2 | Activate needs a trigger with valid settings and ≥ 1 valid step; allowed from the draft or the last published deactivated version. The previous active version becomes archived, atomically; a version changed since the editor loaded it → `conflict`. Deactivate only from active. | activate draft → old active archived |
| WF-3 | Triggers: record event (object; `created`\|`updated`\|`created_or_updated`\|`deleted`; for updates an optional field list, firing only if one of them changed; optional record filter, [QRY-1](query.md)); manual (`global`, or for an object `single` = one run per selected record, `bulk` = one run with all selected); schedule (every N minutes, hours or days at a time, or a cron expression, in UTC); webhook (GET or POST to the workflow's URL, POST body schema). | update `stage` with field list [stage] → runs; update `name` → doesn't |
| WF-4 | A webhook trigger URL contains ≥ 128 random bits, accepts calls only while a version with that trigger is active (else 404), and answers `{run_id}`. | call after deactivate → 404 |
| WF-5 | Record steps: create, update (chosen fields), delete (to trash), upsert, find (filter, sort, limit 1–200). Each writes through [REC](records.md) and emits [EVN-1](events.md) events. | find with limit 500 → 200 |
| WF-6 | Other steps: send email (from a connected account the workflow's creator owns, [MAIL-1](mail.md); to, cc, subject, body, files); HTTP request (GET\|POST\|PUT\|PATCH\|DELETE, headers, body; URL rules of `@specfork/webhooks:HOOK-2`; 10 s timeout); form (pauses until a member submits text, number, date or record fields); filter (stops the branch if no match); if/else (ordered branches with filters, else branch); iterator (over a list, ≤ 10,000 items, optional continue on item failure); delay (until a date not in the past, or a duration ≤ 365 days). | HTTP step to `http://10.0.0.1` → step fails |
| WF-7 | Variables `{{<step>.<path>}}`, `trigger` for the trigger's output, usable in any setting. A setting that is only one reference keeps the value's type; references inside text render as text, objects as JSON. Unknown reference → empty, logged. | `{{trigger.record.amount}}` alone → number |
| WF-8 | Run `{status: not_started\|enqueued\|running\|waiting\|completed\|failed\|stopping\|stopped, version snapshot, trigger payload, steps [{status: not_started\|running\|waiting\|success\|failed\|failed_safely\|skipped, input, output, error}], started_at, ended_at, created_by}`. A run uses the version snapshot it started with. Members can stop a running or waiting run. | activate v2 mid-run → run finishes on v1 |
| WF-9 | Step failure: retried `retries` (0–3) times after 1 s, 5 s, 15 s; then `continue_on_failure` → `failed_safely` and the run goes on, else the run fails. Step logs ≤ 256 KB each; HTTP response bodies kept ≤ 32 KB. | 500 then 200 with retries 1 → success |
| WF-10 | Throttle per workspace: more than 100 run starts per minute stay `not_started` and start later, oldest first; more than 5,000 per hour → run created `failed` with `throttled`. | 150 starts in a minute → 50 start next minute |
| WF-11 | Loop guard: a run triggered by a write from another run carries depth + 1; depth > 5 → the run isn't started and the attempt is logged on the workflow. | workflow updating its own trigger object → stops at depth 5 |
| WF-12 | Steps write as actor `workflow` + workflow name, with full record access; a manual run started by a member acts as that member (their role, actor `manual`). Run `created_by`: member (manual), `workflow` (record event, schedule), `webhook`. | member's manual run tries destroy → step fails 403 |
| WF-13 | Housekeeping: `enqueued` > 1 h → re-queued; `stopping` > 1 h → `stopped`; `running` > 24 h with no step progress → `failed`. Finished runs older than 14 days are deleted, keeping at most 1,000 per workflow. | run 15 days old → gone |

## Out of scope

- Code steps, AI steps, logic functions, calendar-event creation, record picking strategies, multiple triggers per version.

## Decisions

- Loop guard at depth 5 and a 24 h running timeout; the source had neither.
- Webhook trigger URLs are unguessable; the source's URL used workspace and workflow ids and its auth setting wasn't enforced.
