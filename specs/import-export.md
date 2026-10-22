# Import and export

**Status:** Draft · **Prefix:** IO

## Purpose

Bring records in from spreadsheets and take them out as CSV.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| IO-1 | Import one file into one object: CSV (comma or semicolon, UTF-8, header row) or XLSX (first sheet); ≤ 10,000 data rows, else refused before mapping. | 10,001 rows → refused |
| IO-2 | Mapping: each column maps to one field or sub-field, or is skipped; suggestions come from fuzzy match on field name and label. Importable: active, writable, non-system fields plus `id`; many-to-one relations by a unique field of the target (e.g. company by domain); actor fields not importable. | column "Company domain" → suggested `company` by domain |
| IO-3 | Per-cell checks before import: number, UUID, email, URL/domain, date `YYYY-MM-DD`, date-time ISO 8601, phone, country, calling code, select by value or label; booleans `true`\|`false`\|`yes`\|`no`\|`1`\|`0` (any case), anything else invalid. Invalid cells are shown for fixing or the row skipped. | `maybe` in boolean column → flagged |
| IO-4 | Rows repeating a unique value inside the file are flagged before import (links compared after [FLD-5](fields.md) normalisation). | two rows `acme.com` → flagged |
| IO-5 | Import runs as upsert ([REC-7](records.md)) in batches of 200, `created_by.source` = `import`; it can be cancelled between batches; the result gives created, updated and failed counts with row errors. Workflows and webhooks fire per record ([EVN-1](events.md)). | re-import same file → 0 created, all updated |
| IO-6 | Export: CSV of a view (its filter, sort and visible columns) or of selected records, as a background job; one running export per workspace (else `conflict`); stops at 1 GiB or 1 h; download via a single-use link valid 60 s after it's issued. Only fields the requester can read are included. | second export while one runs → `conflict` |
| IO-7 | Export format: UTF-8 with BOM, header row of labels, first column id; composite fields as one column per sub-field ("Label / Sub"); relations as "Label Id"; currency as decimal amount; dates `YYYY-MM-DD`; date-times ISO 8601 UTC; cells starting with `=`, `+`, `-`, `@`, tab or CR are prefixed so spreadsheets don't run them. | name `=1+1` → cell not evaluated |

## Out of scope

- Migration connectors from other CRMs, scheduled exports, XLSX export.

## Decisions

- Invalid booleans are errors; the source silently imported them as false.
