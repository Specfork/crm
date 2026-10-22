# Query

**Status:** Draft · **Prefix:** QRY

## Purpose

Filter, sort, page and aggregate any object's records, and search across objects.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| QRY-1 | A filter is a tree of `and`, `or`, `not` nodes and conditions `{field[.sub_field], op, value}`. Relations: filter many-to-one by id or by the target's fields (one hop); filtering or sorting through one-to-many → `unsupported`. | filter people by `company.name` → works |
| QRY-2 | Operators: text, emails, links, phones (sub-fields): `eq`, `neq`, `in`, `contains`, `starts_with`, `ends_with` (case- and accent-insensitive); number, date, date-time, UUID: `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `in`; boolean: `eq`; select, rating: `in`, `not_in`; multi-select, array: `contains_any`, `is_empty`; all: `is_null`, `is_not_null`. Wrong operator for the type → `bad_filter`. | `amount_micros gt 1000000000` → deals over 1,000 |
| QRY-3 | Empty equals null: `is_null` also matches `""` and `[]`; `neq` and `not_in` never match nulls. | `job_title neq "CEO"` → excludes people without title |
| QRY-4 | Date filters accept relative values: today, past/next N days\|weeks\|months\|years, in the past, in the future, evaluated in the caller's time zone ([OBJ-7](objects.md)). | "next 7 days" on Monday → through next Monday |
| QRY-5 | Sort: ordered list of `{field[.sub_field], dir: asc\|desc, nulls: first\|last (default last)}`; `id` is always the final tie-breaker; text compares case-insensitively; select sorts by option position, not value. | sort by stage → `new` before `won` |
| QRY-6 | Pages: opaque cursors over the sort keys + id; `first` + `after` or `last` + `before` (mixing → `bad_request`); default 60, max 200 (larger → 200); response has `has_next`, `has_previous`, start and end cursors. `total` is returned on request and ignores the cursor. | 130 records, page 60 → 3 pages, total 130 each |
| QRY-7 | Reads may expand relations: to-one fully, to-many up to 60 per parent; a to-many inside a to-many → `too_complex`. | person with company → company fields inline |
| QRY-8 | Aggregates over a filter, optionally grouped by a select or many-to-one field: `count`, `count_empty`, `count_not_empty`, `count_unique`, `count_true`, `count_false`, `percent_empty`, `percent_not_empty`, `min`, `max`, `avg`, `sum`. | sum of amount by stage → one total per stage |
| QRY-9 | Global search spans searchable objects and their searchable fields (label field always; person: name, emails, phones, job title; company: name, domain; opportunity: name; note, task: title, body). Each term is a prefix match, case- and accent-insensitive; records matching all terms rank above those matching some; ties by object: person, company, opportunity, note, task, custom. ≤ 100 per page, cursor-paged; deleted and unreadable records excluded. | "ann ac" → Ann at Acme first |
| QRY-10 | Searchable field types: text, full name, emails, phones, links, address, rich text, UUID. Changes to a record are searchable within 5 s. | rename company → found by new name |

## Out of scope

- Regular-expression filters, vector/semantic search, saved queries outside views.

## Decisions

- Select sorts by option order so pipelines sort by stage order; the source sorted by value text.
- Default `nulls last`; the source defaulted to nulls first.
